<template>
  <div class="app-container">

    <div class="header container">
      <el-form label-position="left" inline class="demo-table-expand">
        <el-form-item label="编号:">
          <el-input v-model="form.num" clearable :placeholder="placeholderNum" style="width:271px;" prefix-icon="el-icon-s-order" />
        </el-form-item>
        <el-form-item label="姓名:">
          <el-input v-model="form.username" clearable :placeholder="placeholderUsername" style="width:271px;" prefix-icon="el-icon-user" />
        </el-form-item>
        <el-form-item label="考勤年月:">
          <el-date-picker v-model="form.yearMonth" type="month" :placeholder="placeholderYearMonth" style="width:271px;" format="yyyy 年 MM 月" value-format="yyyy-MM" />
        </el-form-item>
        <el-form-item label="休息日期:">
          <el-select v-model="form.selected" style="width: 76px;">
            <el-option label="星期" value="week"></el-option>
            <el-option label="日期" value="date"></el-option>
          </el-select>
          <el-date-picker v-show="form.selected === 'date'" v-model="form.dates" type="dates" :picker-options="pickerOptions" :default-value="infoYearMonth" format="yyyy 年 MM 月 dd 日" value-format="yyyy-MM-dd" placeholder="请选择休息日期" style="width: calc(271px - 76px);" @change="handleChange" />
          <el-select v-show="form.selected === 'week'" v-model="form.weeks" clearable multiple collapse-tags placeholder="请选择休息星期" style="width: calc(271px - 76px);" @change="handleChange">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="上班打卡:">
          <el-time-picker is-range v-model="form.startTimes" range-separator="至" :start-placeholder="'默认(' + this.defaultInfo.startTimes[0] + ')'" :end-placeholder="'默认(' + this.defaultInfo.startTimes[1] + ')'" style="width: 271px" value-format="HH:mm:ss" @change="handleChange" />
        </el-form-item>
        <el-form-item label="下班打卡:">
          <el-time-picker is-range v-model="form.endTimes" range-separator="至" :start-placeholder="'默认(' + this.defaultInfo.endTimes[0] + ')'" :end-placeholder="'默认(' + this.defaultInfo.endTimes[1] + ')'" style="width: 271px" value-format="HH:mm:ss" @change="handleChange" />
        </el-form-item>
        <el-form-item label="日期格式">
          <el-radio-group v-model="dateFormat">
            <el-radio label="/" border style="margin: 0;width: calc(271px/2)">
              1994/07/05
            </el-radio>
            <el-radio label="-" border style="margin: 0;width: calc(271px/2)">
              1994-07-05
            </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="文件名称:">
          <!-- <FilenameOption v-model="form.filename" :username="form.username" /> -->
          <el-input v-model="form.filename" clearable :placeholder="placeholderFilename" style="width:271px;" prefix-icon="el-icon-document" />
        </el-form-item>
        <el-form-item>
          <el-button :loading="downloadLoading" type="primary" icon="el-icon-document" @click="handleCreate">
            生成打卡数据
          </el-button>
          <el-button :loading="downloadLoading" :disabled="!(list && list.length)" type="primary" icon="el-icon-document" @click="handleDownload">
            导出为Excel
          </el-button>
        </el-form-item>
      </el-form>
      <!-- <el-row>
        <FilenameOption v-model="filename" :username="username" />
        <AutoWidthOption v-model="autoWidth" />
        <BookTypeOption v-model="bookType" />
      </el-row> -->
    </div>
    <div class="page-container1">
      <el-scrollbar class="page-component__scroll">
        <div class="page-container">
          <el-table :data="list" element-loading-text="Loading..." border fit highlight-current-row>
            <el-table-column align="center" label="序号" width="95">
              <template slot-scope="scope">
                {{ scope.$index + 1 }}
              </template>
            </el-table-column>
            <el-table-column align="center" :label="'编号\n考勤人员编号\n（未来以HR员工编号为准）'" min-width="200">
              <template slot-scope="scope">
                {{ scope.row.num }}
              </template>
            </el-table-column>
            <el-table-column :label="'姓名\n(选填)'" min-width="110" align="center">
              <template slot-scope="scope">
                <el-tag>{{ scope.row.username }}</el-tag>
              </template>
            </el-table-column>
            <!-- <el-table-column label="Readings" width="115" align="center">
              <template slot-scope="scope">
                {{ scope.row.pageviews }}
              </template>
            </el-table-column> -->
            <el-table-column align="center" :label="'打卡时间\nYYYY-MM-DD HH:MM:SS\n（格式不允许修改）'" min-width="220">
              <template slot-scope="scope">
                <i class="el-icon-time" />
                <span>{{ scope.row.date }}</span>
                <span>{{ ' ' }}</span>
                <el-tag type="success">{{ scope.row.date | parseTime('周{a}') }}</el-tag>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </el-scrollbar>
    </div>
  </div>
</template>

<script>
import { parseTime } from '@/utils'
// options components
// import FilenameOption from './components/FilenameOption'
export default {
  name: 'ExcelUtils',
  // components: { FilenameOption },
  data() {
    return {
      pickerOptions: {
        disabledDate: (time) => {
          const range = this.getDateRange(this.infoYearMonth)
          return time.getTime() < new Date(range[0] + ' 00:00:00') || time.getTime() > new Date(range[range.length - 1] + ' 00:00:00')
          // return time.getTime() > Date.now() || time.getTime() < new Date(range[0]) - 5*24*3600*1000
        }
      },
      list: null,
      downloadLoading: false,
      form: {
        num: '',
        username: '',
        yearMonth: '',
        selected: 'week',
        weeks: [0],
        dates: [],
        startTimes: '',
        endTimes: '',
        filename: ''
      },
      defaultInfo: {
        num: 'E100010578',
        username: '明玉钗',
        // 根据当前时间生成
        yearMonth: this.nowYearMonth(),
        startTimes: ['08:47:00', '08:50:00'],
        endTimes: ['21:00:00', '21:10:00']
      },
      dateFormat: '/',
      autoWidth: false,
      bookType: 'xlsx',
      options: [{
          value: 0,
          label: '星期日'
        }, {
          value: 1,
          label: '星期一'
        }, {
          value: 2,
          label: '星期二'
        }, {
          value: 3,
          label: '星期三'
        }, {
          value: 4,
          label: '星期四'
        }, {
          value: 5,
          label: '星期五'
        }, {
          value: 6,
          label: '星期六'
        }]
    }
  },
  computed: {
    placeholderNum() {
      return '请输入编号 (默认:' + this.defaultInfo.num + ')'
    },
    placeholderUsername() {
      return '请输入姓名 (默认:' + this.defaultInfo.username + ')'
    },
    placeholderYearMonth() {
      return '请输入年月 (默认:' + (this.formatYearMonth(this.defaultInfo.yearMonth)) + ')'
    },
    placeholderFilename() {
      return '文件名 (默认:' + this.defalutFilename + ')'
    },
    infoNum: {
      get() {
        if (!this.form.num) {
          return this.defaultInfo.num
        }
        return this.form.num
      }
    },
    infoUsername() {
      if (!this.form.username) {
        return this.defaultInfo.username
      }
      return this.form.username
    },
    infoYearMonth() {
      if (!this.form.yearMonth) {
        return this.defaultInfo.yearMonth
      }
      return this.form.yearMonth
    },
    defalutFilename() {
      const filename = this.infoUsername + '的' + this.getMonth(this.infoYearMonth) + '月考勤表'
      return filename
    },
    infoFilename() {
      if (!this.form.filename) {
        return this.defalutFilename
      }
      return this.form.filename
    },
    infoStartTimes() {
      if (!this.form.startTimes) {
        return this.defaultInfo.startTimes
      }
      return this.form.startTimes
    },
    infoEndTimes() {
      if (!this.form.endTimes) {
        return this.defaultInfo.endTimes
      }
      return this.form.endTimes
    }
  },
  watch: {
    infoYearMonth(val) {
      // 若selected是‘date’，更新选择框数据。
      // 更新dates数据，即若用户重选了年月yearMonth，那么dates数据要准备
      // this.form.selected === 'date' 选择的不是date，也要清除
      const dates = this.form.dates
      if (dates && dates.length) {
        for (let i=dates.length - 1; i>=0; i--) {
          if (dates[i].indexOf(val) === -1) {
            dates.splice(i, 1)
          }
        }
      }
      // 若selectd是‘week’，则根据week生成dates数据（再议）
      // 方案一：date和week分开，date记录下dates剔除，week生成数据时判断剔除
    }
    // selected切换的同时，立即生成dates数据
    // 1.初次加载
    // 2.week 切 date
    // 3.date 切 week
    ,
    defalutFilename(val) {
      this.form.filename = val
    }
  },
  created() {
    this.form.num = this.defaultInfo.num
    this.form.username = this.defaultInfo.username
    this.form.yearMonth = this.defaultInfo.yearMonth
    this.form.startTimes = this.defaultInfo.startTimes
    this.form.endTimes = this.defaultInfo.endTimes
    this.form.filename = this.defalutFilename
  },
  methods: {
    // 根据当前时间生成yearMonth
    nowYearMonth() {
      const date = new Date()
      return date.getFullYear() + '-' + this.getFullDate(date.getMonth() + 1)
    },
    // 文件名月份
    getMonth(yearMonth) {
      return new Date(yearMonth).getMonth() + 1
    },
    // 格式化展示placeholder默认年月值
    formatYearMonth(yearMonth) {
      const date = new Date(yearMonth)
      return date.getFullYear() + ' 年 ' + this.getFullDate(date.getMonth() + 1) + ' 月' 
    },
    // 获取所选年月的月份日期范围
    getDateRange(yearMonth) {
      const date = new Date(yearMonth)
      const year = date.getFullYear()
      const month = date.getMonth() + 1
      // 设置month，实际是设置下个月日期(month + 1)
      date.setMonth(month)
      date.setDate(0)
      // 月末日期
      const lastDay = date.getDate()
      const allDays = []
      for (let i=1; i<=lastDay; i++) {
        allDays.push(year + '-' + this.getFullDate(month) + '-' + this.getFullDate(i))
      }
      return allDays
    },
    // 根据日期范围，休息日（星期）,剔除数据
    weeks2Dates(allDays, weeks) {
      console.log(allDays, weeks)
      if (!weeks || weeks.length === 0) {
        return allDays
      }
      for (let i=allDays.length-1; i>=0; i--) {
        const date = new Date(allDays[i])
        const week = date.getDay()
        if (weeks.indexOf(week) !== -1) {
          allDays.splice(i, 1)
        }
      }
      return allDays
    },
    // 根据日期范围，休息日期（日期），剔除数据
    dates2Dates(allDays, dates) {
      if (!dates || dates.length === 0) {
        return allDays
      }
      for (let i=allDays.length-1; i>=0; i--) {
        if (dates.indexOf(allDays[i]) !== -1) {
          allDays.splice(i, 1)
        }
      }
      return allDays
    },
    handleChange(dates) {
      console.log(dates)
    },
    fetchData() {
      this.list = [{
          'id': 0,
          'title': 'xiaoming',
          'author': 'xiaozhou',
          'pageviews': 111,
          'dispaly_time': '2020-07-30 18:18:18'
      }]
    },
    getFullDate(num) {
      return num < 10 ? '0' + num : num
    },
    getEveryDay(dates, startTimes, endTimes, num, username) {
      const allDays = []
      // 生成随机时间
      const onRandomTimes = this.getRandomTimes(startTimes, dates.length)
      const offRandomTimes = this.getRandomTimes(endTimes, dates.length)
      dates.forEach((ymsDate, index) => {
        allDays.push({
          num: num,
          username: username,
          date: ymsDate + ' ' + onRandomTimes[index]
        })
        allDays.push({
          num: num,
          username: username,
          date: ymsDate + ' '  + offRandomTimes[index]
        })
      })
      return allDays
    },
    getRandomTimes(times, size) {
      // 日期前缀
      const datePrefix = '2020-01-01 '
      const startTime = datePrefix + times[0]
      const endTime = datePrefix + times[1]
      const allTimes = []
      const start = +new Date(startTime)
      const end = +new Date(endTime)
      console.log(start, end)
      for (let i = 1; i <= size; i++) {
        const newTime = new Date(start + Math.random() * (end - start))
        allTimes.push(this.getFullDate(newTime.getHours()) + ':' + this.getFullDate(newTime.getMinutes()) + ':' + this.getFullDate(newTime.getSeconds()))
      }
      return allTimes
    },
    handleCreate() {
      const num = this.infoNum
      const username = this.infoUsername
      // 1.获取年份月份
      const yearMonth = this.infoYearMonth
      // 2.获取日期范围
      const allDays = this.getDateRange(yearMonth)
      // 2.剔除休息日期
      let dates
      if (this.form.selected === 'date') {
        dates = this.dates2Dates(allDays, this.form.dates)
      } else {
        dates = this.weeks2Dates(allDays, this.form.weeks)
      }
      // 3.获取上班打卡范围
      const startTimes = this.infoStartTimes
      // 4.获取下班打卡范围
      const endTimes = this.infoEndTimes
      // 5.生成随机打卡数据
      const data = this.getEveryDay(dates, startTimes, endTimes, num, username)
      if (this.dateFormat === '/') {
        data.forEach(item => {
          item.date = item.date.replace(/-/g, '/')
        })
      }
      this.list = data
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['编号\n考勤人员编号\n（未来以HR员工编号为准）', 
          '姓名\n(选填)', 
          '打卡时间\nYYYY-MM-DD HH:MM:SS\n（格式不允许修改）']
        const filterVal = ['num', 'username', 'date']
        const list = this.list
        const data = this.formatJson(filterVal, list)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: this.infoFilename,
          autoWidth: this.autoWidth,
          bookType: this.bookType
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    }
  }
}
</script>

<style>
.page-component__scroll {
  height: 100%;
}
.page-component__scroll .el-scrollbar__wrap {
  overflow-x: auto;
}
.container .demo-table-expand {
  font-size: 0;
}
.container .demo-table-expand label {
  width: 72px;
  color: #99a9bf;
}
.container .demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: calc(33%);
  height: 40px;
  line-height: 40px;
}
.page-container .el-table .cell {
white-space: pre-line;
}
</style>
<style scoped>
.container, .page-container {
    width: 1140px;
    padding: 10px 0;
    margin: 0 auto;
}
.header {
  /* margin-top: 20px; */
  border-bottom: 1px solid #dcdfe6;
}
.page-container1 {
  height: calc(100vh - 140px);
}
</style>