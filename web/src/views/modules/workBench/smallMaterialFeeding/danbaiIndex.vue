<template>
  <a-card :bordered="false">
    <!-- 筛选区域 -->
    <div style="display:none">
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
        <!-- 当前成品生产订单 -->
      <a-divider orientation="left">
        当前成品生产订单
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
      操作栏
      </a-divider>
      <div>
        <a-form :form="form">
          <a-row :gutter="20">
            <a-col :span="8">
              <a-form-item label="原材料批次" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <a-input
                  @keyup="keyupHandle"
                  v-model="batchSn"
                  :allowClear="true"
                  placeholder="请输入" />
                  <a-alert v-show="batchInfo.isShow" :message="batchInfo.msg" :type="batchInfo.type" show-icon />

              </a-form-item>
            </a-col>
             <a-col :span="8" style="padding-top: 4px;">
            <a-button type="primary" :loading="addLoading" @click="addFeedingClick">添加投料</a-button>
            <a-button style="margin-left: 10px;" :disabled="relateOrderLoading" type="default" :loading="relateOrderLoading" @click="relateOrderClick">关联投料订单</a-button>
          </a-col>
          </a-row>
        </a-form>
      </div>
      <div>
        <a-tabs defaultActiveKey="1" @change="TabChange">
          <a-tab-pane tab="待投料" key="1" >
            <waitFeedingTable @feeding="feedingClick" :data="waitFeedingData" :data1="formulaDetails" :againsList='againsList' @remove="removeHandle"></waitFeedingTable>
          </a-tab-pane>
          <a-tab-pane tab="已投料" key="2">
            <hasFeedingTable :data="hasFeedingData"></hasFeedingTable>
          </a-tab-pane>
        </a-tabs>
      </div>
    </a-spin>
    <!-- 关联订单弹出框 -->
    <feedingOrderModal @ok="relationOk" ref="feedingOrderModal"/>
  </a-card>
</template>

<script>
import { getAction, postAction } from '@/api/manage'
import waitFeedingTable from '@/views/modules/workBench/smallMaterialFeeding/waitFeedingTable'
import hasFeedingTable from '@/views/modules/workBench/smallMaterialFeeding/hasFeedingTable'
import feedingOrderModal from '@/views/modules/workBench/smallMaterialFeeding/feedingOrderModal'
export default {
  components: {
    waitFeedingTable,
    hasFeedingTable,
    feedingOrderModal
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
      formulaDetails: [],
      hasFeedingData: [],
      warehouses: [],
      relateOrderLoading: false,
      batchSn: '',
      againsList: [],
      batchInfo: {
        msg: '',
        isShow: false,
        type: 'success'
      }
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
      let _this = this;
      if(!this.selectedStation || !this.selectedLine) {
        this.$message.error('请选择产线、工位')
        return
      }
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

              if (res.result.length > 0) {
                _this.againsList = res.result;
              }

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
    relationOk(keys) {
      const params = {
        feedOrderId: keys[0],
        productOderId: this.mixinBatchInfo.id
      }
      const _this = this
      this.$confirm({
        title: '提示',
        content: '确定要进行关联订单操作吗?',
        okText: '确定',
        cancelText: '取消',
        onOk() {
          _this.relateOrderLoading = true
          postAction('/customFeedRecord/updateBlendLotOrder', params).then(res => {
            if (res.success) {
              _this.$message.success('操作成功')
            } else{
              _this.$message.warning(res.message);
            }
            _this.relateOrderLoading = false
          }).catch(() => {
            _this.relateOrderLoading = false
          })
        }
      })
    },
    relateOrderClick() {
      // if (!this.mixinBatchInfo.id) {
      //   this.$message.error('请先查询出当前成品生产订单信息')
      //   return
      // }
      this.$refs.feedingOrderModal.visible = true
    },
    keyupHandle(event) {
      if (event.key === 'Enter') {
        if (event.target.value !== '') {
          this.addBatch(event.target.value)
        } else {
          this.$message.error('请先输入或扫描')
        }
      }
    },
    addBatch(batchSn) {
      var _this = this;
      this.addLoading = true;
      this.pageLoading = true
      postAction('/customFeedRecord/getBatchAndCheck',{
        batchSn: batchSn,
        lineId: _this.selectedLine,
        orderCode: _this.mixinBatchInfo.orderCode,
        type:3
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
    addFeedingClick() {
      if(this.batchSn) {
        this.addBatch(this.batchSn)
      } else {
        this.$message.error('请先输入或扫描')
      }
    },
    changeLine(value) {
      this.selectedStation = undefined
      this.formulaDetails = []
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
          this.selectedStation = res.result.records[0] &&  res.result.records[0].id;
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
          this.getFormulaDetails(res.result);
        } else {
          this.mixinBatchInfo = {}
          this.$message.error(res.message)
        }
      }).catch(() => {
        this.pageLoading = false
      })
    },

    getFormulaDetails(data) {
      this.pageLoading = true;
      getAction('/customFeedRecord/listDetailsByCode', {
        orderCode: data.orderCode
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
</style>
