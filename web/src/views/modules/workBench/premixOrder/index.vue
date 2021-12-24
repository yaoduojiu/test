<template>
  <a-card :bordered="false">
    <a-spin :spinning="pageLoading">
      <a-divider orientation="left">
        预混料订单
      </a-divider>
      <div id="batchInfoTable">
        <table>
          <tr>
            <td class="label">订单编号</td>
            <td class="">{{ feedOrderInfo.code ? feedOrderInfo.code : '--' }}</td>
            <td class="label">集合料编码</td>
            <td class="">{{ feedOrderInfo.materialCode ? feedOrderInfo.materialCode : '--' }}</td>
            <td class="label">集合料名称</td>
            <td class="">{{ feedOrderInfo.materialName ? feedOrderInfo.materialName : '--' }}</td>
          </tr>
          <tr>

            <td class="label">状态</td>
            <td class="">{{ feedOrderInfo.status_dictText ? feedOrderInfo.status_dictText : '--' }}</td>
            <td class="label">生产数量</td>
            <td class="">{{ feedOrderInfo.qty ? feedOrderInfo.qty : '--' }}</td>
          </tr>
        </table>
      </div>
    </a-spin>

    <a-tabs defaultActiveKey="1" @change="handleTabPrint" v-model="activeKey">
      <a-tab-pane tab="投料" key="1" >
        <a-form>
          <a-row :gutter="20">
            <a-col :span="8">
              <a-form-item label="原材料批次" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-input
                  @pressEnter="addFeedingClick"
                  v-model="batchSn"
                  placeholder="请先输入或扫描" :allowClear="true"/>
                  <a-alert v-show="batchInfo.isShow" :message="batchInfo.msg" :type="batchInfo.type" show-icon />

              </a-form-item>
            </a-col>
              <a-col :span="8" style="padding-top: 4px;">
            <a-button type="primary" :loading="addLoading" @click="addFeedingClick">添加投料</a-button>
          </a-col>
          </a-row>
        </a-form>
        <a-tabs defaultActiveKey="1">
          <a-tab-pane tab="投料" key="1" >
            <div style="margin-bottom: 20px;">
              <a-button :disabled="waitFeedingData.length === 0" type="primary" @click="feed">确定投料</a-button>
            </div>
            <a-row>
              <a-col :span="16">
                <a-table
                  ref="table"
                  size="middle"
                  :scroll="{x:true}"
                  bordered
                  :rowKey="(record,index)=>{return index}"
                  :columns="columns"
                  :dataSource="waitFeedingData"
                  class="j-table-force-nowrap">
                  <div slot="action" slot-scope="text, record">
                    <a-button type="link" @click="remove(record)">移除</a-button>
                  </div>
                </a-table>
              </a-col>
              <a-col :span="8" style="padding-left:20px">
                <div style="margin-bottom: 20px; font-size: 21px; margin-top: -50px">
                  配方明细
                </div>
                <a-table
                  size="middle"
                  bordered
                  :columns="columns11"
                  :dataSource="formulaDetails"
                  rowKey="materialCode"
                  class="j-table-force-nowrap change-width">
                  <div slot="action" slot-scope="text, record">
                    <a-button type="link" @click="remove(record)">移除</a-button>
                  </div>
                </a-table>
              </a-col>
            </a-row>

          </a-tab-pane>
          <a-tab-pane tab="已投料" key="2" >
            <a-table
              ref="table1"
              size="middle"
              :scroll="{x:true}"
              bordered
              rowKey="batchCode"
              :columns="columns1"
              :dataSource="feededList"
              class="j-table-force-nowrap">

            </a-table>
          </a-tab-pane>
        </a-tabs>
      </a-tab-pane>
      <a-tab-pane tab="打印" key="2" >
      <!-- 条码打印 -->
      <a-form-model layout="inline">
        <a-row :gutter="24">
          <a-col :xl="10" :lg="10" :md="10" :sm="24">
            <div style="display: flex;align-items: center;">
              <div style="width: 80px;">打印机：</div>
               <j-dict-select-tag v-model="selectedPrintName" dictCode="batch_sn_print_name" placeholder="请选择打印机" />
            </div>
          </a-col>
          <a-col :xl="10" :lg="10" :md="10" :sm="24">
            <div style="display: flex;align-items: center;">
              <div style="width: 100px;">打印模板：</div>
               <j-dict-select-tag v-model="selectedPrintTemplate" dictCode="batch_sn_print_btw" placeholder="请选择打印模板" />
            </div>
          </a-col>
        </a-row>
      </a-form-model>
      <div style="height: 60px; margin-top: 20px">
        <a-form :form="form" layout="inline" labelAlign="left">
          <a-form-item label="包装规格" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-input-number
                  :precision="3"
                  :min="0"
                  :max="999999999"
                    v-decorator="[
                    'standardQty',
                    { rules: [{ required: true, message: '请输入包装规格' }] },
                  ]"
                placeholder="请输入" style="width: 100%" />
              </a-form-item>
              <a-form-item label="张数" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-input-number
                  :precision="0"
                  :min="0"
                  :max="999999999"
                    v-decorator="[
                    'sheetQty',
                    { rules: [{ required: true, message: '请输入打印张数' }] },
                  ]"
                placeholder="请输入" style="width: 100%" />
              </a-form-item>
              <a-form-item>
                <a-button type="primary" :loading="printLoading" @click="printClick">条码打印</a-button>
              </a-form-item>
        </a-form>
      </div>
      <div style="margin-top: 10px;">
        <a-row :gutter="30">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <div>已打印张数：<span>{{ printInfo.printQty !== null ? printInfo.printQty : '--' }}</span>张</div>
            <div style="margin-top: 20px;">已打印重量：<span>{{ printInfo.printWeight !== null ? printInfo.printWeight : '--' }}</span>kg</div>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <div>剩余可打印数量：<span>{{ printInfo.remainPrintQty !== null ? printInfo.remainPrintQty : '--' }}</span>张</div>
            <div style="margin-top: 20px;">剩余可打印重量：<span>{{ printInfo.remainPrintWeight !== null ? printInfo.remainPrintWeight : '--' }}</span>kg</div>
          </a-col>
        </a-row>
        <a-tabs defaultActiveKey="1">
          <a-tab-pane tab="已打印" key="1" >
            <hasPrintedTable ref="hasPrintedTable" :printName="selectedPrintName" :printTemplate="selectedPrintTemplate" @re-print="getPrintInfo" ></hasPrintedTable>
          </a-tab-pane>
        </a-tabs>
      </div>
      </a-tab-pane>
    </a-tabs>
  </a-card>
</template>

<script>
import { getAction, postAction, putAction } from '@/api/manage'
import hasPrintedTable from '@/views/modules/workBench/premixOrder/hasPrintedTable'

export default {
  name: 'premixOrder',
   components: {
    hasPrintedTable
  },
  data() {
    return {
      activeKey: "1",
      printLoading: false,
      pageLoading: false,
      addLoading: false,
      selectedPrintName: null,
      selectedPrintTemplate: null,
      printInfo: {},
      form: this.$form.createForm(this),
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
      ],
      feededList: [],
      waitFeedingData: [],
      feedOrderInfo: {},
      formulaDetails: [],
      labelCol: {
        xs: { span: 24 },
        sm: { span: 8 },
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 },
      },
      batchSn: '',
      columns11: [
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
          dataIndex: 'qty'
        },
      ],
       batchInfo: {
        msg: '',
        isShow: false,
        type: 'success'
      }
    }
  },
  created() {
  },
  mounted() {
    this.getFeedOrderInfo();
    this.getPrintInfo();
  },
  methods: {

    printClick() {
      if(!this.selectedPrintName) {
        this.$message.error('请先选择打印机')
        return
      }
      if(!this.selectedPrintTemplate) {
        this.$message.error('请先选择打印模板')
        return
      }
      if(!this.feedOrderInfo.code) {
        this.$message.error('请查询出当前混料批次信息')
        return
      }
      this.form.validateFields((err, values) => {
        if (!err) {
          let httpurl = '/order/premixOrder/print';
          let formData = Object.assign({
          }, values);
          let {standardQty, sheetQty} = formData;
          console.log("表单提交数据",formData)
          this.printLoading = true;
          this.pageLoading = true
          putAction(httpurl,{
            standardQty,
            sheetQty,
            printerName: this.selectedPrintName,
            templatePath: this.selectedPrintTemplate,
            orderCode: this.feedOrderInfo.code
          }).then((res)=>{
            this.printLoading = false
            this.pageLoading = false
            if(res.success){
              this.$message.success('操作成功');
              this.getPrintInfo();
              this.$refs.hasPrintedTable.loadData(1)
              this.$nextTick(() => {
                this.form.setFieldsValue({
                  standardQty: '',
                  sheetQty: ''
                })
              })
            }else{
              this.$message.warning(res.message);
            }
          }).catch(() => {
            this.printLoading = false;
            this.pageLoading = false
          })
        }
      })
    },
    getPrintInfo() {
      getAction('/order/premixOrder/printInfo').then(res => {
        if(res.success) {
          this.printInfo = res.result
        } else {
          this.$message.error(res.message)
        }
      })
    },

    handleTabPrint(key) {
      if (key == 2) {
        this.getPrintRecord();
      }

    },

    getPrintRecord() {
      var _this = this;
      setTimeout(()=>{
        _this.$refs.hasPrintedTable.queryParam = {
          orderId: this.feedOrderInfo.id
        }
        _this.$refs.hasPrintedTable.loadData()
      }, 200)

    },

    feed() {
      var _this = this;
      this.$confirm({
        title: '提示',
        content: '确定要进行投料吗?',
        okText: '确定',
        cancelText: '取消',
        onOk() {
          _this.pageLoading = true
          postAction('/order/premixOrder/feed', {
            feedDataList: _this.waitFeedingData,
            orderCode: _this.feedOrderInfo.code,
          }).then(res => {
            _this.pageLoading = false
            if(res.success === true) {
              _this.$message.success(res.message)
              _this.waitFeedingData = []
              _this.getFeedOrderInfo()

            } else {
              _this.$message.warning(res.message);

            }
          }).catch(() => {
            _this.pageLoading = false
          })
        }
      });
    },

    addFeedingClick() {
      if(this.batchSn) {
        this.addBatch(this.batchSn)
      } else {
        this.$message.error('请先输入或扫描')
      }
    },

    remove(payload) {
      let _this = this;
      this.$confirm({
        title: '警告',
        content: '真的要移除吗?',
        okText: '确定',
        okType: 'danger',
        cancelText: '取消',
        onOk() {
          _this.waitFeedingData.splice(_this.waitFeedingData.findIndex(ele => ele.batchCode === payload.batchCode), 1);
        }
      });
    },

    addBatch(batchSn) {
      this.addLoading = true;
      this.pageLoading = true
      postAction('/order/premixOrder/getBatchAndCheck',{
        batchSn: batchSn,
        materialCode: this.feedOrderInfo.materialCode,
        type:4
      }).then((res)=>{
        this.addLoading = false
        this.pageLoading = false
        if(res.success){
          let isValid = this.formulaDetails.find(item => item.materialCode ===  res.result.materialCode);
          if (!isValid) {
            this.batchInfo = {
              msg: '原材料核对错误. 批次料物料编码：' + res.result.materialCode,
              isShow: true,
              type: 'error'
            }
            return;
          } else {
            this.batchInfo = {
              msg: '原材料核对正确. 批次料物料编码：' + res.result.materialCode,
              isShow: true,
              type: 'success'
            }
          }
          const hasIndex = this.waitFeedingData.findIndex(ele => ele.batchCode === res.result.batchCode)
          if (hasIndex > -1) {
            this.$message.warning(`原材料批次${res.result.batchCode}已添加`);
            return
          }
          this.waitFeedingData.push(res.result);
          this.$message.success(res.message);
          this.batchSn = ''
        }else{
          this.$message.warning(res.message);
        }
        this.batchSn = '';
      }).catch(() => {
        this.addLoading = false;
        this.pageLoading = false
      })
    },



    getFeedOrderInfo() {
      this.pageLoading = true;
      getAction('/order/premixOrder/getByStatus').then(res => {
        this.pageLoading = false
        if(res.success === true && res.result) {
          this.feedOrderInfo = res.result;
          this.getFeedRecord(res.result);
          this.getFormulaDetails(res.result);

        } else {
          this.feedOrderInfo = {}

          this.feededList = []
          this.$message.error(res.message)
        }
      }).catch((e) => {
        this.pageLoading = false;
        this.feededList = []
        this.feedOrderInfo = {}
        this.$message.warning(e.response.data.message);
      });
    },


    getFeedRecord(data) {
      getAction('/order/premixOrder/feedRecord', {
        orderCode: data.code
      }).then(res => {
        if (res.success) {
          this.feededList = res.result
        } else {
          this.$message.error(res.message)
        }
      }).catch((e) => {
        this.pageLoading = false;
        this.$message.warning(e.response.data.message);
      });
    },

    getFormulaDetails(data) {
      this.pageLoading = true;
      getAction('/formula/formulaInfo/listFormulaDetailsByCode', {
        materialCode: data.materialCode
      }).then(res => {
        this.pageLoading = false
        if(res.success === true && res.result) {
          this.formulaDetails = res.result;
        } else {
           this.formulaDetails = []
          this.$message.error(res.message)
        }
      }).catch(() => {
        this.pageLoading = false
      })
    },

  }
};
</script>

<style scoped lang="less">
  #batchInfoTable {
    width: 100%;
    table {
      width: 100%;
      border-collapse: collapse;
      td {
        width: 16.66%;
        height: 34px;
        text-align: center;
        line-height: 34px;
        border: 1px solid #dddddd;
        &.label {
          background-color: #f8f8f8;
        }
      }
    }
  }
  .change-width {
    /deep/ .ant-table-body {
      min-width: 260px!important;
    }
  }
</style>
