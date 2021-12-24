<template>
  <a-modal
    :title="title"
    :width="width"
    @cancel="handleCancel"
    :visible="visible">
    <pick-out-form ref="realForm" @select='handleChange' :disabled="disableSubmit" normal  :dataInfo = "dataInfo"></pick-out-form>
     <template slot="footer">
        <a-button  @click="handleCancel">
          取消
        </a-button>
        <a-button  @click="temporary">
          暂存
        </a-button>
        <a-button  type="primary"  @click="pick">
          直接出库
        </a-button>
      </template>
  </a-modal>
</template>

<script>
import { postAction } from '../../../../api/manage';

  import PickOutForm from './PickOutForm'

  export default {
    name: 'PickOutModal',
    components: {
      PickOutForm
    },
    props: {
       dataInfo: {
        type: Object,
        default: ()=>{},
        required: true
      }
    },
    data () {
      return {
        title:"领料出库",
        // width:800,
        visible: false,
        disableSubmit: false,
        width: 1000,
        selected: []
      }
    },
    watch: {
      visible: function (newQuestion, oldQuestion) {
        if (newQuestion) {
          this.$refs.realForm && this.$refs.realForm.getData();
        }
      }
    },
    methods: {
      // add () {
      //   this.visible=true
      //   this.$nextTick(()=>{
      //     this.$refs.realForm.add();
      //   })
      // },
      // edit (record) {
      //   this.visible=true
      //   this.$nextTick(()=>{
      //     this.$refs.realForm.edit(record);
      //   });
      // },
      close () {
        this.$emit('close');
        this.selected = [];
        this.visible = false;
      },
      handleCancel () {
        this.close()
        this.$refs.realForm.sn = ''
        this.$refs.realForm.data = []
      },
      handleChange(arr) {
        this.selected = arr;
      },
      temporary() {
        if (this.parseData().length === 0) {
          this.$message.error('请先扫描出或者勾选批次')
          return
        } 
        postAction('/api/pickOrder/pc/temporary', this.parseData()).then(res=> {
          if (res.success) {
            this.$message.success(res.message);
            this.close();
            this.$emit('ok')
          } else {
            this.$message.warn(res.message);
          }
        })
      },
      pick() {
        if (this.parseData().length === 0) {
          this.$message.error('请先扫描出或者勾选批次')
          return
        } 
        postAction('/api/pickOrder/pc/pick', this.parseData()).then(res=> {
          if (res.success) {
            this.$message.success(res.message);
            this.close();
            this.$emit('ok')
          } else {
            this.$message.warn(res.message);
          }
        })
      },
      parseData() {
        let results = [];
        this.selected.forEach(item => {
          results.push({
            batchSn: item.batchSn,
            locationCode: item.locationCode,
            pickDetailId: this.dataInfo.id
          })
        });
        return results;
      }
    }
  }
</script>

<style lang="less" scoped>


</style>