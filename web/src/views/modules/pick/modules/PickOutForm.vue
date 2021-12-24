<template>
  <div>
    <a-descriptions title="基本信息">
      <a-descriptions-item label="物料编码">
        {{dataInfo.materialCode}}
      </a-descriptions-item>
      <a-descriptions-item label="物料名称">
        {{dataInfo.materialName}}
      </a-descriptions-item>
      <a-descriptions-item label="领料数量">
        {{dataInfo.pickQty}}
      </a-descriptions-item>
      <a-descriptions-item label="已领数量">
        {{dataInfo.receiveQty}}
      </a-descriptions-item>
      <a-descriptions-item label="审核人">
        {{dataInfo.createBy}}
      </a-descriptions-item>
       <a-descriptions-item label="审核时间">
        {{dataInfo.createTime}}
      </a-descriptions-item>
    </a-descriptions>
    <div style="height: 60px;">
        <a-form :form="form" layout="inline" labelAlign="left">
          <a-form-item label="物料SN" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-input
                   @keyup="keyupHandle"
                   v-model="sn"
                placeholder="请输入" style="width: 100%" />
              </a-form-item>
              <a-form-item>
                <a-button type="primary" @click="addClick">添加</a-button>
              </a-form-item>
        </a-form>
      </div>
    <a-table
      :row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
      :columns="columns"
      :data-source="data"
      bordered
      rowKey="id"
      size="small"
      :pagination=false
      :scroll={y:200}
    >
      <span slot="action" slot-scope="text, record">
        <!-- <a-input-number v-model="record.number"   placeholder="请输入入库数量" style="width: 100%" /> -->
        <a-button type="danger" @click="removeClick(record.id)" size="small">移除</a-button>
      </span>
    </a-table>
    </div>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  const columns = [
    {
      title: '库位',
      dataIndex: 'locationCode',
    },
    {
      title: '批次编号',
      dataIndex: 'code',
    },
    {
      title: '物料编码',
      dataIndex: 'materialCode',
    },
    {
      title: '物料名称',
      dataIndex: 'materialName',
    },
    {
      title: '在库数量',
      dataIndex: 'inQty',
    },
    {
      title: '出库数量',
      dataIndex: 'qty',
    },
    {
      title: '操作',
      dataIndex: 'action',
      align:"center",
      fixed:"right",
      width:147,
      scopedSlots: { customRender: 'action' },
    }
  ];
  export default {
    name: 'PickOutForm',
    components: {
    },
    props: {
      dataInfo: {
        type: Object,
        default: ()=>{},
        required: true
      }
    },
    data () {
      return {
        sn: '',
        data: [],
        columns,
        selectedRowKeys: [],
        url: {
          add: "/stock/stockInOrder/add",
          edit: "/stock/stockInOrder/edit",
          queryById: "/stock/stockInOrder/queryById"
        },
        form: this.$form.createForm(this),
        labelCol: {
          xs: { span: 24 },
          sm: { span: 8 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
      }
    },
    computed: {
      
    },
    mounted () {
      // this.getData();
    },
    methods: {
      // getData() {
      //   this.selectedRowKeys = [];
      //   getAction('api/pickOrder/findBatchInfo', {
      //     materialId: this.dataInfo.materialId
      //   }).then(res => {
      //     if (res.success) {
      //       this.data = res.result;
      //     } else {
      //       this.$messages.warn(res.message);
      //     }
          
          
      //   })
      // },
      removeClick(id) {
        let _this = this
        this.$confirm({
        title: '提示',
        content: '确定要进行移除吗?',
        okText: '确定',
        cancelText: '取消',
        onOk() {
          _this.data = _this.data.filter(ele => ele.id !== id)
        }
      });
      },
      // 根据物料sn查询批次信息
      getBatchInfo(sn) {
        getAction('/batchInfo/batchInfo/getBatchInfoBySn', {
          sn
        }).then(res => {
          if (res.success) {
            if(!(this.data.find(ele => ele.id === res.result.id)))
            this.data.push({
              ...res.result,
              batchSn: this.sn
            });
            this.sn = ''
          } else {
            this.$message.warn(res.message);
          }
        })
      },
      onSelectChange(selectedRowKeys) {
        this.$emit('select', this.data.filter(item=>selectedRowKeys.includes(item.id)));
        this.selectedRowKeys = selectedRowKeys;
      },
      addClick() {
        if(this.sn) {
          this.getBatchInfo(this.sn)
        } else {
          this.$message.error('物料SN不能为空')
        }
      },
      keyupHandle(event) {
        if (event.key === 'Enter') {
          if (event.target.value !== '') {
            this.getBatchInfo(event.target.value)
          } else {
            this.$message.error('物料SN不能为空')
          }
        }
      }
    }
  }
</script>

<style lang="less" scoped>
  .title{
    font-weight: bold;
  }

</style>