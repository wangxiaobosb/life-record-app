<template>
  <view class="container">
    <!-- 未填写信息提示 -->
    <view v-if="!hasInfo" class="empty-page">
      <text class="empty-text">请先在首页完善个人信息</text>
    </view>

    <view v-else>
      <!-- 三餐打卡 -->
      <view class="card">
        <view class="card-title">🍽️ 今日三餐</view>
        
        <view class="meal-item" v-for="meal in meals" :key="meal.type">
          <view class="meal-header">
            <text class="meal-name">{{ meal.name }}</text>
            <button 
              :class="todayCheck[meal.type] ? 'done-btn' : 'check-btn'"
              @click="openMealModal(meal.type)"
            >
              {{ todayCheck[meal.type] ? '已记录' : '记录' }}
            </button>
          </view>
          <view class="meal-content" v-if="todayCheck[meal.type]">
            <text class="content-text">{{ todayMealContent[meal.type] || '已打卡' }}</text>
            <text class="time-text">{{ todayCheckTime[meal.type] }}</text>
          </view>
        </view>
      </view>

      <!-- 体重记录 -->
      <view class="card">
        <view class="card-title">⚖️ 今日体重</view>
        <view class="weight-input">
          <input 
            class="weight-input-field" 
            v-model="todayWeight" 
            placeholder="请输入今日体重（kg）"
            type="digit"
            maxlength="5"
          />
          <button class="save-weight-btn" @click="saveWeight">保存</button>
        </view>
        <view class="weight-hint" v-if="todayWeightRecord">
          <text class="hint-text">今日已记录：{{ todayWeightRecord }} kg，记录时间：{{ todayWeightTime }}</text>
        </view>
      </view>

      <!-- 今日日记 -->
      <view class="card">
        <view class="card-title">📝 今日日记</view>
        <textarea 
          class="diary-input" 
          v-model="todayDiary" 
          placeholder="记录今天发生的事、心情、感悟..."
          maxlength="1000"
        ></textarea>
        <button class="save-diary-btn" @click="saveDiary">保存日记</button>
        <view class="diary-hint" v-if="todayDiarySaved">
          <text class="hint-text">日记已保存，最后修改时间：{{ todayDiaryUpdateTime }}</text>
        </view>
      </view>
    </view>

    <!-- 三餐内容弹窗 -->
    <uni-popup ref="mealPopup" type="bottom">
      <view class="popup-content">
        <view class="popup-title">记录{{ currentMealName }}</view>
        <textarea 
          class="popup-input" 
          v-model="mealContent" 
          placeholder="记录一下吃了什么（可选）"
          maxlength="100"
        ></textarea>
        <view class="popup-buttons">
          <button class="cancel-btn" @click="closeMealModal">取消</button>
          <button class="confirm-btn" @click="confirmMeal">确认打卡</button>
        </view>
      </view>
    </uni-popup>
  </view>
</template>

<script>
export default {
  data() {
    return {
      hasInfo: false,
      meals: [
        { type: 'breakfast', name: '早餐' },
        { type: 'lunch', name: '午餐' },
        { type: 'dinner', name: '晚餐' }
      ],
      todayCheck: { breakfast: false, lunch: false, dinner: false },
      todayMealContent: { breakfast: '', lunch: '', dinner: '' },
      todayCheckTime: { breakfast: '', lunch: '', dinner: '' },
      todayWeight: '',
      todayWeightRecord: '',
      todayWeightTime: '',
      todayDiary: '',
      todayDiarySaved: false,
      todayDiaryUpdateTime: '',
      currentMealType: '',
      currentMealName: '',
      mealContent: ''
    }
  },
  computed: {
    today() {
      const d = new Date()
      return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`
    }
  },
  onShow() {
    this.init()
  },
  methods: {
    init() {
      const savedName = uni.getStorageSync('life_name')
      this.hasInfo = !!savedName
      if (!this.hasInfo) return
      this.loadTodayData()
    },
    loadTodayData() {
      // 加载三餐记录
      const allMeals = uni.getStorageSync('life_all_meals') || {}
      const todayMeals = allMeals[this.today] || {
        check: { breakfast: false, lunch: false, dinner: false },
        content: { breakfast: '', lunch: '', dinner: '' },
        time: { breakfast: '', lunch: '', dinner: '' }
      }
      this.todayCheck = todayMeals.check
      this.todayMealContent = todayMeals.content
      this.todayCheckTime = todayMeals.time

      // 加载体重记录
      const allWeight = uni.getStorageSync('life_all_weight') || {}
      if (allWeight[this.today]) {
        this.todayWeightRecord = allWeight[this.today].value
        this.todayWeightTime = allWeight[this.today].time
        this.todayWeight = this.todayWeightRecord
      } else {
        this.todayWeightRecord = ''
        this.todayWeightTime = ''
        this.todayWeight = ''
      }

      // 加载日记记录
      const allDiary = uni.getStorageSync('life_all_diary') || {}
      if (allDiary[this.today]) {
        this.todayDiary = allDiary[this.today].content
        this.todayDiarySaved = true
        this.todayDiaryUpdateTime = allDiary[this.today].updateTime
      } else {
        this.todayDiary = ''
        this.todayDiarySaved = false
        this.todayDiaryUpdateTime = ''
      }
    },
    // 打开三餐记录弹窗
    openMealModal(type) {
      this.currentMealType = type
      this.currentMealName = this.meals.find(m => m.type === type).name
      this.mealContent = this.todayMealContent[type] || ''
      this.$refs.mealPopup.open()
    },
    closeMealModal() {
      this.$refs.mealPopup.close()
      this.mealContent = ''
    },
    // 确认三餐打卡
    confirmMeal() {
      const now = new Date()
      const timeStr = `${now.getHours().toString().padStart(2,'0')}:${now.getMinutes().toString().padStart(2,'0')}`
      
      this.todayCheck[this.currentMealType] = true
      this.todayMealContent[this.currentMealType] = this.mealContent.trim()
      this.todayCheckTime[this.currentMealType] = timeStr

      // 保存到本地
      const allMeals = uni.getStorageSync('life_all_meals') || {}
      allMeals[this.today] = {
        check: this.todayCheck,
        content: this.todayMealContent,
        time: this.todayCheckTime
      }
      uni.setStorageSync('life_all_meals', allMeals)

      this.closeMealModal()
      uni.showToast({ title: '记录成功', icon: 'success' })
    },
    // 保存体重
    saveWeight() {
      if (!this.todayWeight.trim()) {
        uni.showToast({ title: '请输入体重', icon: 'none' })
        return
      }
      const now = new Date()
      const timeStr = `${now.getHours().toString().padStart(2,'0')}:${now.getMinutes().toString().padStart(2,'0')}`
      
      this.todayWeightRecord = this.todayWeight.trim()
      this.todayWeightTime = timeStr

      // 保存到本地
      const allWeight = uni.getStorageSync('life_all_weight') || {}
      allWeight[this.today] = {
        value: this.todayWeightRecord,
        time: this.todayWeightTime
      }
      uni.setStorageSync('life_all_weight', allWeight)

      uni.showToast({ title: '体重保存成功', icon: 'success' })
    },
    // 保存日记
    saveDiary() {
      const now = new Date()
      const timeStr = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')} ${now.getHours().toString().padStart(2,'0')}:${now.getMinutes().toString().padStart(2,'0')}`
      
      this.todayDiarySaved = true
      this.todayDiaryUpdateTime = timeStr

      // 保存到本地
      const allDiary = uni.getStorageSync('life_all_diary') || {}
      allDiary[this.today] = {
        content: this.todayDiary.trim(),
        createTime: allDiary[this.today]?.createTime || timeStr,
        updateTime: timeStr
      }
      uni.setStorageSync('life_all_diary', allDiary)

      uni.showToast({ title: '日记保存成功', icon: 'success' })
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
.empty-page {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 60vh;
}
.empty-text {
  font-size: 16px;
  color: #6B7280;
}
.card {
  background-color: #FFFFFF;
  border-radius: 16px;
  padding: 24px;
  margin-bottom: 16px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.05);
}
.card-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 20px;
  color: #1F2937;
}
/* 三餐打卡 */
.meal-item {
  padding: 16px 0;
  border-bottom: 1px solid #F3F4F6;
}
.meal-item:last-child {
  border-bottom: none;
  padding-bottom: 0;
}
.meal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}
.meal-name {
  font-size: 16px;
  font-weight: 500;
  color: #1F2937;
}
.check-btn {
  background-color: #3B82F6;
  color: #FFFFFF;
  border: none;
  border-radius: 8px;
  padding: 8px 16px;
  font-size: 14px;
}
.done-btn {
  background-color: #10B981;
  color: #FFFFFF;
  border: none;
  border-radius: 8px;
  padding: 8px 16px;
  font-size: 14px;
}
.meal-content {
  padding-left: 4px;
}
.content-text {
  display: block;
  font-size: 14px;
  color: #4B5563;
  margin-bottom: 4px;
}
.time-text {
  font-size: 12px;
  color: #9CA3AF;
}
/* 体重记录 */
.weight-input {
  display: flex;
  gap: 12px;
  margin-bottom: 12px;
}
.weight-input-field {
  flex: 1;
  height: 48px;
  border: 1px solid #E5E7EB;
  border-radius: 8px;
  padding: 0 16px;
  font-size: 16px;
  background-color: #F9FAFB;
}
.save-weight-btn {
  width: 80px;
  height: 48px;
  background-color: #3B82F6;
  color: #FFFFFF;
  border: none;
  border-radius: 8px;
  font-size: 14px;
}
.weight-hint {
  padding: 12px;
  background-color: #ECFDF5;
  border-radius: 8px;
}
.hint-text {
  font-size: 14px;
  color: #065F46;
}
/* 日记 */
.diary-input {
  width: 100%;
  min-height: 150px;
  border: 1px solid #E5E7EB;
  border-radius: 8px;
  padding: 16px;
  font-size: 16px;
  line-height: 1.6;
  background-color: #F9FAFB;
  margin-bottom: 12px;
}
.save-diary-btn {
  width: 100%;
  height: 48px;
  background-color: #3B82F6;
  color: #FFFFFF;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  margin-bottom: 12px;
}
.diary-hint {
  padding: 12px;
  background-color: #EFF6FF;
  border-radius: 8px;
}
/* 弹窗 */
.popup-content {
  background-color: #FFFFFF;
  border-radius: 16px 16px 0 0;
  padding: 24px;
  padding-bottom: 40px;
}
.popup-title {
  font-size: 18px;
  font-weight: bold;
  text-align: center;
  margin-bottom: 20px;
  color: #1F2937;
}
.popup-input {
  width: 100%;
  min-height: 100px;
  border: 1px solid #E5E7EB;
  border-radius: 8px;
  padding: 16px;
  font-size: 16px;
  background-color: #F9FAFB;
  margin-bottom: 20px;
}
.popup-buttons {
  display: flex;
  gap: 12px;
}
.cancel-btn {
  flex: 1;
  height: 48px;
  background-color: #F3F4F6;
  color: #4B5563;
  border: none;
  border-radius: 8px;
  font-size: 16px;
}
.confirm-btn {
  flex: 1;
  height: 48px;
  background-color: #3B82F6;
  color: #FFFFFF;
  border: none;
  border-radius: 8px;
  font-size: 16px;
}
</style>