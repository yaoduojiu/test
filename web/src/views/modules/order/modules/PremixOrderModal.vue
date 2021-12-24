<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    switchFullscreen
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row>
          <a-col :span="24">
            <a-form-item label="订单编号" :labelCol="labelCol" :wrapperCol="wrapperCol" hidden>
              <a-input v-decorator="['code']" placeholder="请输入订单编号" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="集合料编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['materialCode', validatorRules.materialCode]"
                :trigger-change="true"
                org-fields="code,name"
                dest-fields="materialCode,materialName"
                code="bb_hh_material"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="集合料名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialName']" placeholder="请输入集合料名称" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['status', validatorRules.status]" :trigger-change="true" dictCode="order_status" placeholder="请选择状态" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="订单数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['qty']" placeholder="请输入订单数量" style="width: 100%" />
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </a-spin>
  </j-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: "PremixOrderModal",
    components: {
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        validatorRules: {
          materialCode: {
            rules: [
              { required: true, message: '请输入集合料编码!'},
            ]
          },
          status: {
            rules: [
              { required: true, message: '请输入状态!'},
            ]
          },
        },
        url: {
          add: "/order/premixOrder/add",
          edit: "/order/premixOrder/edit",
        }

      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'code','materialCode','materialName','status','qty'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }

        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'code','materialCode','materialName','status','qty'))
      },


    }
  }
</script>