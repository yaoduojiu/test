<template>
    <!-- table区域-begin -->
    <div>
      <!-- <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div> -->

      <a-table
        ref="table"
        size="small"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        @change="handleChangeInTable"
        class="j-table-force-nowrap">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
      </a-table>
    </div>

</template>
<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { getAction } from '@/api/manage'
  import {filterObj} from '@/utils/util'
export default {
  name: "List",
  mixins:[JeecgListMixin],
  props: ['url', 'param', 'columns'],
  // components: {
  //   PickDetailList,
  //   PickListModal,
  //   PickOutList
  // },
  data () {
    return {
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
      superFieldList:[],
    }
  },
  watch:{
    param:{
      immediate: true,
      handler(val) {
        this.loadData(1);
      }
    }
  },
  methods: {

    loadData(arg) {
      if(!Object.values(this.param)[0]) {
        return;
      }
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1;
      }
      // this.onClearSelected()
      var params = this.getQueryParams();//查询条件
      this.loading = true;
      getAction(this.url, params).then((res) => {
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

    handleChangeInTable(ipagination, filters, sorter) {
      //分页、排序、筛选变化时触发
      if (Object.keys(sorter).length > 0) {
        this.sorter.column = sorter.field
        this.sorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
      }
      this.ipagination = ipagination
      this.loadData()
    },

    getQueryParams() {
      let params = Object.assign({}, this.param, this.queryParam);
      params.pageNo = this.ipagination.current;
      params.pageSize = this.ipagination.pageSize;
      return filterObj(params);
    },

  }
}
</script>

<style scoped>
  .j-table-force-nowrap >>> tr th{
       white-space: nowrap;
  } 
   .j-table-force-nowrap >>> tr td{
       white-space: nowrap;
  } 
</style>