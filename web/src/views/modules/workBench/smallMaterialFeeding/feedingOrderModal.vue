<template>
  <a-modal
    :title="title"
    :confirm-loading="confirmLoading"
    @ok="handleOk"
    :width="width"
    @cancel="handleCancel"
    :visible="visible">
      <a-table
      ref="table"
      size="middle"
      :scroll="{x:true}"
      bordered
      rowKey="id"
      :columns="columns"
      :dataSource="dataSource"
      :loading="dataLoading"
      :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
      class="j-table-force-nowrap">
    </a-table>
  </a-modal>
</template>

<script>
import {getAction} from '@/api/manage'
export default {
  data() {
    return {
      title: '关联投料订单',
      visible: false,
      width: 800,
      confirmLoading: false,
      dataSource: [],
      columns: [
        {
          title:'投料订单号',
          align:"center",
          dataIndex: 'batchNo'
        },
        {
          title:'主原材料编码',
          align:"center",
          dataIndex: 'materialCode'
        },
        {
          title:'主原材料名称',
          align:"center",
          dataIndex: 'materialName'
        }
      ],
      dataLoading: false,
      selectedRowKeys: []
    };
  },
  watch: {
    visible: {
      handler(val) {
        if(val) this.getOrderData()
      },
      immediate: true
    }
  },
  created() {

  },
  mounted() {

  },
  methods: {
    getOrderData() {
      this.dataLoading = true
      getAction('/customFeedRecord/findLastOrder').then(res => {
        if(res.success) {
          this.dataSource = res.result
        } else {
          this.$message.error(res.message)
        }
        this.dataLoading = false
      }).catch(() => {
        this.dataLoading = false
      })
    },
    onSelectChange(selectedRowKeys) {
      if(selectedRowKeys.length > 1) {
        this.$message.error('只能勾选一个投料订单')
        return
      }
      this.selectedRowKeys = selectedRowKeys;
    },
    handleOk() {
      if(this.selectedRowKeys.length === 0) {
        this.$message.error('请先勾选一个投料订单')
        return
      }
      this.$emit('ok', this.selectedRowKeys)
      this.visible = false
      this.selectedRowKeys = []
    },
    handleCancel() {
      this.visible = false
      this.selectedRowKeys = []
    },
  }
};
</script>

<style scoped lang="less">

</style>
