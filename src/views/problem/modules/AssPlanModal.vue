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
          label="评定计划名称"
          hasFeedback >
          <a-input placeholder="请输入评定计划名称" v-decorator="['planname', {rules: [{required: true, message: '请输入评定计划名称！'}, {max: 20,message: '最大长度为20！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="起始时间"
          hasFeedback >
          <a-date-picker v-decorator="[ 'startdate', {rules: [{required: true, message: '请选择起始时间！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="结束时间"
          hasFeedback >
          <a-date-picker v-decorator="[ 'enddate', {rules: [{required: true, message: '请选择结束时间！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="评定组"
          hasFeedback >
         <!-- <a-input placeholder="请输入评定组" v-decorator="['objgroup', {}]" />-->
          <a-select placeholder="请选择评定组" v-decorator="['objgroup', {rules: [{required: true, message: '请选择评定名称！'}]}]" >
            <a-select-option :objgroups="objgroups" v-for="d in objgroups" :value="d">{{d}}</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="负责人"
          hasFeedback >
          <a-input placeholder="请输入负责人姓名" v-decorator="[ 'header', {rules: [{required: true, message: '请输入负责人名称！'}, {max: 10,message: '最大长度为10！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="评定项"
          hasFeedback >
         <!-- <a-input placeholder="请输入评定项" v-decorator="['rulename', {}]" />-->
          <a-select placeholder="请选择评定项" v-decorator="['rulename', {rules: [{required: true, message: '请选择评定项！'}]}]" >
            <a-select-option :asstypes="asstypes" v-for="d in asstypes" :value="d">{{d}}</a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction, getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "AssPlanModal",
    data () {
      return {
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
          add: "/ws/assplan/add",
          edit: "/ws/assplan/edit",
          getGroup: "/ws/assplan/getGroup",
        },
        objgroups:{},
        asstypes:{}
      }
    },
    created () {
      this.loadData();
    },
    methods: {
      loadData() {
        getAction(this.url.getGroup, null).then((res) => {
          if (res.success) {
            this.objgroups = res.result.objgroups;
            this.asstypes = res.result.asstypes;
          }
        })
      },
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'planname','startdate','enddate','objgroup','header','rulename'))
          //时间格式化
          this.form.setFieldsValue({startdate:this.model.startdate?moment(this.model.startdate):null})
          this.form.setFieldsValue({enddate:this.model.enddate?moment(this.model.enddate):null})
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
            //时间格式化
           /* formData.punchTime = formData.punchTime?formData.punchTime.format('YYYY-MM-DD HH:mm:ss'):null;*/
            formData.startdate = formData.startdate?formData.startdate.format():null;
            formData.enddate = formData.enddate?formData.enddate.format():null;
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