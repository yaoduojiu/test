<template>
  <j-modal
    :title="title"
    :width="modalWidth"
    :visible="visible"
    :confirmLoading="confirmLoading"
    switchFullscreen
    wrapClassName="j-popup-modal"
    @ok="handleSubmit"
    @cancel="handleCancel"
    cancelText="关闭">

    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        
        <a-row :gutter="24">
          <slot name="first" :value ='queryParam'></slot>
          
          <template v-if="toggleSearchStatus">
            <slot name="more" :value ='queryParam'></slot>
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

    <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
      <i class="anticon anticon-info-circle ant-alert-icon"></i>
      已选择&nbsp;<a style="font-weight: 600">{{ table.selectedRowKeys.length }}</a>项&nbsp;&nbsp;
      <a style="margin-left: 24px" @click="onClearSelected">清空</a>

      <a v-if="!showSearchFlag" style="margin-left: 24px" @click="onlyReload">刷新</a>
    </div>

    <a-table
      ref="table"
      size="middle"
      bordered
      :rowKey="combineRowKey"
      :columns="columns"
      :dataSource="table.dataSource"
      :pagination="table.pagination"
      :loading="table.loading"
      :rowSelection="{fixed:true,selectedRowKeys: table.selectedRowKeys, onChange: handleChangeInTableSelect, type: this.multi ? 'checkbox': 'radio'}"
      @change="handleChangeInTable"
      style="min-height: 300px"
      :scroll="tableScroll"
      :customRow="clickThenCheck">
    </a-table>


  </j-modal>
</template>

<script>
  import { getAction } from '@/api/manage'
  import {filterObj} from '@/utils/util'
  import { filterMultiDictText } from '@/components/dict/JDictSelectUtil'
  import { httpGroupRequest } from '@/api/GroupRequest.js'

  const MODAL_WIDTH = 1200;
  export default {
    name: 'JPopupURL',
    props: ['multi', 'url', 'title', 'param', 'columns'],
    components:{
    },
    data(){
      return {
        visible:false,
        confirmLoading:false,
        queryInfo:[],
        toggleSearchStatus:false,
        queryParam:{
        },
        table: {
          loading: true,
          // 表头
          dataSource: [],
          // 选择器
          selectedRowKeys: [],
          selectionRows: [],
          // 分页参数
          pagination: {
            current: 1,
            pageSize: 10,
            pageSizeOptions: ['10', '20', '30'],
            showTotal: (total, range) => {
              return range[0] + '-' + range[1] + ' 共' + total + '条'
            },
            showQuickJumper: true,
            showSizeChanger: true,
            total: 0
          }
        },
        modalWidth:MODAL_WIDTH,
        tableScroll:{x:MODAL_WIDTH-100},
        dynamicParam:{}

      }
    },
    mounted() {
      //this.loadColumnsInfo()
    },
    watch: {
      visible() {
        this.searchReset();
      },
    },
    computed:{
      showSearchFlag(){
        return this.queryInfo && this.queryInfo.length>0
      }
    },
    methods:{
      searchQuery() {
        this.loadData(1);
      },
      loadData(arg) {
        if (arg == 1) {
          this.table.pagination.current = 1
        }
        let params = this.getQueryParams();//查询条件
        this.table.loading = true
        let url = `${this.url}`
        //缓存key
        getAction(url, params).then(res => {
          this.table.loading = false
          // console.log("daa",res)
          let data = res.result
          if (data) {
            this.table.pagination.total = Number(data.total)
            this.table.dataSource = data.records
          } else {
            this.table.pagination.total = 0
            this.table.dataSource = []
          }
        })
      },
      getQueryParams() {
        let params = Object.assign({}, this.param, this.queryParam);
        params.pageNo = this.table.pagination.current;
        params.pageSize = this.table.pagination.pageSize;
        return filterObj(params);
      },
      handleChangeInTableSelect(selectedRowKeys, selectionRows) {
        //update-begin-author:taoyan date:2020902 for:【issue】开源online的几个问题 LOWCOD-844
        if(!selectedRowKeys || selectedRowKeys.length==0){
          this.table.selectionRows = []
        }else if(selectedRowKeys.length == selectionRows.length){
          this.table.selectionRows = selectionRows
        }else{
          //当两者长度不一的时候 需要判断
          let keys = this.table.selectedRowKeys
          let rows = this.table.selectionRows;
          //这个循环 添加新的记录
          for(let i=0;i<selectionRows.length;i++){
            let combineKey = this.combineRowKey(selectionRows[i])
            if(keys.indexOf(combineKey)<0){
              //如果 原来的key 不包含当前记录 push
              rows.push(selectionRows[i])
            }
          }
          //这个循环 移除取消选中的数据
          this.table.selectionRows = rows.filter(item=>{
            let combineKey = this.combineRowKey(item)
            return selectedRowKeys.indexOf(combineKey)>=0
          })
        }
        //update-end-author:taoyan date:2020902 for:【issue】开源online的几个问题 LOWCOD-844
        this.table.selectedRowKeys = selectedRowKeys
      },
      handleChangeInTable(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        if (Object.keys(sorter).length > 0) {
          this.sorter.column = sorter.field
          this.sorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
        }
        this.table.pagination = pagination
        this.loadData()
      },
      handleCancel() {
        this.close()
      },
      handleSubmit() {
        if(!this.multi){
          if(this.table.selectionRows && this.table.selectionRows.length>1){
            this.$message.warning("请选择一条记录")
            return false
          }
        }
        if(!this.table.selectionRows || this.table.selectionRows.length==0){
          this.$message.warning("请选择一条记录")
          return false
        }
        this.$emit('ok', this.table.selectionRows);
        this.close()
      },
      close() {
        this.$emit('close');
        this.visible = false;
        this.onClearSelected()
      },
      show(){
        this.visible = true
        this.loadColumnsInfo()
      },
      handleToggleSearch(){
        this.toggleSearchStatus = !this.toggleSearchStatus;
      },
      searchByquery(){
        this.loadData(1);
      },
      onlyReload(){
        this.loadData();
      },
      searchReset(){
        Object.keys(this.queryParam).forEach(key=>{
          this.queryParam[key]=""
        })
        this.loadData(1);
      },
      onClearSelected(){
        this.table.selectedRowKeys = []
        this.table.selectionRows = []
      },
      combineRowKey(record){
        let res = ''
         Object.keys(record).forEach(key=>{
           res+=record[key]
         })
        if(res.length>50){
          res = res.substring(0,50)
        }
        return res
      },

      clickThenCheck(record){
        return {
          on: {
            click: () => {
              let rowKey = this.combineRowKey(record)
              if(!this.table.selectedRowKeys || this.table.selectedRowKeys.length==0){
                let arr1=[],arr2=[]
                arr1.push(record)
                arr2.push(rowKey)
                this.table.selectedRowKeys=arr2
                this.table.selectionRows=arr1
              }else{
                if(this.table.selectedRowKeys.indexOf(rowKey)<0){
                  this.table.selectedRowKeys.push(rowKey)
                  this.table.selectionRows.push(record)
                }else{
                  let rowKey_index = this.table.selectedRowKeys.indexOf(rowKey)
                  this.table.selectedRowKeys.splice(rowKey_index,1);
                  this.table.selectionRows.splice(rowKey_index,1);
                }
              }
            }
          }
        }
      },
      //防止字典中有垃圾数据
      initDictOptionData(dictOptions){
        let obj = { }
        Object.keys(dictOptions).map(k=>{
          obj[k] = dictOptions[k].filter(item=>{
            return item!=null
          });
        });
        this.dictOptions  = obj
      }

    }
  }
</script>

<style scoped>

</style>