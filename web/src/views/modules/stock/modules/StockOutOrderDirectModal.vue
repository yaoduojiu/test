<template>
  <a-modal
    :title="title"
    :confirm-loading="confirmLoading"
    :width="width"
    @cancel="handleCancel"
    :visible="visible">
    <stock-out-order-direct-form ref="realForm" @ok='handleOk'></stock-out-order-direct-form>
    <template slot="footer">
      <a-button  @click="handleCancel">
        取消
      </a-button>
      <a-button  @click="temporary">
        暂存
      </a-button>
      <a-button  type="primary"  @click="outOrder">
        直接出库
      </a-button>
    </template>
  </a-modal>

</template>

<script>

  import StockOutOrderDirectForm from './StockOutOrderDirectForm'

  export default {
    name: 'StockOutOrderDirectModal',
    components: {
      StockOutOrderDirectForm
    },
    data () {
      return {
        title:"出库",
        // width:800,
        visible: false,
        confirmLoading: false,
        width: 1000
      }
    },
    mounted() {
    },
    watch: {
      visible: {
        handler(val, old) {
          if (val) {
            this.$refs.realForm && Object.keys(this.$refs.realForm.form).forEach(key=>{
              this.$refs.realForm.form[key]=""
            })
          }
        },
        // immediate: true
      }
    },
    methods: {
      close () {
        this.$emit('close');
        this.visible = false;
      },
      submitCallback(){
        this.$emit('ok');
        this.visible = false;
      },
      handleCancel () {
        this.close()
      },
      temporary() {
        this.$refs.realForm.temporary();
      },
      outOrder() {
        this.$refs.realForm.outOrder();
      },
      handleOk() {
        this.close();
        this.$emit('ok');
      }

    }
  }
</script>

<style lang="less" scoped>
/** Button按钮间距 */
  // .ant-btn {
  //   margin-left: 30px;
  //   margin-bottom: 30px;
  //   float: right;
  // }
  .drawer-footer{
    position: absolute;
    bottom: -8px;
    width: 100%;
    border-top: 1px solid #e8e8e8;
    padding: 10px 16px;
    text-align: right;
    left: 0;
    background: #fff;
    border-radius: 0 0 2px 2px;
  }
</style>