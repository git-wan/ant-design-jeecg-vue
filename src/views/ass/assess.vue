<template>
  <a-card :bordered="false"><!--:bordered="false"-->

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :span="8">
            <a-form-item label="评定计划名称">
              <a-select v-model="queryParam.planname" placeholder="请输入评定计划名称">
               <!-- <a-select-option value=""> 请选择</a-select-option>-->
                <a-select-option :plannames="plannames" v-for="d in plannames" :value="d.planname">{{d.planname}}</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <a-col :span="6">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <!-- <a-button type="primary" @click="superQuery" icon="filter" style="margin-left: 8px">高级查询</a-button>-->
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->

    <!-- table区域-begin -->
    <div>
      <!--<div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>-->

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"

        @change="handleTableChange">
        <!-- :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"-->
        <span slot="action" slot-scope="text, record">
          <a @click="handleAss(record)">评定</a>
         <!-- <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>-->
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <Assess-modal ref="AssessModal" @ok="modalFormOk"></Assess-modal>

    <!-- 一对多表单区域 -->
    <JeecgDemoTabsModal ref="jeecgDemoTabsModal" @ok="modalFormOk"></JeecgDemoTabsModal>

  </a-card>
</template>

<script>
  import AssessModal from './modules/AssessModal'
  import SuperQueryModal from './modules/SuperQueryModal'
  import JeecgDemoTabsModal from './modules/JeecgDemoTabsModal'
  import { filterObj } from '@/utils/util'
  import { deleteAction, getAction, postAction } from '@/api/manage'
 /* import { initDictOptions, filterDictText } from '@/components/dict/DictSelectUtil'*/
  import { mapGetters } from 'vuex'
  export default {
    name: 'Assess',
    components: {
      AssessModal,
      SuperQueryModal,
      JeecgDemoTabsModal
    },
    data() {
      return {
        description: '评定',
        // 查询条件
        queryParam: {
          status: '未评定'
        },
        plannames:{},
        //字典数组缓存
        sexDictOptions: [],
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: 'center',
            customRender: function(t, r, index) {
              return parseInt(index) + 1
            }
          },
          {
            title: '评定计划名称',
            align: 'center',
            dataIndex: 'planname'
          },
          {
            title: '评定人类别',
            align: 'center',
            dataIndex: 'adjtype'
          },
          {
            title: '评定人',
            align: 'center',
            dataIndex: 'adjuster'
          },
          {
            title: '评定对象',
            align: 'center',
            dataIndex: 'assobject'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: 'center',
            scopedSlots: { customRender: 'action' }
          }
        ],
        //数据集
        dataSource: [],
        // 分页参数
        ipagination: {
          current: 1,
          pageSize: 10,
          pageSizeOptions: ['10', '20', '30'],
          showTotal: (total, range) => {
            return range[0] + '-' + range[1] + ' 共' + total + '条'
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
        isorter: {
          column: 'assobject',
          order: 'desc'
        },
        loading: false,
        selectedRowKeys: [],
        selectedRows: [],
        url: {
          list: '/ws/assess/list',
          getStr: '/ws/assess/getplanname'
        }

      }
    },
    created() {
      this.loadData()
    },
    methods: {
      ...mapGetters(['nickname', 'avatar']),
      loadData() {
        getAction(this.url.getStr, null).then((res) => {
          this.plannames = res.result
        })
      },
      searchQuery() {
        this.ipagination.current = 1
        var params = this.getQueryParams()//查询条件
       /* alert(JSON.stringify(params));*/
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            var  jsons = res.result.records;
            for(var json in jsons){
             if(json.status == "已评定"){
               jsons.remove(json);
             }
            }
            this.dataSource = jsons;
            this.ipagination.total = res.result.total
          }
        })
      },
      getQueryParams() {
        var param = Object.assign({}, this.queryParam, this.isorter)
        param.field = this.getQueryField()
        param.pageNo = this.ipagination.current
        param.pageSize = this.ipagination.pageSize
        return filterObj(param)
      },
      getQueryField() {
        //TODO 字段权限控制
        var str = 'id,'
        this.columns.forEach(function(value, index) {
          str += ',' + value.dataIndex
        })
        return str
      },
      onSelectChange(selectedRowKeys, selectionRows) {
        this.selectedRowKeys = selectedRowKeys
        this.selectionRows = selectionRows
      },
      onClearSelected() {
        this.selectedRowKeys = []
        this.selectionRows = []
      },

      superQuery() {
        this.$refs.superQueryModal.show()
      },
      searchReset() {
        var that = this
        that.queryParam = {}
        that.loadData(1)
      },
      batchDel: function() {
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条记录！')
          return
        } else {
          var ids = ''
          for (var a = 0; a < this.selectedRowKeys.length; a++) {
            ids += this.selectedRowKeys[a] + ','
          }
          var that = this
          this.$confirm({
            title: '确认删除',
            content: '是否删除选中数据?',
            onOk: function() {
              deleteAction(that.url.deleteBatch, { ids: ids }).then((res) => {
                if (res.success) {
                  that.$message.success(res.message)
                  that.loadData()
                  that.onClearSelected()
                } else {
                  that.$message.warning(res.message)
                }
              })
            }
          })
        }
      },
      handleDelete: function(id) {
        var that = this
        deleteAction(that.url.delete, { id: id }).then((res) => {
          if (res.success) {
            that.$message.success(res.message)
            that.loadData()
          } else {
            that.$message.warning(res.message)
          }
        })
      },
      onetomany: function(record) {
        this.$refs.jeecgDemoTabsModal.add()
        this.$refs.jeecgDemoTabsModal.title = '编辑'
      },
      handleAss: function(record) {
        var username = this.nickname();
        var adjuster = record.adjuster;
        if(username==adjuster){
          this.$refs.AssessModal.getData(record)
          this.$refs.AssessModal.title = '评定'
        }else {
          this.$message.warning("你不是此项评定人！")
        }

      },
      handleTableChange(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        console.log(sorter)
        //TODO 筛选
        if (Object.keys(sorter).length > 0) {
          this.isorter.column = sorter.field
          this.isorter.order = 'ascend' == sorter.order ? 'asc' : 'desc'
        }
        this.ipagination = pagination
        this.loadData()
      },
      modalFormOk() {
        // 新增/修改 成功时，重载列表
        this.loadData()
      },
      //跳转单据页面
      jump() {
        this.$router.push({ path: '/jeecg/helloworld' })
      }
    }
  }
</script>
<style scoped>
  .ant-card-body .table-operator {
    margin-bottom: 18px;
  }

  .ant-layout-content {
    margin: 12px 16px 0 !important;
  }

  .ant-table-tbody .ant-table-row td {
    padding-top: 15px;
    padding-bottom: 15px;
  }

  .anty-row-operator button {
    margin: 0 5px
  }

  .ant-btn-danger {
    background-color: #ffffff
  }

  .ant-modal-cust-warp {
    height: 100%
  }

  .ant-modal-cust-warp .ant-modal-body {
    height: calc(100% - 110px) !important;
    overflow-y: auto
  }

  .ant-modal-cust-warp .ant-modal-content {
    height: 90% !important;
    overflow-y: hidden
  }

  /** Button按钮间距 */
  .ant-btn {
    margin-left: 3px
  }
</style>