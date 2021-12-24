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
              <a-input placeholder="请输入物料编码" v-model="queryParam.materialCode"></a-input>
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
                <a-input placeholder="请输入供应商编码" v-model="queryParam.supplierCode"></a-input>
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
                <a-input placeholder="请输入库位编码" v-model="queryParam.locationCode"></a-input>
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
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button @click="handlePrinter" type="primary" icon="printer">打印</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('批次表')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button hidden type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
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
        class="j-table-force-nowrap"
        :scroll="{x:true}"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange, type:'radio'}"
        :customRow="clickThenSelect"
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
      <a-tab-pane tab="批次SN表" key="1" >
        <BatchSnList :mainId="selectedMainId" />
      </a-tab-pane>
    </a-tabs>

    <batch-info-modal ref="modalForm" @ok="modalFormOk"></batch-info-modal>
    <BatchPrintModal :ids="currentId" ref="BatchPrintModal" @ok="printFormOK" />
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import BatchInfoModal from './modules/BatchInfoModal'
  import BatchPrintModal from '@/views/modules/batchInfo/modules/BatchPrintModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import { getAction } from '@/api/manage'
  import BatchSnList from './BatchSnList'
  import {initDictOptions} from '@/components/dict/JDictSelectUtil'
  import '@/assets/less/TableExpand.less'

  export default {
    name: 'BatchInfoList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      BatchSnList,
      BatchInfoModal,
      BatchPrintModal
    },
    data () {
      return {
        description: '批次表管理页面',
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
          },
          {
            title:'供应商编码',
            align:"center",
            dataIndex: 'supplierCode'
          },
          {
            title:'供应商名称',
            align:"center",
            dataIndex: 'supplierName'
          },
          {
            title:'生产日期',
            align:"center",
            dataIndex: 'productionDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'质保日期',
            align:"center",
            dataIndex: 'warrantyDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'来料日期',
            align:"center",
            dataIndex: 'inDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'仓库名称',
            align:"center",
            dataIndex: 'warehouseName'
          },
          {
            title:'库位编码',
            align:"center",
            dataIndex: 'locationCode'
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
          list: "/batchInfo/batchInfo/list",
          delete: "/batchInfo/batchInfo/delete",
          deleteBatch: "/batchInfo/batchInfo/deleteBatch",
          exportXlsUrl: "/batchInfo/batchInfo/exportXls",
          importExcelUrl: "batchInfo/batchInfo/importExcel",
        },
        dictOptions:{
         status:[],
         type:[],
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
      handlePrinter() {
        console.log(this.selectionRows)
        if(this.selectedRowKeys.length !== 1) {
          this.$message.error('请勾选一个批次进行打印')
          return
        }
        this.currentId = this.selectedRowKeys[0]
        this.$refs.BatchPrintModal.visible = true;
        this.$refs.BatchPrintModal.initForm()
      },
      initDictConfig(){
        initDictOptions('batch_status').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'status', res.result)
          }
        })
        initDictOptions('batch_type').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'type', res.result)
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
      printFormOK() {
        this.currentId = '';
        this.onClearSelected();
        this.$refs.BatchPrintModal.visible = false
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
<style scoped>
  @import '~@assets/less/common.less'
</style>