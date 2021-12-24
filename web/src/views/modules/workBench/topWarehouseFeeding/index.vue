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
    <a-spin :spinning="topListLoading">
    <div style=" margin-top: 20px">
      <a-row :gutter="16">

        <a-col v-for="item in topList" :key="item.whCode" :span="8" >
          <a-card :title="item.whCode" :bordered="true" @click.native="setColor(item)" :class="item.status" style="height:350px; margin-bottom: 16px;">
            <p>当前可投: <strong>{{item.materialCode}}/{{item.materialName}}</strong></p>
            <p>最近投料批次: <strong>{{item.batchNo}}</strong></p>
            <p>
              <span v-if="item.qty">投料重量：<strong>{{item.qty}}KG</strong></span> &nbsp;
            </p>
            <p>
              <span v-if="item.latestQty">最近投料重量：<strong>{{item.latestQty}}KG</strong></span> &nbsp;
            </p>
            <p>
              <span v-if="item.dataQty">今日累计：<strong>{{item.dataQty}}KG</strong></span> &nbsp;
            </p>
            <p>
              <span v-if="item.needQty">需求量：<strong>{{item.needQty}}KG</strong></span> &nbsp;
            </p>
            <a-space class="buttons" align="center" size="large">
              <a-button type="primary" style="float: left;" @click="feed(item)">投料</a-button>
              <a-button v-if="item.hide" type="primary" style="float: left;" @click="feedEnd(item)">投料完成</a-button>
              <a-button type="danger" style="float: right;" @click="cancel(item)">撤销</a-button>
            </a-space>
          </a-card>
        </a-col>

      </a-row>
    </div>
    </a-spin>
    <a-modal
      title="吨袋投料"
      :width="600"
      @cancel="handleCancel"
      :visible="modal.visible">
      <template slot="footer">
        <a-button key="back" @click="handleCancel">
          取消
        </a-button>
        <a-button v-show="modal.data.sn && modal.materialName === modal.item.materialName" key="submit" type="primary"  @click="handleOks">
          确定
        </a-button>
      </template>
      <a-form-item label="原料条码">
        <a-input v-model="modal.form.sn" @pressEnter="handleOk" :autoFocus='true' placeholder="扫描输入" />
      </a-form-item>
      <div style="margin-top: -10px">
        <a-alert v-if="modal.data.sn && modal.materialName === modal.item.materialName" message="原料核对正确" type="success"  show-icon />
        <a-alert v-if="modal.data.sn && modal.materialName !== modal.item.materialName" message="原料核对错误" type="error" show-icon />
      </div>

      <div style="margin-top:10px">
        <span style="margin-right:40px" v-if="modal.data.sn">条码：{{modal.data.sn}} </span>
        <span v-if="modal.data.qty">重量：{{modal.data.qty}}KG </span>
      </div>

    </a-modal>
  </a-card>
</template>

<script>
import { deleteAction, getAction, postAction } from '@/api/manage'

export default {
  name: 'topWarehouseFeeding',
  data() {
    return {
      pageLoading: false,
      topListLoading: false,
      lines: [],
      firstLine: undefined,
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
      topList: [],
      modal: {
        visible: false,
        form: {
          "sn": '',
          "id": '',
          "stationCode": "",
          "topBomId": ""
        },
        item: {},
        data: {
          sn:'',
          qty: ''
        },
        materialName:''
      }
    }
  },
  created() {
    this.getLines()
  },
  mounted() {

  },
  methods: {
    setColor(item) {
      if (item.status !== 'done') {
        item.status = 'active';
        this.topList.forEach(data => {
          if (data !== item) {
            data.status = 'default';
          }
        });
        this.$forceUpdate();
      }
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

    feed(item) {
      if(!this.selectedStation || !this.selectedLine) {
        this.$message.error('请选择产线、工位')
        return
      }
      this.modal.visible = true
      this.modal.form.sn = ''
      this.modal.data.sn = ''
      this.modal.data.qty = ''
      this.modal.form.id = this.feedOrderInfo.id
      this.modal.form.stationCode = this.selectedStation;
      this.modal.form.topBomId = item.id
      this.modal.item = item;
    },

    feedEnd(item) {
      let data = JSON.parse(JSON.stringify(this.modal.form));
      data.batchSn = this.modal.data.sn;
      var _this = this;
      delete data.sn;
      postAction('/top/feed', data).then(res => {
        if(res.success === true && res.result) {
          this.$message.success("投料成功!");
          // this.modal.visible = false
          item.status = 'done'
          item.hide = false
          Object.assign(item, res.result);
          this.topList.forEach(data => {
            if (data !== item && data.status === 'done') {
              data.status = 'default';
            }
          });
          _this.$forceUpdate();
          this.getFeedOrderInfo();
          this.getFeedList();
        } else {
          this.$message.error(res.message)
          delete item.qty;
          item.hide = false;
          this.$forceUpdate();
        }
      }).catch((e) => {
        this.pageLoading = false;
        this.$message.warning(e.response.data.message);
        delete item.qty;
        item.hide = false;
        this.$forceUpdate();
      }).finally(()=>{
        Object.keys(this.modal.form).forEach(key => {
          this.modal.form[key] = '';
        })
      })

    },
    handleOks() {
      if (!(this.modal.data.sn && this.modal.data.qty)) {
        this.$message.warning("请输入物料SN并按回车键!");
        return;
      }
      var _this = this;
      getAction('/top/feed/startFeed', {
        "batchNo": this.modal.item.batchNo,
        "id": _this.feedOrderInfo.id,
        "stationCode": _this.selectedStation,
        "topBomId": this.modal.item.id
      }).then(res => {
        if(res.success === true) {
          this.$message.success("吨袋投料开始");
          this.modal.visible = false
          this.modal.item.sn = this.modal.data.sn;
          this.modal.item.qty = this.modal.data.qty;
          this.modal.item.hide = true
        } else {
          this.$message.error(res.message)
        }
      }).catch((e) => {
        this.$message.error(e.response.data.message)
      })
    },

    handleCancel(){
      this.modal.visible=false
      var _this = this;
      getAction('/top/feed/stopFeed', {
        "batchNo": this.modal.item.batchNo,
        "id": _this.feedOrderInfo.id,
        "stationCode": _this.selectedStation,
        "topBomId": this.modal.item.id
      })
    },

    handleOk() {
      if (!this.modal.form.sn) {
        return;
      }


      getAction('/top/feed/getBatchSnAndCheckBySn', {
        sn: this.modal.form.sn,
        materialCode : this.modal.item.materialCode,
        lineId: this.selectedLine
      }).then(res => {
        if(res.success === true && res.result) {
          this.modal.data.sn = res.result.sn;
          this.modal.data.qty = res.result.qty;
          this.modal.materialName = res.result.materialName;
        } else {
          this.$message.error(res.message)
          this.modal.data.sn = '';
          this.modal.data.qty = '';
        }
      }).catch((e) => {
        this.modal.data.sn = '';
        this.modal.data.qty = '';
        this.$message.warning(e.response.data.message);
      }).finally(()=> {
        this.modal.form.sn = ''
      });

    },
    cancel(item){
      var _this = this;
      this.$confirm({
        title: '撤销投料',
        content: '确定撤销当前投料吗?',
        okText: '确定',
        okType: 'danger',
        cancelText: '取消',
        onOk() {
          deleteAction('/top/feed', {
            "batchNo": item.batchNo,
            "id": _this.feedOrderInfo.id,
            "stationCode": _this.selectedStation,
            "topBomId": item.id
          }).then(res=> {
            if (res.success === true) {
              _this.$message.warning('撤销投料成功！');
              item.status = 'active';
              item.qty = '';
              item.hide = false
              item.batchNo = res.result.batchNo;
              _this.$forceUpdate();
            } else {
              _this.$message.error(res.message)
            }
          }).catch((e) => {
            _this.$message.warning(e.response.data.message);
          });
        }
      });
    },

    changeLine(value) {
      this.selectedStation = undefined
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
      getAction('/top/feed/getFeedOrder', {
        lineId: this.selectedLine
      }).then(res => {
        this.pageLoading = false
        if(res.success === true && res.result) {
          this.feedOrderInfo = res.result;
          this.getFeedList()
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

    getFeedList() {
      this.topListLoading = true;
      getAction('/top/feed/findTopBom', {
        lineId: this.selectedLine
      }).then(res => {
        this.topListLoading = false
        if(res.success === true && res.result) {
          this.topList = res.result;
          this.topList = this.topList.filter(item => {
            return this.feedOrderInfo.materialCode == item.materialCode;
          })
          this.topList.forEach(item => {
            item.status = 'default'
            item.hide = false
          })

        } else {
          this.topList = [];
          this.$message.error(res.message)
        }
      }).catch((e) => {
        this.topListLoading = false;
        this.topList = [];

        this.$message.warning(e.response.data.message);
      });
    }
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
    width: 100%;
  }
    .default {
    background-color: #fff;
  }
  .active {
    background-color: yellow;
  }
  .done {
    background-color: #38e238
  }
</style>
