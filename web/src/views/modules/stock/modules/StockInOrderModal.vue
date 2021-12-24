<template>
  <a-modal
    :title="title"
    :confirm-loading="confirmLoading"
    @ok="handleOk"
    :width="width"
    @cancel="handleCancel"
    :visible="visible">
    <stock-in-order-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit" normal></stock-in-order-form>
  </a-modal>
</template>

<script>

  import StockInOrderForm from './StockInOrderForm'

  export default {
    name: 'StockInOrderModal',
    components: {
      StockInOrderForm
    },
    data () {
      return {
        title:"操作",
        // width:800,
        visible: false,
        disableSubmit: false,
        confirmLoading: false,
        width: 800
      }
    },
    methods: {
      add () {
        this.visible=true
        this.$nextTick(()=>{
          this.$refs.realForm.add();
        })
      },
      edit (record) {
        this.visible=true
        this.$nextTick(()=>{
          this.$refs.realForm.edit(record);
        });
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      submitCallback(){
        this.$emit('ok');
        this.visible = false;
      },
      handleOk () {
        this.$refs.realForm.submitForm();
      },
      handleCancel () {
        this.close()
      }
    }
  }
</script>

<style lang="less" scoped>
/** Button按钮间距 */
  .ant-btn {
    margin-left: 30px;
    margin-bottom: 30px;
    float: right;
  }
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