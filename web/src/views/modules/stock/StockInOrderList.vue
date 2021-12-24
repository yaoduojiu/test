<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="入库单编码">
              <a-input placeholder="请输入入库单编码" v-model="queryParam.code"></a-input>
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
              <a-form-item label="入库状态">
                <j-dict-select-tag placeholder="请选择入库状态" v-model="queryParam.inStatus" dictCode="stock_in_status"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="库位编号">
                <a-input placeholder="请输入库位编号" v-model="queryParam.locationCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="批次编号">
                <a-input placeholder="请输入批次编号" v-model="queryParam.batchCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="盘点单号">
                <a-input placeholder="请输入盘点单号" v-model="queryParam.inventoryCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="入库时间">
                <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="queryParam.inTime_begin"></j-date>
                <span class="query-group-split-cust"></span>
                <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="queryParam.inTime_end"></j-date>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="入库操作人">
                <a-input placeholder="请输入入库操作人" v-model="queryParam.inOper"></a-input>
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
      <a-button @click="handleInOrderDirect" type="primary" icon="plus">入库</a-button>
      <a-button @click="handleInOrderExecute" type="primary" icon="plus" :disabled = 'isEnable'>执行入库</a-button>
      <a-button @click="handleInOrderClose" type="primary" icon="plus">关闭</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('入库单')">导出</a-button>
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

    <stock-in-order-modal ref="modalForm" @ok="modalFormOk"></stock-in-order-modal>
    <stock-in-order-direct-modal ref="modalDirectForm" @ok="modalFormOk"></stock-in-order-direct-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import StockInOrderModal from './modules/StockInOrderModal'
  import StockInOrderDirectModal from './modules/StockInOrderDirectModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import { postAction, putAction } from '@/api/manage'

  export default {
    name: 'StockInOrderList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      StockInOrderModal,
      StockInOrderDirectModal
    },
    data () {
      return {
        description: '入库单管理页面',
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
            title:'入库单编码',
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
            title:'入库数量',
            align:"center",
            sorter: true,
            dataIndex: 'inQty'
          },
          {
            title:'入库状态',
            align:"center",
            dataIndex: 'inStatus_dictText'
          },
          {
            title:'库位编号',
            align:"center",
            sorter: true,
            dataIndex: 'locationCode'
          },
          {
            title:'批次编号',
            align:"center",
            sorter: true,
            dataIndex: 'batchCode'
          },
          {
            title:'盘点单号',
            align:"center",
            sorter: true,
            dataIndex: 'inventoryCode'
          },
          {
            title:'入库时间',
            align:"center",
            sorter: true,
            dataIndex: 'inTime',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'入库操作人',
            align:"center",
            dataIndex: 'inOper'
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
          list: "/stock/stockInOrder/list",
          delete: "/stock/stockInOrder/delete",
          deleteBatch: "/stock/stockInOrder/deleteBatch",
          exportXlsUrl: "/stock/stockInOrder/exportXls",
          importExcelUrl: "stock/stockInOrder/importExcel",
          orderInDirect: "/stock/inOrder/direct",
          orderInExecute: "/stock/inOrder/execute",
          orderInClose: "/stock/inOrder/close",
        },
        dictOptions:{},
        superFieldList:[],
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
            return this.selectedRowKeys.includes(item.id) && item.inStatus === "WAIT"
          });
          return !(selected.length > 0);
        }
        return true;
      }
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'code',text:'入库单编码',dictCode:''})
        fieldList.push({type:'popup',value:'materialCode',text:'物料编码', popup:{code:'bb_hh_material',field:'id',orgFields:'id',destFields:'material_id'}})
        fieldList.push({type:'string',value:'materialName',text:'物料名称',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'inQty',text:'入库数量',dictCode:''})
        fieldList.push({type:'string',value:'inStatus',text:'入库状态',dictCode:'stock_in_status'})
        fieldList.push({type:'string',value:'locationCode',text:'库位编号',dictCode:''})
        fieldList.push({type:'string',value:'batchCode',text:'批次编号',dictCode:''})
        fieldList.push({type:'string',value:'inventoryCode',text:'盘点单号',dictCode:''})
        fieldList.push({type:'date',value:'inTime',text:'入库时间'})
        fieldList.push({type:'string',value:'inOper',text:'入库操作人',dictCode:''})
        fieldList.push({type:'string',value:'createBy',text:'创建人',dictCode:''})
        fieldList.push({type:'datetime',value:'createTime',text:'创建日期'})
        fieldList.push({type:'string',value:'updateBy',text:'更新人',dictCode:''})
        fieldList.push({type:'datetime',value:'updateTime',text:'更新日期'})
        this.superFieldList = fieldList
      },
      handleInOrderDirect() {
        this.$refs.modalDirectForm.visible = true;
      },
      handleInOrderExecute() {
        var _this = this;
        if (this.selectedRowKeys.length > 0) {
          this.$confirm({
            title: '确定要执行入库吗?',
            onOk() {
              _this.inOrderExecute();
            },
            onCancel() {},
          });
        }

      },

      inOrderExecute() {
         putAction(this.url.orderInExecute, this.selectedRowKeys).then(res => {
          if(res.success){
            this.$message.success(res.message);
            this.modalFormOk();
          }else{
            this.$message.warning(res.message);
          }
        })
      },

      handleInOrderClose() {

      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>