<template>
  <a-card :bordered="false">
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
         <a-col :xl="10" :lg="11" :md="12" :sm="24">
            <a-form-item label="日期">
              <j-date placeholder="请选择开始日期" class="query-group-cust" v-model="queryParam.updateTimeBegin"></j-date>
              <span class="query-group-split-cust"></span>
              <j-date placeholder="请选择结束日期" class="query-group-cust" v-model="queryParam.updateTimeEnd"></j-date>
            </a-form-item>
          </a-col>

          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <div>
      <a-list :grid="{ gutter: 16, xs: 1, sm: 2, md: 2, lg: 2.5}" :data-source="dataSource">
        <a-list-item slot="renderItem" slot-scope="item">
          <a-card :title="item.code">
            <a-table rowKey="code" size="small" :columns="columns" :data-source="item.materialList" :pagination=false bordered>
              <template slot="imgSlot">
                no
              </template>
            </a-table>
          </a-card>
        </a-list-item>
      </a-list>
      <template>
        <a-pagination :current="ipagination.current" :total="ipagination.total" @change="onChange" />
      </template>
    </div>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
  import { getAction} from '@/api/manage'

  export default {
    name: 'locationList',
    mixins:[JeecgListMixin, mixinDevice],
    data () {
      return {
        url: {
          list: "/api/stock/by/location",
        },
        columns: [
          {
            title:'物料编码',
            align:"center",
            dataIndex: 'code'
          },
          {
            title:'在库数量',
            align:"center",
            dataIndex: 'qty'
          },
          {
            title:'入库数量',
            align:"center",
            dataIndex: 'inQty'
          },
          {
            title:'出库数量',
            align:"center",
            dataIndex: 'outQty'
          }
        ],
        ipagination:{
          current: 1,
          pageSize: 12,
          total: 0
        },
      }
    },
    created() {
    },
    onChange(page) {
      loadData(page);
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
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
        params.pageNo = this.ipagination.current;
        params.pageSize = this.ipagination.pageSize;
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
      },
    }
  }
</script>
<style lang="less" scoped>
  @import '~@assets/less/common.less';
  .item {
    h3 {
      text-align: center;
    }
    /deep/ th {
      border: 1px solid #f00;
    }
  }

 .item {
   /deep/ .ant-empty {
     display: none;
   }
 }



</style>