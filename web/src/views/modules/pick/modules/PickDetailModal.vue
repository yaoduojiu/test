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
            <a-form-item hidden label="物料ID" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialId']" placeholder="请输入物料ID" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['materialCode', validatorRules.materialCode]"
                :trigger-change="true"
                org-fields="id,code,name"
                dest-fields="materialId,materialCode,materialName"
                code="bb_hh_material"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialName']" placeholder="请输入物料名称" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="领料数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['pickQty', validatorRules.pickQty]" placeholder="请输入领料数量" style="width: 100%" />
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
    name: "PickDetailModal",
    components: {
    },
    props:{
      mainId:{
        type:String,
        required:false,
        default:''
      }
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
              { required: true, message: '请输入物料编码!'},
            ]
          },
          pickQty: {
            rules: [
              { required: true, message: '请输入领料数量!'},
            ]
          },
        },
        url: {
          add: "/pick/pickList/addPickDetail",
          edit: "/pick/pickList/editPickDetail",
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
          this.form.setFieldsValue(pick(this.model,'pickId','materialId','materialCode','materialName','pickQty','receiveQty','createBy','createTime','updateBy','updateTime','sysOrgCode','remark'))
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
            formData['pickId'] = this.mainId
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
        this.form.setFieldsValue(pick(row,'pickId','materialId','materialCode','materialName','pickQty','receiveQty','createBy','createTime','updateBy','updateTime','sysOrgCode','remark'))
      },


    }
  }
</script>
