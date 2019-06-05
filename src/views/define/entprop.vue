<template>
  <a-card :bordered="false"><!--:bordered="false"-->

    <!-- 查询区域 -->
<!--    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :span="6">
            <a-form-item label="时间">
              <a-date-picker :disabledDate="disabledDate"   v-model="queryParam.SALEDATE" />
            </a-form-item>
          </a-col>

          <a-col :span="6" >
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
          &lt;!&ndash;    <a-button type="primary" @click="superQuery" icon="filter" style="margin-left: 8px">高级查询</a-button>&ndash;&gt;
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>-->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
<!--      <a-button @click="handleAdd" type="primary" icon="plus">新增属性值</a-button>-->
     <!-- <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>-->

    </div>

    <!-- table区域-begin -->
    <div>
    <!--  <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
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
    <Propvalue-modal ref="PropvalueModal" @ok="modalFormOk"></Propvalue-modal>

    <!-- 一对多表单区域 -->
    <JeecgDemoTabsModal ref="jeecgDemoTabsModal" @ok="modalFormOk"></JeecgDemoTabsModal>

    <!-- 高级查询区域 -->
    <superQueryModal ref="superQueryModal" @ok="modalFormOk" @handleSuperQuery="handleSuperQuery"></superQueryModal>
  </a-card>
</template>

<script>
  import PropvalueModal from './modules/PropvalueModal'
  import SuperQueryModal from './modules/SuperQueryModal'
  import JeecgDemoTabsModal from './modules/JeecgDemoTabsModal'
  import {filterObj} from '@/utils/util'
  import {deleteAction, getAction, postAction} from '@/api/manage'
  import moment from 'moment'
/*  import {initDictOptions, filterDictText} from '@/components/dict/DictSelectUtil'*/
  export default {
    name: "entprop",
    components: {
      PropvalueModal,
      SuperQueryModal,
      JeecgDemoTabsModal,
    },
    data() {
      return {
        description: '用户管理页面',
        // 查询条件
        queryParam: {},
        //字典数组缓存
        sexDictOptions: [],
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key: 'rowIndex',
            width: 60,
            align: "center",
            customRender: function (t, r, index) {
              return parseInt(index) + 1;
            }
          },
          {
            title: '实物编号',
            align: "center",
            dataIndex: 'entityno'
          },
          {
            title: '实体姓名',
            align: "center",
            dataIndex: 'entityname'
          },
          {
            title: '属性编号',
            align: "center",
            dataIndex: 'propertyno'
          },
          {
            title: '属性名称',
            align: "center",
            dataIndex: 'propertyname'
          },
          {
            title: '属性操作',
            align: "center",
            dataIndex: 'valueflag'
          },
          {
            title: '属性取值类型',
            align: "center",
            dataIndex: 'valuetype'
          },
          {
            title: '属性值',
            align: "center",
            dataIndex: 'propertyvalue'
          },
          {
            title: '自动录入',
            align: "center",
            dataIndex: 'autoentry'
          },
          {
            title: '备注',
            align: "center",
            dataIndex: 'note'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
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
            return range[0] + "-" + range[1] + " 共" + total + "条"
          },
          showQuickJumper: true,
          showSizeChanger: true,
          total: 0
        },
   /*     isorter: {
          column: 'score',
          order: 'desc',
        },*/
        loading: false,
        selectedRowKeys: [],
        selectedRows: [],
        url: {
          list: "/ws/entprop/list",
          delete:"/ws/entprop/delete"
        },

      }
    },
    created() {
      this.loadData();
      //初始化字典配置
     // this.initDictConfig();
    },
    methods: {

      exportXls(){
        var SALEDATE = this.getQueryParams().SALEDATE.format('YYYY-MM-DD');
        /*let paramsStr = encodeURI(JSON.stringify());*/
        let url = `${window._CONFIG['domianURL']}/ws/salereport/exportXls?SALEDATE=${SALEDATE}`;
        window.location.href = url;
      },

      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        var params = this.getQueryParams();//查询条件
      /*  var SALEDATE = params.SALEDATE.toString();
        alert(typeof  SALEDATE);*/
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.records;
          }
        })
      },
      handleSuperQuery(arg) {//高级查询方法
        let params = {'superQueryParams':encodeURI(JSON.stringify(arg))};
        getAction(this.url.list, params).then((res) => {
          if (res.success) {
            this.dataSource = res.result.records;
            this.ipagination.total = res.result.total;
          }else{
            that.$message.warn(res.message);
          }
        })
      },
      /*initDictConfig() {
        //初始化字典 - 性别
        initDictOptions('sex').then((res) => {
          if (res.success) {
            this.sexDictOptions = res.result;
          }
        });
      },*/
      getQueryParams() {
        // if(this.queryParam.SALEDATE==null){
        //   this.$message.warning('请选择时间！');
        //   return  false;
        // }
        var param = Object.assign({}, this.queryParam, this.isorter);
        param.field = this.getQueryField();
        param.pageNo = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        return filterObj(param);
      },
      getQueryField() {
        //TODO 字段权限控制
        var str = "id,";
        this.columns.forEach(function (value, index) {
          str += "," + value.dataIndex;
        });
        return str;
      },
      onSelectChange(selectedRowKeys, selectionRows) {
        this.selectedRowKeys = selectedRowKeys;
        this.selectionRows = selectionRows;
      },
      onClearSelected() {
        this.selectedRowKeys = [];
        this.selectionRows = [];
      },
      searchQuery() {
        this.loadData(1);
      },
      superQuery() {
        this.$refs.superQueryModal.show();
      },
      searchReset() {
        var that = this;
        that.queryParam = {}
        that.loadData(1);
      },
      batchDel: function () {
        if (this.selectedRowKeys.length <= 0) {
          this.$message.warning('请选择一条记录！');
          return;
        } else {
          var ids = "";
          for (var a = 0; a < this.selectedRowKeys.length; a++) {
            ids += this.selectedRowKeys[a] + ",";
          }
          var that = this;
          this.$confirm({
            title: "确认删除",
            content: "是否删除选中数据?",
            onOk: function () {
              deleteAction(that.url.deleteBatch, {ids: ids}).then((res) => {
                if (res.success) {
                  that.$message.success(res.message);
                  that.loadData();
                  that.onClearSelected();
                } else {
                  that.$message.warning(res.message);
                }
              });
            }
          });
        }
      },
      handleDelete: function (id) {
        var that = this;
        deleteAction(that.url.delete, {id: id}).then((res) => {
          if (res.success) {
            that.$message.success(res.message);
            that.loadData();
          } else {
            that.$message.warning(res.message);
          }
        });
      },
      handleEdit: function (record) {
        this.$refs.PropvalueModal.edit(record);
        this.$refs.PropvalueModal.title = "编辑";
      },
      onetomany: function (record) {
        this.$refs.jeecgDemoTabsModal.add();
        this.$refs.jeecgDemoTabsModal.title = "编辑";
      },
  /*    handleAdd: function () {
        this.$refs.PropvalueModal.add();
        this.$refs.PropvalueModal.title = "新增";
      },*/
      handleTableChange(pagination, filters, sorter) {
        //分页、排序、筛选变化时触发
        console.log(sorter);
        //TODO 筛选
        if (Object.keys(sorter).length > 0) {
          this.isorter.column = sorter.field;
          this.isorter.order = "ascend" == sorter.order ? "asc" : "desc"
        }
        this.ipagination = pagination;
        this.loadData();
      },
      modalFormOk() {
        // 新增/修改 成功时，重载列表
        this.loadData();
      },

      disabledDate(current) {
        // Can not select days before today and today
        return current && current > moment().endOf('day');
      },
      //跳转单据页面
      jump() {
        this.$router.push({path: '/jeecg/helloworld'})
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