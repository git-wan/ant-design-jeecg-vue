<template>
  <a-card :bordered="false"><!--:bordered="false"-->

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :sm="6" :md="12">
            <a-form-item label="评定人">
              <a-input placeholder="请输入名称查询" v-model="queryParam.adjuster"></a-input>
            </a-form-item>
          </a-col>
          <a-col :sm="6" :md="12">
            <a-form-item label="评定对象">
              <a-input placeholder="请输入名称查询" v-model="queryParam.assobject"></a-input>
            </a-form-item>
          </a-col>
          <a-col :sm="6" :md="12">
            <a-form-item label="评定组">
              <!--<DictSelectTag v-model="queryParam.assgroup" placeholder="请输入用户性别" dictCode="sex"/>-->
              <a-select v-model="queryParam.assgroup" placeholder="请输入评定组">
                <a-select-option :assgroups="assgroups" v-for="(value,index) in assgroups" :value="value">{{value}}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <a-col :span="6">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <!--<a-button type="primary" @click="superQuery" icon="filter" style="margin-left: 8px">高级查询</a-button>-->
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <!--  <a-button type="primary" icon="plus" @click="jump">创建单据</a-button>
        <a-button type="primary" icon="plus" @click="onetomany">一对多</a-button>-->

      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{
        selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange">

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <AssObj-modal ref="AssObjModal" @ok="modalFormOk"></AssObj-modal>

    <!-- 一对多表单区域 -->
    <JeecgDemoTabsModal ref="jeecgDemoTabsModal" @ok="modalFormOk"></JeecgDemoTabsModal>

    <!-- 高级查询区域 -->
    <superQueryModal ref="superQueryModal" @ok="modalFormOk" @handleSuperQuery="handleSuperQuery"></superQueryModal>
  </a-card>
</template>

<script>
  import AssObjModal from './modules/AssObjModal'
  import SuperQueryModal from './modules/SuperQueryModal'
  import JeecgDemoTabsModal from './modules/JeecgDemoTabsModal'
  import { filterObj } from '@/utils/util'
  import { deleteAction, getAction, postAction } from '@/api/manage'
 /* import { initDictOptions, filterDictText } from '@/components/dict/DictSelectUtil'*/

  export default {
    name: 'AssObj',
    components: {
      AssObjModal,
      SuperQueryModal,
      JeecgDemoTabsModal
    },
    data() {
      return {
        description: '评定组页面',
        // 查询条件
        queryParam: {},
        //字典数组缓存
        sexDictOptions: [],
        // 表头
        assgroups: {},
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
            title: '评定人',
            align: 'center',
            dataIndex: 'adjuster'
            //hidden:true
          },
          {
            title: '评定对象',
            align: 'center',
            dataIndex: 'assobject'
          },
          {
            title: '评定人类别',
            align: 'center',
            dataIndex: 'adjtype'
          },
          {
            title: '评定组',
            align: 'center',
            dataIndex: 'assgroup'
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
          column: 'assgroup',
          order: 'desc'
        },
        loading: false,
        selectedRowKeys: [],
        selectedRows: [],
        url: {
          list: '/ws/assobj/list',
          delete: '/ws/assobj/delete',
          deleteBatch: '/ws/assobj/deleteBatch'
        }

      }
    },
    created() {
      this.loadData()
      //初始化字典配置
     // this.initDictConfig()
    },
    methods: {
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1
        }
        var params = this.getQueryParams()//查询条件
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.records
            this.ipagination.total = res.result.total//assgroups
            //json数组
            var jsons = res.result.records;
            //alert(jsons)
            //jsons=JSON.parse(jsons);
         //   alert(typeof jsons)
            var set = new Set();
            for (var i in jsons){
              set.add(jsons[i].assgroup);
            }
            this.assgroups= set;
          }
        })
      },
      searchQuery() {
        this.loadData(1)
      },
      handleSuperQuery(arg) {//高级查询方法
        let params = { 'superQueryParams': encodeURI(JSON.stringify(arg)) }
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.records
            this.ipagination.total = res.result.total
          } else {
            that.$message.warn(res.message)
          }
        })
      },
  /*    initDictConfig() {
        //初始化字典 - 性别
        initDictOptions('sex').then((res) => {
          if (res.success) {
            this.sexDictOptions = res.result
          }
        })
      },*/
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
      handleEdit: function(record) {
        this.$refs.AssObjModal.edit(record)
        this.$refs.AssObjModal.title = '编辑'
      },
      onetomany: function(record) {
        this.$refs.jeecgDemoTabsModal.add()
        this.$refs.jeecgDemoTabsModal.title = '编辑'
      },
      handleAdd: function() {
        this.$refs.AssObjModal.add()
        this.$refs.AssObjModal.title = '新增'
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