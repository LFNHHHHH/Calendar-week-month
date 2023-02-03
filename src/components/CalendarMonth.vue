<template>
  <div class="calendar">
    <div class="calendar-top">
      <div @click="checkoutPreMonth">上一月</div>
      <div>{{ `${selectData.year}-${selectData.month}-${selectData.day}` }}</div>
      <div @click="checkoutNextMonth">下一月</div>
    </div>
    <ul class="week-area">
      <li class="week-item" v-for="(item, index) in weekArr" :key="index">{{ item }}</li>
    </ul>

    <ul class="data-area">
      <li
        class="calendar-item"
        v-for="(item, index) in dataArr" :key="index"
        @click="checkoutDate(item)"
      >
        <span
          :class="[{
            red: item.isSelected, 
            clddd: item.type != 'normal'
          }]"
        >{{ item.isSelected ? '😊' : item.day }}</span>
      </li>
    </ul>
  </div>
</template>

<script>
// import dayjs from 'dayjs'
export default {
  name: 'CalendarMonth',
  data () {
    return {
      weekArr: ['周日', '周一', '周二', '周三', '周四', '周五', '周六'],
      dataArr: [],

      selectData: {}
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
      const preInfo = this.getPreMonth()
      const nextInfo = this.getNextMonth()

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
      if (selectData.type !== 'normal') return // 非有效日期不可点选

      this.selectData.day = selectData.day // 对选中日期赋值

      // 查找当前选中日期的索引
      const oldSelectIndex = this.dataArr.findIndex(item => item.isSelected && item.type === 'normal')
      // 查找新切换日期的索引
      const newSelectIndex = this.dataArr.findIndex(item => item.day === selectData.day && item.type === 'normal')

      // 更改isSelected值
      if (this.dataArr[oldSelectIndex]) this.$set(this.dataArr[oldSelectIndex], 'isSelected', false)
      if (this.dataArr[newSelectIndex]) this.$set(this.dataArr[newSelectIndex], 'isSelected', true)
    },
    checkoutPreMonth() { // 点击上个月按钮
      let { year, month } = this.selectData
      if (month === 1) {
        year -= 1
        month = 12
      } else {
        month -= 1
      }

      this.selectData = { year, month, day: 1 }
      this.dataArr = this.getMonthData(this.selectData)
    },
    checkoutNextMonth() { // 点击下个月按钮
      let { year, month } = this.selectData
      if (month === 12) {
        year += 1
        month = 1
      } else {
        month += 1
      }

      this.selectData = { year, month, day: 1 }
      this.dataArr = this.getMonthData(this.selectData)
    },
    getPreMonth(date) { // // 获取前一个月的年月信息
      let { year, month } = date || this.selectData
      if (month === 1) {
        year -= 1
        month = 12
      } else {
        month -= 1
      }
      return { year, month }
    },
    getNextMonth() { // // 获取后一个月的年月信息
      let { year, month } = this.selectData
      if (month === 12) {
        year += 1
        month = 1
      } else {
        month += 1
      }
      return { year, month }
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