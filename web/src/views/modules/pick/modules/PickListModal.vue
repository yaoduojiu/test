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
            <a-form-item label="订单编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['orderCode', validatorRules.orderCode]"
                :trigger-change="true"
                org-fields="code,id,material_code,material_name,material_id"
                dest-fields="orderCode,orderId,materialCode,materialName,materialId"
                code="order_popup"
                @callback="popupCallback"
                />
            </a-form-item>
            <a-form-item hidden>
              <a-input v-decorator="['orderId']" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item hidden label="物料ID" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialId']" placeholder="请输入物料ID" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item hidden label="物料编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialCode']" placeholder="请输入物料编码" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item hidden label="物料名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialName']" placeholder="请输入物料名称" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="领料人" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-search-select-tag v-decorator="['pickUser', validatorRules.pickUser]" dict="sys_user,realname,id"  />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-textarea v-decorator="['remarks']" rows="4" placeholder="请输入备注" />
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
    name: "PickListModal",
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
          orderCode: {
            rules: [
              { required: true, message: '请输入订单编号!'},
            ]
          },
          pickUser: {
            rules: [
              { required: true, message: '请选择领料人!'},
            ]
          },
        },
        url: {
          add: "/pick/pickList/add",
          edit: "/pick/pickList/edit",
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
          this.form.setFieldsValue(pick(this.model,'code','orderCode','orderId','materialId','materialCode','materialName','pickUser','remarks','status','createBy','createTime','updateBy','updateTime'))
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
        this.form.setFieldsValue(pick(row,'code','orderCode','orderId','materialId','materialCode','materialName','pickUser','remarks','status','createBy','createTime','updateBy','updateTime'))
      },

      
    }
  }
</script>