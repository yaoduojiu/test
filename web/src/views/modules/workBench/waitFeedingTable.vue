<template>
  <div>
    <div style="margin-bottom: 20px;">
      <a-button :disabled="data.length === 0" type="primary" @click="sureFeedingClick">确定投料</a-button>
    </div>
    <a-table
      ref="table"
      size="middle"
      :scroll="{x:true}"
      bordered
      rowKey="batchCode"
      :columns="columns"
      :dataSource="data"
      class="j-table-force-nowrap">
      <div slot="action" slot-scope="text, record">
        <a-button type="link" @click="remove(record)">移除</a-button>
      </div>
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
          title:'原材料编码',
          align:"center",
          dataIndex: 'materialCode'
        },
        {
          title:'原材料名称',
          align:"center",
          dataIndex: 'materialName'
        },
        {
          title:'待投数量',
          align:"center",
          dataIndex: 'qty'
        },
        {
          title:'可投数量',
          align:"center",
          dataIndex: 'remainderQty'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align:"center",
          fixed:"right",
          width:100,
          scopedSlots: { customRender: 'action' }
        }
      ]
    };
  },
  created() {

  },
  mounted() {

  },
  methods: {
    remove(record) {
      console.log(record)
      this.$emit('remove', record);
    },
    sureFeedingClick() {
      this.$emit('feeding')
    }
  }
};
</script>

<style scoped lang="scss">

</style>
