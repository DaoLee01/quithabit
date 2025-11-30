<template>
  <view class="settings-container">
    <!-- 1. 彩蛋埋点：绑定点击事件到标题栏 -->
    <view class="settings-header" @click="handleHeaderClick">
      <text class="settings-title">设置</text>
    </view>
    
    <view class="settings-list">
      <view class="setting-item">
        <text class="setting-label">自定义习惯名称</text>
        <input class="setting-input" v-model="customHabitName" @blur="saveCustomName" placeholder="例如：练琴" />
      </view>

      <view class="setting-item" @click="navigateToReminderSettings">
        <text class="setting-label">通知提醒设置</text>
        <text class="setting-arrow">></text>
      </view>
      <view class="setting-item">
        <text class="setting-label">开启所有提醒</text>
        <switch :checked="dailyReminder" @change="toggleReminder" />
      </view>
      <view class="setting-item">
        <text class="setting-label">显示激励语录</text>
        <switch :checked="showMotivation" @change="toggleMotivation" />
      </view>
      <view class="setting-item" @click="showResetConfirm">
        <text class="setting-label">重置数据</text>
        <text class="setting-arrow">></text>
      </view>
      
      <!-- 2. 关于应用 -->
      <view class="setting-item" @click="showAbout">
        <text class="setting-label">关于应用</text>
        <text class="setting-arrow">></text>
      </view>
    </view>
    
    <!-- 3. 数据备份与恢复 -->
    <view class="backup-section">
      <button class="backup-btn" @click="exportData">导出数据</button>
      <button class="restore-btn" @click="importData">导入数据</button>
    </view>
    
    <!-- 密码输入弹窗 -->
    <view v-if="showPasswordModal" class="modal-mask">
      <view class="modal-content">
        <text class="modal-title">🔐 开发者模式</text>
        <input 
          password 
          type="number" 
          class="password-input" 
          v-model="inputPassword" 
          placeholder="请输入通行码" 
          maxlength="4"
        />
        <view class="modal-buttons">
          <button class="modal-cancel" @click="showPasswordModal = false">取消</button>
          <button class="modal-confirm" @click="checkPassword">进入</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      dailyReminder: false,
      showMotivation: true,
      customHabitName: '',
      
      // 彩蛋相关变量
      clickCount: 0,
      lastClickTime: 0,
      showPasswordModal: false,
      inputPassword: '',
      SECRET_CODE: '2025' // 密码
    }
  },
  onLoad() { this.loadSettings() },
  methods: {
    loadSettings() {
      try {
        const settings = uni.getStorageSync('appSettings') || {}
        this.dailyReminder = settings.dailyReminder || false
        this.showMotivation = settings.showMotivation !== false
        this.customHabitName = settings.customHabitName || '自律打卡'
      } catch (e) { console.error(e) }
    },
    saveSettings() {
      try {
        const settings = {
          dailyReminder: this.dailyReminder,
          showMotivation: this.showMotivation,
          customHabitName: this.customHabitName || '自律打卡'
        }
        uni.setStorageSync('appSettings', settings)
      } catch (e) { console.error(e) }
    },
    saveCustomName() { this.saveSettings(); uni.showToast({ title: '名称已修改', icon: 'none' }) },
    toggleReminder(e) {
      this.dailyReminder = e.detail.value; this.saveSettings()
      if(this.dailyReminder) uni.showToast({ title: '已开启提醒功能', icon: 'success' })
      else uni.showToast({ title: '已暂停所有提醒', icon: 'none' })
    },
    toggleMotivation(e) { this.showMotivation = e.detail.value; this.saveSettings() },
    navigateToReminderSettings() { uni.navigateTo({ url: '/pages/reminder-settings/reminder-settings' }) },
    
    showResetConfirm() {
      uni.showModal({
        title: '重置数据',
        content: '确定要重置所有数据吗？此操作不可撤销。',
        confirmColor: '#FF3B30',
        success: (res) => { if(res.confirm) this.resetData() }
      })
    },
    
    resetData() {
      try {
        // 清空所有业务数据
        uni.removeStorageSync('habitRecord'); uni.removeStorageSync('quitRecord')
        uni.removeStorageSync('fitnessRecord'); uni.removeStorageSync('readingRecord')
        uni.removeStorageSync('wordsRecord'); uni.removeStorageSync('customRecord')
        uni.removeStorageSync('struggleRecords'); uni.removeStorageSync('struggleHistory')
        uni.removeStorageSync('readingHistory'); uni.removeStorageSync('wordHistory')
        uni.removeStorageSync('userCustomBooks'); uni.removeStorageSync('reminderSettings')
        uni.removeStorageSync('lastRecordDate'); uni.removeStorageSync('appSettings')
        
        uni.showToast({ title: '数据已重置', icon: 'success' })
        setTimeout(() => { uni.reLaunch({ url: '/pages/index/index' }) }, 1500)
      } catch (e) { uni.showToast({ title: '重置失败', icon: 'none' }) }
    },
    
    // --- 彩蛋触发逻辑 ---
    handleHeaderClick() {
        const now = new Date().getTime()
        if (now - this.lastClickTime > 1000) this.clickCount = 0
        this.clickCount++
        this.lastClickTime = now
        
        if (this.clickCount >= 5) {
            this.clickCount = 0
            this.inputPassword = ''
            this.showPasswordModal = true
        }
    },
    
    checkPassword() {
        if (this.inputPassword === this.SECRET_CODE) {
            this.showPasswordModal = false
            uni.showToast({ title: '开发者模式已解锁', icon: 'success' })
            setTimeout(() => { uni.navigateTo({ url: '/pages/debug/debug' }) }, 500)
        } else {
            uni.showToast({ title: '密码错误', icon: 'none' })
            this.inputPassword = ''
        }
    },

    showAbout() {
      uni.showModal({
        title: '关于健康习惯助手',
        content: '版本: 3.6.1\n\n作者: b站up主 道系青年Lee\n\n功能特色：\n• 戒除习惯追踪（严苛模式）\n• 健身/读书/单词打卡（宽容模式）\n• 自定义习惯养成\n• 智能统计与图表\n• 成就奖赏机制\n\n欢迎大家到b站看我的视频，如果有任何建议，欢迎在视频下留言反馈！\n\n注意：如有严重困扰，建议寻求专业心理咨询师的帮助。',
        showCancel: false, confirmText: '支持作者'
      })
    },

    // --- 数据导出逻辑 (V3.7 新增) ---
    exportData() {
      uni.showLoading({ title: '正在打包...' })
      try {
        const backup = {
          version: '3.7',
          timestamp: new Date().getTime(),
          data: {
            quitRecord: uni.getStorageSync('quitRecord'),
            fitnessRecord: uni.getStorageSync('fitnessRecord'),
            readingRecord: uni.getStorageSync('readingRecord'),
            wordsRecord: uni.getStorageSync('wordsRecord'),
            customRecord: uni.getStorageSync('customRecord'),
            
            struggleHistory: uni.getStorageSync('struggleHistory'),
            readingHistory: uni.getStorageSync('readingHistory'),
            wordHistory: uni.getStorageSync('wordHistory'),
            
            userCustomBooks: uni.getStorageSync('userCustomBooks'),
            reminderSettings: uni.getStorageSync('reminderSettings'),
            appSettings: uni.getStorageSync('appSettings')
          }
        }
        
        const jsonStr = JSON.stringify(backup)
        // 使用特殊的头尾标记，方便识别
        const saveCode = `【健康习惯助手存档】\n${jsonStr}\n【结束】`
        
        uni.hideLoading()
        
        uni.setClipboardData({
          data: saveCode,
          success: () => {
            uni.showModal({
              title: '导出成功',
              content: '存档码已复制到剪贴板！\n\n请立即粘贴到【微信文件传输助手】或【手机备忘录】中保存。\n\n以后换机时，复制这段文字点击导入即可。',
              showCancel: false,
              confirmText: '知道了'
            })
          }
        })
        
      } catch (e) {
        uni.hideLoading()
        console.error(e)
        uni.showToast({ title: '导出失败', icon: 'none' })
      }
    },
    
    // --- 数据导入逻辑 (V3.7 新增) ---
    importData() {
      uni.getClipboardData({
        success: (res) => {
          const clipboardContent = res.data
          
          if (!clipboardContent || !clipboardContent.includes('【健康习惯助手存档】')) {
            uni.showModal({
              title: '无效存档',
              content: '剪贴板里没有检测到有效的存档码。\n\n请先去备忘录复制完整的存档文字，再点导入。',
              showCancel: false
            })
            return
          }
          
          // 提取 JSON
          let jsonStr = clipboardContent.replace('【健康习惯助手存档】', '').replace('【结束】', '').trim()
          
          try {
            const backup = JSON.parse(jsonStr)
            
            if (!backup.data) {
              throw new Error('数据结构损坏')
            }
            
            const dateStr = new Date(backup.timestamp).toLocaleString()
            
            uni.showModal({
              title: '⚠️ 高能预警',
              content: `检测到存档时间：${dateStr}\n\n导入将【彻底覆盖】当前手机上的所有进度，且无法撤销！\n\n确定要覆盖吗？`,
              confirmText: '确定覆盖',
              confirmColor: '#FF3B30',
              success: (modalRes) => {
                if (modalRes.confirm) {
                  this.executeImport(backup.data)
                }
              }
            })
            
          } catch (e) {
            uni.showToast({ title: '存档码损坏或解析失败', icon: 'none' })
          }
        }
      })
    },
    
    executeImport(data) {
      try {
        uni.showLoading({ title: '正在恢复...' })
        
        for (const key in data) {
          if (data[key] !== null && data[key] !== undefined) {
            uni.setStorageSync(key, data[key])
          }
        }
        
        uni.hideLoading()
        uni.showToast({ title: '恢复成功！', icon: 'success' })
        
        setTimeout(() => {
          uni.reLaunch({ url: '/pages/index/index' })
        }, 1500)
        
      } catch (e) {
        uni.hideLoading()
        uni.showToast({ title: '写入存储失败', icon: 'none' })
      }
    }
  }
}
</script>

<style>
.settings-container { padding: 20px; min-height: 100vh; background: #f5f5f5; }
.settings-header { text-align: center; margin-bottom: 30px; padding: 10px; /* 增加点击热区 */ }
.settings-title { font-size: 24px; font-weight: bold; color: #333; }
.settings-list { background: white; border-radius: 12px; overflow: hidden; margin-bottom: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
.setting-item { display: flex; justify-content: space-between; align-items: center; padding: 20px; border-bottom: 1px solid #f0f0f0; }
.setting-item:last-child { border-bottom: none; }
.setting-label { font-size: 16px; color: #333; }
.setting-arrow { font-size: 16px; color: #999; }
.setting-input { text-align: right; color: #007AFF; font-weight: bold; width: 150px; }
.backup-section { display: flex; gap: 10px; }
.backup-btn, .restore-btn { flex: 1; padding: 12px; border-radius: 8px; font-size: 14px; border: 1px solid #007AFF; background: white; color: #007AFF; }
.restore-btn { background: #007AFF; color: white; }

/* 密码弹窗样式 */
.modal-mask { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center; z-index: 2000; }
.modal-content { background: white; width: 80%; max-width: 300px; padding: 25px; border-radius: 15px; display: flex; flex-direction: column; }
.modal-title { font-size: 18px; font-weight: bold; text-align: center; margin-bottom: 20px; }
.password-input { 
    width: 100%; height: 44px; border: 1px solid #ddd; border-radius: 8px; 
    text-align: center; margin-bottom: 20px; font-size: 18px; box-sizing: border-box; letter-spacing: 5px;
}
.modal-buttons { display: flex; gap: 10px; }
.modal-cancel { background: #f8f9fa; color: #666; flex: 1; padding: 10px; border-radius: 8px; }
.modal-confirm { background: #007AFF; color: white; flex: 1; padding: 10px; border-radius: 8px; }
</style>