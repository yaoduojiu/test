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
                <a-input placeholder="请输入产成品物料编码" v-model="queryParam.materialCode"/>
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

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
       <a-button @click="handlePrinter" type="primary" icon="printer">打印</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('生产订单')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
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
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange, type:'radio'}"
        :customRow="clickThenSelect"
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

    <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="配料明细" key="1" >
        <OrderDetailsList :mainId="selectedMainId" />
      </a-tab-pane>
      <a-tab-pane tab="投料记录" key="2" forceRender>
        <FeedRecordList :mainId="selectedMainId" />
      </a-tab-pane>
    </a-tabs>

    <blend-lot-modal ref="modalForm" @ok="modalFormOk"></blend-lot-modal>
    <PrintModal ref="PrintModal" :ids="currentId" @ok="printFormOK"></PrintModal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import BlendLotModal from './modules/BlendLotModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import PrintModal from '@/views/modules/orderBlendLot/modules/PrintModal'
  import { getAction } from '@/api/manage'
  import OrderDetailsList from './OrderDetailsList'
  import FeedRecordList from './FeedRecordList'
  import {initDictOptions} from '@/components/dict/JDictSelectUtil'


  export default {
    name: 'BlendLotList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      OrderDetailsList,
      FeedRecordList,
      BlendLotModal,
      PrintModal
    },
    data () {
      return {
        description: '生产订单管理页面',
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
            dataIndex: 'status_dictText',
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
            title:'产线',
            align:"center",
            dataIndex: 'lineId_dictText',
          },
          {
            title:'投料订单',
            align:"center",
            dataIndex: 'orderId_dictText',
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
          },
          {
            title:'蛋白配方名称',
            align:"center",
            dataIndex: 'dbFormulaname'
          },
          {
            title:'蛋白配方重量',
            align:"center",
            dataIndex: 'dbFormulaweiht'
          },
          {
            title:'蛋白总批次',
            align:"center",
            dataIndex: 'dbTotalbatches'
          },
          {
            title:'蛋白打包状态',
            align:"center",
            dataIndex: 'dbStatus_dictText',
          },
          {
            title:'添加剂1配方名称',
            align:"center",
            dataIndex: 'db1Formulaname'
          },
          {
            title:'添加剂1配方重量',
            align:"center",
            dataIndex: 'db1Formulaweiht'
          },
          {
            title:'添加剂1总批次',
            align:"center",
            dataIndex: 'db1Totalbatches'
          },
          {
            title:'添加剂1打包状态',
            align:"center",
            dataIndex: 'db1Status_dictText',
          },
          {
            title:'添加剂2配方名称',
            align:"center",
            dataIndex: 'db2Formulaname'
          },
          {
            title:'添加剂2配方重量',
            align:"center",
            dataIndex: 'db2Formulaweiht'
          },
          {
            title:'添加剂2总批次',
            align:"center",
            dataIndex: 'db2Totalbatches'
          },
          {
            title:'添加剂2打包状态',
            align:"center",
            dataIndex: 'db2Status_dictText',
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
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/orderBlendLot/blendLot/list",
          delete: "/orderBlendLot/blendLot/delete",
          deleteBatch: "/orderBlendLot/blendLot/deleteBatch",
          exportXlsUrl: "/orderBlendLot/blendLot/exportXls",
          importExcelUrl: "orderBlendLot/blendLot/importExcel",
        },
        dictOptions:{
         status:[],
         lineId:[],
         orderId:[],
         dbStatus:[],
         db1Status:[],
         db2Status:[],
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
        superFieldList:[],
        currentId: ''
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      printFormOK() {
        this.currentId = '';
        this.onClearSelected();
        this.$refs.PrintModal.visible = false
      },
      handlePrinter() {
        console.log(this.selectionRows)
        if(this.selectedRowKeys.length !== 1) {
          this.$message.error('请勾选一个生产订单进行操作')
          return
        }
        this.currentId = this.selectedRowKeys[0];
        this.$refs.PrintModal.visible = true;
        this.$refs.PrintModal.initForm()
      },
      initDictConfig(){
        initDictOptions('order_status').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'status', res.result)
          }
        })
        initDictOptions('line,line_name,id').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'lineId', res.result)
          }
        })
        initDictOptions('hh_order,code,id').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'orderId', res.result)
          }
        })
        initDictOptions('pack_status').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'dbStatus', res.result)
          }
        })
        initDictOptions('pack_status').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'db1Status', res.result)
          }
        })
        initDictOptions('pack_status').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'db2Status', res.result)
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
      },
      onSelectChange(selectedRowKeys, selectionRows) {
        this.selectedMainId=selectedRowKeys[0]
        this.selectedRowKeys = selectedRowKeys;
        this.selectionRows = selectionRows;
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
        fieldList.push({type:'string',value:'materialId',text:'物料ID',dictCode:''})
        fieldList.push({type:'string',value:'status',text:'状态',dictCode:'order_status'})
        fieldList.push({type:'BigDecimal',value:'qty',text:'生产数量',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'actualQty',text:'实际数量',dictCode:''})
        fieldList.push({type:'string',value:'lineId',text:'产线',dictCode:'line,line_name,id'})
        fieldList.push({type:'string',value:'orderId',text:'投料订单',dictCode:'hh_order,code,id'})
        fieldList.push({type:'date',value:'startTime',text:'开始时间'})
        fieldList.push({type:'date',value:'endTime',text:'结束时间'})
        fieldList.push({type:'string',value:'dbFormulaname',text:'蛋白配方名称',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'dbFormulaweiht',text:'蛋白配方重量',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'dbTotalbatches',text:'蛋白总批次',dictCode:''})
        fieldList.push({type:'string',value:'dbStatus',text:'蛋白打包状态',dictCode:'pack_status'})
        fieldList.push({type:'string',value:'db1Formulaname',text:'添加剂1配方名称',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'db1Formulaweiht',text:'添加剂1配方重量',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'db1Totalbatches',text:'添加剂1总批次',dictCode:''})
        fieldList.push({type:'string',value:'db1Status',text:'添加剂1打包状态',dictCode:'pack_status'})
        fieldList.push({type:'string',value:'db2Formulaname',text:'添加剂2配方名称',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'db2Formulaweiht',text:'添加剂2配方重量',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'db2Totalbatches',text:'添加剂2总批次',dictCode:''})
        fieldList.push({type:'string',value:'db2Status',text:'添加剂2打包状态',dictCode:'pack_status'})
        fieldList.push({type:'string',value:'createBy',text:'创建人',dictCode:''})
        fieldList.push({type:'datetime',value:'createTime',text:'创建日期'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>