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
            <a-form-item label="仓库编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['code', validatorRules.code]" placeholder="请输入仓库编号" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="仓库名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['name', validatorRules.name]" placeholder="请输入仓库名称" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="仓库类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="radio" v-decorator="['type', validatorRules.type]" :trigger-change="true" dictCode="material_type" placeholder="请选择仓库类型" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="可见范围" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-multi-select-tag type="list_multi" v-decorator="['users']" :trigger-change="true" dictCode="sys_user,realname,id" placeholder="请选择可见范围" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['remarks']" placeholder="请输入备注" ></a-input>
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
    name: "WarehouseModal",
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
          code: {
            rules: [
              { required: true, message: '请输入仓库编号!'},
              { validator: (rule, value, callback) => validateDuplicateValue('hh_warehouse', 'code', value, this.model.id, callback)},
            ]
          },
          name: {
            rules: [
              { required: true, message: '请输入仓库名称!'},
              { validator: (rule, value, callback) => validateDuplicateValue('hh_warehouse', 'name', value, this.model.id, callback)},
            ]
          },
          type: {
            rules: [
              { required: true, message: '请输入仓库类型!'},
            ]
          },
          status: {
            initialValue:"Y",
            rules: [
            ]
          },
        },
        url: {
          add: "/warehouse/warehouse/add",
          edit: "/warehouse/warehouse/edit",
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
          this.form.setFieldsValue(pick(this.model,'code','name','type','users','remarks','status','createBy','createTime','updateBy','updateTime'))
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
                that.$store.dispatch('Warehouse');
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
        this.form.setFieldsValue(pick(row,'code','name','type','users','remarks','status','createBy','createTime','updateBy','updateTime'))
      },

      
    }
  }
</script>