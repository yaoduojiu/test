<template>
  <j-modal
    :title="title"
    :width="width"
    placement="right"
    :closable="false"
    @cancel="handleCancel"
    cancelText="关闭"
    okText="打印"
    @ok="submitForm"
    :visible="visible">
    <a-spin :spinning="printLoading">
      <a-form :form="form">
          <a-row>
            <a-col :span="24">
              <a-form-item label="打印机" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <j-dict-select-tag type="list" v-decorator="['printerName', validatorRules.printerName]" :trigger-change="true" dictCode="batch_print_name" placeholder="请选择打印机" />
              </a-form-item>
            </a-col>
            <a-col :span="24">
              <a-form-item label="打印模板" :labelCol="labelCol" :wrapperCol="wrapperCol">
                <j-dict-select-tag type="list" v-decorator="['printBtw', validatorRules.printBtw]" :trigger-change="true" dictCode="batch_print_btw" placeholder="请选择打印模板" />
              </a-form-item>
            </a-col>
            <a-col :span="24">
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
            </a-col>
            <a-col :span="24">
              <a-form-item label="打印张数" :labelCol="labelCol" :wrapperCol="wrapperCol">
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
            </a-col>
          </a-row>
        </a-form>
    </a-spin>
  </j-modal>
</template>

<script>
  import { httpAction, getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  export default {
    name: 'BatchPrintModal',
    props: {
      ids: {
        required: true,
        default: ''
      }
    },
    data () {
      return {
        title:"操作",
        width:600,
        visible: false,
        disablePrint: false,
        printLoading: false,
        form: this.$form.createForm(this),
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        validatorRules: {
          printBtw: {
            rules: [
              { required: true, message: '请选择打印模板!'},
            ]
          },
          printerName: {
            rules: [
              { required: true, message: '请选择打印机!'},
            ]
          }
        }
      }
    },
    methods: {
      initForm () {
        this.form.resetFields();
        this.model = {}
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'printBtw', 'printerName', 'standardQty', 'sheetQty'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.printLoading = true;
            let formData = Object.assign(this.model, {id: this.ids}, values);
            console.log("表单提交数据",formData)
            httpAction('/batchInfo/batchInfo/printBatch',formData,'post').then((res)=>{
              that.printLoading = false;
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.printLoading = false;
            })
          }
         
        })
      },
      printCallback(){
        this.$emit('ok');
        this.visible = false;
      },
      handleOk () {
        this.$refs.realForm.printForm();
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