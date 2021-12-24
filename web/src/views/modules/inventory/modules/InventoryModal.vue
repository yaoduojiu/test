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
            <a-form-item label="库位" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-multi-select-tag type="list_multi" v-decorator="['locationId']" :trigger-change="true" dictCode="hh_location,code,id" placeholder="请选择库位" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-multi-select-tag type="list_multi" v-decorator="['materialId']" :trigger-change="true" dictCode="hh_material,name,id" placeholder="请选择物料" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['remarks']" placeholder="请输入备注" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="盘点类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-select mode="multiple" placeholder="请选择盘点类型" default-value="type" style="width:100%" @change="handleTypeChange">
                <a-select-option value="物料盘点">
                  物料盘点
                </a-select-option>
                <a-select-option value="库位盘点">
                  库位盘点
                </a-select-option>
              </a-select>
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
    name: "InventoryModal",
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
        },
        url: {
          add: "/inventory/inventory/add",
          edit: "/inventory/inventory/edit",
        }

      }
    },
    created () {
    },
    methods: {
      handleTypeChange() {

      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'no','warehostCode','warehouseName','type','status','locationId','materialId','remarks'))
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
        this.form.setFieldsValue(pick(row,'no','warehostCode','warehouseName','type','status','locationId','materialId','remarks','createBy','createTime'))
      },


    }
  }
</script>