<template>
  <a-card :bordered="false" :class="'cust-erp-sub-tab'">
    <!-- 操作按钮区域 -->

    <!-- table区域-begin -->
    <div class="ant-tables">
      <a-table
        class="ant-tables"
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :loading ='loading'
        :scroll="{x:true}"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
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

      </a-table>
    </div>

  </a-card>
</template>

<script>
import { getAction } from '../../../api/manage';

  export default {
    name: "PickOutList",
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
            this.queryParam['pickCode'] = val
            this.loadData(1);
          }
        }
      }
    },
    data () {
      return {
        description: '领料单管理页面',
        disableMixinCreated:true,
        loading: false,
        // 表头
        columns: [
          {
            title: 'ID',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
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
          },
          {
            title:'盘点单号',
            align:"center",
            dataIndex: 'inventoryCode'
          },
          {
            title:'出库时间',
            align:"center",
            dataIndex: 'outTime'
          },
          {
            title:'出库人',
            align:"center",
            dataIndex: 'outUser'
          },
        ],
        url: {
          list: "/api/pickOrder/getStockOutOrderByCode",
          // delete: "/pick/pickList/deletePickDetail",
          // deleteBatch: "/pick/pickList/deleteBatchPickDetail",
          // exportXlsUrl: "/pick/pickList/exportPickDetail",
          // importUrl: "/pick/pickList/importPickDetail",
        },
        dictOptions:{
         status:[],
        },
        superFieldList:[],
        dataInfo: {},
        queryParam: {
        },
        dataSource: [],
        ipagination:{
          current: 1,
          pageSize: 5,
          pageSizeOptions: ['5', '10', '20'],
          showTotal: (total, range) => {
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
      }
    },
    created() {
    },
    computed: {
    },
    methods: {
      handleTableChange(pagination, filters, sorter) {
        this.ipagination = pagination;
        this.loadData();
      },
      handlePickOut() {
        if (this.selectedRowKeys.length === 1) {
          console.log(this.selectedRowKeys);
          this.dataInfo = this.dataSource.find(item => this.selectedRowKeys.includes(item.id));
          this.$refs.pickOutModalForm.visible = true;
        }
        
      },
      clearList(){
        this.dataSource=[]
        this.ipagination.current = 1
      },
      loadData(page) {
        this.loading = true;
        let data = Object.assign({pageNo:page || this.ipagination.current, pageSize: this.ipagination.pageSize}, this.queryParam);
        getAction('/api/pickOrder/getStockOutOrderByCode', data).then(res => {
          this.dataSource = res.result.records;
          this.ipagination.total = res.result.total;
          this.ipagination.pageSize = res.result.size;
          this.loading = false;
        });
      },
    }
  }
</script>
<style scoped>
  .ant-tables >>> tr th{
       white-space: nowrap;
  } 
   .ant-tables >>> tr td{
       white-space: nowrap;
  } 
  
</style>
