<template>
  <a-card :bordered="false" :class="'cust-erp-sub-tab'">
    <!-- 操作按钮区域 -->
    <div class="table-operator">
    </div>

    <!-- table区域-begin -->
    <div>
      <!-- <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div> -->

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        class="deeps"
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
        </template>

      </a-table>
    </div>
  </a-card>
</template>

<script>

  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import {getAction} from '@/api/manage.js'

  export default {
    name: "InOrderList",
    mixins: [JeecgListMixin],
    props:{
      mainCode:{
        type:String,
        default:'',
        required:false
      }
    },
    watch:{
      mainCode:{
        immediate: true,
        handler(val) {
          if(!this.mainCode){
            this.clearList()
          }else{
            this.queryParam['inventoryCode'] = val
            this.loadData(1);
          }
        }
      }
    },
    data () {
      return {
        disableMixinCreated:true,
        // 表头
        columns: [
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
            dataIndex: 'createTime'
          },
          // {
          //   title:'更新人',
          //   align:"center",
          //   dataIndex: 'updateBy'
          // },
          // {
          //   title:'更新日期',
          //   align:"center",
          //   dataIndex: 'updateTime'
          // },
          // {
          //   title: '操作',
          //   dataIndex: 'action',
          //   align:"center",
          //   fixed:"right",
          //   width:147,
          //   scopedSlots: { customRender: 'action' }
          // }
        ],
        dataSource: [],
        url: {
          list: "/inventory/inventory/findInOrderByInventoryCode"
        },
        dictOptions:{
         type:[],
         status:[],
         locationId:[],
         materialId:[],
        },
        superFieldList:[],
      }
    },
    methods: {
      clearList(){
        this.dataSource=[]
        this.selectedRowKeys=[]
        this.ipagination.current = 1
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
        var params = this.getQueryParams();//查询条件
        params.inventoryCode= this.mainCode;
        this.loading = true;
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            //update-begin---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
            this.dataSource = res.result.records||res.result;
            if(res.result.total)
            {
              this.ipagination.total = res.result.total;
            }
            //update-end---author:zhangyafei    Date:20201118  for：适配不分页的数据列表------------
          }
          if(res.code===510){
            this.$message.warning(res.message)
          }
          this.loading = false;
        })
      }
    }
  }
</script>
<style scoped>
  .deeps >>> tr th{
       white-space: nowrap;
  } 
   .deeps >>> tr td{
       white-space: nowrap;
  } 
</style>
