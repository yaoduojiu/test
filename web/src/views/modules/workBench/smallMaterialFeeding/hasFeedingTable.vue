<template>
  <div>
    <a-table
      ref="table"
      size="middle"
      :scroll="{x:true}"
      bordered
      rowKey="batchCode"
      :columns="columns"
      :dataSource="data"
      class="j-table-force-nowrap">
    </a-table>
  </div>
</template>

<script>
export default {
  props: {
    data: {
      type: Array,
      default: function() {
        return []
      }
    }
  },
  data() {
    return {
      columns: [
        {
          title:'原材料批次',
          align:"center",
          dataIndex: 'batchCode'
        },
        {
          title:'原材料物料编码',
          align:"center",
          dataIndex: 'materialCode'
        },
        {
          title:'原材料名称',
          align:"center",
          dataIndex: 'materialName'
        },
        {
          title:'物料SN',
          align:"center",
          dataIndex: 'batchSn'
        },
        {
          title:'投料数量',
          align:"center",
          dataIndex: 'feedQty'
        },
      ]
    };
  },
  created() {

  },
  mounted() {

  },
  methods: {
  }
};
</script>

<style scoped lang="scss">

</style>
