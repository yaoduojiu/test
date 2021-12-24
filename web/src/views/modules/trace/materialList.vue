<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="批次编码">
              <a-input placeholder="请输入批次编码" v-model="queryParam.code"></a-input>
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
              <a-form-item label="状态">
                <j-dict-select-tag placeholder="请选择状态" v-model="queryParam.status" dictCode="batch_status"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="供应商编码">
                <j-popup placeholder="请选择供应商编码" v-model="queryParam.supplierCode" code="pp_supplier" org-fields="code,name" dest-fields="supplier_code,supplier_name" :field="getPopupField('supplier_code,supplier_name')"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="供应商名称">
                <a-input placeholder="请输入供应商名称" v-model="queryParam.supplierName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="生产日期">
                <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="queryParam.productionDate_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="queryParam.productionDate_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="质保日期">
                <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="queryParam.warrantyDate_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="queryParam.warrantyDate_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="来料日期">
                <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="queryParam.inDate_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="queryParam.inDate_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="仓库名称">
                <a-input placeholder="请输入仓库名称" v-model="queryParam.warehouseName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="库位编码">
                <a-input placeholder="请输入库位编码" v-model="queryParam.localtionCode"></a-input>
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
            title:'批次编码',
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
            title:'数量',
            align:"center",
            sorter: true,
            dataIndex: 'qty'
          },
          {
            title:'状态',
            align:"center",
            dataIndex: 'status_dictText'
          },
          {
            title:'批次类型',
            align:"center",
            dataIndex: 'type_dictText'
          },
          {
            title:'在库数量',
            align:"center",
            dataIndex: 'inQty'
          }

        ],
      url: {
        list: "/batchInfo/batchInfo/list",
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
          url: 'retrospect/material/findFeedRecordList',
          param: 'batchId',
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
          url: 'retrospect/material/findStockInOrderList',
          param: 'batchCode',
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
          url: 'retrospect/material/findStockOutOrderList',
          param: 'batchCode',
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
        {
          title: '生产订单',
          url: 'retrospect/material/findBlendLotList',
          param: 'batchCode',
          columns: [
            {
              title:'订单编号',
              align:"center",
              dataIndex: 'orderId_dictText'
            },
            {
              title:'混料批次号',
              align:"center",
              dataIndex: 'blendLotId'
            },
            {
              title:'产品物料编码',
              align:"center",
              dataIndex: 'materialCode'
            },
            {
              title:'产品名称',
              align:"center",
              dataIndex: 'materialName'
            },
            {
              title:'状态',
              align:"center",
              dataIndex: 'status'
            },
            {
              title:'生产数量',
              align:"center",
              dataIndex: 'qty'
            },
            {
              title:'实际数量',
              align:"center",
              dataIndex: 'actualQty'
            },
            {
              title:'开始时间',
              align:"center",
              dataIndex: 'startTime'
            },
            {
              title:'结束时间',
              align:"center",
              dataIndex: 'endTime'
            }
          ],

        },
        {
          title: '投料订单',
          url: 'retrospect/material/findOrderRecordList',
          param: 'batchId',
          columns: [
            {
              title:'订单编号',
              align:"center",
              dataIndex: 'code'
            },
            {
              title:'产品物料编码',
              align:"center",
              dataIndex: 'materialCode'
            },
            {
              title:'产品名称',
              align:"center",
              dataIndex: 'lineName'
            },
            {
              title:'状态',
              align:"center",
              dataIndex: 'status_dictText'
            },
            {
              title:'订单数量',
              align:"center",
              dataIndex: 'qty'
            }
          ],

        }
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
        if (item === 'batchId') {
          return {
            batchId: this.selectedId
          }
        } else {
          return {
            batchCode: this.selectedCode
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
      this.selectedId = selectionRows[0].id;
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
      fieldList.push({type:'string',value:'code',text:'批次编码',dictCode:''})
      fieldList.push({type:'string',value:'materialId',text:'物料编码ID',dictCode:''})
      fieldList.push({type:'popup',value:'materialCode',text:'物料编码', popup:{code:'bb_hh_material',field:'id',orgFields:'id',destFields:'material_id'}})
      fieldList.push({type:'string',value:'materialName',text:'物料名称',dictCode:''})
      fieldList.push({type:'BigDecimal',value:'qty',text:'数量',dictCode:''})
      fieldList.push({type:'string',value:'status',text:'状态',dictCode:'batch_status'})
      fieldList.push({type:'string',value:'type',text:'批次类型',dictCode:'batch_type'})
      fieldList.push({type:'BigDecimal',value:'inQty',text:'在库数量',dictCode:''})
      fieldList.push({type:'popup',value:'supplierCode',text:'供应商编码', popup:{code:'pp_supplier',field:'code',orgFields:'code',destFields:'supplier_code'}})
      fieldList.push({type:'string',value:'supplierName',text:'供应商名称',dictCode:''})
      fieldList.push({type:'date',value:'productionDate',text:'生产日期'})
      fieldList.push({type:'date',value:'warrantyDate',text:'质保日期'})
      fieldList.push({type:'date',value:'inDate',text:'来料日期'})
      fieldList.push({type:'string',value:'warehouseName',text:'仓库名称',dictCode:''})
      fieldList.push({type:'string',value:'locationCode',text:'库位编码',dictCode:''})
      fieldList.push({type:'string',value:'createBy',text:'创建人',dictCode:''})
      fieldList.push({type:'datetime',value:'createTime',text:'创建日期'})
      fieldList.push({type:'string',value:'updateBy',text:'更新人',dictCode:''})
      fieldList.push({type:'datetime',value:'updateTime',text:'更新日期'})
      this.superFieldList = fieldList
    }
  }
}
</script>