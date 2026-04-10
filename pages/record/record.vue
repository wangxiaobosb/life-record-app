<template>
  <view class="container">
    <!-- 日历头部 -->
    <view class="calendar-header">
      <button class="arrow-btn" @click="prevMonth">‹</button>
      <text class="month-text">{{ currentYear }}年{{ currentMonth }}月</text>
      <button class="arrow-btn" @click="nextMonth">›</button>
    </view>

    <!-- 日历主体 -->
    <view class="calendar-card">
      <view class="week-header">
        <view class="week-item" v-for="w in weekList" :key="w">{{ w }}</view>
      </view>
      <view class="calendar-grid">
        <view class="day-item empty" v-for="i in firstDayEmpty" :key="'e-'+i"></view>
        <view 
          class="day-item" 
          v-for="d in monthDays" 
          :key="d"
          :class="{ 
            today: isToday(d), 
            hasRecord: hasRecord(d)
          }"
          @click="selectDate(d)"
        >
          {{ d }}
          <view class="dot" v-if="hasRecord(d)"></view>
        </view>
      </view>
    </view>

    <!-- 选中日期的详情 -->
    <view class="detail-card" v-if="selectedDate">
      <view class="detail-title">{{ selectedDate }} 记录</view>
      
      <!-- 三餐记录 -->
      <view class="detail-section">
        <view class="section-title">🍽️ 三餐记录</view>
        <view class="meal-detail" v-for="meal in meals" :key="meal.type">
          <text class="meal-name">{{ meal.name }}</text>
          <text class="meal-status" :class="detailMeals.check[meal.type] ? 'done' : 'undone'">
            {{ detailMeals.check[meal.type] ? `${detailMeals.content[meal.type] || '已打卡'} ${detailMeals.time[meal.type]}` : '未打卡' }}
          </text>
        </view>
      </view>

      <!-- 体重记录 -->
      <view class="detail-section" v-if="detailWeight">
        <view class="section-title">⚖️ 体重记录</view>
        <text class="weight-text">{{ detailWeight.value }} kg，记录时间：{{ detailWeight.time }}</text>
      </view>

      <!-- 日记记录 -->
      <view class="detail-section" v-if="detailDiary">
        <view class="section-title">📝 日记</view>
        <text class="diary-text">{{ detailDiary.content }}</text>
        <text class="diary-time">最后修改：{{ detailDiary.updateTime }}</text>
      </view>

      <view class="empty-detail" v-if="!hasAnyRecord">
        <text>当日暂无任何记录</text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      weekList: ['日', '一', '二', '三', '四', '五', '六'],
      currentYear: new Date().getFullYear(),
      currentMonth: new Date().getMonth() + 1,
      selectedDate: null,
      meals: [
        { type: 'breakfast', name: '早餐' },
        { type: 'lunch', name: '午餐' },
        { type: 'dinner', name: '晚餐' }
      ],
      detailMeals: { check: {}, content: {}, time: {} },
      detailWeight: null,
      detailDiary: null
    }
  },
  computed: {
    monthDays() {
      return new Date(this.currentYear, this.currentMonth, 0).getDate()
    },
    firstDayEmpty() {
      return new Date(this.currentYear, this.currentMonth - 1, 1).getDay()
    },
    today() {
      const d = new Date()
      return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`
    },
    hasAnyRecord() {
      return this.detailMeals.check.breakfast || this.detailMeals.check.lunch || 
             this.detailMeals.check.dinner || this.detailWeight || this.detailDiary
    }
  },
  onShow() {
    this.selectedDate = null
  },
  methods: {
    prevMonth() {
      this.currentMonth--
      if (this.currentMonth < 1) {
        this.currentMonth = 12
        this.currentYear--
      }
      this.selectedDate = null
    },
    nextMonth() {
      this.currentMonth++
      if (this.currentMonth > 12) {
        this.currentMonth = 1
        this.currentYear++
      }
      this.selectedDate = null
    },
    isToday(day) {
      const dateStr = `${this.currentYear}-${String(this.currentMonth).padStart(2,'0')}-${String(day).padStart(2,'0')}`
      return dateStr === this.today
    },
    // 判断当天是否有任何记录
    hasRecord(day) {
      const dateStr = `${this.currentYear}-${String(this.currentMonth).padStart(2,'0')}-${String(day).padStart(2,'0')}`
      const allMeals = uni.getStorageSync('life_all_meals') || {}
      const allWeight = uni.getStorageSync('life_all_weight') || {}
      const allDiary = uni.getStorageSync('life_all_diary') || {}
      
      const hasMeal = allMeals[dateStr] && (allMeals[dateStr].check.breakfast || allMeals[dateStr].check.lunch || allMeals[dateStr].check.dinner)
      const hasWeight = !!allWeight[dateStr]
      const hasDiary = !!allDiary[dateStr]
      
      return hasMeal || hasWeight || hasDiary
    },
    // 选择日期查看详情
    selectDate(day) {
      const dateStr = `${this.currentYear}-${String(this.currentMonth).padStart(2,'0')}-${String(day).padStart(2,'0')}`
      this.selectedDate = dateStr
      
      // 加载三餐记录
      const allMeals = uni.getStorageSync('life_all_meals') || {}
      this.detailMeals = allMeals[dateStr] || {
        check: { breakfast: false, lunch: false, dinner: false },
        content: { breakfast: '', lunch: '', dinner: '' },
        time: { breakfast: '', lunch: '', dinner: '' }
      }

      // 加载体重记录
      const allWeight = uni.getStorageSync('life_all_weight') || {}
      this.detailWeight = allWeight[dateStr] || null

      // 加载日记记录
      const allDiary = uni.getStorageSync('life_all_diary') || {}
      this.detailDiary = allDiary[dateStr] || null
    }
  }
}
</script>

<style scoped>
.container {
  min-height: 100vh;
  background-color: #F8F9FA;
  padding: 20px 16px;
  padding-bottom: 80px;
}
/* 日历头部 */
.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px;
  background-color: #FFFFFF;
  border-radius: 12px;
  margin-bottom: 16px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.05);
}
.arrow-btn {
  width: 44px;
  height: 44px;
  border: none;
  background: none;
  font-size: 20px;
  color: #3B82F6;
  padding: 0;
}
.month-text {
  font-size: 18px;
  font-weight: bold;
  color: #1F2937;
}
/* 日历卡片 */
.calendar-card {
  background-color: #FFFFFF;
  border-radius: 12px;
  padding: 16px;
  margin-bottom: 20px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.05);
}
.week-header {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  margin-bottom: 12px;
}
.week-item {
  text-align: center;
  font-size: 14px;
  color: #9CA3AF;
  padding: 8px 0;
  font-weight: 500;
}
.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 6px;
}
.day-item {
  aspect-ratio: 1/1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  border-radius: 8px;
  position: relative;
  background-color: #F9FAFB;
  color: #4B5563;
}
.day-item.today {
  background-color: #3B82F6;
  color: #FFFFFF;
}
.day-item.hasRecord {
  background-color: #EFF6FF;
}
.dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: #3B82F6;
  position: absolute;
  bottom: 6px;
}
.day-item.today .dot {
  background-color: #FFFFFF;
}
/* 详情卡片 */
.detail-card {
  background-color: #FFFFFF;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.05);
}
.detail-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 20px;
  color: #1F2937;
  padding-bottom: 12px;
  border-bottom: 1px solid #F3F4F6;
}
.detail-section {
  margin-bottom: 24px;
}
.section-title {
  font-size: 16px;
  font-weight: 500;
  margin-bottom: 12px;
  color: #374151;
}
.meal-detail {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #F9FAFB;
}
.meal-name {
  font-size: 15px;
  color: #4B5563;
}
.meal-status {
  font-size: 14px;
}
.meal-status.done {
  color: #10B981;
}
.meal-status.undone {
  color: #9CA3AF;
}
.weight-text {
  font-size: 15px;
  color: #4B5563;
}
.diary-text {
  display: block;
  font-size: 15px;
  line-height: 1.6;
  color: #4B5563;
  margin-bottom: 8px;
}
.diary-time {
  font-size: 12px;
  color: #9CA3AF;
}
.empty-detail {
  text-align: center;
  padding: 30px 0;
  font-size: 14px;
  color: #9CA3AF;
}
</style>