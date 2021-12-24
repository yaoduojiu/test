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
    <!-- 添加投料 -->
    <a-divider orientation="left">
    操作栏
    </a-divider>
    <div>
      <a-form>
        <a-row :gutter="20">
          <a-col :span="8">
            <a-form-item label="原材料批次" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input
                @pressEnter="feed"
                v-model="batchSn"
                placeholder="请先输入或扫描" :allowClear="true"/>

            </a-form-item>
          </a-col>
            <a-col :span="8" style="padding-top: 4px;">
          <a-button type="primary" :loading="addLoading" @click="feed">添加投料</a-button>
        </a-col>
        </a-row>
      </a-form>
    </div>
    <a-spin :spinning="pageLoading">
      <a-divider orientation="left">
        当前投料订单
      </a-divider>
      <div id="batchInfoTable">
        <table>
          <tr>
            <td class="label">订单编号</td>
            <td class="">{{ feedOrderInfo.code ? feedOrderInfo.code : '--' }}</td>
            <td class="label">主原料</td>
            <td class="">{{ feedOrderInfo.materialCode ? feedOrderInfo.materialCode : '--' }}</td>
            <td class="label">主原料名称</td>
            <td class="">{{ feedOrderInfo.materialName ? feedOrderInfo.materialName : '--' }}</td>
          </tr>
          <tr>
            <td class="label">状态</td>
            <td class="">{{ feedOrderInfo.status_dictText ? feedOrderInfo.status_dictText : '--' }}</td>
            <td class="label">订单数量</td>
            <td class="">{{ feedOrderInfo.qty ? feedOrderInfo.qty : '--' }}</td>
            <td class="label">原料批次</td>
            <td class="">{{ feedOrderInfo.batchNo ? feedOrderInfo.batchNo : '--' }}</td>
          </tr>
        </table>
      </div>
    </a-spin>

    <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="已投料" key="1" >
        <a-table
          size="middle"
          :scroll="{x:true}"
          bordered
          :rowKey="(record,index)=>{return index}"
          :columns="columns"
          :dataSource="data"
          class="j-table-force-nowrap">
        </a-table>
      </a-tab-pane>
    </a-tabs>
    <a-modal v-model="visible" title="订单">
      <template slot="footer">
        <a-button key="back" @click="visible=false">
          关闭
        </a-button>
      </template>
      <div style="font-size: 16px; font-weight: bold">
      <p>订单编号： {{feedOrderInfo.code}}</p>
      <p>原料名称： {{feedOrderInfo.materialName}}</p>
      <p>订单数量： {{feedOrderInfo.qty}}</p>
      <p>状态： {{feedOrderInfo.status_dictText}}</p>
      <p>原料批次： {{feedOrderInfo.batchNo}}</p>
      </div>
    </a-modal>
  </a-card>
</template>

<script>
import { getAction, postAction, deleteAction } from '@/api/manage'
export default {
  name: 'topWarehouseFeeding',
  data() {
    return {
      pageLoading: false,
      addLoading: false,
      visible: false,
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
          title:'投料类型',
          align:"center",
          dataIndex: 'type_dictText'
        },
        {
          title:'投料人',
          align:"center",
          dataIndex: 'createBy'
        },
        {
          title:'投料时间',
          align:"center",
          dataIndex: 'createTime'
        },
      ],
      data: [],
      lines: [],
      selectedLine: undefined,
      stations: [],
      selectedStation: undefined,
      feedOrderInfo: {},
      labelCol: {
        xs: { span: 24 },
        sm: { span: 8 },
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 },
      },
      warehouses: [],
      batchSn: '',
      form: {
        batchSn: '',
        id: '',
        stationCode: '',
      },
    }
  },
  created() {
    this.getLines()
  },
  mounted() {

  },
  methods: {
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

    feed() {
      let _this = this,
        batchInfo = {};
      if(!this.selectedStation || !this.selectedLine) {
        this.$message.error('请选择产线、工位')
        return
      }
      if (!this.batchSn) {
        this.$message.warning('请先输入或扫描原材料批次!');
        return;
      }
      this.form.batchSn = this.batchSn
      this.addLoading = true;
      this.form.id = this.feedOrderInfo.id
      this.form.stationCode = this.selectedStation;
      getAction('/storage/feed/getBatchInfoBySn', {
        sn: this.batchSn,
        materialCode: this.feedOrderInfo.materialCode,
        lineId: this.selectedLine,
        type: 1
      }).then(res => {
        let cString = '';
        if (res.code === 200 && res.success === true) {
          batchInfo = res.result;
          return res.result.materialCode === this.feedOrderInfo.materialCode;
        } else {
          this.$message.warning(res.message);
          this.addLoading = false;
          return Promise.reject();
        }
      }).then((flag) => {
         if (flag) {
          this.$confirm({
            title: '确定投料',
            content: (
              <div style="font-size: 16px">
                <p style="font-weight: bold">原材料核对正确</p>
                <p>原料代码：{batchInfo.materialCode}</p>
                <p>入库时间：{batchInfo.createTime}</p>
                <p>数量：{batchInfo.qty}</p>
              </div>
            ),
            okText: '投料',
            cancelText: '取消',
            onOk() {
              postAction('/storage/feed', {..._this.form}).then(res => {
                if(res.success === true && res.result) {
                  _this.$message.warning("投料成功!");
                  _this.feedOrderInfo.batchNo = res.result.batchNo;
                } else {
                  _this.$message.error(res.message)
                }
              }).catch((e) => {
                _this.$message.warning(e.response.data.message);
              }).finally(()=>{
                _this.addLoading = false;
                Object.keys(this.modal.form).forEach(key => {
                  _this.form[key] = '';
                })
              });
            },
            onCancel() {
              _this.addLoading = false;
            }
          });
        } else {
          this.$error({
            title: '确定投料',
             content: (
              <div style="font-size: 16px">
                <p style="font-weight: bold">原材料核对错误</p>
                <p>原料代码：{batchInfo.materialCode}</p>
                <p>入库时间：{batchInfo.createTime}</p>
                <p>数量：{batchInfo.qty}</p>
              </div>
            ),
            okText: '取消',
            okType: 'danger',
            onOk() {
              _this.addLoading = false;
            }
          });
        }
      }).finally(()=> {
        this.batchSn = ''
      });
    },

    changeLine(value) {
      this.selectedStation = undefined
      this.data = []
      this.getStations()
      this.getFeedOrderInfo()
    },
    changeStation(value) {
      this.getWareHouses(value)
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
          this.selectedStation = res.result.records[0] &&  res.result.records[0].id;
        }
      })
    },
    // 获取当前生成订单
    getFeedOrderInfo() {
      this.pageLoading = true;
      getAction('/storage/feed/getFeedOrder', {
        lineId: this.selectedLine
      }).then(res => {
        this.pageLoading = false
        if(res.success === true && res.result) {
          this.feedOrderInfo = res.result;
          this.visible = true;
          this.getFeedRecord(res.result);
        } else {
          this.feedOrderInfo = {}
          this.$message.error(res.message)
        }
      }).catch((e) => {
        this.pageLoading = false;
        this.feedOrderInfo = {}
        this.$message.warning(e.response.data.message);
      });
    },

    getFeedRecord(data) {
      getAction('/storage/feed/feedRecord', {
        orderCode: data.code
      }).then(res => {
        if (res.success) {
          this.data = res.result
        }
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
  .buttons {
    clear: both;
    padding-top: 10px;
  }
    .default {
    background-color: #fff;
  }
  .active {
    background-color: yellow;
  }
  .done {
    background-color: green;
  }
</style>
