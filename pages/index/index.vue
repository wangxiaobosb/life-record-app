<template>
  <view class="container">
    <!-- 未填写信息时的引导页 -->
    <view v-if="!hasInfo" class="welcome-page">
      <view class="welcome-card">
        <view class="welcome-icon">⏳</view>
        <view class="welcome-title">记录你的生命时光</view>
        <view class="welcome-sub">每一秒都值得被记录</view>
        
        <input 
          class="input" 
          v-model="name" 
          placeholder="请输入你的姓名"
          maxlength="10"
        />

        <!-- 快速年份选择器 -->
        <view class="date-picker">
          <view class="date-header">
            <button class="year-btn" @click="prevYear">‹</button>
            <text class="year-text">{{ selectYear }}年</text>
            <button class="year-btn" @click="nextYear">›</button>
          </view>
          
          <view class="month-grid">
            <view 
              class="month-item" 
              v-for="m in 12" 
              :key="m"
              :class="{ active: selectMonth === m }"
              @click="selectMonth = m"
            >
              {{ m }}月
            </view>
          </view>

          <view class="day-grid">
            <view class="week-item" v-for="w in weekList" :key="w">{{ w }}</view>
            <view class="day-item empty" v-for="i in firstDayEmpty" :key="'e-'+i"></view>
            <view 
              class="day-item" 
              v-for="d in monthDays" 
              :key="d"
              :class="{ active: selectDay === d, disabled: isFuture(d) }"
              @click="!isFuture(d) && (selectDay = d)"
            >
              {{ d }}
            </view>
          </view>
        </view>

        <button 
          class="start-btn" 
          :disabled="!name || !selectDay"
          @click="saveInfo"
        >
          开始记录
        </button>
      </view>
    </view>

    <!-- 已填写信息后的生命计时页 -->
    <view v-else class="home-page">
      <view class="life-card">
        <view class="life-icon">✨</view>
        <view class="age-text">
          你已经 <text class="age-num">{{ age }}</text> 岁了
        </view>
        <view class="sub-text">剩下的日子里，你大约可以</view>

        <view class="data-grid">
          <view class="data-item">
            <view class="data-icon">🍚</view>
            <text class="data-num">{{ remainEat }}</text>
            <text class="data-desc">顿饭</text>
          </view>
          <view class="data-item">
            <view class="data-icon">😴</view>
            <text class="data-num">{{ remainSleep }}</text>
            <text class="data-desc">小时觉</text>
          </view>
          <view class="data-item">
            <view class="data-icon">🎉</view>
            <text class="data-num">{{ remainWeekend }}</text>
            <text class="data-desc">个周末</text>
          </view>
          <view class="data-item">
            <view class="data-icon">🏖️</view>
            <text class="data-num">{{ remainHoliday }}</text>
            <text class="data-desc">天假期</text>
          </view>
        </view>
      </view>

      <view class="tip-card">
        <text class="tip-text">💡 点击底部「打卡」，记录今天的三餐和体重</text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      hasInfo: false,
      name: '',
      selectYear: new Date().getFullYear(),
      selectMonth: new Date().getMonth() + 1,
      selectDay: null,
      weekList: ['日', '一', '二', '三', '四', '五', '六'],
      age: '0.000000',
      remainEat: 0,
      remainSleep: 0,
      remainWeekend: 0,
      remainHoliday: 0
    }
  },
  computed: {
    monthDays() {
      return new Date(this.selectYear, this.selectMonth, 0).getDate()
    },
    firstDayEmpty() {
      return new Date(this.selectYear, this.selectMonth - 1, 1).getDay()
    },
    today() {
      const d = new Date()
      return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`
    }
  },
  onLoad() {
    this.init()
  },
  methods: {
    init() {
      const savedName = uni.getStorageSync('life_name')
      const savedBirth = uni.getStorageSync('life_birth')
      if (savedName && savedBirth) {
        this.name = savedName
        const [y, m, d] = savedBirth.split('-').map(Number)
        this.selectYear = y
        this.selectMonth = m
        this.selectDay = d
        this.hasInfo = true
        this.startTimer()
      }
    },
    prevYear() {
      this.selectYear--
      this.selectDay = null
    },
    nextYear() {
      if (this.selectYear < new Date().getFullYear()) {
        this.selectYear++
        this.selectDay = null
      }
    },
    isFuture(day) {
      const selectDate = new Date(this.selectYear, this.selectMonth - 1, day)
      return selectDate > new Date()
    },
    saveInfo() {
      if (!this.name || !this.selectDay) return
      const birthDate = `${this.selectYear}-${String(this.selectMonth).padStart(2,'0')}-${String(this.selectDay).padStart(2,'0')}`
      uni.setStorageSync('life_name', this.name)
      uni.setStorageSync('life_birth', birthDate)
      this.hasInfo = true
      this.startTimer()
      uni.showToast({ title: '欢迎开始记录', icon: 'success' })
    },
    startTimer() {
      this.updateLifeData()
      setInterval(() => {
        this.updateLifeData()
      }, 1000)
    },
    updateLifeData() {
      const birth = new Date(uni.getStorageSync('life_birth'))
      const now = new Date()
      const ageMs = now.getTime() - birth.getTime()
      const yearMs = 365.25 * 24 * 60 * 60 * 1000
      
      this.age = (ageMs / yearMs).toFixed(6)
      const aliveDays = Math.floor(ageMs / (1000 * 60 * 60 * 24))
      const remainDays = 78 * 365 - aliveDays
      
      this.remainEat = (remainDays * 3).toLocaleString()
      this.remainSleep = Math.floor(remainDays * 8 / 24).toLocaleString()
      this.remainWeekend = Math.floor(remainDays / 7 * 2).toLocaleString()
      this.remainHoliday = Math.floor(remainDays / 365 * 115).toLocaleString()
    }
  }
}
</script>

<style scoped>
.container {
  min-height: 100vh;
  background-color: #F8F9FA;
  padding-bottom: 80px;
}
/* 欢迎页 */
.welcome-page {
  padding: 40px 20px;
}
.welcome-card {
  background-color: #FFFFFF;
  border-radius: 20px;
  padding: 40px 24px;
  max-width: 500px;
  margin: 0 auto;
  box-shadow: 0 4px 20px rgba(0,0,0,0.05);
}
.welcome-icon {
  font-size: 60px;
  text-align: center;
  margin-bottom: 20px;
}
.welcome-title {
  font-size: 24px;
  font-weight: bold;
  text-align: center;
  margin-bottom: 8px;
  color: #1F2937;
}
.welcome-sub {
  font-size: 14px;
  color: #6B7280;
  text-align: center;
  margin-bottom: 30px;
}
.input {
  width: 100%;
  height: 52px;
  border: 1px solid #E5E7EB;
  border-radius: 12px;
  padding: 0 16px;
  font-size: 16px;
  margin-bottom: 24px;
  background-color: #F9FAFB;
}
/* 日期选择器 */
.date-picker {
  border: 1px solid #E5E7EB;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 30px;
  background-color: #F9FAFB;
}
.date-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}
.year-btn {
  width: 44px;
  height: 44px;
  border: none;
  background: none;
  font-size: 20px;
  color: #3B82F6;
  padding: 0;
}
.year-text {
  font-size: 18px;
  font-weight: bold;
  color: #1F2937;
}
.month-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
  margin-bottom: 16px;
}
.month-item {
  text-align: center;
  padding: 10px 0;
  border-radius: 8px;
  font-size: 14px;
  background-color: #FFFFFF;
  color: #4B5563;
}
.month-item.active {
  background-color: #3B82F6;
  color: #FFFFFF;
}
.day-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 4px;
}
.week-item {
  text-align: center;
  font-size: 12px;
  color: #9CA3AF;
  padding: 6px 0;
}
.day-item {
  aspect-ratio: 1/1;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  border-radius: 6px;
  background-color: #FFFFFF;
}
.day-item.active {
  background-color: #3B82F6;
  color: #FFFFFF;
}
.day-item.disabled {
  color: #D1D5DB;
}
.start-btn {
  width: 100%;
  height: 52px;
  background: linear-gradient(135deg, #3B82F6 0%, #60A5FA 100%);
  color: #FFFFFF;
  border: none;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 500;
}
.start-btn[disabled] {
  background: #D1D5DB;
}

/* 首页 */
.home-page {
  padding: 20px 16px;
}
.life-card {
  background-color: #FFFFFF;
  border-radius: 20px;
  padding: 30px 24px;
  margin-bottom: 16px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.05);
}
.life-icon {
  font-size: 40px;
  text-align: center;
  margin-bottom: 16px;
}
.age-text {
  font-size: 22px;
  text-align: center;
  margin-bottom: 8px;
  color: #1F2937;
}
.age-num {
  font-size: 36px;
  font-weight: bold;
  color: #3B82F6;
}
.sub-text {
  font-size: 14px;
  color: #6B7280;
  text-align: center;
  margin-bottom: 30px;
}
.data-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}
.data-item {
  text-align: center;
  padding: 16px;
  background-color: #F8FAFC;
  border-radius: 12px;
}
.data-icon {
  font-size: 24px;
  margin-bottom: 8px;
}
.data-num {
  display: block;
  font-size: 20px;
  font-weight: bold;
  color: #1F2937;
  margin-bottom: 4px;
}
.data-desc {
  font-size: 14px;
  color: #6B7280;
}
.tip-card {
  background-color: #EFF6FF;
  border-radius: 12px;
  padding: 16px;
  text-align: center;
}
.tip-text {
  font-size: 14px;
  color: #1E40AF;
}
</style>