<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container >
      <a-form :model="form" slot="detail">
        <a-row :gutter="20">
          <a-col :span="12">
            <a-form-item label="库位">
              <a-select v-model="form.locationId" value="item.id">
                <a-select-option v-for="item in locationList"  :key="item.id">
                  {{item.code}}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="物料SN">
              <a-input v-model="form.batchSn" @pressEnter="addToList"  placeholder="扫描输入" />
            </a-form-item>
          </a-col>

          <a-col :span="24" style="text-align: right">
            <a-button type="primary" @click="addToList">添加</a-button>
          </a-col>
        </a-row>
      </a-form>
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
      </a-table>
    </j-form-container>
  </a-spin>
  
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import { validateDuplicateValue } from '@/utils/util'
  import { postAction, putAction } from '@/api/manage'
  // import JPopup from './JPopups'

  export default {
    name: 'StockInOrderDirectForm',
    // components: {
    //   JPopup
    // },
    props: {
     
    },
    data () {
      return {
        form: {
          locationId: '',
          batchSn: '',
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
            dataIndex: 'selectedRow.qty'
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
          locationId: this.form.locationId,
          batchTypeEnum: this.form.batchType
        }
      },
      disable() {
        return !(this.form.locationId && this.form.batchType);
      },
      formList() {
        let formList = [];
        this.list.forEach(item => {
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
        postAction('/stock/inOrder/temporary', this.formList).then(res=> {
          if(res.success){
              this.$message.success(res.message);
              this.list = [];
              this.$emit('ok');
            }else{
              this.$message.warning(res.message);
            }
        });
      },
      inOrder() {
        if (this.formList.length === 0) return;
        postAction('/stock/inOrder/direct', this.formList).then(res=> {
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
        if (!this.form.locationId) {
          this.$message.warning('请选择库位');
          return;
        }
        if (!this.form.batchSn) {
          this.$message.warning('请输入物料SN');
          return;
        }
        getAction('/batchInfo/batchInfo/getBatchInfoBySn', {
          sn: this.form.batchSn
        }).then(res => {
          if (res.code !== 200) {
            this.form.batchSn = '';
            this.$message.warning(res.message);
            return;
          }
          let data = Object.assign({}, {selectedRow: res.result}, {form: this.form});
          data = JSON.parse(JSON.stringify(data));
          data.selectedRow.locationCode = this.locationList.find(item => item.id === data.form.locationId)["code"];
          this.list.push(data);
          this.form.batchSn = '';
          // Object.keys(this.form).forEach(key => {
          
          //   this.form[key] = '';
          // })
        });
      },

      combineRowKey(record) {
        return record.form.locationId + record.selectedRow.id;
      }

    }
  }
</script>