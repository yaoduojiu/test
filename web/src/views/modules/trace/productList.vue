<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
     <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="订单编号">
              <a-input placeholder="请输入订单编号" v-model="queryParam.orderCode"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="混料批次号">
              <a-input placeholder="请输入混料批次号" v-model="queryParam.blendLotId"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="产成品物料编码">
                <j-popup placeholder="请选择产成品物料编码" v-model="queryParam.materialCode" code="material_product" org-fields="code,name,id" dest-fields="material_code,material_name,material_id" :field="getPopupField('material_code,material_name,material_id')"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="产品名称">
                <a-input placeholder="请输入产品名称" v-model="queryParam.materialName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="状态">
                <j-dict-select-tag placeholder="请选择状态" v-model="queryParam.status" dictCode="order_status"/>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="开始时间">
                <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="queryParam.startTime_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="queryParam.startTime_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="结束时间">
                <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="queryParam.endTime_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="queryParam.endTime_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="创建日期">
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择开始时间" class="query-group-cust" v-model="queryParam.createTime_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" placeholder="请选择结束时间" class="query-group-cust" v-model="queryParam.createTime_end"></j-date>
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

    <!-- table区域-begin -->
    <div>
      <!-- <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div> -->

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
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange,  type:'radio'}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
      </a-table>
    </div>

    <a-tabs defaultActiveKey="1">
      <a-tab-pane v-for="(item, index) in tabList" :tab="item.title" :key="index" >
        <List :url="item.url" :param='getParam(item.param)' :columns="item.columns"></List>
      </a-tab-pane>
    </a-tabs>

  </a-card>
</template>
<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { getAction } from '@/api/manage'
  import List from './list'
  import {initDictOptions,filterMultiDictText} from '@/components/dict/JDictSelectUtil'
export default {
  name: "MaterialList",
  mixins:[JeecgListMixin],
  components: {
   List
  },
  data () {
    return {
      // 表头
      columns: [
          {
            title:'订单编号',
            align:"center",
            sorter: true,
            dataIndex: 'orderCode'
          },
          {
            title:'混料批次号',
            align:"center",
            sorter: true,
            dataIndex: 'blendLotId'
          },
          {
            title:'产成品物料编码',
            align:"center",
            sorter: true,
            dataIndex: 'materialCode'
          },
          {
            title:'产品名称',
            align:"center",
            sorter: true,
            dataIndex: 'materialName'
          },
          {
            title:'状态',
            align:"center",
            sorter: true,
            dataIndex: 'status_dictText'
          },
          {
            title:'生产数量',
            align:"center",
            sorter: true,
            dataIndex: 'qty'
          },
          {
            title:'实际数量',
            align:"center",
            sorter: true,
            dataIndex: 'actualQty'
          },
          {
            title:'开始时间',
            align:"center",
            sorter: true,
            dataIndex: 'startTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'结束时间',
            align:"center",
            sorter: true,
            dataIndex: 'endTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          }
      ],
      url: {
        list: "/orderBlendLot/blendLot/list",
        // delete: "/pick/pickList/delete",
        // deleteBatch: "/pick/pickList/deleteBatch",
        // exportXlsUrl: "/pick/pickList/exportXls",
        // importExcelUrl: "pick/pickList/importExcel",
      },
      dictOptions:{
        status:[],
      },
      /* 分页参数 */
      ipagination:{
        current: 1,
        pageSize: 5,
        pageSizeOptions: ['5', '10', '50'],
        showTotal: (total, range) => {
          return range[0] + "-" + range[1] + " 共" + total + "条"
        },
        showQuickJumper: true,
        showSizeChanger: true,
        total: 0
      },
      selectedMainId:'',
      selectedCode: '',
      selectedId: '',
      superFieldList:[],
      tabList: [
        {
          title: '投料记录',
          url: 'retrospect/product/findFeedRecordList',
          param: 'blendLotId',
          columns: [
            {
              title:'生产订单编号',
              align:"center",
              dataIndex: 'orderCode'
            },
            {
              title:'混料批次号',
              align:"center",
              dataIndex: 'blendLotId'
            },
            {
              title:'原材料批次',
              align:"center",
              dataIndex: 'batchCode'
            },
            {
              title:'原材料物料编码',
              align:"center",
              dataIndex: 'materialCode'
            },
            {
              title:'原材料物料名称',
              align:"center",
              dataIndex: 'materialName'
            },
            {
              title:'投料数量',
              align:"center",
              dataIndex: 'feedQty'
            },
            {
              title:'投料类型',
              align:"center",
              dataIndex: 'type_dictText'
            },
            {
              title:'投料工位',
              align:"center",
              dataIndex: 'stationCode'
            },
            {
              title:'投料仓号',
              align:"center",
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
              dataIndex: 'createTime'
            },
          ],

        },
        {
          title: '入库记录',
          url: 'retrospect/product/findStockInOrderList',
          param: 'blendLotId',
          columns: [
            {
              title:'入库单编号',
              align:"center",
              dataIndex: 'code'
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
              title:'入库数量',
              align:"center",
              dataIndex: 'inQty'
            },
            {
              title:'状态',
              align:"center",
              dataIndex: 'inStatus_dictText'
            },
            {
              title:'入库类型',
              align:"center",
              dataIndex: 'batchType_dictText'
            },
            {
              title:'仓库编号',
              align:"center",
              dataIndex: 'warehouseCode'
            },
            {
              title:'批次编号',
              align:"center",
              dataIndex: 'batchCode'
            },
            {
              title:'盘点单号',
              align:"center",
              dataIndex: 'inventoryCode'
            },
            {
              title:'入库时间',
              align:"center",
              dataIndex: 'inTime'
            }
          ],

        },
        {
          title: '出库记录',
          url: 'retrospect/product/findStockOutOrderList',
          param: 'blendLotId',
          columns: [
            {
              title:'出库单编号',
              align:"center",
              dataIndex: 'code'
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
              title:'出库类型',
              align:"center",
              dataIndex: 'outType_dictText'
            },
            {
              title:'出库数量',
              align:"center",
              dataIndex: 'outQty'
            },
            {
              title:'状态',
              align:"center",
              dataIndex: 'status_dictText'
            },
            {
              title:'仓库名称',
              align:"center",
              dataIndex: 'warehouseName'
            },
            {
              title:'库位编号',
              align:"center",
              dataIndex: 'locationCode'
            },
            {
              title:'批次编号',
              align:"center",
              dataIndex: 'batchCode'
            }
          ],

        },
      ]
    }
  },
  created() {
    this.getSuperFieldList();
  },
  computed: {
    importExcelUrl: function(){
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    },
    getParam(item) {
      return function(item) {
        if (item === 'blendLotId') {
          return {
            blendLotId: this.selectedId
          }
        }
      }
    }
  },
  methods: {
    initDictConfig(){
      initDictOptions('sys_user,realname,id').then((res) => {
        if (res.success) {
          this.$set(this.dictOptions, 'pickUser', res.result)
        }
      })
      initDictOptions('pick_list_status').then((res) => {
        if (res.success) {
          this.$set(this.dictOptions, 'status', res.result)
        }
      })
    },
    clickThenSelect(record) {
      return {
        on: {
          click: () => {
            this.onSelectChange(record.id.split(","), [record]);
          }
        }
      }
    },
    onClearSelected() {
      this.selectedRowKeys = [];
      this.selectionRows = [];
      this.selectedMainId=''
      this.selectedCode = ''
    },
    onSelectChange(selectedRowKeys, selectionRows) {
      this.selectedMainId=selectedRowKeys[0]
      this.selectedRowKeys = selectedRowKeys;
      this.selectionRows = selectionRows;
      this.selectedCode = selectionRows[0].code;
      this.selectedId = selectionRows[0].blendLotId;
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
      this.onClearSelected()
      var params = this.getQueryParams();//查询条件
      this.loading = true;
      getAction(this.url.list, params).then((res) => {
        if (res.success) {
          this.dataSource = res.result.records;
          this.ipagination.total = res.result.total;
        }
        if(res.code===510){
          this.$message.warning(res.message)
        }
        this.loading = false;
      })
    },
     getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'orderCode',text:'订单编号',dictCode:''})
        fieldList.push({type:'string',value:'blendLotId',text:'混料批次号',dictCode:''})
        fieldList.push({type:'popup',value:'materialCode',text:'产成品物料编码', popup:{code:'material_product',field:'code',orgFields:'code',destFields:'material_code'}})
        fieldList.push({type:'string',value:'materialName',text:'产品名称',dictCode:''})
        fieldList.push({type:'string',value:'status',text:'状态',dictCode:'order_status'})
        fieldList.push({type:'BigDecimal',value:'qty',text:'生产数量',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'actualQty',text:'实际数量',dictCode:''})
        fieldList.push({type:'date',value:'startTime',text:'开始时间'})
        fieldList.push({type:'date',value:'endTime',text:'结束时间'})
        fieldList.push({type:'string',value:'createBy',text:'创建人',dictCode:''})
        fieldList.push({type:'datetime',value:'createTime',text:'创建日期'})
        this.superFieldList = fieldList
      }
  }
}
</script>