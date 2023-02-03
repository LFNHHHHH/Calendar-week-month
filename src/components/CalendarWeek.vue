<template>
  <div class="calendar">
    <div class="calendar-top">
      <div @click="checkoutPreWeek">上一周</div>
      <div>{{ `${selectData.year}-${selectData.month}-${selectData.day}` }}</div>
      <div @click="checkoutNextWeek">下一周</div>
    </div>
    <ul class="week-area">
      <li class="week-item" v-for="(item, index) in weekArr" :key="index">{{ item }}</li>
    </ul>

    <ul class="data-area">
      <li
        class="calendar-item"
        v-for="(item, index) in weekData" :key="index"
        @click="checkoutDate(item)"
      >
        <span
          :class="[{
            red: item.isSelected, 
          }]"
        >{{ item.day }}</span>
      </li>
    </ul>
  </div>
</template>

<script>
// import dayjs from 'dayjs'
export default {
  name: 'CalendarComponent',
  data () {
    return {
      weekArr: ['周日', '周一', '周二', '周三', '周四', '周五', '周六'],
      dataArr: [],
      weekData: [],

      selectData: {},
    }
  },
  created () {
    this.init()
  },
  methods: {
    init () { // 初始化
      // 获取当前日期
      this.getCurrentDate()

      // 获取指定月份数据
      this.dataArr = this.getMonthData(this.selectData)

      // 获取本周数据
      this.getWeekData()
    },
    getCurrentDate () { // 获取当前日期
      this.selectData = {
        year: new Date().getFullYear(),
        month: new Date().getMonth() + 1,
        day: new Date().getDate(),
      }
    },
    getMonthData (data) { // 获取指定月份数据
      const { year, month, day } = data
      let dataArr = []
      let daysInMonth = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]

      // 闰年处理
      if ((year % 4 === 0 && year % 100 !== 0) || year % 400 === 0) {
        daysInMonth[1] = 29
      }

      const monthStartWeekDay = new Date(year, month - 1, 1).getDay() // 当前月份1号是周几
      const monthEndWeekDay = new Date(year, month, 1).getDay() // 下一月1号是周几

      // 数据填充
      const preInfo = this.getPreMonth(data)
      const nextInfo = this.getNextMonth(data)

      for (let i = 0; i < monthStartWeekDay; i++) {
        let preObj = {
          type: 'pre',
          day: daysInMonth[preInfo.month - 1] - (monthStartWeekDay - i - 1),
          month: preInfo.month,
          year: preInfo.year,
        }
        dataArr.push(preObj)
      }

      for (let i = 0; i < daysInMonth[month - 1]; i++) {
        let emptyObj = {
          type: 'normal',
          day: i + 1,
          month,
          year,
          isSelected: Number(day) === i + 1
        }
        dataArr.push(emptyObj)
      }

      if (!monthEndWeekDay) return dataArr // 周日值为 0，此时不需要渲染下面数据
      for (let i = 0; i < 7 - monthEndWeekDay; i++) {
        let nextObj = {
          type: 'next',
          day: i + 1,
          month: nextInfo.month,
          year: nextInfo.year,
        }
        dataArr.push(nextObj)
      }

      return dataArr
    },
    checkoutDate(selectData) { // 切换点选日期（更改当前数组中子元素的isSelected）
      this.selectData = selectData // 对选中日期赋值

      this.dataArr.forEach(item => item.isSelected = false)
      this.weekData.forEach(item => item.isSelected = false)

      const index = this.dataArr.findIndex(item => item.day === selectData.day)
      const index2 = this.weekData.findIndex(item => item.day == selectData.day)
      if (this.dataArr[index]) this.$set(this.dataArr[index], 'isSelected', true)
      if (this.weekData[index2]) this.$set(this.weekData[index2], 'isSelected', true)
    },
    getInfoOfWeekView(selectedIndex, length) { // 获取处理周视图所需的位置信息
      const indexOfLine = Math.ceil((selectedIndex + 1) / 7) // 当前行数
      const totalLine = Math.ceil(length / 7) // 总行数
      const sliceStart = (indexOfLine - 1) * 7 // 当前行左端索引
      const sliceEnd = sliceStart + 7 // 当前行右端索引

      return { indexOfLine, totalLine, sliceStart, sliceEnd }
    },
    getWeekData () { // 获取本周数据信息
      const selectedIndex = this.dataArr.findIndex(item => item.isSelected)
      const { sliceStart, sliceEnd } = this.getInfoOfWeekView(selectedIndex, this.dataArr.length)
      this.weekData = this.dataArr.slice(sliceStart, sliceEnd)
    },
    checkoutPreWeek() { // 点击上一周按钮
      const selectedIndex = this.dataArr.findIndex(item => item.isSelected)
      const { indexOfLine, sliceStart, sliceEnd } = this.getInfoOfWeekView(selectedIndex, this.dataArr.length)

      // 前一周数据
      let weekData = []
      if (indexOfLine === 1) {
        let preDataArr = []
        // 如果当前时间已经是上一月数据了，就不需要在获取上一月数据了
        if (this.dataArr[selectedIndex].type == 'normal') {
          preDataArr = this.getMonthData(this.getPreMonth(), true)
        } else {
          preDataArr = this.getMonthData(this.selectData, true)
        }
        const preDay = this.dataArr[0].day - 1 || preDataArr[preDataArr.length - 1].day
        const preIndex = preDataArr.findIndex(item => item.day === preDay && item.type === 'normal')
        const { sliceStart: preSliceStart, sliceEnd: preSliceEnd } = this.getInfoOfWeekView(preIndex, preDataArr.length)
        this.dataArr = preDataArr
        weekData = preDataArr.slice(preSliceStart, preSliceEnd)
      } else {
        weekData = this.dataArr.slice(sliceStart - 7, sliceEnd - 7)
      }
      this.setSelected(weekData)
    },
    checkoutNextWeek() { // 点击下一周按钮
      const selectedIndex = this.dataArr.findIndex(item => item.isSelected)
      const { indexOfLine, sliceStart, sliceEnd, totalLine } = this.getInfoOfWeekView(selectedIndex, this.dataArr.length)

      // 后一周数据 
      let weekData = []
      if (indexOfLine >= totalLine) {
        const nextDataArr = this.getMonthData(this.getNextMonth(), true)
        const nextDay = this.dataArr[this.dataArr.length - 1].type === 'normal' ? 1 : this.dataArr[this.dataArr.length - 1].day + 1
        const nextIndex = nextDataArr.findIndex(item => item.day === nextDay)
        const { sliceStart: nextSliceStart, sliceEnd: nextSliceEnd } = this.getInfoOfWeekView(nextIndex, nextDataArr.length)
        this.dataArr = nextDataArr
        weekData = nextDataArr.slice(nextSliceStart, nextSliceEnd)
      } else {
        weekData = this.dataArr.slice(sliceStart + 7, sliceEnd + 7)
      }
      this.setSelected(weekData)
    },
    setSelected (weekData) { // 设置 selected
      weekData[0].isSelected = true
      this.dataArr.forEach(item => item.isSelected = false)
      let index = this.dataArr.findIndex(item => item.day == weekData[0].day && item.month == weekData[0].month)
      if (this.dataArr[index]) this.$set(this.dataArr[index], 'isSelected', true)
      this.selectData = weekData[0]
      this.weekData = weekData
    },
    getPreMonth(date, appointDay = 1) { // // 获取前一个月的年月信息
      let { year, month } = date || this.selectData
      if (month === 1) {
        year -= 1
        month = 12
      } else {
        month -= 1
      }
      return { year, month, day: appointDay }
    },
    getNextMonth(date, appointDay = 1) { // // 获取后一个月的年月信息
      let { year, month } = date || this.selectData
      if (month === 12) {
        year += 1
        month = 1
      } else {
        month += 1
      }
      return { year, month, appointDay}
    },
  }
}
</script>

<style scoped>
ul {padding: 0;}
li {padding: 10px 0;}
.red {color: red; font-weight: bold;}
.clddd {color: #ddd;}
.calendar-top {
  width: 100%;
  display: flex;
  justify-content: space-around;
  align-items: center;
  cursor: pointer;
}




.calendar {overflow-x: hidden;}
.week-area {
  width: 100%;
  display: flex;
}
.week-item, .calendar-item {
  display: flex;
  flex: 0 0 14.285%;
  align-items: center;
  justify-content: center;
}
.data-area {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  cursor: pointer;
}
</style>