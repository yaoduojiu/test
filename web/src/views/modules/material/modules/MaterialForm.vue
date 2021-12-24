<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="物料编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['code', validatorRules.code]" placeholder="请输入物料编码"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['name', validatorRules.name]" placeholder="请输入物料名称"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-multi-select-tag type="list_multi" v-decorator="['type', validatorRules.type]" :trigger-change="true" dictCode="material_type" placeholder="请选择物料类型" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料单位" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['unit']" :trigger-change="true" dictCode="air_china_unit" placeholder="请选择物料单位" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="包装型号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['packingCode']" placeholder="请输入包装型号" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="产品标准编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['standardCode']" placeholder="请输入产品标准编号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="成分含量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['formula']" placeholder="请输入成分含量"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="原料组成" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['bom']" placeholder="请输入原料组成"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="使用说明" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['instructions']" placeholder="请输入使用说明"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="产品功效" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['productEfficacy']" placeholder="请输入产品功效"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="保质期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['quality']" placeholder="请输入保质期"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="储存条件" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['storageConditions']" placeholder="请输入储存条件"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['remarks']" placeholder="请输入备注"  ></a-input>
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
    name: 'MaterialForm',
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
          code: {
            rules: [
              { required: true, message: '请输入物料编码!'},
              { validator: (rule, value, callback) => validateDuplicateValue('hh_material', 'code', value, this.model.id, callback)},
            ]
          },
          name: {
            rules: [
              { required: true, message: '请输入物料名称!'},
            ]
          },
          type: {
            rules: [
              { required: true, message: '请输入物料类型!'},
            ]
          },
        },
        url: {
          add: "/material/material/add",
          edit: "/material/material/edit",
          queryById: "/material/material/queryById"
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
          this.form.setFieldsValue(pick(this.model,'code','name','type','unit', 'packingCode','standardCode', 'formula','bom','instructions','productEfficacy','quality','storageConditions','remarks','createBy','createTime','updateBy','updateTime'))
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
        this.form.setFieldsValue(pick(row,'code','name','type','unit', 'packingCode','standardCode', 'formula','bom','instructions','productEfficacy','quality','storageConditions','remarks','createBy','createTime','updateBy','updateTime'))
      },
    }
  }
</script>