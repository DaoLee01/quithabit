<template>
  <view class="stats-container">
    <view class="stats-header">
      <text class="stats-title">数据统计</text>
    </view>
    
    <view class="habit-selector">
      <view 
        v-for="habit in habits" 
        :key="habit.id"
        :class="['habit-option', selectedHabit === habit.id ? 'habit-option-active' : '']"
        @click="selectHabit(habit.id)"
      >
        {{ habit.name }}
      </view>
    </view>
    
    <view class="time-range-selector">
      <view 
        v-for="range in timeRanges" 
        :key="range.id"
        :class="['time-range-option', selectedTimeRange === range.id ? 'time-range-option-active' : '']"
        @click="selectTimeRange(range.id)"
      >
        {{ range.name }}
      </view>
    </view>
    
    <view class="overview-cards">
      <view class="overview-card">
        <text class="overview-value">{{ currentStats.successCount }}</text>
        <text class="overview-label">{{ selectedHabit === 'quit' ? '当前连续成功' : '累计成功天数' }}</text>
      </view>
      <view class="overview-card">
        <text class="overview-value">{{ currentStats.daysTotal }}</text>
        <text class="overview-label">挑战/经过天数</text>
      </view>
      <view class="overview-card">
        <text class="overview-value">{{ currentStats.successRate }}%</text>
        <text class="overview-label">当前完成率</text>
      </view>
      <view class="overview-card">
        <text class="overview-value">{{ currentStats.bestStreak }}</text>
        <text class="overview-label">历史最佳连胜</text>
      </view>
    </view>
    
    <view class="chart-section">
      <text class="section-title">
        {{ timeRanges.find(r => r.id === selectedTimeRange).name }}趋势图
      </text>
      
      <!-- 1. 月度图表 -->
      <view v-if="selectedTimeRange === 'month'" class="chart-container">
        <view v-for="day in monthData" :key="day.day" class="chart-day-sm">
          <view 
            :class="['chart-bar', day.success ? 'chart-success' : 'chart-empty']"
            :style="{ height: day.success ? '40px' : '10px' }"
          ></view>
          <text class="chart-label" v-if="day.day % 5 === 0 || day.day === 1">{{ day.day }}</text>
        </view>
      </view>

      <!-- 2. 季度图表 -->
      <view v-if="selectedTimeRange === 'quarter'" class="chart-container-wide">
        <view v-for="m in quarterData" :key="m.label" class="chart-col-wide">
           <text class="bar-value">{{ m.rate }}%</text>
           <view class="bar-bg"><view class="bar-fill" :style="{ height: m.rate + '%' }"></view></view>
           <text class="chart-label">{{ m.label }}</text>
        </view>
      </view>

      <!-- 3. 年度图表 -->
      <view v-if="selectedTimeRange === 'year'" class="chart-container">
        <view v-for="m in yearData" :key="m.label" class="chart-day-sm">
           <view class="bar-bg-sm"><view class="bar-fill" :style="{ height: m.rate + '%' }"></view></view>
           <text class="chart-label">{{ m.label }}</text>
        </view>
      </view>

      <!-- 4. 全部图表 -->
      <view v-if="selectedTimeRange === 'total'" class="chart-container-wide">
        <view v-for="y in totalData" :key="y.label" class="chart-col-wide">
           <text class="bar-value">{{ y.rate }}%</text>
           <view class="bar-bg"><view class="bar-fill" :style="{ height: y.rate + '%' }"></view></view>
           <text class="chart-label">{{ y.label }}</text>
        </view>
      </view>

      <view class="stats-feedback">
        <text class="feedback-text">{{ getFeedbackText() }}</text>
      </view>
    </view>
    
    <view class="detailed-stats">
      <text class="section-title">详细记录</text>
      <view class="stat-item">
        <text class="stat-label">开始记录日期:</text>
        <text class="stat-value">{{ globalStartDate || '暂无' }}</text>
      </view>
      <view class="stat-item">
        <text class="stat-label">当前状态:</text>
        <text class="stat-value">{{ currentStreak }} 天 {{ selectedHabit === 'quit' ? '连胜' : '累计' }}</text>
      </view>
      <view class="stat-item" v-if="lastRecordInfo">
        <text class="stat-label">最近记录:</text>
        <text class="stat-value">{{ lastRecordInfo }}</text>
      </view>
    </view>
    
  </view>
</template>

<script>
export default {
  data() {
    return {
      selectedHabit: 'quit',
      selectedTimeRange: 'month',
      habits: [
        { id: 'quit', name: '戒除' },
        { id: 'fitness', name: '健身' },
        { id: 'reading', name: '读书' },
        { id: 'words', name: '单词' },
        { id: 'custom', name: '自定义' }
      ],
      customHabitName: '自定义',
      
      timeRanges: [
        { id: 'month', name: '本月' },
        { id: 'quarter', name: '本季' },
        { id: 'year', name: '本年' },
        { id: 'total', name: '全部' }
      ],
      currentStats: { successCount: 0, daysTotal: 0, successRate: 0, bestStreak: 0 },
      globalStartDate: '',
      currentStreak: 0,
      lastRecordInfo: '',
      
      validHistoryArray: [], 
      
      monthData: [], quarterData: [], yearData: [], totalData: []
    }
  },
  
  onLoad() { this.loadSettings(); this.loadStats() },
  onShow() { this.loadSettings(); this.loadStats() },
  
  methods: {
    loadSettings() {
        const settings = uni.getStorageSync('appSettings') || {}
        this.customHabitName = settings.customHabitName || '自定义'
    },
    
    selectHabit(habitId) { this.selectedHabit = habitId; this.loadHabitStats(habitId) },
    selectTimeRange(rangeId) { this.selectedTimeRange = rangeId; this.calculateRangeStats() },
    loadStats() { this.loadHabitStats(this.selectedHabit) },
    
    loadHabitStats(habitType) {
      try {
        const record = uni.getStorageSync(`${habitType}Record`) || {}
        const rawHistory = record.history || []
        
        // 1. 数据清洗 + 【核心修复：去重】
        // Set 去重是解决 "11月29日打了2次" 导致统计虚高的关键
        const now = new Date()
        const bufferTime = now.getTime() + 86400000 
        const uniqueHistory = [...new Set(rawHistory)] // 去重
        
        this.validHistoryArray = uniqueHistory.filter(d => new Date(d).getTime() <= bufferTime)
        
        // 2. 基础数据重算
        let streak = 0
        
        if (habitType === 'quit') {
            // 严苛模式：倒推连胜
            const sorted = [...this.validHistoryArray].sort((a,b) => new Date(b) - new Date(a))
            if (sorted.length > 0) {
                const lastDate = new Date(sorted[0]); lastDate.setHours(0,0,0,0)
                const today = new Date(); today.setHours(0,0,0,0)
                const diffDays = Math.floor((today - lastDate) / (1000 * 60 * 60 * 24))
                
                if (diffDays <= 1) {
                    streak = 1
                    for (let i = 1; i < sorted.length; i++) {
                        const p = new Date(sorted[i-1]); p.setHours(0,0,0,0)
                        const c = new Date(sorted[i]); c.setHours(0,0,0,0)
                        if (Math.floor((p-c)/(1000*60*60*24)) === 1) streak++
                        else break
                    }
                }
            }
            this.currentStreak = streak
        } else {
            // 宽容模式：累计天数
            this.currentStreak = this.validHistoryArray.length
        }
        
        this.currentStats.bestStreak = record.bestStreak || 0
        if (record.startDate) this.globalStartDate = record.startDate
        
        if (habitType === 'fitness' && record.lastType) this.lastRecordInfo = record.lastType
        else if (habitType === 'reading' && record.lastContent) this.lastRecordInfo = record.lastContent
        else if (habitType === 'words' && record.lastCount) this.lastRecordInfo = `${record.lastCount}个`
        else this.lastRecordInfo = record.lastSuccess || '无'
        
        this.generateChartData()
        this.calculateRangeStats()
        
      } catch (e) { console.error(e) }
    },
    
    calculateRangeStats() {
      const now = new Date(); now.setHours(0,0,0,0)
      const currentYear = now.getFullYear()
      const currentMonth = now.getMonth()
      
      let successCount = 0
      let daysTotal = 1 
      
      // 1. 【戒除类】严苛模式
      if (this.selectedHabit === 'quit') {
        if (this.currentStreak === 0) {
            this.currentStats.successCount = 0; this.currentStats.daysTotal = 1; this.currentStats.successRate = 0; return
        }
        
        const streakStartDate = new Date(now)
        streakStartDate.setDate(now.getDate() - (this.currentStreak - 1))
        
        let windowStart = null
        let windowEnd = new Date(now)
        
        if (this.selectedTimeRange === 'month') windowStart = new Date(currentYear, currentMonth, 1)
        else if (this.selectedTimeRange === 'quarter') { const q = Math.floor(currentMonth / 3); windowStart = new Date(currentYear, q * 3, 1) }
        else if (this.selectedTimeRange === 'year') windowStart = new Date(currentYear, 0, 1)
        else windowStart = new Date(this.globalStartDate || now)
        
        let effectiveStart = streakStartDate > windowStart ? streakStartDate : windowStart
        const diffTime = windowEnd - effectiveStart
        daysTotal = Math.floor(diffTime / (1000 * 60 * 60 * 24)) + 1
        if (daysTotal < 1) daysTotal = 1
        
        successCount = this.validHistoryArray.filter(d => {
            const date = new Date(d); date.setHours(0,0,0,0)
            return date >= effectiveStart && date <= windowEnd
        }).length
        
      } 
      // 2. 【其他类】宽容模式
      else {
        const userStartDate = this.globalStartDate ? new Date(this.globalStartDate) : new Date()
        let effectiveStart = null
        let windowEnd = new Date(now)
        
        if (this.selectedTimeRange === 'month') effectiveStart = new Date(currentYear, currentMonth, 1)
        else if (this.selectedTimeRange === 'quarter') { const q = Math.floor(currentMonth / 3); effectiveStart = new Date(currentYear, q * 3, 1) }
        else if (this.selectedTimeRange === 'year') effectiveStart = new Date(currentYear, 0, 1)
        else effectiveStart = userStartDate
        
        if (userStartDate > effectiveStart) effectiveStart = userStartDate
        
        const diffTime = windowEnd - effectiveStart
        daysTotal = Math.floor(diffTime / (1000 * 60 * 60 * 24)) + 1
        if (daysTotal < 1) daysTotal = 1
        
        successCount = this.validHistoryArray.filter(d => {
            const date = new Date(d); date.setHours(0,0,0,0)
            return date >= effectiveStart && date <= windowEnd
        }).length
      }
      
      this.currentStats.successCount = successCount
      this.currentStats.daysTotal = daysTotal
      
      if (daysTotal > 0) {
        let rate = Math.round((successCount / daysTotal) * 100)
        if (rate > 100) rate = 100
        this.currentStats.successRate = rate
      } else {
        this.currentStats.successRate = 0
      }
    },
    
    generateChartData() {
      const now = new Date()
      const currentYear = now.getFullYear()
      const currentMonth = now.getMonth()
      
      let strictStreakDates = []
      if (this.selectedHabit === 'quit' && this.currentStreak > 0) {
          for(let i=0; i<this.currentStreak; i++) {
              const d = new Date(); d.setDate(d.getDate() - i)
              strictStreakDates.push(d.toDateString())
          }
      }
      
      const isSuccess = (dateString) => {
          if (this.selectedHabit === 'quit') {
              return this.validHistoryArray.includes(dateString) && strictStreakDates.includes(dateString)
          } else {
              return this.validHistoryArray.includes(dateString)
          }
      }
      
      // 1. 月
      const daysInMonth = new Date(currentYear, currentMonth + 1, 0).getDate()
      this.monthData = []
      for (let day = 1; day <= daysInMonth; day++) {
        const dateString = new Date(currentYear, currentMonth, day).toDateString()
        this.monthData.push({ day: day, success: isSuccess(dateString) })
      }
      
      // 2. 季 (简化算法，仅展示比例)
      this.quarterData = []
      const currentQuarter = Math.floor(currentMonth / 3)
      for (let i = 0; i < 3; i++) {
        const m = currentQuarter * 3 + i
        const totalDays = new Date(currentYear, m + 1, 0).getDate()
        let successDays = 0
        for(let d=1; d<=totalDays; d++) {
            if (isSuccess(new Date(currentYear, m, d).toDateString())) successDays++
        }
        const rate = Math.round((successDays / totalDays) * 100)
        this.quarterData.push({ label: `${m+1}月`, rate: rate })
      }
      
      // 3. 年
      this.yearData = []
      for (let i = 0; i < 12; i++) {
        const totalDays = new Date(currentYear, i + 1, 0).getDate()
        let successDays = 0
        for(let d=1; d<=totalDays; d++) {
            if (isSuccess(new Date(currentYear, i, d).toDateString())) successDays++
        }
        const rate = Math.round((successDays / totalDays) * 100)
        this.yearData.push({ label: `${i+1}`, rate: rate })
      }
      
      // 4. 全部
      this.totalData = []
      const startYear = 2025; const targetYears = [startYear, startYear + 1, startYear + 2] 
      targetYears.forEach(y => {
        const daysTotal = (y % 4 === 0 && y % 100 !== 0) || (y % 400 === 0) ? 366 : 365; let successDays = 0
        const sourceArray = this.selectedHabit === 'quit' ? strictStreakDates : this.validHistoryArray
        successDays = sourceArray.filter(d => new Date(d).getFullYear() === y).length
        const rate = Math.round((successDays / daysTotal) * 100)
        this.totalData.push({ label: `${y}`, rate: rate })
      })
    },
    
    getFeedbackText() {
      const rate = this.currentStats.successRate
      if (this.selectedHabit === 'quit' && this.currentStreak === 0) return '重新开始，这次一定行！'
      if (rate >= 90) return '太棒了！你的执行力惊人！'
      if (rate >= 70) return '做得不错，继续保持！'
      if (rate >= 40) return '还可以，但仍有提升空间。'
      return '万事开头难，别灰心，从今天开始坚持！'
    }
  }
}
</script>

<style>
/* 样式保持不变 */
.stats-container { padding: 20px; min-height: 100vh; background: #f5f5f5; padding-bottom: 20px; }
.stats-header { text-align: center; margin-bottom: 20px; }
.stats-title { font-size: 24px; font-weight: bold; color: #333; }
.habit-selector, .time-range-selector { display: flex; background: white; border-radius: 12px; padding: 4px; margin-bottom: 15px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
.habit-option, .time-range-option { flex: 1; text-align: center; padding: 10px; border-radius: 8px; font-size: 14px; transition: all 0.3s ease; }
.habit-option-active { background: #007AFF; color: white; }
.time-range-option-active { background: #4CD964; color: white; }
.overview-cards { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 20px; }
.overview-card { background: white; padding: 15px; border-radius: 12px; text-align: center; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
.overview-value { font-size: 24px; font-weight: bold; color: #007AFF; display: block; }
.overview-label { font-size: 12px; color: #666; display: block; margin-top: 5px; }
.chart-section { background: white; padding: 20px; border-radius: 12px; margin-bottom: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
.section-title { font-size: 18px; font-weight: bold; margin-bottom: 15px; display: block; }
.stats-feedback { background: #f0f9ff; padding: 10px; border-radius: 8px; text-align: center; margin-top: 15px; }
.feedback-text { color: #007AFF; font-size: 14px; font-weight: bold; }
.chart-container { display: flex; justify-content: space-between; align-items: end; height: 100px; padding-bottom: 10px; }
.chart-day-sm { display: flex; flex-direction: column; align-items: center; flex: 1; margin: 0 1px; }
.chart-bar { width: 6px; border-radius: 3px 3px 0 0; margin-bottom: 5px; transition: all 0.3s ease; }
.chart-success { background: #4CD964; }
.chart-empty { background: #f0f0f0; }
.bar-bg-sm { width: 6px; height: 80px; background: #f0f0f0; border-radius: 3px; display: flex; align-items: end; margin-bottom: 5px; }
.chart-container-wide { display: flex; justify-content: space-around; align-items: end; height: 120px; }
.chart-col-wide { display: flex; flex-direction: column; align-items: center; width: 40px; }
.bar-bg { width: 20px; height: 80px; background: #f0f0f0; border-radius: 10px; display: flex; align-items: end; margin: 5px 0; overflow: hidden; }
.bar-fill { width: 100%; background: #007AFF; transition: height 0.5s ease; }
.bar-value { font-size: 10px; color: #007AFF; font-weight: bold; }
.chart-label { font-size: 10px; color: #666; }
.detailed-stats { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
.stat-item { display: flex; justify-content: space-between; align-items: center; padding: 12px 0; border-bottom: 1px solid #f0f0f0; }
.stat-item:last-child { border-bottom: none; }
.stat-item .stat-label { font-size: 14px; color: #666; }
.stat-item .stat-value { font-size: 14px; color: #333; font-weight: bold; }
</style>