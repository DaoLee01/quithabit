<template>
  <view class="debug-container">
    <view class="status-bar">
      <text class="status-text">🛡️ 沙盒模式已开启</text>
      <text class="status-sub">当前操作不会丢失真实数据 (已备份)</text>
    </view>

    <view class="control-panel">
      <text class="panel-title">第一步：设定天数</text>
      <input class="day-input" type="number" v-model="debugDays" placeholder="输入模拟天数" />
      
      <text class="panel-title">第二步：选择场景</text>
      <view class="grid-box">
        <button class="action-btn success" @click="applyData('quit', false)">
          ✨ 戒除：连胜 {{ debugDays }} 天
        </button>
        <button class="action-btn warning" @click="applyData('quit', true)">
          💔 戒除：断签 (原{{ debugDays }}天)
        </button>
        <button class="action-btn info" @click="applyData('fitness', false)">
          💪 健身：累计 {{ debugDays }} 天
        </button>
        <button class="action-btn normal" @click="simulateNewUser">
          👶 模拟新用户 (清空)
        </button>
      </view>
    </view>

    <view class="safe-exit">
      <button class="restore-btn" @click="restoreAndExit">♻️ 还原真实数据并退出</button>
      <text class="tip-text">注意：如果不点击还原直接杀后台，下次启动需手动还原数据</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      debugDays: 30,
      dataKeys: [
        'habitRecord', 'quitRecord', 'fitnessRecord', 'readingRecord', 
        'wordsRecord', 'struggleRecords', 'struggleHistory', 'readingHistory'
      ]
    }
  },
  
  onLoad() {
    this.backupRealData()
  },
  
  methods: {
    // 1. 进页面自动备份
    backupRealData() {
      // 检查是否已经有备份（防止重复备份导致备份了假数据）
      const hasBackup = uni.getStorageSync('__DEBUG_HAS_BACKUP__')
      if (hasBackup) return 

      this.dataKeys.forEach(key => {
        const data = uni.getStorageSync(key)
        if (data) {
          uni.setStorageSync('backup_' + key, data)
        }
      })
      uni.setStorageSync('__DEBUG_HAS_BACKUP__', true)
      console.log('真实数据已备份')
    },

    // 2. 还原数据
    restoreAndExit() {
      try {
        this.dataKeys.forEach(key => {
          const backupData = uni.getStorageSync('backup_' + key)
          if (backupData) {
            uni.setStorageSync(key, backupData)
            uni.removeStorageSync('backup_' + key) // 还原后清除备份
          } else {
            // 如果原本没数据，还原时应该清除现有的假数据
            uni.removeStorageSync(key)
          }
        })
        uni.removeStorageSync('__DEBUG_HAS_BACKUP__')
        
        uni.showToast({ title: '数据已还原', icon: 'success' })
        setTimeout(() => {
          uni.navigateBack()
        }, 1500)
      } catch (e) {
        uni.showToast({ title: '还原失败，请勿操作', icon: 'none' })
      }
    },

    // 3. 生成测试数据 (和之前逻辑一致)
    applyData(habitType, isBroken) {
      const days = parseInt(this.debugDays)
      if (isNaN(days) || days < 0) return
      
      this.generateFakeData(habitType, days, isBroken)
      
      uni.showToast({ title: '数据已生成，请去首页查看', icon: 'none' })
      // 这里不自动跳回首页，允许用户连续生成不同数据
      // 用户可以手动切到底部Tab查看，或者我们提供一个查看按钮
      setTimeout(() => {
          uni.switchTab({ url: '/pages/index/index' })
      }, 1000)
    },

    simulateNewUser() {
      this.dataKeys.forEach(key => uni.removeStorageSync(key))
      uni.showToast({ title: '已重置为空白状态', icon: 'none' })
      setTimeout(() => {
          uni.switchTab({ url: '/pages/index/index' })
      }, 1000)
    },

    generateFakeData(habitType, days, isBroken) {
      const history = []
      const today = new Date()
      const lastSuccessDate = new Date(today)
      if (isBroken) lastSuccessDate.setDate(today.getDate() - 2) 
      
      for (let i = 0; i < days; i++) {
        const d = new Date(lastSuccessDate)
        d.setDate(lastSuccessDate.getDate() - i)
        history.push(d.toDateString())
      }
      
      const startDate = new Date(lastSuccessDate)
      startDate.setDate(lastSuccessDate.getDate() - (days - 1))

      const record = {
        streakDays: days,
        totalDays: days,
        monthSuccess: Math.min(days, today.getDate()),
        bestStreak: days,
        lastSuccess: lastSuccessDate.toDateString(),
        history: history,
        startDate: startDate.toDateString()
      }
      uni.setStorageSync(`${habitType}Record`, record)
    }
  }
}
</script>

<style>
.debug-container { padding: 20px; min-height: 100vh; background: #1a1a1a; color: #fff; }
.status-bar { background: #333; padding: 15px; border-radius: 8px; margin-bottom: 30px; border-left: 5px solid #4CD964; }
.status-text { font-size: 16px; font-weight: bold; display: block; margin-bottom: 5px; color: #4CD964; }
.status-sub { font-size: 12px; color: #aaa; }

.control-panel { background: #2c2c2c; padding: 20px; border-radius: 12px; margin-bottom: 30px; }
.panel-title { font-size: 14px; color: #ccc; margin-bottom: 10px; display: block; }
.day-input { background: #444; color: white; padding: 10px; border-radius: 6px; margin-bottom: 20px; text-align: center; font-size: 18px; font-weight: bold; }

.grid-box { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
.action-btn { font-size: 12px; padding: 15px 0; border-radius: 8px; border: none; color: white; width: 100%; }
.success { background: #4CD964; color: #000; }
.warning { background: #FF9500; color: #000; }
.info { background: #007AFF; }
.normal { background: #666; }

.safe-exit { margin-top: 20px; text-align: center; }
.restore-btn { background: #FF3B30; color: white; padding: 15px; border-radius: 50px; font-weight: bold; width: 100%; margin-bottom: 10px; }
.tip-text { font-size: 10px; color: #666; }
</style>