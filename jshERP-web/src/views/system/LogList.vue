<template>
  <a-row :gutter="24">
    <a-col :md="24">
      <a-card :style="cardStyle" :bordered="false">
        <!-- 查询区域 -->
        <div class="table-page-search-wrapper">
          <a-form layout="inline" @keyup.enter.native="searchQuery">
            <a-row :gutter="24">
              <a-col :md="6" :sm="24">
                <a-form-item label="操作模块" :labelCol="labelCol" :wrapperCol="wrapperCol">
                  <a-input placeholder="请输入操作模块" v-model="queryParam.operation"></a-input>
                </a-form-item>
              </a-col>
              <a-col :md="6" :sm="24">
                <a-form-item label="操作详情" :labelCol="labelCol" :wrapperCol="wrapperCol">
                  <a-input placeholder="请输入操作详情" v-model="queryParam.content"></a-input>
                </a-form-item>
              </a-col>
              <a-col :md="6" :sm="24">
                <a-form-item label="创建时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
                  <a-range-picker
                    style="width: 100%"
                    v-model="queryParam.createTimeRange"
                    format="YYYY-MM-DD"
                    :placeholder="['开始时间', '结束时间']"
                    @change="onDateChange"
                    @ok="onDateOk"
                  />
                </a-form-item>
              </a-col>
              <a-col :md="6" :sm="24" >
                <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                  <a-button type="primary" @click="searchQuery">查询</a-button>
                  <a-button style="margin-left: 8px" @click="searchReset">重置</a-button>
                  <a @click="handleToggleSearch" style="margin-left: 8px">
                    {{ toggleSearchStatus ? '收起' : '展开' }}
                    <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
                  </a>
                </span>
              </a-col>
            </a-row>
            <template v-if="toggleSearchStatus">
              <a-row :gutter="24">
                <a-col :md="6" :sm="24">
                  <a-form-item label="操作员" :labelCol="labelCol" :wrapperCol="wrapperCol">
                    <a-input placeholder="请输入操作员账号或姓名" v-model="queryParam.userInfo"></a-input>
                  </a-form-item>
                </a-col>
                <a-col :md="6" :sm="24">
                  <a-form-item label="操作IP" :labelCol="labelCol" :wrapperCol="wrapperCol">
                    <a-input placeholder="请输入操作IP" v-model="queryParam.clientIp"></a-input>
                  </a-form-item>
                </a-col>
                <a-col :md="6" :sm="24" v-if="isManage">
                  <a-form-item label="租户账号" :labelCol="labelCol" :wrapperCol="wrapperCol">
                    <a-input placeholder="请输入租户账号" v-model="queryParam.tenantLoginName"></a-input>
                  </a-form-item>
                </a-col>
                <a-col :md="6" :sm="24" v-if="isManage">
                  <a-form-item label="租户类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
                    <a-select v-model="queryParam.tenantType" placeholder="请选择租户类型">
                      <a-select-option value="">请选择</a-select-option>
                      <a-select-option value="0">试用租户</a-select-option>
                      <a-select-option value="1">付费租户</a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
              </a-row>
            </template>
          </a-form>
        </div>
        <!-- table区域-begin -->
        <a-table
          ref="table"
          bordered
          size="middle"
          rowKey="id"
          :columns="columns"
          :dataSource="dataSource"
          :pagination="ipagination"
          :scroll="scroll"
          :loading="loading"
          @change="handleTableChange">
          <!-- 字符串超长截取省略号显示-->
          <span slot="content" slot-scope="text, record">
              <j-ellipsis :value="text" :length="40"/>
            </span>
        </a-table>
        <!-- table区域-end -->
      </a-card>
    </a-col>
  </a-row>
</template>
<!-- f r o m 7 5  2 7 1  8 9 2 0 -->
<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JEllipsis from '@/components/jeecg/JEllipsis'
  import { getFormatDate, getPrevMonthFormatDate } from '@/utils/util'
  import {getAction } from '@/api/manage'
  import moment from 'moment'

  export default {
    name: "LogList",
    mixins:[JeecgListMixin],
    components: {
      JEllipsis
    },
    data () {
      return {
        // 查询条件
        queryParam: {
          operation:'',
          content:'',
          userInfo: '',
          clientIp:'',
          tenantLoginName:'',
          tenantType:'',
          beginTime: getPrevMonthFormatDate(1),
          endTime: getFormatDate(),
          createTimeRange: [moment(getPrevMonthFormatDate(1)), moment(getFormatDate())],
        },
        tabKey: "1",
        isManage: false,
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:40,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {title: '操作模块', dataIndex: 'operation', width: 120, align: "left"},
          {title: '操作详情', dataIndex: 'content', scopedSlots: { customRender: 'content' }, width: 360, align:"left" },
          {title: '操作员账号', dataIndex: 'loginName', width: 80, align: "left"},
          {title: '操作员姓名', dataIndex: 'userName', width: 80, align: "left"},
          {title: '操作IP', dataIndex: 'clientIp', width: 100, align: "left"},
          {title: '操作时间', dataIndex: 'createTimeStr', width: 110, align: "left"}
        ],
        operateColumn:
        {
          title: '操作类型',
          dataIndex: 'operateType_dictText',
          align:"center",
        },
        labelCol: {
          span: 5
        },
        wrapperCol: {
          span: 18,
          offset: 1
        },
        url: {
          list: "/log/list",
        }
      }
    },
    created() {
      this.initUserInfo()
    },
    methods: {
      onDateChange: function (value, dateString) {
        this.queryParam.beginTime=dateString[0]
        this.queryParam.endTime=dateString[1]
        if(dateString[0] && dateString[1]) {
          this.queryParam.createTimeRange = [moment(dateString[0]), moment(dateString[1])]
        }
      },
      onDateOk(value) {
        console.log(value);
      },
      searchReset() {
        this.queryParam = {
          operation:'',
          content:'',
          userInfo: '',
          clientIp:'',
          tenantLoginName:'',
          tenantType:'',
          beginTime: getPrevMonthFormatDate(1),
          endTime: getFormatDate(),
          createTimeRange: [moment(getPrevMonthFormatDate(1)), moment(getFormatDate())],
        }
        this.loadData(1);
      },
      initUserInfo() {
        getAction('/user/getUserSession').then((res)=>{
          if(res.code === 200){
            let user = res.data.user
            if(user.tenantId === 0) {
              this.isManage = true
            }
          }else{
            this.$message.warning(res.data)
          }
        })
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>