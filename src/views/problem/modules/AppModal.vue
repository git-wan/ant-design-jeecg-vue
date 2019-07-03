<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
      
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="应用维护标题"
          hasFeedback >
          <a-input placeholder="请输入应用维护标题" v-decorator="['maintainname', {rules: [{required: true, message: '请评应用维护标题！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="维护开始时间"
          hasFeedback >
          <a-date-picker format="YYYY-MM-DD HH:mm:ss" :showTime="{ defaultValue: moment('00:00:00', 'HH:mm:ss') }" v-decorator="[ 'sdate', {rules: [{required: true, message: '维护开始时间！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="维护结束时间"
          hasFeedback >
          <a-date-picker format="YYYY-MM-DD HH:mm:ss" :showTime="{ defaultValue: moment('00:00:00', 'HH:mm:ss') }" v-decorator="[ 'edate', {rules: [{required: true, message: '维护结束时间！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="维护人员"
          hasFeedback >
          <a-input placeholder="请输入维护人员" v-decorator="['maintainer', {rules: [{required: true, message: '请输入负责人名称！'}, {max: 50,message: '最大长度为50！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="维护内容"
          hasFeedback >
          <a-textarea placeholder="请输入维护内容" :autosize="{ minRows: 4, maxRows: 7 }" v-decorator="['maintainnote', {rules: [{required: true, message: '请输入维护内容！'}]}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="维护结果"
          hasFeedback >
          <a-select placeholder="请选择维护结果" v-decorator="['maintainresult', {rules: [{required: true, message: '请选择维护结果！'}]}]" >
            <a-select-option  value="维护成功">维护成功</a-select-option>
            <a-select-option  value="维护失败">维护失败</a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "AppModal",
    data () {
      return {
        moment,
        title:"操作",
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
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/ws/apprecord/add",
          edit: "/ws/apprecord/edit",
        },
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
          this.form.setFieldsValue(pick(this.model,'maintainname','assobject','sdate','edate','maintainer','maintainresult'))
          this.form.setFieldsValue({sdate:this.model.sdate?moment(this.model.sdate,'YYYY-MM-DD HH:mm:ss'):null})
          this.form.setFieldsValue({edate:this.model.edate?moment(this.model.edate,'YYYY-MM-DD HH:mm:ss'):null})
          //时间格式化
        });
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
            formData.sdate = formData.sdate?formData.sdate.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.edate = formData.edate?formData.edate.format('YYYY-MM-DD HH:mm:ss'):null;
            //时间格式化
            console.log(formData)
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


    }
  }
</script>

<style scoped>

</style>