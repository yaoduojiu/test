<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="出库单编码">
              <a-input placeholder="请输入出库单编码" v-model="queryParam.code"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="物料编码">
              <a-input placeholder="请输入物料编码" v-model="queryParam.materialCode"/>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="物料名称">
                <a-input placeholder="请输入物料名称" v-model="queryParam.materialName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="物料SN">
                <a-input placeholder="请输入物料SN" v-model="queryParam.batchSn"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="状态">
                <j-dict-select-tag placeholder="请选择状态" v-model="queryParam.status" dictCode="stock_out_status"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="出库类型">
                <j-dict-select-tag placeholder="请选择出库类型" v-model="queryParam.outType" dictCode="stock_out_type"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="仓库名称">
                <a-input placeholder="请输入仓库名称" v-model="queryParam.warehouseName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="库位编号">
                <j-popup placeholder="请选择库位编号" v-model="queryParam.locationCode" code="bb_hh_location" org-fields="id,code" dest-fields="location_id,location_code" :field="getPopupField('location_id,location_code')"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="批次编码">
                <a-input placeholder="请输入批次编码" v-model="queryParam.batchCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="领料单号">
                <a-input placeholder="请输入领料单号" v-model="queryParam.pickCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="盘点单号">
                <a-input placeholder="请输入盘点单号" v-model="queryParam.inventoryCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="出库时间">
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择开始时间" class="query-group-cust" v-model="queryParam.outTime_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择结束时间" class="query-group-cust" v-model="queryParam.outTime_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="出库操作人">
                <a-input placeholder="请输入出库操作人" v-model="queryParam.outUser"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="创建日期">
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择开始时间" class="query-group-cust" v-model="queryParam.createTime_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择结束时间" class="query-group-cust" v-model="queryParam.createTime_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="更新日期">
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择开始时间" class="query-group-cust" v-model="queryParam.updateTime_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择结束时间" class="query-group-cust" v-model="queryParam.updateTime_end"></j-date>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleOutOrderDirect" type="primary" icon="plus">出库</a-button>
      <a-button @click="handleOut" type="primary" icon="check" :disabled = 'isEnable'>执行出库</a-button>
      <a-button @click="handlePrint" type="primary" icon="printer" :disabled='isEnablePrint'>出库打印</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('出库单')">导出</a-button>
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
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
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
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>

    <stock-out-order-modal ref="modalForm" @ok="modalFormOk"></stock-out-order-modal>
    <stock-out-order-direct-modal ref="modalDirectForm" @ok="modalFormOk"></stock-out-order-direct-modal>
    <a-modal
      :title="popup.title"
      :confirm-loading="popup.confirmLoading"
      :width="popup.width"
      @cancel="popup.visible = false"
      :visible="popup.visible">
      <a-form :form="form" >
      <a-row>
          <a-col :span="12">
            <a-form-item label="打印机" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['printerName', validatorRules.printBtw]" :trigger-change="true" dictCode="print_out_batch_printer_name" placeholder="请选择打印机" />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="打印模板" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['printBtw', validatorRules.printerName]" :trigger-change="true" dictCode="print_out_batch_print_btw" placeholder="请选择打印模板" />
            </a-form-item>
          </a-col>
          <a-col :span="12" style="padding-left: 25px">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-checkbox v-decorator="['isPrintParent', validatorRules.isPrintParent]" >
                是否打印结余标签
              </a-checkbox>
            </a-form-item>
          </a-col>
      </a-row>
      </a-form>
      <template slot="footer">
        <a-button  @click="handlePopupCancel">
          取消
        </a-button>
        <a-button  type="primary"  @click="handleClickPrint">
          打印
        </a-button>
      </template>
    </a-modal>
  </a-card>

</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import StockOutOrderModal from './modules/StockOutOrderModal'
  import StockOutOrderDirectModal from './modules/StockOutOrderDirectModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import { postAction, putAction } from '@/api/manage'
  export default {
    name: 'StockOutOrderList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      StockOutOrderModal,
      StockOutOrderDirectModal
    },
    data () {
      return {
        description: '出库单管理页面',
        form: this.$form.createForm(this),
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        validatorRules: {
          printBtw: {
            rules: [
              { required: true, message: '请选择打印机!'},
            ]
          },
          printerName: {
            rules: [
              { required: true, message: '请选择打印机模板!'},
            ]
          },
          isPrintParent: {
            rules: [
              { required: false, message: '请选择是否打印结余标签!'},
            ]
          },
        },
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
            title:'出库单编码',
            align:"center",
            sorter: true,
            dataIndex: 'code'
          },
          {
            title:'物料编码',
            align:"center",
            sorter: true,
            dataIndex: 'materialCode'
          },
          {
            title:'物料名称',
            align:"center",
            sorter: true,
            dataIndex: 'materialName'
          },
          {
            title:'物料SN',
            align:"center",
            sorter: true,
            dataIndex: 'batchSn'
          },
          {
            title:'状态',
            align:"center",
            sorter: true,
            dataIndex: 'status_dictText'
          },
          {
            title:'仓库名称',
            align:"center",
            sorter: true,
            dataIndex: 'warehouseName'
          },
          {
            title:'库位编号',
            align:"center",
            sorter: true,
            dataIndex: 'locationCode'
          },
          {
            title:'批次编码',
            align:"center",
            sorter: true,
            dataIndex: 'batchCode'
          },
          {
            title:'领料单号',
            align:"center",
            sorter: true,
            dataIndex: 'pickCode'
          },
          {
            title:'盘点单号',
            align:"center",
            sorter: true,
            dataIndex: 'inventoryCode'
          },
          {
            title:'出库数量',
            align:"center",
            sorter: true,
            dataIndex: 'outQty'
          },
          {
            title:'出库时间',
            align:"center",
            sorter: true,
            dataIndex: 'outTime'
          },
          {
            title:'出库操作人',
            align:"center",
            dataIndex: 'outUser'
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
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/stock/stockOutOrder/list",
          delete: "/stock/stockOutOrder/delete",
          deleteBatch: "/stock/stockOutOrder/deleteBatch",
          exportXlsUrl: "/stock/stockOutOrder/exportXls",
          importExcelUrl: "stock/stockOutOrder/importExcel",

        },
        dictOptions:{},
        superFieldList:[],
        popup: {
          visible: false,
          title: '打印',
          confirmLoading: true,
          width: 800
        }
      }
    },
    created() {
      this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },

      isEnable: function() {
        // return true;
        if (this.selectedRowKeys.length> 0) {
          let selected = this.dataSource.filter(item=>{
            return this.selectedRowKeys.includes(item.id) && item.status === "WAIT"
          });
          return !(selected.length > 0);
        }
        return true;
      },

      isEnablePrint: function() {
        return !(this.selectedRowKeys.length> 0);
      }
    },
    methods: {
      handleOutOrderDirect() {
        this.$refs.modalDirectForm.visible = true;
      },
      handlePopupCancel() {
        this.popup.visible = false
      },
      handlePrint() {
        this.popup.visible = true
        this.form.resetFields();
      },
      handleClickPrint() {
        this.form.validateFields((err, values) => {
          if (!err) {
            console.log('Received values of form: ', values);
            let ids =  this.selectedRowKeys.join(',');
            let params = {
              ...values,
              ids
            };
            postAction('/stock/outOrder/printOutBatch', params).then(res=>{
              if(res.success){
                this.$message.success(res.message);
                this.popup.visible = false;
              }else{
                this.$message.warning(res.message);
              }
            })
          }
        });
      },
      handleOut() {
        putAction('/stock/outOrder/execute', this.selectedRowKeys).then(res=>{
          if(res.success){
            this.$message.success(res.message);
            this.selectedRowKeys = [];
            this.modalFormOk();
          }else{
            this.$message.warning(res.message);
          }
        });
      },
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'code',text:'出库单编码',dictCode:''})
        fieldList.push({type:'popup',value:'materialCode',text:'物料编码', popup:{code:'bb_hh_material',field:'id',orgFields:'id',destFields:'material_id'}})
        fieldList.push({type:'string',value:'materialName',text:'物料名称',dictCode:''})
        fieldList.push({type:'string',value:'status',text:'状态',dictCode:'stock_out_status'})
        fieldList.push({type:'string',value:'outType',text:'出库类型',dictCode:'stock_out_type'})
        fieldList.push({type:'popup',value:'warehouseCode',text:'仓库编号', popup:{code:'bb_hh_warehouse',field:'id',orgFields:'id',destFields:'warehouse_id'}})
        fieldList.push({type:'string',value:'warehouseName',text:'仓库名称',dictCode:''})
        fieldList.push({type:'popup',value:'locationCode',text:'库位编号', popup:{code:'bb_hh_location',field:'id',orgFields:'id',destFields:'location_id'}})
        fieldList.push({type:'string',value:'batchCode',text:'批次编码',dictCode:''})
        fieldList.push({type:'string',value:'pickCode',text:'领料单号',dictCode:''})
        fieldList.push({type:'string',value:'inventoryCode',text:'盘点单号',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'outQty',text:'出库数量',dictCode:''})
        fieldList.push({type:'datetime',value:'outTime',text:'出库时间'})
        fieldList.push({type:'string',value:'outUser',text:'出库操作人',dictCode:''})
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
  @import '~@assets/less/common.less';
</style>