<template>
  <div>
     <a-card :bordered="false">
       <div class="table-operator">
         <a-upload
          style="display: inline-block"
          name="file"
          :showUploadList="false"
          :multiple="true"
          :action="uploadUrl"
          :headers="headers"
          @change="handleChange"
        >
          <a-button type="primary">上传文件</a-button>
        </a-upload>
        <a-button type="primary" :loading="cLoading" @click="moreCalculate">批量计算</a-button>
        <a-button type="primary" :loading="acLoading" @click="allCalculate">全部重新计算</a-button>
        <a-button type="primary" @click="moreDownloadFile">批量下载</a-button>
        <a-button type="primary" @click="allDownloadFile">全部下载</a-button>
        <a-button type="danger" @click="doClearClick">清空</a-button>
       </div>
       <div>
         <a-table
          ref="table"
          size="middle"
          bordered
          rowKey="id"
          :columns="columns"
          :dataSource="dataSource"
          :pagination="false"
          :loading="loading"
          :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
          class="j-table-force-nowrap">

          <template slot="originFileName" slot-scope="text, record">
            <div class="table-cell">
              <span class="">{{ record.originFileName }}</span>
              <a-icon @click="previewClick(record, 1)" class="icon-button" type="eye" />
              <a-icon @click="simpleDownloadFile([record.id], 1)" class="icon-button" type="download" />
            </div>
          </template>
          <template slot="targetValue" slot-scope="text, record">
            <div>
              <a-input v-model="record.targetValue" type="number"></a-input>
            </div>
          </template>
          <template slot="downloadFileName" slot-scope="text, record">
            <div class="table-cell" v-if="record.downloadFileName">
              <span class="">{{ record.downloadFileName }}</span>
              <a-icon @click="previewClick(record, 2)" class="icon-button" type="eye" />
              <a-icon class="icon-button" @click="simpleDownloadFile([record.id], 2)" type="download" />
            </div>
            <div v-else>请先计算</div>
          </template>
          <span slot="action" slot-scope="text, record">
            <div class="table-row-right-button">
              <a-button type="primary" @click="changeCalculate([record])">计算</a-button>
            </div>
          </span>
        </a-table>
       </div>
     </a-card>
     <!-- 预览弹出框 -->
     <!-- <a-modal>
       <xlsx-read :file="previewFile">
          <xlsx-table />
        </xlsx-read>
     </a-modal> -->
     <a-modal
      width="1000px"
      title="文件预览"
      :closable="false"
      :visible="visible"
    >
    <a-spin :spinning="spinning">
      <div class="xlsx-wrapper" style="height: 500px; max-width:1000px; overflow-y: auto">
        <xlsx-read :file="previewFile">
          <xlsx-table />
        </xlsx-read>
      </div>
    </a-spin>
    <div slot="footer" style="text-align:center">
        <a-button type="primary" @click="visible = false">关闭</a-button>
    </div>
    </a-modal>
  </div>
</template>

<script>
import Vue from 'vue'
import { ACCESS_TOKEN } from '@/store/mutation-types'
import {getAction, putAction, postAction, deleteAction, downFile} from '@/api/manage'
import { XlsxRead, XlsxTable } from 'vue-xlsx/dist/vue-xlsx.es'
import lodash from 'lodash'
export default {
  components: {
    XlsxRead,
    XlsxTable
  },
  data() {
    return {
      loading: false,
      columns: [
        {
          title:'上传文件',
          align:"center",
          dataIndex: 'originFileName',
          scopedSlots: { customRender: 'originFileName' }
        },
        {
          title:'目标修正值',
          align:"center",
          dataIndex: 'targetValue',
          scopedSlots: { customRender: 'targetValue' }
        },
        {
          title:'计算结果',
          align:"center",
          dataIndex: 'downloadFileName',
          scopedSlots: { customRender: 'downloadFileName' }
        },
        {
          title: '操作',
          dataIndex: 'action',
          align:"center",
          fixed:"right",
          width:147,
          scopedSlots: { customRender: 'action' }
        },
      ],
      dataSource: [],
      headers: {
        'X-Access-Token': 'authorization-text'
      },
      uploadUrl: '',
      previewFile: '',
      visible: false,
      spinning: false,
      selectedRowKeys: [],
      selectionRows: [],
      cLoading: false,
      acLoading: false
    };
  },
  created() {
    this.uploadUrl = window._CONFIG['domianURL'] + '/svc/xpSvcRecord/uploadFile'
    const token = Vue.ls.get(ACCESS_TOKEN)
    if (token) {
      this.headers[ 'X-Access-Token' ] = token // 让每个请求携带自定义 token 请根据实际情况自行修改
    }
  },
  mounted() {

  },
  methods: {
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedRowKeys = selectedRowKeys;
      this.selectionRows = selectionRows;
    },
    // 全部计算
    async allCalculate() {
      if (this.dataSource.length > 0) {
        this.acLoading = true
        try {
          await this.changeCalculate(this.dataSource);
        } finally {
          this.acLoading = false
        }
      }
    },
    // 批量计算
    async moreCalculate() {
      if (this.selectionRows.length === 0) {
        this.$message.error('请先勾选数据')
        return
      }
      this.cLoading = true;
      try {
        await this.changeCalculate(this.selectionRows);
      } finally {
        this.cLoading = false;
        this.selectionRows = [];
        this.selectedRowKeys = [];
      }
    },
    allDownloadFile() {
      if (this.dataSource.length == 0) {
        this.$message.error('暂无数据')
        return
      }
      let ids = this.dataSource.map(ele => ele.id)
      this.simpleDownloadFile(ids, 2);
    },
    // 批量下载
    moreDownloadFile() {
      if (this.selectionRows.length === 0) {
        this.$message.error('请先勾选数据')
        return
      }
      let rows = lodash.cloneDeep(this.selectionRows)
      let ids = rows.map(ele => ele.id)
      this.simpleDownloadFile(ids, 2);
    },
    dataURLtoFile(dataurl, filename) {//将base64转换为文件
      let bstr = window.atob(dataurl);
      let n = bstr.length;
      let u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      } 
      return new Blob([u8arr], { type: 'text/csv' })
    },
    // 文件预览点击事件
    previewClick(data, type) {
      this.spinning = true;
      let params = {
        id: data.id,
        type
      }
      getAction('/svc/xpSvcRecord/preview', params).then(res => {
        if(res.success === false) {
          this.$message.error(res.message);
          return
        }
        let filename = type === 1 ? data.originFileName : data.downloadFileName;
        let objectUrl = this.dataURLtoFile(res.result, filename)
        this.previewFile = objectUrl;
        this.visible = true;
        this.spinning = false;
      }).catch(error => {
        this.spinning = false;
      })
    },
    // 上传之后的处理
    handleChange(info) {
      const {file} = info;
      if (file && file.response && file.response.success === false) {
        this.$message.error(file.response.message)
      }
      if (file && file.response && file.response.code == 200) {
        this.dataSource.push({
          originFileObj: file.originFileObj,
          ...file.response.result
        })
      }
    },
    // 下载文件
    async simpleDownloadFile(ids, type) {
      let idsStr = ids.join(',');
      const link = document.createElement("a");
      link.href = `${window._CONFIG['domianURL']}/svc/xpSvcRecord/download?id=${idsStr}&type=${type}`;
      link.target = "_blank"
      link.click();
    },
    // 转换计算
    async changeCalculate(data) {
      const payload = {
        svcRecords: data
      }
      const res = await postAction('/svc/xpSvcRecord/transform', payload);
      if(res.success === false) {
        this.$message.error(res.message);
        return
      }
      let ids = res.result.map(ele => ele.id)
      for (let index = 0; index < this.dataSource.length; index++) {
        const element = this.dataSource[index];
        if (ids.includes(element.id)) {
          const newData = res.result.filter(item => item.id == element.id)[0]
          this.$set(this.dataSource, index, newData)
        }
      }
      this.$message.success('操作成功')
      return Promise.resolve(res)
      try {
      } catch (error) {
        return Promise.reject(error)
      }
    },
    doClearClick() {
      this.$confirm({
        title: '提示',
        content: '真的要进行清空操作吗 ?',
        onOk:() => {
          this.dataSource = [];
          this.selectedRowKeys = [];
          this.selectionRows = [];
        },
        onCancel() {
        },
      });
    }
  }
};
</script>

<style scoped lang="less">
  @import '~@assets/less/common.less';
  .xlsx-wrapper {
    /deep/ table {
      border-collapse: collapse;
      td {
        border: 1px solid #dddddd;
        padding: 0 10px;
        height: 40px;
        line-height: 40px;
      }
    }
  }
  .icon-button {
    margin-left: 10px;
    cursor: pointer;
    font-size: 20px;
    color: #999999;
    &:hover {
      color: #308bd8;
      transition: .3ms;
    }
  }
  .table-cell {
    display: flex;
    align-items: center;
  }
  .table-row-right-button {
    .ant-btn {
      margin: 0 8px 0 0;
    }
  }
</style>
