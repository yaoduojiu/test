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
              <div style="width: 60px;">工位：</div>
               <a-select @change="changeStation" v-model="selectedStation" placeholder="请选择">
                <a-select-option
                  v-for="station in stations"
                  :key="station.id"
                  :value="station.id">
                  {{ station.name }}
                </a-select-option>
              </a-select>
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
      <!-- 添加投料 -->
      <a-divider orientation="left">
        添加投料
      </a-divider>
      <div>
        <a-row style="margin-bottom: 10px;">
          <a-col style="padding-left: 4%;" :span="8">
              <a-button type="primary" :loading="addLoading" @click="addFeedingClick">添加投料</a-button>
            </a-col>
        </a-row>
        <a-form :form="form">
          <a-row>
            <a-col :span="8">
              <a-form-item label="原材料批次" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-input
                  v-decorator="[
                    'batchOrderCode',
                    {rules: [{ required: true, message: '请输入原材料批次' }]}
                  ]"
                  placeholder="请输入" />
              </a-form-item>
            </a-col>
            <a-col :span="8">
              <a-form-item label="仓号" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-input
                  v-decorator="[
                    'warehouseCode',
                    {rules: [{ required: true, message: '请输入仓号' }]}
                  ]"
                  placeholder="请输入" />
                  <!-- <a-select v-decorator="[
                    'warehouseCode',
                    {rules: [{ required: true, message: '请输入仓号' }]}
                  ]" v-model="selectedStation" placeholder="请选择">
                <a-select-option
                  v-for="warehouse in warehouses"
                  :key="warehouse.code"
                  :value="warehouse.code">
                  {{ warehouse.code }}
                </a-select-option>
              </a-select> -->
              </a-form-item>
            </a-col>
            <a-col :span="8">
              <a-form-item label="投料数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-input-number
                  :precision="0"
                  :min="0"
                  :max="999999999"
                    v-decorator="[
                    'qty',
                    { rules: [{ required: true, message: '请输入投料数量' }] },
                  ]"
                placeholder="请输入" style="width: 100%" />
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
      <!-- 投料展示 -->
      <a-divider orientation="left">
        投料信息
      </a-divider>
      <div>
        <a-tabs defaultActiveKey="1" @change="TabChange">
          <a-tab-pane tab="待投料" key="1" >
            <waitFeedingTable @feeding="feedingClick" :data="waitFeedingData" @remove="removeHandle"></waitFeedingTable>
          </a-tab-pane>
          <a-tab-pane tab="已投料" key="2">
            <hasFeedingTable :data="hasFeedingData"></hasFeedingTable>
          </a-tab-pane>
        </a-tabs>
      </div>
    </a-spin>
  </a-card>
</template>

<script>
import { getAction, postAction } from '@/api/manage'
import waitFeedingTable from '@/views/modules/workBench/waitFeedingTable'
import hasFeedingTable from '@/views/modules/workBench/hasFeedingTable'
export default {
  components: {
    waitFeedingTable,
    hasFeedingTable
  },
  name: 'WorkBench',
  data() {
    return {
      pageLoading: false,
      lines: [],
      selectedLine: undefined,
      stations: [],
      selectedStation: undefined,
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
      addLoading: false,
      waitFeedingData: [],
      hasFeedingData: [],
      warehouses: []
    }
  },
  created() {
    this.getLines()
  },
  mounted() {

  },
  methods: {
    removeHandle(payload) {
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
    feedingClick() {
      if(!this.selectedStation || !this.selectedLine) {
        this.$message.error('请选择产线、工位')
        return
      }
      let _this = this;
      this.$confirm({
        title: '提示',
        content: '确定要进行投料吗?',
        okText: '确定',
        cancelText: '取消',
        onOk() {
          _this.pageLoading = true
          postAction('/customFeedRecord/feed', {
            lineId: _this.selectedLine,
            feedDataList: _this.waitFeedingData,
            orderCode: _this.mixinBatchInfo.orderCode,
            stationId: _this.selectedStation,
            blendLotCode: _this.mixinBatchInfo.blendLotId
          }).then(res => {
            _this.pageLoading = false
            if(res.success === true) {
              _this.$message.success(res.message)
              _this.waitFeedingData = []
              _this.getMixinBatchInfo()
            } else {
              _this.$message.warning(res.message);
            }
          }).catch(() => {
            _this.pageLoading = false
          })
        }
      });
    },
    // 获取已投料信息
    getHasFeedingData({orderCode, blendLotId}) {
      getAction('/customFeedRecord/feed', {blendLot: blendLotId, orderCode}).then(res => {
        if(res.success === true) {
          this.hasFeedingData = res.result;
        }
      })
    },
    // 获取仓位
    getWareHouses(parentId) {
      getAction('/station/station/listStationWarehouseByMainId', {
        parentId,
        pageNo: 1,
        pageSize: 99999999
      }).then(res => {
        if (res.success) {
          this.warehouses = res.result.records
        }
      })
    },
    TabChange() {},
    addFeedingClick() {
      this.form.validateFields((err, values) => {
        if (!err) {
          let httpurl = '/customFeedRecord/getBatch';
          let formData = Object.assign({
          }, values);
          if(formData.batchOrderCode.indexOf('-') === -1) {
            this.$message.error('原材料批次输入的格式不正确')
            return
          }
          let {batchOrderCode, ...others} = formData;
          let [batchCode, outOrderCode] = batchOrderCode.split('-')
          console.log("表单提交数据",formData)
          this.addLoading = true;
          this.pageLoading = true
          postAction(httpurl,{
            batchCode,
            outOrderCode,
            ...others
          }).then((res)=>{
            this.addLoading = false
            this.pageLoading = false
            if(res.success){
              const hasIndex = this.waitFeedingData.findIndex(ele => ele.batchCode === res.result.batchCode)
              if (hasIndex > -1) {
                this.$message.warning(`原材料批次${res.result.batchCode}已添加`);
                return
              }
              this.waitFeedingData.push(res.result);
              this.$message.success(res.message);
              this.$nextTick(() => {
                this.form.setFieldsValue({
                  qty: '',
                  batchCode: '',
                  warehouseCode: ''
                })
              })
            }else{
              this.$message.warning(res.message);
            }
          }).catch(() => {
            this.addLoading = false;
            this.pageLoading = false
          })
        }
      })
    },
    changeLine(value) {
      this.selectedStation = undefined
      this.getStations()
      this.getMixinBatchInfo()
    },
    changeStation(value) {
      this.getWareHouses(value)
    },
    // 获取产线
    getLines() {
      getAction('/line/line/list', {
        pageNo: 1,
        pageSize: 99999999,
        keyword: '蛋白'
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
      getAction('/customFeedRecord/getBlendLot', {
        lineId: this.selectedLine
      }).then(res => {
        this.pageLoading = false
        if(res.success === true && res.result) {
          this.mixinBatchInfo = res.result;
          this.getHasFeedingData(res.result)
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
