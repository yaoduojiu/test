<template>
  <a-card :bordered="false">
    <!-- 筛选区域 -->
    <div>
      <a-form-model layout="inline">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24" hidden>
            <div style="display: flex;align-items: center;">
              <div style="width: 60px;">产线：</div>
              <a-select v-model="selectedLine" @change="changeLine" placeholder="请选择">
                <a-select-option
                  v-for="line in lines"
                  :key="line.id"
                  :value="line.id">
                  {{ line.lineName }}
                </a-select-option>
              </a-select>
            </div>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <div style="display: flex;align-items: center;">
              <div style="width: 80px;">打印机：</div>
               <j-dict-select-tag v-model="selectedPrintName" dictCode="batch_sn_print_name" placeholder="请选择打印机" />
            </div>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <div style="display: flex;align-items: center;">
              <div style="width: 100px;">打印模板：</div>
               <j-dict-select-tag v-model="selectedPrintTemplate" dictCode="batch_sn_print_btw" placeholder="请选择打印模板" />
            </div>
          </a-col>
        </a-row>
      </a-form-model>
    </div>
    <a-spin :spinning="pageLoading">
        <!-- 当前混料批次 -->
      <a-divider orientation="left">
        当前混料批次
      </a-divider>
      <div id="batchInfoTable">
        <table>
          <tr>
            <td class="label">订单编号</td>
            <td class="">{{ mixinBatchInfo.orderCode ? mixinBatchInfo.orderCode : '--' }}</td>
            <td class="label">混料批次号</td>
            <td class="">{{ mixinBatchInfo.blendLotId ? mixinBatchInfo.blendLotId : '--' }}</td>
            <td class="label">产成品物料编码</td>
            <td class="">{{ mixinBatchInfo.materialCode ? mixinBatchInfo.materialCode : '--' }}</td>
          </tr>
          <tr>
            <td class="label">产成品名称</td>
            <td class="">{{ mixinBatchInfo.materialName ? mixinBatchInfo.materialName : '--' }}</td>
            <td class="label">状态</td>
            <td class="">{{ mixinBatchInfo.status_dictText ? mixinBatchInfo.status_dictText : '--' }}</td>
            <td class="label">生产数量</td>
            <td class="">{{ mixinBatchInfo.qty ? mixinBatchInfo.qty : '--' }}</td>
          </tr>
          <tr>
            <td class="label">实际数量</td>
            <td class="">{{ mixinBatchInfo.actualQty ? mixinBatchInfo.actualQty : '--' }}</td>
            <td class="label">开始时间</td>
            <td class="">{{ mixinBatchInfo.startTime ? mixinBatchInfo.startTime : '--' }}</td>
            <td class="label">结束时间</td>
            <td class="">{{ mixinBatchInfo.endTime ? mixinBatchInfo.endTime : '--' }}</td>
          </tr>
        </table>
      </div>
      <!-- 条码打印 -->
      <a-divider orientation="left">
        条码打印
      </a-divider>
      <div style="height: 60px;">
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
              <a-form-item label="包装型号" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-input
                    :maxLength="30"
                    v-decorator="[
                    'packingCode',
                    { rules: [{ required: true, message: '请输入' }] },
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
            <hasPrintedTable :printName="selectedPrintName" :printTemplate="selectedPrintTemplate" @re-print="getPrintInfo" ref="hasPrintedTable"></hasPrintedTable>
          </a-tab-pane>
        </a-tabs>
      </div>
    </a-spin>
  </a-card>
</template>

<script>
import { getAction, putAction } from '@/api/manage'
import hasPrintedTable from '@/views/modules/workBench/barcodePrint/hasPrintedTable'
export default {
  components: {
    hasPrintedTable
  },
  name: 'BarcodePrint',
  data() {
    return {
      pageLoading: false,
      lines: [],
      selectedLine: undefined,
      selectedPrintName: null,
      selectedPrintTemplate: null,
      stations: [],
      // selectedPrintType: undefined,
      mixinBatchInfo: {},
      formModel: {},
      form: this.$form.createForm(this),
      labelCol: {
        xs: { span: 24 },
        sm: { span: 8 },
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 },
      },
      printLoading: false,
      waitFeedingData: [],
      hasFeedingData: [],
      printInfo: {},
      printTypeOptions: [
        {
          value: 'DBORDERNUMBER',
          label: '蛋白'
        },
        {
          value: 'DB1ORDERNUMBER',
          label: '添加剂1'
        },
        {
          value: 'DB2ORDERNUMBER',
          label: '添加剂2'
        }
      ]
    }
  },
  created() {
    this.getLines()
    // this.getPrintInfo()
  },
  mounted() {

  },
  methods: {
    getPrintInfo() {
      getAction('/workbench/print/getBlendLot', {
        lineId : this.selectedLine
      }).then(ress=> {
        if (ress.result && ress.result.id) {
          getAction('/workbench/print/printInfo', {
            id : ress.result.id
          }).then(res => {
            if(res.success) {
              this.printInfo = res.result
            } else {
              this.$message.error(res.message)
            }
          })
        } else {
          this.printInfo = {};
        }
      });

    },
    printClick() {
      if(!this.selectedLine) {
        this.$message.error('请先选择产线')
        return
      }
      if(!this.selectedPrintName) {
        this.$message.error('请先选择打印机')
        return
      }
      if(!this.selectedPrintTemplate) {
        this.$message.error('请先选择打印模板')
        return
      }
      if(!this.mixinBatchInfo.blendLotId) {
        this.$message.error('请查询出当前混料批次信息')
        return
      }
      this.form.validateFields((err, values) => {
        if (!err) {
          let httpurl = '/workbench/print/print';
          let formData = Object.assign({
          }, values);
          let {standardQty, sheetQty, packingCode} = formData;
          console.log("表单提交数据",formData)
          this.printLoading = true;
          this.pageLoading = true
          putAction(httpurl,{
            standardQty,
            sheetQty,
            packingCode,
            lineId: this.selectedLine,
            blendLotCode: this.mixinBatchInfo.blendLotId,
            printerName: this.selectedPrintName,
            templatePath: this.selectedPrintTemplate,
            orderCode: this.mixinBatchInfo.orderCode
          }).then((res)=>{
            this.printLoading = false
            this.pageLoading = false
            if(res.success){
              this.$message.success('操作成功');
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
    changeLine(value) {
      if(this.selectedLine) {
        this.getMixinBatchInfo()
        this.getPrintInfo()
      }
    },
    // 获取产线
    getLines() {
      getAction('/line/line/list', {
        pageNo: 1,
        pageSize: 99999999,
        keyword: '添加剂'
      }).then(res => {
        if(res.success === true) {
          this.lines = res.result.records;
          this.selectedLine = res.result.records[0] &&  res.result.records[0].id;
          this.changeLine();
        }
      })
    },
    // 获取工位
    getStations() {
      getAction('/station/station/list', {
        pageNo: 1,
        pageSize: 99999999,
        lineId: this.selectedLine
      }).then(res => {
        if(res.success === true) {
          this.stations = res.result.records;
        }
      })
    },
    // 获取混料批次信息
    getMixinBatchInfo() {
      this.pageLoading = true;
      getAction('/workbench/print/getBlendLot', {
        lineId: this.selectedLine,
      }).then(res => {
        this.pageLoading = false
        if(res.success === true && res.result) {
          this.mixinBatchInfo = res.result;
          this.$refs.hasPrintedTable.queryParam = {
            orderId: res.result.id
          }
          this.$refs.hasPrintedTable.loadData()
        } else {
          this.mixinBatchInfo = {}
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
</style>
