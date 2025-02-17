<template>
  <page-header-wrapper>
    <a-card :bordered="false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="48">
            <a-col :md="12" :lg="6" :sm="24">
              <a-form-item label="用户筛选">
                <a-select v-model="queryParam.accountType" placeholder="请选择用户类型" :default-value="{ key: '0' }">
                  <a-select-option :value="0">真实用户</a-select-option>
                  <a-select-option :value="1">模拟用户</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>

            <a-col :md="12" :lg="6" :sm="24">
              <a-form-item label="下级代理">
                <a-select v-model="queryParam.agentId" placeholder="请选择下级代理">
                  <a-select-option v-for="(item, index) in agentlist" :key="index" :value="item.id">{{ item.agentName }}
                  </a-select-option>
                </a-select>
              </a-form-item>
            </a-col>

            <a-col :md="12" :lg="6" :sm="24">
              <a-form-item label="真实姓名">
                <a-input v-model="queryParam.realName" style="width: 100%" placeholder="请填写真实姓名" />
              </a-form-item>
            </a-col>

            <a-col :md="12" :lg="6" :sm="24">
              <a-form-item label="用户手机">
                <a-input v-model="queryParam.phone" style="width: 100%" placeholder="请填写用户手机号" />
              </a-form-item>
            </a-col>

          </a-row>
          <a-row :gutter="48">
            <a-col :md="12" :lg="6" :sm="24">
              <a-form-item>
                <span class="table-page-search-submitButtons">
                  <a-button icon="redo" @click="() => this.queryParam = {}">重置</a-button>
                  <a-button type="primary" icon="search" style="margin-left: 8px"
                    @click="queryParam.pageNum = 1, pagination.current = 1,getuserList()">查询</a-button>
                  <a-button type="primary" icon="plus" style="margin-left: 8px"
                    @click="$refs.adduserdialog.addUserdialog = true">添加账户</a-button>
                </span>
              </a-form-item>
            </a-col>
          </a-row>
        </a-form>
      </div>
    </a-card>
    <a-card :bordered="false">
      <a-table bordered :loading="loading" :pagination="pagination" @change="handleTableChange" :columns="columns"
        :data-source="dataList" rowKey="phone">
        <span slot="agentName" slot-scope="text,record">
          <template>
            <div>
              <span>{{ record.agentName }}（{{ record.agentId }}）</span>
            </div>
          </template>
        </span>
        <span slot="isLock" slot-scope="text,record">
          <template>
            <div>
              <a-tag :color="record.isLock == 0 ? 'green' : 'red'">{{ record.isLock == 0 ? '可交易' : '不可交易' }}</a-tag>
            </div>
          </template>
        </span>
        <span slot="isLogin" slot-scope="text,record">
          <template>
            <div>
              <a-tag :color="record.isLogin == 0 ? 'green' : 'red'">{{ record.isLogin == 0 ? '可登陆' : '不可登陆' }}</a-tag>
            </div>
          </template>
        </span>
        <span slot="isOnline" slot-scope="text,record">
          <template>
            <div>
              <div :title="record.isOnline == null ? 'offline': 'online'" :color="record.isOnline == !null ? 'green' : 'red'">{{ record.isOnline == !null ? '🟢' : '🔴' }}</div>
            </div>
          </template>
        </span>
        <template slot="action" slot-scope="text,record">
          <a slot="action" href="javascript:;"
            @click="currentDetails = record; $refs.detailuserdialog.userDialog = true">用户详情</a>
          <a-divider type="vertical" />
          <a slot="action" href="javascript:;" @click="$refs.EditUserinfodialog.getEditorder(record)">编辑</a>
          <a-divider type="vertical" />
          <a slot="action" href="javascript:;" @click="$refs.editUserbankdialog.getbankinfo(record)">银行卡</a>
          <a-divider type="vertical" />
          <a slot="action" href="javascript:;" @click="$refs.editCapitaluserdialog.getEditorder(record)">资金</a>
          <a-divider type="vertical" />
          <a slot="action" href="javascript:;"
            @click="currentDetails = record; $refs.audituserdialog.userDialog = true">实名审核</a>
          <a-divider type="vertical" />
          <a slot="action" href="javascript:;" @click="getDeluser(record)">删除</a>
        </template>
      </a-table>
    </a-card>
    <adduserdialog ref="adduserdialog" :getinit='getinit' :agentlist="agentlist"></adduserdialog>
    <detailuserdialog ref="detailuserdialog" :currentDetails='currentDetails'></detailuserdialog>
    <EditUserinfodialog ref="EditUserinfodialog" :getinit='geteditinit' :agentlist="agentlist"></EditUserinfodialog>
    <editUserbankdialog ref="editUserbankdialog" :getinit='geteditinit'></editUserbankdialog>
    <editCapitaluserdialog ref="editCapitaluserdialog" :getinit='geteditinit'></editCapitaluserdialog>
    <audituserdialog ref="audituserdialog" :currentDetails='currentDetails' :getinit='geteditinit'></audituserdialog>
  </page-header-wrapper>
</template>
<script>
import { usermanag, nextagent, userdelete } from '@/api/home'
import adduserdialog from './components/adduserdialog'
import detailuserdialog from './components/detailuserdialog'
import EditUserinfodialog from './components/EditUserinfodialog'
import editUserbankdialog from './components/editUserbankdialog'
import editCapitaluserdialog from './components/editCapitaluserdialog'
import audituserdialog from './components/audituserdialog'
import moment from 'moment'
export default {
  name: 'agentlist',
  components: {
    adduserdialog,
    detailuserdialog,
    EditUserinfodialog,
    editUserbankdialog,
    editCapitaluserdialog,
    audituserdialog
  },
  data() {
    return {
      queryParam: {
        agentId: undefined,
        realName: '',
        phone: '',
        pageNum: 1,
        pageSize: 10,
        accountType: 0,
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 7 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 13 }
      },
      columns: [
        {
          title: '用户ID',
          dataIndex: 'id',
          align: "center",
        },
        {
          title: '所属代理（Id）',
          scopedSlots: { customRender: 'agentName' },
          align: "center",
        },
        {
          title: '手机号',
          dataIndex: 'phone',
          align: "center",
        },
        {
          title: '真实姓名',
          dataIndex: 'realName',
          align: "center",
        },
        {
          title: '总资金',
          dataIndex: 'userAmt',
          align: "center",
          customRender: (text, row, index) => {
            return text.toFixed(2)
          }
        },
        {
          title: '认证信息',
          dataIndex: 'isActive',
          align: "center",
          customRender: (text, row, index) => {
            return text == 0 ? '待认证' : text == 1 ? '待审核' : text == 2 ? '认证成功' : text == 3 ? '驳回' : ''
          }
        },
        {
          title: '交易状态',
          dataIndex: 'isLock',
          align: "center",
          scopedSlots: { customRender: 'isLock' },
        },
        {
          title: '登录状态',
          dataIndex: 'isLogin',
          align: "center",
          scopedSlots: { customRender: 'isLogin' },
        },
        {
          title: '在线状态',
          dataIndex: 'isOnline',
          align: "center",
          scopedSlots: { customRender: 'isOnline' },
        },
        {
          title: '注册时间',
          dataIndex: 'regTime',
          align: "center",
          customRender: (text, row, index) => {
            return moment(text).format('YYYY-MM-DD HH:mm:ss')
          }
        },
        {
          title: '操作',
          key: 'action',
          align: "center",
          scopedSlots: { customRender: 'action' },
        },
      ],//表头
      dataList: [],
      pagination: {
        total: 0,
        pageSize: 10,//每页中显示10条数据
        current: 1,
        showSizeChanger: true,
        pageSizeOptions: ["10", "20", "50", "100"],//每页中显示的数据
        onShowSizeChange: (current, pageSize) => this.onSizeChange(current, pageSize), // 改变每页数量时更新显示
        onChange: (page, pageSize) => this.onPageChange(page, pageSize),//点击页码事件
        showTotal: total => `共有 ${total} 条数据`,  //分页中显示总的数据
      },
      loading: false,
      agentqueryParam: {
        pageNum: 1,
        pageSize: 100,
      },
      agentlist: [],
      currentDetails: {}
    }
  },
  created() {
    this.getuserList()
    this.getagentlist()
  },
  methods: {
    getDeluser(val) {
      var that = this
      this.$confirm({
        title: '提示',
        content: '确认删除该用户吗？此操作不可恢复，将删除该用户所有的充值、提现与持仓记录！',
        onOk() {
          var data = {
            userId: val.id,
          }
          userdelete(data).then(res => {
            if (res.status == 0) {
              that.$message.success({ content: res.msg, duration: 2 });
              that.getinit()
            } else {
              that.$message.error({ content: res.msg });
            }
          })
        },
        onCancel() {
          console.log('Cancel');
        },
      });
    },
    getinit() {
      this.queryParam = {
        agentId: undefined,
        realName: '',
        phone: '',
        pageNum: 1,
        pageSize: 10,
        accountType: 0,
      }
      this.pagination.current = 1
      this.getuserList()
    },
    geteditinit() {
      this.getuserList()
    },
    getUserOnlineStatus() {

    },
    getuserList() {
      var that = this;
      this.loading = true
      usermanag(this.queryParam).then(res => {
        this.dataList = res.data.list
        this.pagination.total = res.data.total
        setTimeout(() => {
          that.loading = false
        }, 500);
      })
    },
    getagentlist() {
      nextagent(this.agentqueryParam).then(res => {
        this.agentlist = res.data.list
      })
    },
    onPageChange(page, pageSize) {
      this.queryParam.pageNum = page
      this.pagination.current = page
      this.getuserList()
    },
    onSizeChange(current, pageSize) {
      this.queryParam.pageNum = current
      this.pagination.current = page
      this.queryParam.pageSize = pageSize
      this.getuserList()
    },
    handleTableChange() {

    }
  }
}
</script>