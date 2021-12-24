<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="物料编码ID" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialId']" placeholder="请输入物料编码ID" disabled ></a-input>
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
              <a-input v-decorator="['materialName']" placeholder="请输入物料名称" disabled ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['qty', validatorRules.qty]" placeholder="请输入数量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="批次类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['type', validatorRules.type]" :trigger-change="true" dictCode="batch_type" placeholder="请选择批次类型" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="供应商编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['supplierCode']"
                :trigger-change="true"
                org-fields="code,name"
                dest-fields="supplierCode,supplierName"
                code="pp_supplier"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="供应商名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['supplierName']" placeholder="请输入供应商名称"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="生产日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择生产日期" v-decorator="['productionDate']" :trigger-change="true" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="质保日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择质保日期" v-decorator="['warrantyDate']" :trigger-change="true" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="来料日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择来料日期" v-decorator="['inDate']" :trigger-change="true" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col v-if="showFlowSubmitButton" :span="24" style="text-align: center">
            <a-button @click="submitForm">提 交</a-button>
          </a-col>
        </a-row>
      </a-form>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'BatchInfoForm',
    components: {
    },
    props: {
      //流程表单data
      formData: {
        type: Object,
        default: ()=>{},
        required: false
      },
      //表单模式：true流程表单 false普通表单
      formBpm: {
        type: Boolean,
        default: false,
        required: false
      },
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
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
        confirmLoading: false,
        validatorRules: {
          materialCode: {
            rules: [
              { required: true, message: '请输入物料编码!'},
            ]
          },
          qty: {
            rules: [
              { required: true, message: '请输入数量!'},
            ]
          },
          type: {
            rules: [
              { required: true, message: '请输入批次类型!'},
            ]
          },
        },
        url: {
          add: "/batchInfo/batchInfo/add",
          edit: "/batchInfo/batchInfo/edit",
          queryById: "/batchInfo/batchInfo/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return false
          }
          return true
        }
        return this.disabled
      },
      showFlowSubmitButton(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return true
          }
        }
        return false
      }
    },
    created () {
      //如果是流程中表单，则需要加载流程表单data
      this.showFlowData();
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
          this.form.setFieldsValue(pick(this.model,'code','materialId','materialCode','materialName','qty','status','type','inQty','supplierCode','supplierName','productionDate','warrantyDate','inDate','warehouseName','localtionCode','createBy','createTime','updateBy','updateTime'))
        })
      },
      //渲染流程表单数据
      showFlowData(){
        if(this.formBpm === true){
          let params = {id:this.formData.dataId};
          getAction(this.url.queryById,params).then((res)=>{
            if(res.success){
              this.edit (res.result);
            }
          });
        }
      },
      submitForm () {
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
            })
          }
         
        })
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'code','materialId','materialCode','materialName','qty','status','type','inQty','supplierCode','supplierName','productionDate','warrantyDate','inDate','warehouseName','localtionCode','createBy','createTime','updateBy','updateTime'))
      },
    }
  }
</script>