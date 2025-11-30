<script>
export default {
  onLaunch: function() {
    console.log('App Launch')
    // 启动时尝试迁移旧版本数据
    this.migrateOldData()
  },
  onShow: function() {
    console.log('App Show')
  },
  onHide: function() {
    console.log('App Hide')
  },
  methods: {
    /**
     * 数据迁移脚本
     * 目的：兼容 V1.0/V2.0 的 'habitRecord' 数据结构
     * 逻辑：如果发现有旧数据，且没有新版 'quitRecord' 数据，则搬运过去
     */
    migrateOldData() {
      try {
        // 1. 读取旧版本数据
        const oldData = uni.getStorageSync('habitRecord')
        
        // 2. 读取新版本数据 (检查【戒除】习惯是否已经有记录)
        const hasNewData = uni.getStorageSync('quitRecord')
        
        // 3. 只有当：有旧数据 && 没新数据 时，才执行迁移，避免覆盖用户新打的卡
        if (oldData && (!hasNewData || !hasNewData.lastSuccess)) {
          console.log('检测到旧版本数据，开始迁移...', oldData)
          
          // 构建符合 V3.0 标准的新结构
          const newRecord = {
            // 核心数据搬运
            streakDays: oldData.streakDays || 0,
            bestStreak: oldData.bestStreak || oldData.streakDays || 0,
            totalDays: oldData.monthSuccess || oldData.streakDays || 0, // 旧版可能混用，尽量保留最大值
            
            // 辅助数据初始化
            currentMonthSuccess: 0, 
            progressPercent: 0,
            targetDays: 30,
            targetLabel: '全月',
            
            // 时间记录
            lastSuccess: oldData.lastSuccess || '',
            startDate: oldData.startDate || new Date().toDateString(),
            
            // 历史数组重建 (旧版没有 history 数组，我们只能把最后一次成功的时间放进去)
            history: [] 
          }
          
          // 如果旧版有最后打卡时间，将其加入历史数组，保证统计图至少有一个点
          if (newRecord.lastSuccess) {
              newRecord.history.push(newRecord.lastSuccess)
          }

          // 4. 写入新保险箱 'quitRecord'
          uni.setStorageSync('quitRecord', newRecord)
          
          // 5. 提示用户
          console.log('数据迁移成功！')
          // 这里可以弹窗提示，也可以悄悄完成，为了不打扰用户，通常悄悄完成即可
          // 只有在调试时才弹窗
          // uni.showModal({
          //   title: '升级成功',
          //   content: '您的旧版打卡记录已自动迁移至【戒除】习惯。',
          //   showCancel: false
          // })
        }
      } catch (e) {
        console.error('数据迁移失败:', e)
      }
    }
  }
}
</script>

<style>
/* 全局样式 */
page {
  background-color: #f8f8f8;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

/* 解决按钮默认样式，让按钮更像是一个可点击的区域而不是传统按钮 */
button {
  margin: 0;
  padding: 0;
  background: none;
  border: none;
  outline: none;
}

button::after {
  border: none;
}

/* 解决一些机型上的滚动条丑陋问题 */
::-webkit-scrollbar {
  width: 0;
  height: 0;
  color: transparent;
}
</style>