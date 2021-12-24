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
              <a-select mode="multiple" @change="locationChange" placeholder="请选择库位"
                v-decorator="[
                  'locationId',
                  { rules: [{ required: false, message: '请选择库位' }] },
                ]"
              >
                <a-select-option v-for="(item, index) in locationData" :key="index" :value="item.id">
                  {{ item.code }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-select
                v-decorator="[
                  'materialId',
                  { rules: [{ required: true, message: '请选择物料' }] },
                ]"
               mode="multiple" placeholder="请选择物料">
                <a-select-option v-for="(item, index) in materialData" :key="index" :value="item.id">
                  {{ item.name }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['remarks']" placeholder="请输入备注" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="盘点类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-select
                v-decorator="[
                  'type',
                  { rules: [{ required: true, message: '请选择盘点类型' }] },
                ]"
               placeholder="请选择盘点类型" style="width:100%" @change="handleTypeChange">
                <a-select-option value="MATERIAL">
                  物料盘点
                </a-select-option>
                <a-select-option value="LOCATION">
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
  import { getAction } from '@/api/manage'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: "InventoryFormDialog",
    components: {
    },
    props: {
      locationData: {
        type: Array,
        default: function() {
          return []
        }
      }
    },
    watch: {
      visible: {
        handler(value) {
          if (value) {
          }
        }
      }
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
        formModel: {},
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
        },
        materialData: []
      }
    },
    created () {
      this.getALLMaterial()
    },
    methods: {
      // 获取物料信息
      getALLMaterial() {
        getAction('/material/material/list', {
          data: {
            pageNo: 1,
            pageSize: 999999999
          }
        }).then(res => {
          if (res.code === 200) {
            this.materialData = res.result.records;
          }
        })
      },
      // 根据库位查询物料
      getPartMaterial(locationId) {
        getAction('/inventory/inventory/listMaterial', {
          data: {
            locationId,
            pageNo: 1,
            pageSize: 999999999
          }
        }).then(res => {
          if (res.code === 200) {
            this.materialData = res.result.records;
          }
        })
      },
      locationChange(value) {
        console.log(value, 'val')
        this.form.setFieldsValue({
          materialId: []
        })
        if (value && value.length > 0) {
          const locationId = value.join(',');
          this.getPartMaterial(locationId)
        } else {
          this.getALLMaterial()
        }
      },
      handleTypeChange(value) {

      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.formModel = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.formModel,'type', 'locationId','materialId','remarks'))
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
            // that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.formModel.id){
              httpurl = "/inventory/inventory/add"
              method = 'post'
            }else{
              httpurl = "inventory/inventory/edit"
              method = 'put';
            }
            let formData = Object.assign(this.formModel, values);
            if (formData.materialId) {
              formData.materialId = formData.materialId.join(',')
            }
            if (formData.locationId) {
              formData.locationId = formData.locationId.join(',')
            }
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
      }
    }
  }
</script>