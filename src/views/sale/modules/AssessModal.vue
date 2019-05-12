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
          label="评定主键"
          hidden=true
          hasFeedback>
          <a-input type="hidden" placeholder="" v-decorator="['id', {}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="评定人类别"
          hidden=true
          hasFeedback>
          <a-input type="hidden" placeholder="" v-decorator="['adjtype', {}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="评定人"
          hasFeedback>
          <a-input placeholder="请输入评定人姓名" read-only="read-only" v-decorator="['adjuster', {}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="评定对象"
          hasFeedback>
          <a-input placeholder="请输入评定人对象姓名" read-only="read-only" v-decorator="['assobject', {}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="评定时间"
          hasFeedback>
          <a-date-picker  :disabled=true v-decorator="[ 'assdate', {initialValue:moment()}]"/>
        </a-form-item>
        <a-form-item
          v-for="d in assinfos"
          :assinfos="assinfos"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          :label="d.scoregroup"
          hasFeedback>
          <a-input-number :min="0" :max="d.score" v-decorator="[`${d.id}`, {rules: [{required: true, message: '请输入分数！'}]}]"/>
          <!-- :min="0" :max="d.score"  :v-decorator="['d.scoregroup', {}]"-->
          <a-textarea v-model="d.scoreinfo" read-only="read-only" :autosize="{ minRows: 2, maxRows: 6 }"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="评语"
          hasFeedback>
          <a-input placeholder="请输入评语" v-decorator="['remark', {rules: [{required: true, message: '请输入评语！'}]}]"/>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction, getAction, postAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from 'moment'
  import { mapGetters } from 'vuex'

  export default {
    name: 'AssessModal',
    data() {
      return {
        title: '操作',
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 }
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {
          onAss: '/ws/assess/onAss',
          addAss: '/ws/assess/addAss'
        },

        assinfos: []
      }
    },
    created() {
    },
    methods: {
      ...mapGetters(['nickname', 'avatar']),
      moment,
      getData(record) {
        var params = {}
        var me = this;
        // params.username = this.nickname();
        params.planname = record.planname
        /* var username = {{ nickname() }};*/
        /* params.userName =record.planname;*/
        getAction(this.url.onAss, params).then((res) => {
          if (res.success) {
            this.assinfos = res.result
            this.visible = true
            me.model = Object.assign({}, record)
            me.$nextTick(() => {
              me.form.setFieldsValue(pick(this.model, 'adjuster', 'assobject','id',"adjtype"))
            })
          }
        })
      },
/*      add() {
        this.edit({})
      },
      edit(record) {
        this.form.resetFields()
        this.model = Object.assign({}, record)
        this.visible = true
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model, 'name', 'keyWord', 'sex', 'age', 'email', 'content'))
          //时间格式化
          this.form.setFieldsValue({ punchTime: this.model.punchTime ? moment(this.model.punchTime, 'YYYY-MM-DD HH:mm:ss') : null })
          this.form.setFieldsValue({ birthday: this.model.birthday ? moment(this.model.birthday) : null })
        })
      },*/
      close() {
        this.$emit('close')
        this.visible = false
      },
      handleOk() {
        const that = this
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true
            let httpurl = this.url.addAss
            let method = 'post'
            let formData = Object.assign(this.model, values)
            //时间格式化
            formData.assdate = formData.assdate ? formData.assdate.format() : null
            console.log(formData)
            //var params = JSON.stringify(formData)
            httpAction(httpurl, formData, method).then((res) => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
              } else {
                that.$message.warning(res.message)
              }
            }).finally(() => {
              that.confirmLoading = false
              that.close()
            })
          }
        })
      },
      handleCancel() {
        this.close()
      }
    }
  }
</script>

<style scoped>

</style>