<template>
<a-row>
  <a-col :span="16">
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
  </a-col>
  <a-col :span="8" style="padding-left:20px">
    <div style="margin-bottom: 20px; font-size: 21px">
      配方明细
    </div>
   <a-table
      size="middle"
      :scroll="{x:true}"
      bordered
      :columns="columns1"
      :dataSource="data1"
      rowKey="materialCode"
      class="j-table-force-nowrap change-width">
      <div slot="action" slot-scope="text, record">
        <a-button type="link" @click="remove(record)">移除</a-button>
      </div>
    </a-table>
  </a-col>
</a-row>

</template>

<script>
export default {
  props: {
    data: {
      type: Array,
      default: function() {
        return []
      }
    },
    data1: {
      type: Array,
      default: function() {
        return []
      }
    },
    againsList: {
      type: Array,
      default: function() {
        return []
      }
    },
  },
  watch: {
    againsList(newName, oldName) {
      this.$refs['table'].$el.querySelector('tr.ant-table-row').style.background = '';
      let batchCodes = [];
      newName.forEach(element => {
        this.data.forEach(item => {
          if (item.batchSn === element) {
            batchCodes.push(item.batchCode)
          }
        })

      });
      batchCodes.forEach(item => {
        console.log(this.$refs['table'].$el.querySelector('tr[data-row-key="' + item + '"]').classList);
        this.$refs['table'].$el.querySelector('tr[data-row-key="' + item + '"]').style.background = "#f6f6b7"
      })
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
          dataIndex: 'qty'
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
      columns1: [
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
          title:'需求数量',
          align:"center",
          dataIndex: 'theoryQty'
        },
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

<style scoped lang="less">
  .change-width {
    /deep/ .ant-table-body {
      min-width: 260px!important;
    }
  } 
</style>
