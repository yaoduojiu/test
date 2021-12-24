<template>
  <a-card :bordered="false" :class="'cust-erp-sub-tab'">
    <!-- 操作按钮区域 -->
    <div class="table-operator" v-if="mainId">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button :disabled="selectedRowKeys.length === 0" @click="handlePickOut" type="primary" icon="plus">领料出库</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('领料明细')">导出</a-button>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel">
          <a-button hidden type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :scroll="{x:true}"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
        </span>

      </a-table>
    </div>

    <pickDetail-modal ref="modalForm" @ok="modalFormOk" :mainId="mainId"></pickDetail-modal>
    <pickOut-modal ref="pickOutModalForm" @ok="modalFormOk" :dataInfo="dataInfo"></pickOut-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import PickDetailModal from './modules/PickDetailModal'
  import PickOutModal from './modules/PickOutModal'

  export default {
    name: "PickDetailList",
    mixins:[JeecgListMixin],
    components: { PickDetailModal,  PickOutModal},
    props:{
      mainId:{
        type:String,
        default:'',
        required:false
      }
    },
    watch:{
      mainId:{
        immediate: true,
        handler(val) {
          if(!this.mainId){
            this.clearList()
          }else{
            this.queryParam['pickId'] = val
            this.loadData(1);
          }
        }
      }
    },
    data () {
      return {
        description: '领料单管理页面',
        disableMixinCreated:true,
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'物料编码',
            align:"center",
            dataIndex: 'materialCode'
          },
          {
            title:'物料名称',
            align:"center",
            dataIndex: 'materialName'
          },
          {
            title:'领料数量',
            align:"center",
            dataIndex: 'pickQty'
          },
          {
            title:'已领数量',
            align:"center",
            dataIndex: 'receiveQty'
          },
          {
            title:'创建人',
            align:"center",
            dataIndex: 'createBy'
          },
          {
            title:'创建日期',
            align:"center",
            sorter: true,
            dataIndex: 'createTime'
          },
          {
            title:'更新人',
            align:"center",
            dataIndex: 'updateBy'
          },
          {
            title:'更新日期',
            align:"center",
            sorter: true,
            dataIndex: 'updateTime'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' },
          }
        ],
        url: {
          list: "/pick/pickList/listPickDetailByMainId",
          delete: "/pick/pickList/deletePickDetail",
          deleteBatch: "/pick/pickList/deleteBatchPickDetail",
          exportXlsUrl: "/pick/pickList/exportPickDetail",
          importUrl: "/pick/pickList/importPickDetail",
        },
        dictOptions:{
         status:[],
        },
        superFieldList:[],
        dataInfo: {}
      }
    },
    created() {
      this.getSuperFieldList();
    },
    computed: {
      importExcelUrl(){
        return `${window._CONFIG['domianURL']}/${this.url.importUrl}/${this.mainId}`;
      }
    },
    methods: {
      handlePickOut() {
        if (this.selectedRowKeys.length === 1) {
          console.log(this.selectedRowKeys);
          this.dataInfo = this.dataSource.find(item => this.selectedRowKeys.includes(item.id));
          this.$refs.pickOutModalForm.visible = true;
        } else {
          this.$message.warning('请勾选一项进行操作')
        }

      },
      clearList(){
        this.dataSource=[]
        this.selectedRowKeys=[]
        this.ipagination.current = 1
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'code',text:'领料单号',dictCode:''})
        fieldList.push({type:'popup',value:'orderCode',text:'订单编号', popup:{code:'order_popup',field:'code',orgFields:'code',destFields:'order_code'}})
        fieldList.push({type:'string',value:'materialCode',text:'物料编码',dictCode:''})
        fieldList.push({type:'string',value:'materialName',text:'物料名称',dictCode:''})
        fieldList.push({type:'sel_search',value:'pickUser',text:'领料人',dictTable:'sys_user', dictText:'realname', dictCode:'id'})
        fieldList.push({type:'Text',value:'remarks',text:'备注',dictCode:''})
        fieldList.push({type:'string',value:'status',text:'状态',dictCode:'pick_list_status'})
        fieldList.push({type:'string',value:'createBy',text:'创建人',dictCode:''})
        fieldList.push({type:'datetime',value:'createTime',text:'创建日期'})
        fieldList.push({type:'string',value:'updateBy',text:'更新人',dictCode:''})
        fieldList.push({type:'datetime',value:'updateTime',text:'更新日期'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>
