<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container >
      <a-form :model="form" slot="detail">
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item label="物料SN">
              <a-input v-model="form.batchSn" @pressEnter="addToList" placeholder="扫描输入" />
            </a-form-item>
          </a-col>

          <a-col :span="6">
            <a-form-item label=" ">
              <a-button type="primary" @click="addToList">添加</a-button>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </j-form-container>
    <a-table v-show="this.list.length > 0"
      ref="table"
      size="small"
      bordered
      :rowKey="combineRowKey"
      :columns="columns"
      :pagination="false"
      :dataSource="list">
      <span slot="action" slot-scope="text, list">
        <a @click="deleteList(list)">删除</a>
      </span>
      <span slot="qty" slot-scope="text, list">
        <a-input type="number" style="text-align:center" v-model="list.selectedRow.qty" v-on:keyup="e => handleChange(e.target.value, list)" @change="e => handleChange(e.target.value, list)"/>
      </span>
    </a-table>
  </a-spin>
  
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import { validateDuplicateValue } from '@/utils/util'
  import { postAction, putAction } from '@/api/manage'
  import JPopup from './JPopups'

  export default {
    name: 'StockOutOrderDirectForm',
    components: {
      JPopup
    },
    props: {
     
    },
    data () {
      return {
        form: {
          batchSn: '',
          locationId: ''
        },
        list: [],
        columns: [
          {
            title:'库位',
            align:"center",
            dataIndex: 'selectedRow.locationCode'
          },
          {
            title:'批次编号',
            align:"center",
            dataIndex: 'selectedRow.code'
          },
          {
            title:'物料编码',
            align:"center",
            dataIndex: 'selectedRow.materialCode'
          },
          {
            title:'物料名称',
            align:"center",
            dataIndex: 'selectedRow.materialName'
          },
          {
            title:'物料SN',
            align:"center",
            dataIndex: 'form.batchSn'
          },
          {
            title:'数量',
            align:"center",
            dataIndex: 'selectedRow.inQty'
          },
          {
            title:'出库数量',
            align:"center",
            dataIndex: 'selectedRow.qty',
            scopedSlots: { customRender: 'qty' },
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        selectedRow: {},
        locationList: [],
        model: {},
        confirmLoading: false,

      }
    },
    computed: {
      param() {
        return {
          locationId: this.form.locationId
        }
      },
      disable() {
        return !(this.form.locationId && this.form.batchType);
      },
      formList() {
        let formList = [];
        this.list.forEach(item => {
          item.form.qty = item.selectedRow.qty;
          formList.push(item.form);
        });
        return formList;
      }
    },
    mounted () {
      //如果是流程中表单，则需要加载流程表单data
      // this.showFlowData();
      this.listLocationByUserId();
      
    },
    methods: {
      temporary() {
        if (this.formList.length === 0) return;
        postAction('/stock/outOrder/temporary', this.formList).then(res=> {
          if(res.success){
              this.$message.success(res.message);
              this.list = [];
              this.$emit('ok');
            }else{
              this.$message.warning(res.message);
            }
        });
      },
      outOrder() {
        if (this.formList.length === 0) return;
        postAction('/stock/outOrder/direct', this.formList).then(res=> {
          if(res.success){
              this.$message.success(res.message);
              this.list = [];
              this.$emit('ok');
            }else{
              this.$message.warning(res.message);
            }
        });
      },
      deleteList(record) {
        var newList = [];
        this.list.forEach(item => {
          if (item !== record) {
            newList.push(item);
          }
        });

        this.list = newList;
      },
      listLocationByUserId() {
        getAction('/warehouse/warehouse/listLocationByUserId').then(res => {
          if (res.success) {
            this.locationList = res.result;
          }
        })
      },
      addToList () {
        if (!this.form.batchSn) {
          this.$message.warning('物料SN不能为空');
          return;
        }

        getAction('/stock/inOrder/getBatchSn', {
          batchSn: this.form.batchSn
        }).then(res => {
          if (res.code !== 200) {
            this.form.batchSn = '';
            this.$message.warning(res.message);
            return;
          }
          this.form.locationId = res.result.locationId;
          let data = Object.assign({}, {selectedRow: res.result}, {form: this.form});
          data.selectedRow.qty = data.selectedRow.inQty;
          data = JSON.parse(JSON.stringify(data));
          // data.form.locationCode = this.locationList.find(item => item.id === data.form.locationId)["code"];
          this.list.push(data);
          this.form.batchSn = '';
          // Object.keys(this.form).forEach(key => {
          
          //   this.form[key] = '';
          // })
        });

      },

      handleChange(value, list) {

        if (value > list.selectedRow.inQty) {
          this.$message.warning('出库数量不能大于库存');
          list.selectedRow.qty = list.selectedRow.inQty;
        }
      },

      combineRowKey(record) {
        return record.selectedRow.id;
      }

    }
  }
</script>