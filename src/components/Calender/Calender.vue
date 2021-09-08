<template>
  <div class="calendar">
    <div class="header">
      <div class="header-day" @click="checkoutCurrentDate">
        <div style="margin: auto">
          {{ today.getDate() }}
          <div class="header-week">
            {{ getWeekName(today) }}
          </div>
        </div>
      </div>
      <div class="header-year-month">
        <span title="上一个月" class="arrow" @click="handlePreMonth()">
          <i class="arrow-left-1">◆</i>
          <i class="arrow-left-2">◆</i>
        </span>
        <div>{{ selectData.year }}年{{ selectData.month }}月</div>
        <span title="下一个月" class="arrow" @click="handleNextMonth()">
          <i class="arrow-right-1">◆</i>
          <i class="arrow-right-2">◆</i>
        </span>
      </div>
    </div>
    <ul class="week-box">
      <li
        v-for="(item, index) in weekArr"
        :key="index"
        class="week-item">
        <span class="week-font calendar-item">
          <div style="margin: auto">
            {{ item }}
          </div>
        </span>
      </li>
    </ul>
    <div class="data-container" :style="{height: `${lineNum * itemHeight}px`}">
      <div
        class="month-area">
        <div class="banner-area">
          <ul class="data-area">
            <li
              v-for="(item, index) in dataArr"
              :key="index"
              :class="[
                 'data-item',
                { 'selected': item.isSelected },
                { 'other-item': item.type !== 'normal' },
                { 'today': item.date === `${today.getFullYear()}-${today.getMonth() + 1}-${today.getDate()}` },
                { 'has-day': dayList.has(item.date) }
              ]"
              :style="{ height: `${itemHeight}px`}"
              @click="checkoutDate(item)">
              <div class="data-font calendar-item"
                   :style="{'background-color': `${colorMap[groupMap[[item.date]]]} !important`}">
                <div style="margin: auto">
                  {{ item.day }}
                </div>
              </div>
            </li>
          </ul>
          <div class="background-month" :style="{height: `${lineNum * itemHeight}px`}">
            <span>{{ selectData.month }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="foot">
      <slot />
    </div>

  </div>
</template>

<script>
export default {
  name: 'Calender',
  props: {
    // 标记日期 {date: yyyy-MM-dd, group: xxx}
    markList: {
      type: Array,
      default() {
        return []
      }
    },
    colorGroup: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      // 星期数组
      weekArr: ['日', '一', '二', '三', '四', '五', '六'],
      // 当前可视区域数据
      dataArr: [],
      today: new Date(),
      // 选中日期信息 -> year, month, day， date
      selectData: {},
      // 子元素行高
      itemHeight: 30,
      // 当前视图总行数
      lineNum: 0,

      // 标记日期
      dayList: new Set(),
      // 标记日期组
      groupMap: {},
      // 组颜色集合
      colorMap: {}
    }
  },
  watch: {
    markList() {
      this.initDayList()
    },
    selectData: {
      handler: 'dayChange',
      immediate: true,
      deep: true
    }
  },
  created() {
    this.checkoutCurrentDate()
    this.initDayList()
  },
  methods: {
    initDayList() {
      if (this.colorGroup) {
        const set = new Set(this.markList.map(v => v.date))
        // 初始化日期组
        this.markList.forEach(v => {
          this.groupMap[v.date] = v.group
        })

        const groupSet = new Set(this.markList.map(v => v.group))

        // 初始化组随机颜色 获取颜色 日期 -> 组 -> 颜色
        if (groupSet.size) {
          groupSet.forEach(group => {
            this.colorMap[group] = this.getRandomSafeColor()
          })
        } else {
          this.colorMap = {}
        }
        this.dayList = set
      } else {
        this.dayList = new Set(this.markList)
      }
    },
    // 今日
    checkoutCurrentDate() {
      this.getCurrentDate()
      this.initDayInfo()
    },
    // 获取当前日期
    getCurrentDate() {
      this.today = new Date()
      this.selectData = {
        year: this.today.getFullYear(),
        month: this.today.getMonth() + 1,
        day: this.today.getDate()
      }
    },
    // 点选切换日期
    checkoutDate(selectData) {
      if (selectData.type === 'pre' || selectData.type === 'next') {
        this.selectData = selectData
        this.initDayInfo()
        return
      }
      this.selectData.day = selectData.day
      const oldSelectIndex = this.dataArr.findIndex(item => item.isSelected && item.type === 'normal')
      const newSelectIndex = this.dataArr.findIndex(item => item.day === selectData.day && item.type === 'normal')
      if (this.dataArr[oldSelectIndex]) {
        this.dataArr[oldSelectIndex].isSelected = false
      }
      if (this.dataArr[newSelectIndex]) {
        this.dataArr[newSelectIndex].isSelected = true
      }
    },
    // 切换上一月
    handlePreMonth(date = undefined) {
      this.selectData = this.getPreMonth(date, 1)
      this.initDayInfo()
    },
    // 切换下一月
    handleNextMonth(date = undefined) {
      this.selectData = this.getNextMonth(date, 1)
      this.initDayInfo()
    },
    // 获取前一个月的年月日信息
    getPreMonth(date = undefined, appointDay = undefined) {
      date = date || this.selectData
      let { year, month } = date
      if (month === 1) {
        year -= 1
        month = 12
      } else {
        month -= 1
      }
      return { year, month, day: appointDay || date.day }
    },
    // 获取一个月的年月日信息
    getNextMonth(date = undefined, appointDay = undefined) {
      date = date || this.selectData
      let { year, month } = date
      if (month === 12) {
        year += 1
        month = 1
      } else {
        month += 1
      }
      return { year, month, day: appointDay || date.day }
    },
    initDayInfo() {
      this.dataArr = this.getMonthData(this.selectData)
      this.lineNum = Math.ceil(this.dataArr.length / 7)
    },
    // 获取指定月份数据
    getMonthData(date, unSelected = false) {
      const { year, month, day } = date
      const dataArr = []
      const daysInMonth = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
      if ((year % 4 === 0 && year % 100 !== 0) || year % 400 === 0) {
        daysInMonth[1] = 29
      }
      const monthStartWeekDay = new Date(year, month - 1, 1).getDay()
      const monthEndWeekDay = new Date(year, month, 1).getDay() || 7
      const preInfo = this.getPreMonth(date)
      const nextInfo = this.getNextMonth(date)
      for (let i = 0; i < monthStartWeekDay; i++) {
        const preObj = {
          type: 'pre',
          day: daysInMonth[preInfo.month - 1] - (monthStartWeekDay - i - 1),
          month: preInfo.month,
          year: preInfo.year
        }
        preObj.date = `${preObj.year}-${preObj.month}-${preObj.day}`
        dataArr.push(preObj)
      }
      for (let i = 0; i < daysInMonth[month - 1]; i++) {
        const itemObj = {
          type: 'normal',
          day: i + 1,
          month,
          year,
          isSelected: day === i + 1 && !unSelected
        }
        itemObj.date = `${itemObj.year}-${itemObj.month}-${itemObj.day}`
        dataArr.push(itemObj)
      }
      for (let i = 0; i < 7 - monthEndWeekDay; i++) {
        const nextObj = {
          type: 'next',
          day: i + 1,
          month: nextInfo.month,
          year: nextInfo.year
        }
        nextObj.date = `${nextObj.year}-${nextObj.month}-${nextObj.day}`
        dataArr.push(nextObj)
      }
      return dataArr
    },
    getWeekName(date) {
      const day = date.getDay()
      switch (day) {
        case 0:
          return '星期天'
        case 1:
          return '星期一'
        case 2:
          return '星期二'
        case 3:
          return '星期三'
        case 4:
          return '星期四'
        case 5:
          return '星期五'
        case 6:
          return '星期六'
        default:
          return ''
      }
    },
    getRandomSafeColor() {
      const base = ['00', '33', '66', '99', 'CC', 'FF']
      const len = base.length
      const bg = []
      const random = Math.ceil(Math.random() * 215 + 1)
      let res
      for (let r = 0; r < len; r++) {
        for (let g = 0; g < len; g++) {
          for (let b = 0; b < len; b++) {
            bg.push('#' + base[r].toString() + base[g].toString() + base[b].toString())
          }
        }
      }
      for (let i = 0; i < bg.length; i++) {
        res = bg[random]
      }
      return res
    },
    dayChange() {
      this.$emit('onDayChange', `${this.selectData.year}-${this.selectData.month}-${this.selectData.day}`)
    }
  }
}
</script>

<style lang="scss" scoped>
.calendar {
  overflow-x: hidden;
  border: 1px #F3F2F2 solid;
  width: 100%;
}

.header {
  height: 80px;
  padding: 0 5px;
  display: flex;
}

.foot {
  padding: 4px;
}

.header-day {
  font-size: 36px;
  color: #E52355;
  width: 30%;
  display: flex;
  border-right: 1px #F3F2F2 solid;
  cursor: pointer;
  text-align: center;
}

.header-week {
  font-size: 12px;
  color: #BCB8B9;
  font-weight: bold;
}

.header-year-month {
  display: flex;
  margin: auto;

  i {
    font-style: normal;
    cursor: pointer;
  }

  .arrow {
    font-size: 24px;

    /** 双击不选取 **/
    -moz-user-select:none;/*火狐*/
    -webkit-user-select:none;/*webkit浏览器*/
    -ms-user-select:none;/*IE10*/
    -khtml-user-select:none;/*早期浏览器*/
    user-select:none;
  }

  .arrow-left-1 {
    color: #000000;
    position: absolute;
  }

  .arrow-left-2 {
    color: #FFFFFF;
    left: 2px;
    position: relative;
  }

  .arrow-right-1 {
    color: #000000;
    position: absolute;
  }

  .arrow-right-2 {
    color: #FFFFFF;
    right: 2px;
    position: relative;
  }

  div {
    color: #E52355;
    width: 96px;
    text-align: center;
    padding-bottom: 5px;
    padding-top: 5px;
  }
}

.calendar-item {
  display: flex;
  width: 25px;
  height: 25px;
  border-radius: 50%;
}

.selected .calendar-item {
  background-image: url('https://cdn.jsdelivr.net/gh/nitmali/picture-bed@master/self/select-day.png');
  background-size: 100% 100%;
}

.has-day:not(.selected, .today) .calendar-item {
  background-color: rgba(82, 212, 140, 1);
  color: #FFFFFF !important;
  border-radius: 50%;
}

.today:not(.selected) .calendar-item {
  background-color: #0e96f6;
  color: #FFFFFF;
  border-radius: 50%;
}

.week-box {
  margin: unset;
  padding-left: unset;
  width: 100%;
  display: flex;
  border-top: 1px #F3F2F2 solid;
  border-bottom: 1px #F3F2F2 solid;
}

.week-item {
  flex: 0 0 14.285%;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 30px;
}

.week-font {
  font-size: 12px;
  color: #1890ff;
  font-weight: 500;
}

.data-container {
  overflow: hidden;
  position: relative;
  padding-bottom: 12px;
  border-bottom: 1px #F3F2F2 solid;
}

.banner-area {
  display: flex;
}

.data-area {
  margin: unset;
  padding-left: unset;
  width: 100%;
  height: 100%;
  display: flex;
  flex-flow: row wrap;
  position: absolute;
  z-index: 99;
}

.data-item {
  flex: 0 0 14.285%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.data-font {
  color: #2b4450;
  font-size: 12px;
  font-weight: 400;
}

.other-item .data-font {
  color: #ccc;
}

.background-month {
  width: 100%;
  position: relative;
  text-align: center;
  color: #F0F0F0;
  //opacity: 0.8;
  display: flex;
  font-size: 124px;
  font-weight: 900;
  z-index: 98;

  span {
    margin: auto;
  }
}
</style>
