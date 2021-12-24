<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="订单编号">
              <j-search-select-tag placeholder="请选择订单编号" v-model="queryParam.orderCode"/>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="混料批次号">
              <a-input placeholder="请输入混料批次号" v-model="queryParam.blendLotId"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="原材料批次">
                <j-popup placeholder="请选择原材料批次" v-model="queryParam.batchCode" code="batch_popup" org-fields="code,id,material_id,material_code,material_name" dest-fields="batch_code,batch_id,material_id,material_code,material_name" :field="getPopupField('batch_code,batch_id,material_id,material_code,material_name')"/>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="物料编码">
                <a-input placeholder="请输入物料编码" v-model="queryParam.materialCode"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="物料名称">
                <a-input placeholder="请输入物料名称" v-model="queryParam.materialName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="投料类型">
                <j-dict-select-tag placeholder="请选择投料类型" v-model="queryParam.type" dictCode="feed_type"/>
              </a-form-item>
            </a-col>
            <a-col :xl="10" :lg="11" :md="12" :sm="24">
              <a-form-item label="投料时间">
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
      <a-button type="primary" icon="download" @click="handleExportXls('投料记录')">导出</a-button>
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

    <feed-record-modal ref="modalForm" @ok="modalFormOk"></feed-record-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import FeedRecordModal from './modules/FeedRecordModal'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'

  export default {
    name: 'FeedRecordList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      FeedRecordModal
    },
    data () {
      return {
        description: '投料记录管理页面',
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
            title:'投料类型',
            align:"center",
            sorter: true,
            dataIndex: 'type_dictText'
          },
          {
            title:'产线',
            align:"center",
            dataIndex: 'lineId_dictText',
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
            title:'物料SN',
            align:"center",
            sorter: true,
            dataIndex: 'batchSn'
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
          list: "/orderFeedRecord/feedRecord/list",
          delete: "/orderFeedRecord/feedRecord/delete",
          deleteBatch: "/orderFeedRecord/feedRecord/deleteBatch",
          exportXlsUrl: "/orderFeedRecord/feedRecord/exportXls",
          importExcelUrl: "orderFeedRecord/feedRecord/importExcel",

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
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'sel_search',value:'orderId',text:'订单编号',dictTable:'hh_order', dictText:'code', dictCode:'id'})
        fieldList.push({type:'string',value:'blendLotId',text:'混料批次号',dictCode:''})
        fieldList.push({type:'popup',value:'batchCode',text:'原材料批次', popup:{code:'batch_popup',field:'code',orgFields:'code',destFields:'batch_code'}})
        fieldList.push({type:'string',value:'materialCode',text:'物料编码',dictCode:''})
        fieldList.push({type:'string',value:'materialName',text:'物料名称',dictCode:''})
        fieldList.push({type:'BigDecimal',value:'feedQty',text:'投料数量',dictCode:''})
        fieldList.push({type:'string',value:'type',text:'投料类型',dictCode:'feed_type'})
        fieldList.push({type:'string',value:'lineId',text:"产线",dictTable:'line', dictText:'line_name', dictCode:'id'})
        fieldList.push({type:'string',value:'stationCode',text:'投料工位',dictCode:''})
        fieldList.push({type:'string',value:'stationWarehouseCode',text:'投料仓号',dictCode:''})
        fieldList.push({type:'string',value:'createBy',text:'投料人',dictCode:''})
        fieldList.push({type:'datetime',value:'createTime',text:'投料时间'})
        fieldList.push({type:'string',value:'outOrderCode',text:'出库单编码',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>