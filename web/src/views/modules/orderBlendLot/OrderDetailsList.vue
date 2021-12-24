<template>
  <a-card :bordered="false" :class="'cust-erp-sub-tab'">
    <!-- 操作按钮区域 -->
    <div class="table-operator" v-if="mainId">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('配料明细')">导出</a-button>
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

    <orderDetails-modal ref="modalForm" @ok="modalFormOk" :mainId="mainId"></orderDetails-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import OrderDetailsModal from './modules/OrderDetailsModal'

  export default {
    name: "OrderDetailsList",
    mixins:[JeecgListMixin],
    components: { OrderDetailsModal },
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
            this.queryParam['orderId'] = val
            this.loadData(1);
          }
        }
      }
    },
    data () {
      return {
        description: '投料订单管理页面',
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
            title:'原材料物料编码',
            align:"center",
            sorter: true,
            dataIndex: 'materialCode'
          },
          {
            title:'原材料物料名称',
            align:"center",
            sorter: true,
            dataIndex: 'materialName'
          },
          {
            title:'理论数量',
            align:"center",
            sorter: true,
            dataIndex: 'theoryQty'
          },
          {
            title:'实际数量',
            align:"center",
            sorter: true,
            dataIndex: 'actualQty'
          },
          {
            title:'偏差数量',
            align:"center",
            sorter: true,
            dataIndex: 'deviationQty'
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
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' },
          }
        ],
        url: {
          list: "/orderBlendLot/blendLot/listOrderDetailsByMainId",
          delete: "/orderBlendLot/blendLot/deleteOrderDetails",
          deleteBatch: "/orderBlendLot/blendLot/deleteBatchOrderDetails",
          exportXlsUrl: "/orderBlendLot/blendLot/exportOrderDetails",
          importUrl: "/orderBlendLot/blendLot/importOrderDetails",
        },
        dictOptions:{
         status:[],
         lineId:[],
         orderId:[],
        },
        superFieldList:[],
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
      clearList(){
        this.dataSource=[]
        this.selectedRowKeys=[]
        this.ipagination.current = 1
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'orderCode',text:'订单编号',dictCode:''})
        fieldList.push({type:'string',value:'blendLotId',text:'混料批次号',dictCode:''})
        fieldList.push({type:'popup',value:'materialCode',text:'产成品物料编码', popup:{code:'	material_product',field:'code',orgFields:'code',destFields:'materrial_code'}})
        fieldList.push({type:'string',value:'materialName',text:'产品名称',dictCode:''})
        fieldList.push({type:'string',value:'materialId',text:'物料ID',dictCode:''})
        fieldList.push({type:'string',value:'status',text:'状态',dictCode:'order_status'})
        fieldList.push({type:'BigDecimal',value:'qty',text:'生产数量',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'actualQty',text:'实际数量',dictCode:''})
        fieldList.push({type:'string',value:'lineId',text:'产线',dictCode:'line,line_name,id'})
        fieldList.push({type:'string',value:'orderId',text:'投料订单',dictCode:'hh_order,code,id'})
        fieldList.push({type:'date',value:'startTime',text:'开始时间'})
        fieldList.push({type:'date',value:'endTime',text:'结束时间'})
        fieldList.push({type:'string',value:'createBy',text:'创建人',dictCode:''})
        fieldList.push({type:'datetime',value:'createTime',text:'创建日期'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>
