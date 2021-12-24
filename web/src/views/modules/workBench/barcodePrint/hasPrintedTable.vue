<template>
  <div>
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
      @change="handleTableChange"
      class="j-table-force-nowrap">
      <div slot="action" slot-scope="text, record">
        <a-button type="link" @click="rePrintClick(record)">重新打印</a-button>
      </div>
    </a-table>
  </div>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'
export default {
  mixins: [JeecgListMixin],
  props: ['printName', 'printTemplate'],
  data() {
    return {
      columns: [
        {
          title:'SN条码',
          align:"center",
          dataIndex: 'sn'
        },
        {
          title:'包装数量',
          align:"center",
          dataIndex: 'qty'
        },
        {
          title:'生产时间',
          align:"center",
          dataIndex: 'createTime'
        },
        {
          title:'打印时间',
          align:"center",
          dataIndex: 'printTime'
        },
        {
          title:'打印类型',
          align:"center",
          dataIndex: 'printType'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align:"center",
          fixed:"right",
          width:100,
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: "/workbench/print/list"
      },
      disableMixinCreated: true
    };
  },
  created() {

  },
  mounted() {

  },
  methods: {
    rePrintClick(record) {
      if(!this.printName) {
        this.$message.error('请先选择打印机')
        return
      }
      if(!this.printTemplate) {
        this.$message.error('请先选择打印模板')
        return
      }
      const _this = this
      this.$confirm({
        title: '提示',
        content: '确定要重新打印吗?',
        okText: '确定',
        cancelText: '取消',
        onOk() {
          _this.loading = true
          getAction(`/workbench/print/reprint?id=${record.id}&printerName=${_this.printName}&templatePath=${_this.printTemplate}`).then(res => {
            if(res.success) {
              _this.loading = false
              _this.$message.success('操作成功')
              _this.loadData()
              _this.$emit('re-print')
            } else {
              _this.$message.warning(res.message)
            }
            _this.loading = false
          }).catch(() => {
            _this.loading = false
          })
        }
      })
    }
  }
};
</script>

<style scoped lang="scss">

</style>
