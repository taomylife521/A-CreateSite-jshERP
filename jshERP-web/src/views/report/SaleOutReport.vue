<!-- from 7 5 2 7 1 8 9 2 0 -->
<template>
  <a-row :gutter="24">
    <a-col :md="24">
      <a-card :style="cardStyle" :bordered="false">
        <!-- 查询区域 -->
        <div class="table-page-search-wrapper">
          <a-form layout="inline" @keyup.enter.native="searchQuery">
            <a-row :gutter="24">
              <a-col :md="6" :sm="24">
                <a-form-item label="商品信息" :labelCol="labelCol" :wrapperCol="wrapperCol">
                  <a-input placeholder="请输入条码、名称、助记码、规格、型号等信息" v-model="queryParam.materialParam"></a-input>
                </a-form-item>
              </a-col>
              <a-col :md="6" :sm="24">
                <a-form-item label="单据日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
                  <a-range-picker
                    style="width: 100%"
                    v-model="queryParam.createTimeRange"
                    format="YYYY-MM-DD"
                    :placeholder="['开始时间', '结束时间']"
                    @change="onDateChange"
                  />
                </a-form-item>
              </a-col>
              <a-col :md="6" :sm="24">
                <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                  <a-button type="primary" @click="searchQuery">查询</a-button>
                  <a-button style="margin-left: 8px" v-print="'#reportPrint'" icon="printer">打印</a-button>
                  <a-button style="margin-left: 8px" @click="exportExcel" icon="download">导出</a-button>
                  <a @click="handleToggleSearch" style="margin-left: 8px">
                    {{ toggleSearchStatus ? '收起' : '展开' }}
                    <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
                  </a>
                </span>
              </a-col>
              <a-col :md="6" :sm="24">
                <a-form-item>
                  <span>实际销售金额：{{realityPriceTotal}}</span>
                </a-form-item>
              </a-col>
            </a-row>
            <template v-if="toggleSearchStatus">
              <a-row :gutter="24">
                <a-col :md="6" :sm="24">
                  <a-form-item label="客户" :labelCol="labelCol" :wrapperCol="wrapperCol">
                    <a-select placeholder="请选择客户" v-model="queryParam.organId"
                              :dropdownMatchSelectWidth="false" showSearch allow-clear optionFilterProp="children">
                      <a-select-option v-for="(item,index) in cusList" :key="index" :value="item.id">
                        {{ item.supplier }}
                      </a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :md="6" :sm="24">
                  <a-form-item label="仓库" :labelCol="labelCol" :wrapperCol="wrapperCol">
                    <a-select
                      optionFilterProp="children"
                      :dropdownMatchSelectWidth="false"
                      showSearch allow-clear style="width: 100%"
                      placeholder="请选择仓库"
                      v-model="queryParam.depotId">
                      <a-select-option v-for="(depot,index) in depotList" :value="depot.id" :key="index">
                        {{ depot.depotName }}
                      </a-select-option>
                    </a-select>
                  </a-form-item>
                </a-col>
                <a-col :md="6" :sm="24" v-if="orgaTree.length">
                  <a-form-item label="机构" :labelCol="labelCol" :wrapperCol="wrapperCol">
                    <a-tree-select style="width:100%" allow-clear :treeData="orgaTree"
                                   v-model="queryParam.organizationId" placeholder="请选择机构">
                    </a-tree-select>
                  </a-form-item>
                </a-col>
                <a-col :md="6" :sm="24">
                  <a-form-item label="商品类别" :labelCol="labelCol" :wrapperCol="wrapperCol">
                    <a-tree-select style="width:100%" :dropdownStyle="{maxHeight:'200px',overflow:'auto'}" allow-clear
                                   :treeData="categoryTree" v-model="queryParam.categoryId" placeholder="请选择商品类别">
                    </a-tree-select>
                  </a-form-item>
                </a-col>
              </a-row>
            </template>
          </a-form>
        </div>
        <!-- table区域-begin -->
        <section ref="print" id="reportPrint">
          <a-table
            bordered
            ref="table"
            size="middle"
            rowKey="id"
            :columns="columns"
            :dataSource="dataSource"
            :components="handleDrag(columns)"
            :pagination="false"
            :scroll="scroll"
            :loading="loading"
            @change="handleTableChange">
            <span slot="customTitle">
              <a-popover trigger="click" placement="right">
                <template slot="content">
                  <a-checkbox-group @change="onColChange" v-model="settingDataIndex" :defaultValue="settingDataIndex">
                    <a-row style="width: 600px">
                      <template v-for="(item,index) in defColumns">
                        <template>
                          <a-col :span="6">
                            <a-checkbox :value="item.dataIndex" v-if="item.dataIndex==='rowIndex'" disabled></a-checkbox>
                            <a-checkbox :value="item.dataIndex" v-if="item.dataIndex!=='rowIndex'">
                              <j-ellipsis :value="item.title" :length="10"></j-ellipsis>
                            </a-checkbox>
                          </a-col>
                        </template>
                      </template>
                    </a-row>
                    <a-row style="padding-top: 10px;">
                      <a-col>
                        恢复默认列配置：<a-button @click="handleRestDefault" type="link" size="small">恢复默认</a-button>
                      </a-col>
                    </a-row>
                  </a-checkbox-group>
                </template>
                <a-icon type="setting" />
              </a-popover>
            </span>
          </a-table>
          <a-row :gutter="24" style="margin-top: 8px;text-align:right;">
            <a-col :md="24" :sm="24">
              <a-pagination @change="paginationChange" @showSizeChange="paginationShowSizeChange"
                size="small"
                show-size-changer
                :showQuickJumper="true"
                :current="ipagination.current"
                :page-size="ipagination.pageSize"
                :page-size-options="ipagination.pageSizeOptions"
                :total="ipagination.total"
                :show-total="(total, range) => `共 ${total-Math.ceil(total/ipagination.pageSize)} 条`">
                <template slot="buildOptionText" slot-scope="props">
                  <span>{{ props.value-1 }}条/页</span>
                </template>
              </a-pagination>
            </a-col>
          </a-row>
        </section>
        <!-- table区域-end -->
      </a-card>
    </a-col>
  </a-row>
</template>
<script>
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { getMpListShort, getPrevMonthFormatDate, getFormatDate } from '@/utils/util'
  import {getAction} from '@/api/manage'
  import {findBySelectCus, queryMaterialCategoryTreeList, getAllOrganizationTreeByUser} from '@/api/api'
  import JEllipsis from '@/components/jeecg/JEllipsis'
  import moment from 'moment'
  import Vue from 'vue'
  export default {
    name: "SaleOutReport",
    mixins:[JeecgListMixin],
    components: {
      JEllipsis
    },
    data () {
      return {
        labelCol: {
          span: 5
        },
        wrapperCol: {
          span: 18,
          offset: 1
        },
        queryParam: {
          materialParam:'',
          beginTime: getPrevMonthFormatDate(3),
          endTime: getFormatDate(),
          createTimeRange: [moment(getPrevMonthFormatDate(3)), moment(getFormatDate())],
          organId: undefined,
          depotId: undefined,
          organizationId: undefined,
          mpList: getMpListShort(Vue.ls.get('materialPropertyList')),
        },
        ipagination:{
          pageSize: 11,
          pageSizeOptions: ['11', '21', '31', '101', '201']
        },
        cusList: [],
        depotList: [],
        orgaTree: [],
        categoryTree:[],
        realityPriceTotal: '',
        tabKey: "1",
        pageName: 'saleOutReport',
        // 默认索引
        defDataIndex:['rowIndex','barCode','materialName','materialStandard','materialModel','materialUnit',
          'outSum','outSumPrice','inSum','inSumPrice','outInSumPrice'],
        // 默认列
        defColumns: [
          {
            dataIndex: 'rowIndex', width:60, align:"center", slots: { title: 'customTitle' },
            customRender:function (t,r,index) {
              return (t !== '合计') ? (parseInt(index) + 1) : t
            }
          },
          {title: '条码', dataIndex: 'barCode', sorter: (a, b) => a.barCode - b.barCode, width: 160},
          {title: '名称', dataIndex: 'materialName', width: 160, ellipsis:true},
          {title: '规格', dataIndex: 'materialStandard', width: 80, ellipsis:true},
          {title: '型号', dataIndex: 'materialModel', width: 80, ellipsis:true},
          {title: '颜色', dataIndex: 'materialColor', width: 60, ellipsis:true},
          {title: '品牌', dataIndex: 'materialBrand', width: 80, ellipsis:true},
          {title: '制造商', dataIndex: 'materialMfrs', width: 80, ellipsis:true},
          {title: '扩展1', dataIndex: 'otherField1', width: 80, ellipsis:true},
          {title: '扩展2', dataIndex: 'otherField2', width: 80, ellipsis:true},
          {title: '扩展3', dataIndex: 'otherField3', width: 80, ellipsis:true},
          {title: '单位', dataIndex: 'materialUnit', width: 80, ellipsis:true},
          {title: '销售数量', dataIndex: 'outSum', sorter: (a, b) => a.outSum - b.outSum, width: 80},
          {title: '销售金额', dataIndex: 'outSumPrice', sorter: (a, b) => a.outSumPrice - b.outSumPrice, width: 80},
          {title: '退货数量', dataIndex: 'inSum', sorter: (a, b) => a.inSum - b.inSum, width: 80},
          {title: '退货金额', dataIndex: 'inSumPrice', sorter: (a, b) => a.inSumPrice - b.inSumPrice, width: 80},
          {title: '实际销售金额', dataIndex: 'outInSumPrice', sorter: (a, b) => a.outInSumPrice - b.outInSumPrice, width: 100}
        ],
        url: {
          list: "/depotItem/saleOut"
        }
      }
    },
    created () {
      this.initCustomer()
      this.getDepotData()
      this.loadAllOrgaData()
      this.loadCategoryTreeData()
      this.initColumnsSetting()
      this.handleChangeOtherField(0)
    },
    methods: {
      moment,
      create(){
      },
      getQueryParams() {
        let param = Object.assign({}, this.queryParam, this.isorter);
        param.monthTime = this.queryParam.monthTime;
        param.field = this.getQueryField();
        param.currentPage = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize-1;
        return param;
      },
      onDateChange: function (value, dateString) {
        this.queryParam.beginTime=dateString[0]
        this.queryParam.endTime=dateString[1]
        if(dateString[0] && dateString[1]) {
          this.queryParam.createTimeRange = [moment(dateString[0]), moment(dateString[1])]
        }
      },
      loadData(arg) {
        //加载数据 若传入参数1则加载第一页的内容
        if (arg === 1) {
          this.ipagination.current = 1;
        }
        let params = this.getQueryParams();//查询条件
        this.loading = true;
        getAction(this.url.list, params).then((res) => {
          if (res.code===200) {
            this.dataSource = res.data.rows;
            this.ipagination.total = res.data.total;
            this.tableAddTotalRow(this.columns, this.dataSource)
            this.realityPriceTotal = res.data.realityPriceTotal
          } else if(res.code===510){
            this.$message.warning(res.data)
          } else {
            this.$message.warning(res.data.message)
          }
          this.loading = false;
        })
      },
      initCustomer() {
        let that = this;
        findBySelectCus({}).then((res)=>{
          if(res) {
            that.cusList = res
          }
        });
      },
      getDepotData() {
        getAction('/depot/findDepotByCurrentUser').then((res)=>{
          if(res.code === 200){
            this.depotList = res.data;
          }else{
            this.$message.info(res.data);
          }
        })
      },
      loadAllOrgaData(){
        let that = this
        let params = {}
        getAllOrganizationTreeByUser(params).then((res)=>{
          if(res){
            that.orgaTree = res
          }
        })
      },
      loadCategoryTreeData(){
        let that = this;
        let params = {};
        params.id='';
        queryMaterialCategoryTreeList(params).then((res)=>{
          if(res){
            that.categoryTree = [];
            for (let i = 0; i < res.length; i++) {
              let temp = res[i];
              that.categoryTree.push(temp);
            }
          }
        })
      },
      searchQuery() {
        if(this.queryParam.beginTime == '' || this.queryParam.endTime == ''){
          this.$message.warning('请选择单据日期！')
        } else {
          this.loadData(1);
        }
      },
      exportExcel() {
        let list = []
        let mpStr = getMpListShort(Vue.ls.get('materialPropertyList'))
        let head = '条码,名称,规格,型号,颜色,品牌,制造商,' + mpStr + ',单位,销售数量,销售金额,退货数量,退货金额,实际销售金额'
        for (let i = 0; i < this.dataSource.length; i++) {
          let item = []
          let ds = this.dataSource[i]
          item.push(ds.barCode, ds.materialName, ds.materialStandard, ds.materialModel, ds.materialColor, ds.materialBrand,
            ds.materialMfrs, ds.otherField1, ds.otherField2, ds.otherField3, ds.materialUnit, ds.outSum,
            ds.outSumPrice, ds.inSum, ds.inSumPrice, ds.outInSumPrice)
          list.push(item)
        }
        let tip = '单据日期：' + this.queryParam.beginTime + '~' + this.queryParam.endTime
        this.handleExportXlsPost('销售统计', '销售统计', head, tip, list)
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>