<template>
  <a-card :bordered="false" :class="'cust-erp-sub-tab'">

    <!-- table区域-begin -->
    <div>

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

    <feedRecord-modal ref="modalForm" @ok="modalFormOk" :mainId="mainId"></feedRecord-modal>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import FeedRecordModal from './modules/FeedRecordModal'

  export default {
    name: "FeedRecordList",
    mixins:[JeecgListMixin],
    components: { FeedRecordModal },
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
        description: '生产订单管理页面',
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
            title:'生产订单号',
            align:"center",
            sorter: true,
            dataIndex: 'blendLotId'
          },
          {
            title:'原材料批次',
            align:"center",
            sorter: true,
            dataIndex: 'batchCode'
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
            title:'投料数量',
            align:"center",
            sorter: true,
            dataIndex: 'feedQty'
          },
          {
            title:'投料类型',
            align:"center",
            sorter: true,
            dataIndex: 'type_dictText',
          },
          {
            title:'投料工位',
            align:"center",
            sorter: true,
            dataIndex: 'stationCode'
          },
          {
            title:'投料仓号',
            align:"center",
            sorter: true,
            dataIndex: 'stationWarehouseCode'
          },
          {
            title:'投料人',
            align:"center",
            dataIndex: 'createBy'
          },
          {
            title:'投料时间',
            align:"center",
            sorter: true,
            dataIndex: 'createTime'
          },
          {
            title:'出库单编码',
            align:"center",
            dataIndex: 'outOrderCode'
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
          list: "/orderBlendLot/blendLot/listFeedRecordByMainId",
          delete: "/orderBlendLot/blendLot/deleteFeedRecord",
          deleteBatch: "/orderBlendLot/blendLot/deleteBatchFeedRecord",
          exportXlsUrl: "/orderBlendLot/blendLot/exportFeedRecord",
          importUrl: "/orderBlendLot/blendLot/importFeedRecord",
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
        fieldList.push({type:'string',value:'blendLotId',text:'生产订单号',dictCode:''})
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
