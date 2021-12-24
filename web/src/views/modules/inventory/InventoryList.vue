<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="盘点单号">
              <a-input placeholder="请输入盘点单号" v-model="queryParam.no"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="仓库类型">
              <j-dict-select-tag placeholder="请选择仓库类型" v-model="queryParam.type" dictCode="inventory_type"/>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="状态">
                <j-dict-select-tag placeholder="请选择状态" v-model="queryParam.status" dictCode="inventory_status"/>
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
      <a-button type="primary" @click="handleSend">签发</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('盘点单')">导出</a-button>
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

    <a-tabs defaultActiveKey="1" @change="TabChange">
      <a-tab-pane tab="盘点清单" key="1" >
        <InventoryDetailList :selectRow="selectionRows[0]" :mainId="selectedMainId" />
      </a-tab-pane>
      <a-tab-pane tab="入库单" key="2">
        <InOrderList ref="InOrderList" :mainCode="selectedMainCode" />
      </a-tab-pane>
      <a-tab-pane tab="出库单" key="3" >
        <OutOrderList ref="OutOrderList" :mainCode="selectedMainCode" />
      </a-tab-pane>
    </a-tabs>

    <!-- <inventory-modal ref="modalForm" @ok="modalFormOk"></inventory-modal> -->
    <InventoryFormDialog :location-data="locationData" ref="formRef" @ok="modalFormOk"></InventoryFormDialog>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import InventoryModal from './modules/InventoryModal'
  import InventoryFormDialog from '@/views/modules/inventory/modules/InventoryFormDialog'
  import { getAction } from '@/api/manage'
  import InventoryDetailList from './InventoryDetailList'
  import InOrderList from '@/views/modules/inventory/modules/InOrderList'
  import OutOrderList from '@/views/modules/inventory/modules/OutOrderList'
  import {initDictOptions,filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import '@/assets/less/TableExpand.less'

  export default {
    name: "InventoryList",
    mixins:[JeecgListMixin],
    components: {
      InventoryDetailList,
      InventoryModal,
      InOrderList,
      OutOrderList,
      InventoryFormDialog
    },
    data () {
      return {
        description: '盘点单管理页面',
        // 表头
        columns: [
          {
            title:'盘点单号',
            align:"center",
            dataIndex: 'no'
          },
          {
            title:'仓库编码',
            align:"center",
            dataIndex: 'warehostCode'
          },
          {
            title:'仓库名称',
            align:"center",
            dataIndex: 'warehouseName'
          },
          {
            title:'仓库类型',
            align:"center",
            dataIndex: 'type_dictText',
          },
          {
            title:'状态',
            align:"center",
            dataIndex: 'status_dictText',
          },
          {
            title:'库位',
            align:"center",
            dataIndex: 'locationId_dictText',
          },
          {
            title:'物料',
            align:"center",
            dataIndex: 'materialId_dictText',
          },
          {
            title:'备注',
            align:"center",
            dataIndex: 'remarks'
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
          list: "/inventory/inventory/list",
          delete: "/inventory/inventory/delete",
          deleteBatch: "/inventory/inventory/deleteBatch",
          exportXlsUrl: "/inventory/inventory/exportXls",
          importExcelUrl: "inventory/inventory/importExcel",
        },
        dictOptions:{
         type:[],
         status:[],
         locationId:[],
         materialId:[],
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
        selectedMainCode: '',
        superFieldList:[],
        locationData: []
      }
    },
    created() {
      this.getSuperFieldList();
      this.getLocationList()
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    methods: {
      TabChange(value) {
        if (value === '2') {
          this.$nextTick(() => {
            this.$refs.InOrderList.loadData(1)
          })
          
        }
        if (value === '3') {
          this.$nextTick(() => {
            this.$refs.OutOrderList.loadData(1)
          })
        }
      },
      handleAdd() {
        this.$refs.formRef.add()
      },
      handleEdit(item) {
        this.$refs.formRef.edit(item)
      },
      // 获取库位列表 （当前设置的库位下）
      getLocationList() {
        getAction('/warehouse/warehouse/listLocationByUserId').then(res => {
          console.log(res, 'res')
          if (res.code === 200) {
            this.locationData = res.result;
          }
        })
      },
      initDictConfig(){
        initDictOptions('inventory_type').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'type', res.result)
          }
        })
        initDictOptions('inventory_status').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'status', res.result)
          }
        })
        initDictOptions('').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'locationId', res.result)
          }
        })
        initDictOptions('').then((res) => {
          if (res.success) {
            this.$set(this.dictOptions, 'materialId', res.result)
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
      handleSend() {
        if (this.selectionRows.length === 0) {
          this.$message.error('请勾选盘点单')
          return
        }
        if (this.selectionRows[0].status !== 'FINISHED') {
          this.$message.error('当前勾选的盘点单未完成盘点')
          return
        }
        let ids = this.selectedRowKeys.join(',');
        this.$confirm({
          title: '提示',
          content: '确定要进行签发吗？',
          onOk: () => {
            getAction('/inventory/inventory/issue', {
              id: ids
            }).then(res => {
              if (res.code === 200) {
                this.$message.success('操作成功')
              } else {
                this.$message.error(res.message)
              }
            })
          }
        })
      },
      onClearSelected() {
        this.selectedRowKeys = [];
        this.selectionRows = [];
        this.selectedMainId=''
        this.selectedMainCode = ''
      },
      onSelectChange(selectedRowKeys, selectionRows) {
        this.selectedMainId=selectedRowKeys[0]
        this.selectedMainCode = selectionRows[0].no
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
  @import '~@assets/less/common.less'
</style>