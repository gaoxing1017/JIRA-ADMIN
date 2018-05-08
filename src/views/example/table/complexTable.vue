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
    <template>
      <span>{{$t('select.title')}}:</span>
      <el-select v-model="selectedOption" :placeholder="$t('select.placeholder')">
        <el-option
          v-for="item in selectOptions"
          :key="item.value"
          :label="item.text"
          :value="item.value">
        </el-option>
      </el-select>
    </template>
    <template>
      <el-button class="filter-item" type="primary" v-waves icon="el-icon-search" @click="handleFilter">{{$t('table.search')}}</el-button>
      <el-button class="filter-item" type="primary" :loading="downloadLoading" v-waves icon="el-icon-download" @click="handleDownload">{{$t('table.export')}}</el-button>
    </template>
    </div>

    <el-table :key='tableKey' :data="list" v-loading="listLoading" :element-loading-text="$t('table.loading')" show-summary stripe fit highlight-current-row
      style="width: 100%" :default-sort = "{prop: 'bugCount', order: 'descending'}">
      <el-table-column align="center" :label="$t('table.id')" width="70" type="index">
      </el-table-column>
      <el-table-column prop="name" width="120px" align="center" :label="$t('table.developer')" :formatter="nameFormat"
      :filters="developerFilterOptions" :filter-method="filterHandler">
      </el-table-column>
      <el-table-column prop="storyPoints" width="100px" align="center" :label="$t('table.storyPoint')" sortable >
      </el-table-column>
      <el-table-column prop="bugStoryPoints" width="120px" align="center" :label="$t('table.bugStoryPoint')" sortable >
      </el-table-column>
      <el-table-column prop="bugCount" width="100px" align="center" :label="$t('table.bugCount')" sortable >
      </el-table-column>
      <el-table-column prop="returnTime" width="100px" align="center" :label="$t('table.returnCount')" sortable >
      </el-table-column>
      <el-table-column prop="delayTaskCount" width="100px" align="center" :label="$t('table.delayCount')" sortable >
      </el-table-column>
      <el-table-column align="center" :label="$t('table.detail')" min-width="230" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="small" plain @click="handleStoryDetail(scope.row)">{{$t('table.storyPoint')}}</el-button>
          <el-button type="primary" size="small" plain @click="handleSubtaskDetail(scope.row)">{{$t('table.subTask')}}</el-button>
          <el-button type="primary" size="small" plain @click="handleBugDetail(scope.row)">{{$t('table.bugCount')}}</el-button>
          <el-button type="primary" size="small" plain @click="handleDelayDetail(scope.row)">{{$t('table.delayCount')}}</el-button>
          <el-button type="primary" size="small" plain @click="handleReturnDetail(scope.row)">{{$t('table.returnCount')}}</el-button>
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
import { fetchList } from '@/api/article'
import waves from '@/directive/waves' // 水波纹指令

const calendarTypeOptions = [
  { key: 'CN', display_name: 'China' },
  { key: 'US', display_name: 'USA' },
  { key: 'JP', display_name: 'Japan' },
  { key: 'EU', display_name: 'Eurozone' }
]
const developerOptions = [
  { name: 'weixiaokang', chinese: '韦晓康' },
  { name: 'huangaoxin', chinese: '黄奥鑫' },
  { name: 'zhoushuyan', chinese: '周淑琰' },
  { name: 'wangbingfu', chinese: '王丙付' },
  { name: 'zhangyizhi', chinese: '章逸之' },
  { name: 'ouyangbo', chinese: '欧阳礡' },
  { name: 'zengqinggui', chinese: '曾庆贵' },
  { name: 'gongjunqing', chinese: '龚俊卿' },
  { name: 'luozheng', chinese: '罗蒸' },
  { name: 'zhengxiang', chinese: '郑翔' },
  // 服务端
  { name: 'caizhongheng', chinese: '蔡忠亨' },
  { name: 'wangpengfei', chinese: '王鹏飞' },
  { name: 'liangfei', chinese: '梁飞' },
  { name: 'lijiaren', chinese: '李佳任' },
  { name: 'xufeiyang', chinese: '徐飞阳' },
  { name: 'taoyemeng', chinese: '陶冶梦' },
  { name: 'huminghu', chinese: '胡明湖' },
  { name: 'zhuwenjie', chinese: '朱文杰' },
  { name: 'caihong', chinese: '蔡红' },
  { name: 'fengyan', chinese: '冯岩' },

  // 安卓
  { name: 'jinsheng', chinese: '金圣' },
  { name: 'wangzhonghao', chinese: '王忠皓' },
  { name: 'wangdapeng', chinese: '汪大鹏' },
  { name: 'pengchenwei', chinese: '彭晨伟' },
  { name: 'luying', chinese: '陆嬴' },
  { name: 'douyeliang', chinese: '窦烨良' },
  { name: 'zhangjin', chinese: '张金' },
  { name: 'ruanyitai', chinese: '阮宜太' },
  { name: 'mashoujun', chinese: '马守君' },
  { name: 'shibing', chinese: '石冰' }

]
// arr to obj ,such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'complexTable',
  directives: {
    waves
  },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: null,
      listLoading: false,
      QueryDate: [],
      listQuery: {
        startTime: '',
        endTime: '',
        type: 0
      },
      selectedOption: { text: '徐汇', value: 0 },
      selectOptions: [{ text: '徐汇', value: 0 },
        { text: '安卓', value: 1 },
        { text: '服务端', value: 2 }],
      developerFilterOptions: [],
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
    for (const i in developerOptions) {
      this.developerFilterOptions.push({ text: developerOptions[i].chinese, value: developerOptions[i].name })
    }
  },
  methods: {
    getList() {
      this.listLoading = true
      fetchList(this.listQuery).then(response => {
        this.list = response.data.data
        this.total = response.data.total
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.startTime = this.QueryDate[0]
      this.listQuery.endTime = this.QueryDate[1]
      this.listQuery.type = this.selectedOption
      this.getList()
    },
    nameFormat(row, col) {
      for (const i in developerOptions) {
        if (developerOptions[i].name === row.name) {
          return developerOptions[i].chinese
        }
      }
      return row.name
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['姓名', '故事点', 'Bug故事点', 'BUG数', '打回数', '逾期数', '故事明细', '子任务明细', 'BUG明细', '打回明细', '逾期明细']
        const filterVal = ['name', 'storyPoints', 'bugStoryPoints', 'bugCount', 'returnTime', 'delayTaskCount', 'issues', 'subTasks', 'bugs', 'returnList', 'delayList']
        const data = this.formatJson(filterVal, this.list)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: '开发KPI统计',
          autoWidth: true
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'name') {
          for (const i in developerOptions) {
            if (developerOptions[i].name === v.name) {
              return developerOptions[i].chinese
            }
          }
          return v.name
        } else {
          return v[j]
        }
      }))
    },
    handleStoryDetail(row) {
      this.pvData = row.issues
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.storyPoint'
    },
    handleSubtaskDetail(row) {
      this.pvData = row.subTasks
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.subTask'
    },
    handleBugDetail(row) {
      this.pvData = row.bugs
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.bugCount'
    },
    handleDelayDetail(row) {
      this.pvData = row.delayList
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.delayCount'
    },
    handleReturnDetail(row) {
      this.pvData = row.returnList
      this.dialogPvVisible = true
      this.dialogPvTitle = 'table.returnCount'
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
