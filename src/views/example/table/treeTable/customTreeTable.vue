<template>
<div class="app-container calendar-list-container">
    <div class="filter-container">
    <template>
        <span>{{$t('datePick.title')}}:</span>
        <el-date-picker
          v-model="QueryDate"
          type="daterange"
          unlink-panels
          range-separator="-"
          :start-placeholder="$t('datePick.startPlaceholder')"
          :end-placeholder="$t('datePick.endPlaceholder')"
          value-format="yyyy-MM-dd"
          :picker-options="pickerOptions2">
        </el-date-picker>
    </template>
    <!-- <template>
      <span>{{$t('select.title')}}:</span>
      <el-select v-model="selectedMembers" multiple filterable :placeholder="$t('select.placeholder')">
        <el-option
          v-for="item in members"
          :key="item.name"
          :label="item.chinese"
          :value="item.name">
        </el-option>
      </el-select>
    </template> -->
    <template>
      <el-button class="filter-item" type="primary" v-waves icon="el-icon-search" @click="handleFilter">{{$t('table.search')}}</el-button>
      <el-button class="filter-item" type="primary" :loading="downloadLoading" v-waves icon="el-icon-download" @click="handleDownload">{{$t('table.export')}}</el-button>
    </template>
    </div>

    <el-table :key='tableKey' :data="list" v-loading="listLoading" :element-loading-text="$t('table.loading')" show-summary stripe fit highlight-current-row
      style="width: 100%" :default-sort = "{prop: 'bugCount', order: 'descending'}">
      <el-table-column align="center" :label="$t('table.id')" width="100" type="index">
      </el-table-column>
      <el-table-column prop="name" width="120px" align="center" :label="$t('table.tester')" :formatter="nameFormat"
      :filters="membersFilterOptions" :filter-method="filterHandler">
      </el-table-column>
      <el-table-column prop="findBugCount" width="100px" align="center" :label="$t('table.bugCount')" sortable >
      </el-table-column>
      <el-table-column prop="featureBugCount" width="100px" align="center" :label="$t('table.featureBugCount')" sortable >
      </el-table-column>
      <el-table-column prop="devBugCount" width="100px" align="center" :label="$t('table.devBugCount')" sortable >
      </el-table-column>
      <el-table-column prop="testBugCount" width="120px" align="center" :label="$t('table.testBugCount')" sortable >
      </el-table-column>
      <el-table-column prop="otherBugCount" width="120px" align="center" :label="$t('table.otherBugCount')" sortable >
      </el-table-column>
      <el-table-column align="center" :label="$t('table.detail')" min-width="230" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="small" plain @click="handleBugDetail(scope.row)">{{$t('table.bugCount')}}</el-button>
          <el-button type="primary" size="small" plain @click="handleFeatureDetail(scope.row)">{{$t('table.featureBugCount')}}</el-button>
          <el-button type="primary" size="small" plain @click="handleDevDetail(scope.row)">{{$t('table.devBugCount')}}</el-button>
          <el-button type="primary" size="small" plain @click="handleTestDetail(scope.row)">{{$t('table.testBugCount')}}</el-button>
          <el-button type="primary" size="small" plain @click="handleotherDetail(scope.row)">{{$t('table.otherBugCount')}}</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog :title="$t(dialogPvTitle)" align="center" :visible.sync="dialogPvVisible" width="350px">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column align="center" :label="$t('table.jiraNo')">
            <template slot-scope="scope">
               <span @click="handleJiraDetail(scope.row)"><a href="#">{{scope.row}}</a></span>
            </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">{{$t('table.confirm')}}</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
/**
  Auth: Lei.j1ang
  Created: 2018/1/19-14:54
*/
import treeTable from '@/components/TreeTable'
import { fetchTestKpi } from '@/api/article'

const calendarTypeOptions = [
  { key: 'CN', display_name: 'China' },
  { key: 'US', display_name: 'USA' },
  { key: 'JP', display_name: 'Japan' },
  { key: 'EU', display_name: 'Eurozone' }
]
const memberOptions = [
  { name: 'zhangling', chinese: '张玲' },
  { name: 'wufei', chinese: '吴菲' },
  { name: 'liufanghua', chinese: '刘芳华' },
  { name: 'wujinpeng', chinese: '吴金鹏' },
  { name: 'zhouxinhui', chinese: '周昕惠' },
  { name: 'hexueqin', chinese: '何雪琴' },
  { name: 'qiupeiteng', chinese: '仇培腾' },
  { name: 'chengjie', chinese: '程杰' }
]
// arr to obj ,such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'customTreeTableDemo',
  components: { treeTable },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: null,
      listLoading: false,
      QueryDate: [],
      listQuery: {
        startTime: '',
        endTime: ''
      },
      selectedMembers: [],
      members: memberOptions,
      membersFilterOptions: [],
      showReviewer: false,
      dialogPvVisible: false,
      dialogPvTitle: 'table.bugCount',
      pvData: [],
      downloadLoading: false,
      pickerOptions2: {
        shortcuts: [{
          text: '最近一周',
          onClick(picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '最近一个月',
          onClick(picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
            picker.$emit('pick', [start, end])
          }
        }, {
          text: '最近三个月',
          onClick(picker) {
            const end = new Date()
            const start = new Date()
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
            picker.$emit('pick', [start, end])
          }
        }]
      }
    }
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type]
    }
  },
  created() {
    for (const i in memberOptions) {
      this.membersFilterOptions.push({ text: memberOptions[i].chinese, value: memberOptions[i].name })
    }
  },
  methods: {
    getList() {
      this.listLoading = true
      fetchTestKpi(this.listQuery).then(response => {
        this.list = response.data.data
        this.total = response.data.total
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.startTime = this.QueryDate[0]
      this.listQuery.endTime = this.QueryDate[1]
      this.getList()
    },
    nameFormat(row, col) {
      for (const i in memberOptions) {
        if (memberOptions[i].name === row.name) {
          return memberOptions[i].chinese
        }
      }
      return row.name
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['姓名', 'Bug数', 'featureBug数', 'devBug数', 'testBug数', 'onlineBug数', 'BUG明细', 'feature明细', 'dev明细', 'test明细', '其他明细']
        const filterVal = ['name', 'findBugCount', 'featureBugCount', 'devBugCount', 'testBugCount', 'onlineBugCount', 'bugList', 'featureBugList', 'devBugList', 'testBugList', 'otherBugList']
        const data = this.formatJson(filterVal, this.list)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: '测试KPI统计',
          autoWidth: true
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'name') {
          for (const i in memberOptions) {
            if (memberOptions[i].name === v.name) {
              return memberOptions[i].chinese
            }
          }
          return v.name
        } else {
          return v[j]
        }
      }))
    },
    handleBugDetail(row) {
      this.pvData = row.bugList
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.bugCount'
    },
    handleFeatureDetail(row) {
      this.pvData = row.featureBugList
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.featureBugCount'
    },
    handleDevDetail(row) {
      this.pvData = row.devBugList
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.devBugCount'
    },
    handleTestDetail(row) {
      this.pvData = row.testBugList
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.testBugCount'
    },
    handleotherDetail(row) {
      this.pvData = row.otherBugList
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.otherBugCount'
    },
    handleJiraDetail(row) {
      window.open('http://j.kyee.com.cn/browse/' + row)
    },
    filterHandler(value, row, column) {
      const property = column['property']
      return row[property] === value
    }
  }
}
</script>
