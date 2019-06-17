<template>
  <a-modal
    :title="title"
    :width="600"
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
          label="值班人"
          hasFeedback>
          <a-input placeholder="请输入评定人对象姓名" read-only="read-only" v-decorator="['watcher', {}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="值班日期"
          hasFeedback>
          <a-date-picker  :disabled=true v-decorator="[ 'rdate', {initialValue:moment()}]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="值班类型"
          hasFeedback>
          <a-select v-decorator="['dutytype']">
            <a-select-option value="早班">早班</a-select-option>
            <a-select-option value="晚班">晚班</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="网络室空调温度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['netc' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="网络室空调湿度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['netm' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="服务器室空调1温度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['serverc1' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="服务器室空调1湿度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['serverm1' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="服务器室空调2温度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['serverc2' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="服务器室空调2湿度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['serverm2' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="服务器室空调3温度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['serverc3' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="服务器室空调3湿度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['serverm3' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="电源室温度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['powerc' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="电源室湿度"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['powerm' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="市电1电压"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['cityv1' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="市电1频率"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['cityh1' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="市电2电压"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['cityv2' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="市电2频率"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['cityh2' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups1电压"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['upsv1' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups1频率"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['upsh1' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups1负载率"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['upsl1' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups1指示灯">
          <a-switch v-model="upshint1">
            <a-icon type="check" slot="checkedChildren"/>
            <a-icon type="close" slot="unCheckedChildren"/>
          </a-switch>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups2电压"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['upsv2' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups2频率"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['upsh2' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups2负载率"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['upsl2' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups2指示灯"
          hasFeedback>
          <a-switch v-model="upshint2">
            <a-icon type="check" slot="checkedChildren"/>
            <a-icon type="close" slot="unCheckedChildren"/>
          </a-switch>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups3电压"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['upsv3' ]"/>
        </a-form-item>
        <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="ups3频率"
        hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['upsh3' ]"/>
      </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups3负载率"
          hasFeedback>
          <a-input-number :min="1" :max="100" placeholder="" v-decorator="['upsl3' ]"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="ups3指示灯"
          hasFeedback>
          <a-switch v-model="upshint3">
            <a-icon type="check" slot="checkedChildren"/>
            <a-icon type="close" slot="unCheckedChildren"/>
          </a-switch>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="异常日志"
          hasFeedback >
          <a-textarea placeholder="请输入异常日志" v-decorator="['errorlog']" :autosize="{ minRows: 4, maxRows: 8 }"/>
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from 'moment'
  import ACol from 'ant-design-vue/es/grid/Col'
  import { mapGetters } from 'vuex'
  export default {
    name: 'DutyModal',
    components: { ACol },
    data: function() {
      return {
        upshint1: true,
        upshint2: true,
        upshint3: true,
        title: '操作',
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 6 }
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 }
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules: {},
        url: {
          add: '/ws/duty/add',
          edit: '/ws/duty/edit'
        }
      }
    },
    created() {
    },
    methods: {
      ...mapGetters(['nickname', 'avatar']),
      moment,
      add() {
        this.edit({})
      },
      edit(record) {
        this.form.resetFields()
        this.model = Object.assign({}, record)
        if(record.length>0){
          this.upshint1 = record.upshint1;
          this.upshint2 = record.upshint2;
          this.upshint3 = record.upshint3;
        }
        this.model.watcher = this.nickname();
        this.model.rdate = moment();
        this.visible = true
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'watcher','rdate','netc','netm','serverc1','serverm1','serverc2','serverm2','serverc3','serverm3','powerc','powerm','cityv1','cityh1','cityv2','cityh2','upsv1','upsh1','upsl1','upsv2','upsh2','upsl2','upsv3','upsh3','upsl3','errorlog'))
          this.form.setFieldsValue({rdate: this.model.rdate ? moment(this.model.rdate, 'YYYY-MM-DD HH:mm:ss') : null})
        })
      },
      close() {
        this.$emit('close')
        this.visible = false
      },
      handleOk() {
        const that = this
       /* alert(this.d)
        return false*/
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true
            let httpurl = ''
            let method = ''
            if (!this.model.id) {
              httpurl += this.url.add
              method = 'post'
            } else {
              httpurl += this.url.edit
              method = 'put'
            }
            this.model.upshint1 = this.upshint1;
            this.model.upshint2 = this.upshint2;
            this.model.upshint3 = this.upshint3;
            let formData = Object.assign(this.model, values)
            formData.rdate = formData.rdate ? formData.rdate.format('YYYY-MM-DD HH:mm:ss') : null;
            console.log(formData)
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