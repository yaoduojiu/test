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
            <a-form-item label="原材料物料编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['materialCode', validatorRules.materialCode]"
                :trigger-change="true"
                org-fields="code,name,id"
                dest-fields="materialCode,materialName,materialId"
                code="material_rew"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="原材料物料名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialName']" placeholder="请输入原材料物料名称" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="理论数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['theoryQty']" placeholder="请输入理论数量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="实际数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['actualQty']" placeholder="请输入实际数量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="偏差数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['deviationQty']" placeholder="请输入偏差数量" style="width: 100%" />
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
    name: "OrderDetailsModal",
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
              { required: true, message: '请输入原材料物料编码!'},
            ]
          },
        },
        url: {
          add: "/orderBlendLot/blendLot/addOrderDetails",
          edit: "/orderBlendLot/blendLot/editOrderDetails",
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
          this.form.setFieldsValue(pick(this.model,'orderCode','orderId','materialCode','materialName','materialId','theoryQty','actualQty','deviationQty','createBy','createTime','updateBy','updateTime','sysOrgCode'))
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
            formData['orderId'] = this.mainId
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
        this.form.setFieldsValue(pick(row,'orderCode','orderId','materialCode','materialName','materialId','theoryQty','actualQty','deviationQty','createBy','createTime','updateBy','updateTime','sysOrgCode'))
      },


    }
  }
</script>
