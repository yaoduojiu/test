<template>
  <div>
    <!-- 操作按钮区域 -->
    <div class="table-operator">
    </div>

    <!-- table区域-begin -->
    <div>
      <div style="padding-bottom: 10px">
        <a-button v-if="mainId && selectRow" @click="addDetailClick" type="primary" size="small">新增盘点明细</a-button>
      </div>
      <!-- <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div> -->

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        class="deeps"
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
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleCheck(record)">盘点</a>
          <!-- <a-divider type="vertical" />
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm> -->
        </span>

      </a-table>
    </div>

    <InventoryDetailCheck @ok="handleCheckOk" ref="InventoryDetailCheck" :model="currentModel"></InventoryDetailCheck>
    <!-- 新增盘点明细弹出框 -->
    <j-modal
    title="新增盘点明细"
    width="600px"
    :visible="addVisible"
    :confirmLoading="addLoading"
    @ok="handleAddOk"
    @cancel="handleAddCancel"
    cancelText="关闭">
      <a-spin :spinning="addLoading">
        <a-form :form="form">
          <a-row>
            <a-col :span="24">
              <a-form-item label="批次" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-select  @change="batchChange" placeholder="请选择批次" 
                  v-decorator="[
                  'batchId',
                    { rules: [{ required: true, message: '请选择批次' }] },
                  ]"
                >
                  <a-select-option v-for="(item, index) in batchData" :key="index" :value="item.id">
                    {{ item.code }}
                  </a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
          </a-row>
          <a-row>
          <a-col :span="24">
            <a-form-item label="盘点数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number
                :precision="0"
                :min="0"
                :max="999999999"
                 v-decorator="[
                  'qty',
                  { rules: [{ required: true, message: '请输入盘点数量' }] },
                ]"
              placeholder="请输入盘点数量" style="width: 100%" />
            </a-form-item>
          </a-col>
        </a-row>
        </a-form>
      </a-spin>
    </j-modal>
  </div>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import InventoryDetailCheck from '@/views/modules/inventory/modules/InventoryDetailCheck'
  import {getAction, postAction, httpAction} from '@/api/manage.js'

  export default {
    name: "InventoryDetailList",
    mixins: [JeecgListMixin],
    components: { InventoryDetailCheck },
    props:{
      mainId:{
        type:String,
        default:'',
        required: true
      },
      selectRow: {
        required: true
      }
    },
    watch:{
      mainId:{
        immediate: true,
        handler(val) {
          console.log(val, 'val')
          if(!this.mainId){
            this.clearList()
          }else{
            this.queryParam['inventoryId'] = val
            this.loadData(1);
          }
        }
      }
    },
    data () {
      return {
        addVisible: false,
        addLoading: false,
        currentModel: {},
        formModel: {},
        form: this.$form.createForm(this),
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        description: '盘点单管理页面',
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
            title:'批次编号',
            align:"center",
            dataIndex: 'code'
          },
          {
            title: '物料编码',
            align: 'center',
            dataIndex: 'materialCode'
          },
          {
            title: '物料名称',
            align: 'center',
            dataIndex: 'materialName'
          },
          {
            title: '库位',
            align: 'center',
            dataIndex: 'locationCode'
          },
          {
            title: '在库数量',
            align: 'center',
            dataIndex: 'inQty'
          },
          {
            title: '盘点数量',
            align: 'center',
            dataIndex: 'qty'
          },
          {
            title: '盘点结果',
            align: 'center',
            dataIndex: 'status_dictText'
          },
          {
            title: '创建时间',
            align: 'center',
            dataIndex: 'createTime'
          },
          {
            title: '创建人',
            align: 'center',
            dataIndex: 'createBy'
          },
          {
            title: '更新时间',
            align: 'center',
            dataIndex: ''
          },
          {
            title: '更新人',
            align: 'center',
            dataIndex: ''
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
        dataSource: [],
        url: {
          list: "/inventory/inventory/listInventoryDetailByMainId",
          delete: "/inventory/inventory/deleteInventoryDetail",
          deleteBatch: "/inventory/inventory/deleteBatchInventoryDetail",
          exportXlsUrl: "/inventory/inventory/exportInventoryDetail",
          importUrl: "/inventory/inventory/importInventoryDetail",
        },
        dictOptions:{
         type:[],
         status:[],
         locationId:[],
         materialId:[],
        },
        superFieldList:[],
        batchData: []
      }
    },
    computed: {
      importExcelUrl(){
        return `${window._CONFIG['domianURL']}/${this.url.importUrl}/${this.mainId}`;
      }
    },
    mounted() {
      this.getBatchData()
    },
    methods: {
      addDetailClick() {
        this.addVisible = true;
      },
      batchChange(value) {
        this.form.setFieldsValue({
          batchId: value
        })
      },
      // 获取批次信息
      getBatchData() {
        console.log('5555555555555555')
        const params = {
          pageSize: 9999999,
          pageNo: 1,
          type: 'PURCHASE'
        }
        getAction('/batchInfo/batchInfo/list', params).then(res => {
          if(res.success === true) {
            this.batchData = res.result.records;
          }
        })
      },
      handleAddOk() {
        let that = this;
        this.form.validateFields((err, values) => {
          if (!err) {
            that.addLoading = true;
            let httpurl = '/inventory/inventory/addInventoryDetail';
            let method = 'post';
            let formData = Object.assign({
              inventoryId: this.selectRow.id,
              locationId: this.selectRow.locationId
            }, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                this.loadData(1)
              }else{
                that.$message.warning(res.message);
              }
              that.addLoading = false
                that.form.setFieldsValue({
                qty: '',
                batchId: ''
              })
              this.addVisible = false
            }).finally(() => {
              that.form.setFieldsValue({
                qty: '',
                batchId: ''
              })
              that.addLoading = false;
            })
          }
        })
      },
      handleAddCancel() {
        this.form.setFieldsValue({
          qty: '',
          batchId: ''
        })
        this.addVisible = false;
      },
      handleCheckOk() {
        this.loadData(1);
      },
      handleCheck(item) {
        this.currentModel = {...item};
        this.$refs.InventoryDetailCheck.visible = true;
      },
      loadData(arg) {
        if(!this.url.list){
          this.$message.error("请设置url.list属性!")
          return
        }
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
        params.inventoryId= this.mainId;
        this.loading = true;
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            //update-begin---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
            this.dataSource = res.result.records||res.result;
            if(res.result.total)
            {
              this.ipagination.total = res.result.total;
            }
            //update-end---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
          }
          if(res.code===510){
            this.$message.warning(res.message)
          }
          this.loading = false;
        })
      },
      clearList(){
        this.dataSource=[]
        this.selectedRowKeys=[]
        this.ipagination.current = 1
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'no',text:'盘点单号',dictCode:''})
        fieldList.push({type:'string',value:'warehostCode',text:'仓库编码',dictCode:''})
        fieldList.push({type:'string',value:'warehouseName',text:'仓库名称',dictCode:''})
        fieldList.push({type:'string',value:'type',text:'仓库类型',dictCode:'inventory_type'})
        fieldList.push({type:'string',value:'status',text:'状态',dictCode:'inventory_status'})
        fieldList.push({type:'list_multi',value:'locationId',text:'库位',dictTable:'', dictText:'', dictCode:''})
        fieldList.push({type:'list_multi',value:'materialId',text:'物料',dictTable:'', dictText:'', dictCode:''})
        fieldList.push({type:'Text',value:'remarks',text:'备注',dictCode:''})
        fieldList.push({type:'string',value:'createBy',text:'创建人',dictCode:''})
        fieldList.push({type:'datetime',value:'createTime',text:'创建日期'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  .deeps >>> tr th{
       white-space: nowrap;
  } 
   .deeps >>> tr td{
       white-space: nowrap;
  } 
</style>
