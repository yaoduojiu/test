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
            <a-form-item label="程序包名" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['appPackageName', validatorRules.appPackageName]" placeholder="请输入程序包名" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="App版本" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['appVersion', validatorRules.appVersion]" placeholder="请输入App版本" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="程序地址" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-upload v-decorator="['url', validatorRules.url]" :trigger-change="true"  ></j-upload>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="版本简介" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['appRemarks']" placeholder="请输入版本简介" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="启用状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-switch v-decorator="['status', validatorRules.status]"  ></j-switch>
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
    name: "AppVersionModal",
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
          appPackageName: {
            rules: [
              { required: true, message: '请输入程序包名!'},
            ]
          },
          appVersion: {
            rules: [
              { required: true, message: '请输入App版本!'},
              { validator: (rule, value, callback) => validateDuplicateValue('app_version', 'app_version', value, this.model.id, callback)},
            ]
          },
          url: {
            rules: [
              { required: true, message: '请输入程序地址!'},
            ]
          },
          status: {
            initialValue:"Y",
            rules: [
            ]
          },
        },
        url: {
          add: "/app/appInfo/addAppVersion",
          edit: "/app/appInfo/editAppVersion",
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
          this.form.setFieldsValue(pick(this.model,'appId','appPackageName','appVersion','url','appRemarks','status','createBy','createTime','updateBy','updateTime','sysOrgCode'))
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
            formData['appId'] = this.mainId
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
        this.form.setFieldsValue(pick(row,'appId','appPackageName','appVersion','url','appRemarks','status','createBy','createTime','updateBy','updateTime','sysOrgCode'))
      },


    }
  }
</script>
