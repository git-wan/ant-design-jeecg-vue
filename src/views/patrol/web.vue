<template>
  <a-card :bordered="false"><!--:bordered="false"-->
    <a-spin tip="Loading..." :spinning="spinning">
      <div class="spin-content">
    <div class="table-operator">
      <a-button @click="loadData" type="primary" icon="search">巡查</a-button>
    </div>

    <!-- table区域-begin -->
    <div>
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
        <template
          slot="status"
          slot-scope="status">
          <a-badge :status="status" :text="status | statusFilter"/>
        </template>
      <!--  <span slot="action" slot-scope="text, record">
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
        </span>-->

      </a-table>
    </div>
      </div>
    </a-spin>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <Error-modal ref="ErrorModal" @ok="modalFormOk"></Error-modal>


  </a-card>
</template>

<script>
  import ErrorModal from './modules/ErrorModal'
  import {filterObj} from '@/utils/util'
  import {deleteAction, getAction, postAction} from '@/api/manage'
  import moment from 'moment'
  /*  import {initDictOptions, filterDictText} from '@/components/dict/DictSelectUtil'*/
  export default {
    name: "AssInfo",
    components: {
      ErrorModal,
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
            title: '实物名称',
            align: "center",
            dataIndex: 'entityname'
          },
          {
            title: '属性值',
            align: "center",
            dataIndex: 'propertyvalue'
          },
          {
            title: '属性取值',
            align: "center",
            dataIndex: 'propertychar'
          },
          {
            title: '状态',
            align: "center",
            dataIndex: 'status',
            scopedSlots: { customRender: 'status' },
          },
         /* {
            title: '操作',
            dataIndex: 'action',
            align: "center",
            scopedSlots: {customRender: 'action'},
          }*/
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
        spinning:false,
        loading: false,
        selectedRowKeys: [],
        selectedRows: [],
        url: {
          list: "/ws/web/webStatus",
        },
      }
    },
    created() {
    /*  this.loadData();*/
      //初始化字典配置
      // this.initDictConfig();
    },
    methods: {
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        //var params = this.getQueryParams();//查询条件
        /*  var SALEDATE = params.SALEDATE.toString();
          alert(typeof  SALEDATE);*/
        this.spinning = true;
        getAction(this.url.list, null).then((res) => {
          // if (res.success) {
         // this.dataSource = res;
          // alert(JSON.stringify(res))
          // }
          if (res.success) {
            this.dataSource = res.result;
            var errorModel = this.$refs.ErrorModal;
            var datas = errorModel.dataSource;
            datas.length = 0 ;
            var jsons = res.result;
            for (var i  in jsons) {
              if(jsons[i].status=="error"){
                datas.push(jsons[i])
              }
            }
            this.spinning = false;
            if(datas.length>0){
              errorModel.visible=true
            }

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
        if(this.queryParam.SALEDATE==null){
          this.$message.warning('请选择时间！');
          return  false;
        }
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
        this.$refs.AssInfoModal.edit(record);
        this.$refs.AssInfoModal.title = "编辑";
      },
      onetomany: function (record) {
        this.$refs.jeecgDemoTabsModal.add();
        this.$refs.jeecgDemoTabsModal.title = "编辑";
      },
      handleAdd: function () {
        window.location.reload();
      },
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
    },
    filters: {
      statusFilter(status) {
        const statusMap = {
          'success': 'OK',
          'failed': 'ERROR'
        }
        return statusMap[status]
      }
    },
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
  .spin-content{
    border: 1px solid #91d5ff;
    background-color: #e6f7ff;
    padding: 30px;
  }
</style>