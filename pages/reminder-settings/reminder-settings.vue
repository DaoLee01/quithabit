<template>
  <view class="reminder-container">
    <view class="header">
      <text class="title">通知提醒设置</text>
    </view>
    
    <view class="reminder-list">
      <view 
        class="reminder-item" 
        v-for="reminder in reminders" 
        :key="reminder.id"
        @longpress="handleLongPress(reminder)"
      >
        <view class="reminder-info">
          <text class="reminder-time">{{ reminder.time }}</text>
          <view class="label-row">
            <text class="reminder-label">{{ reminder.label }}</text>
            <!-- 视觉标记 -->
            <text v-if="reminder.id > 5" class="custom-tag">自定义</text>
          </view>
        </view>
        <switch :checked="reminder.enabled" @change="toggleReminder(reminder.id, $event)" />
      </view>
    </view>
    
    <view class="custom-reminder">
      <text class="section-title">添加新提醒</text>
      <view class="time-picker">
        <picker mode="time" :value="customTime" @change="onTimeChange">
          <view class="time-input">{{ customTime || '点击选择时间' }}</view>
        </picker>
        <button class="add-btn" @click="addCustomReminder" :disabled="!customTime">添加</button>
      </view>
    </view>
    
    <view class="reminder-tips">
      <text class="tips-title">💡 提醒小贴士</text>
      <text class="tips-content">
        1. 这里的开关仅控制是否向日历写入新事件，关闭开关不会自动删除日历里已存在的日程。
        2. <text style="font-weight:bold;color:#007AFF;">长按</text> 自定义提醒可删除。
      </text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      customTime: '',
      reminders: [
        { id: 1, time: '08:00', label: '早晨提醒', enabled: false },
        { id: 2, time: '12:00', label: '午间提醒', enabled: false },
        { id: 3, time: '20:00', label: '晚间提醒', enabled: false },
        { id: 4, time: '21:00', label: '睡前准备', enabled: false },
        { id: 5, time: '22:00', label: '休息提醒', enabled: false }
      ]
    }
  },
  
  onLoad() {
    this.loadReminderSettings()
  },
  
  methods: {
    loadReminderSettings() {
      try {
        const settings = uni.getStorageSync('reminderSettings')
        if (settings && settings.length > 0) {
          this.reminders = settings
        }
      } catch (e) { console.error(e) }
    },
    
    saveReminderSettings() {
      try {
        uni.setStorageSync('reminderSettings', this.reminders)
      } catch (e) { console.error(e) }
    },
    
    toggleReminder(id, e) {
      const reminder = this.reminders.find(r => r.id === id)
      if (reminder) {
        reminder.enabled = e.detail.value
        this.saveReminderSettings()
        
        if (reminder.enabled) {
            this.requestCalendarPermission(reminder)
        } else {
            uni.showToast({ title: '已关闭 (请去日历清理旧日程)', icon: 'none' })
        }
      }
    },
    
    // --- 新增：长按删除逻辑 ---
    handleLongPress(reminder) {
        // 固定 ID (1-5) 不可删除
        if (reminder.id <= 5) return 
        
        uni.showModal({
            title: '删除提醒',
            content: `确定要删除 ${reminder.time} 的提醒吗？\n(注意：系统日历中的日程需手动删除)`,
            confirmColor: '#FF3B30',
            success: (res) => {
                if (res.confirm) {
                    this.reminders = this.reminders.filter(r => r.id !== reminder.id)
                    this.saveReminderSettings()
                    uni.showToast({ title: '已删除', icon: 'success' })
                }
            }
        })
    },
    
    // 1. 请求权限
    requestCalendarPermission(reminder) {
        // #ifdef APP-PLUS
        if (plus.os.name === 'Android') {
            plus.android.requestPermissions(
                ['android.permission.READ_CALENDAR', 'android.permission.WRITE_CALENDAR'],
                (e) => {
                    if (e.deniedAlways.length > 0 || e.deniedPresent.length > 0) {
                        uni.showModal({ title: '权限提示', content: '请在设置中开启日历权限。', showCancel: false })
                    } else {
                        this.addCalendarEvent(reminder)
                    }
                }
            )
        } else {
            this.addCalendarEvent(reminder)
        }
        // #endif
    },

    // 2. 智能获取日历ID
    getCalendarId(main) {
        let calendarId = null
        try {
            const Uri = plus.android.importClass('android.net.Uri')
            const calendarUri = Uri.parse("content://com.android.calendar/calendars")
            const resolver = main.getContentResolver()
            const cursor = plus.android.invoke(resolver, 'query', calendarUri, null, null, null, null)
            
            if (cursor != null) {
                if (plus.android.invoke(cursor, 'moveToFirst')) {
                    do {
                        const idIndex = plus.android.invoke(cursor, 'getColumnIndex', '_id')
                        const nameIndex = plus.android.invoke(cursor, 'getColumnIndex', 'account_name')
                        const id = plus.android.invoke(cursor, 'getString', idIndex)
                        const name = plus.android.invoke(cursor, 'getString', nameIndex)
                        
                        if (calendarId === null) calendarId = id
                        if (name && (name.includes('local') || name.includes('account'))) {
                            calendarId = id
                            break 
                        }
                    } while (plus.android.invoke(cursor, 'moveToNext'))
                }
                plus.android.invoke(cursor, 'close')
            }
        } catch (e) { console.error('ID获取失败', e) }
        return calendarId || '1'
    },

    // 3. 写入日历
    addCalendarEvent(reminder) {
      // #ifdef APP-PLUS
      if (plus.os.name !== 'Android') return
      
      try {
          const main = plus.android.runtimeMainActivity()
          const ContentValues = plus.android.importClass('android.content.ContentValues')
          const Uri = plus.android.importClass('android.net.Uri')
          const TimeZone = plus.android.importClass('java.util.TimeZone')
          const Calendar = plus.android.importClass('java.util.Calendar')
          
          const calId = this.getCalendarId(main)
          const [hours, minutes] = reminder.time.split(':').map(Number)
          const cal = Calendar.getInstance()
          cal.set(Calendar.HOUR_OF_DAY, hours)
          cal.set(Calendar.MINUTE, minutes)
          cal.set(Calendar.SECOND, 0)
          
          const now = Calendar.getInstance()
          if (cal.before(now)) cal.add(Calendar.DATE, 1)
          
          const startMillis = cal.getTimeInMillis()
          const endMillis = startMillis + 10 * 60 * 1000
          
          const eventsUri = Uri.parse("content://com.android.calendar/events")
          const values = new ContentValues()
          values.put("calendar_id", calId) 
          values.put("title", "健康习惯: " + reminder.label)
          values.put("description", "坚持就是胜利！")
          values.put("dtstart", startMillis)
          values.put("dtend", endMillis)
          values.put("eventTimezone", TimeZone.getDefault().getID())
          values.put("hasAlarm", 1)
          
          const resolver = main.getContentResolver()
          const newEventUri = plus.android.invoke(resolver, 'insert', eventsUri, values)
          
          if (newEventUri) {
              const eventID = parseFloat(newEventUri.getLastPathSegment())
              const remindersUri = Uri.parse("content://com.android.calendar/reminders")
              const reminderValues = new ContentValues()
              reminderValues.put("event_id", eventID)
              reminderValues.put("minutes", 0)
              reminderValues.put("method", 1) 
              
              plus.android.invoke(resolver, 'insert', remindersUri, reminderValues)
              uni.showToast({ title: '已添加到日历', icon: 'success' })
          } else {
              uni.showToast({ title: '写入失败(日历无法访问)', icon: 'none' })
          }
      } catch (e) {
          console.error("Native Error:", e)
          uni.showModal({ title: '错误', content: e.message, showCancel: false })
      }
      // #endif
    },
    
    onTimeChange(e) { this.customTime = e.detail.value },
    addCustomReminder() {
      if (!this.customTime) return
      const newReminder = { id: Date.now(), time: this.customTime, label: '自定义提醒', enabled: true }
      this.reminders.push(newReminder)
      this.saveReminderSettings()
      this.requestCalendarPermission(newReminder)
      this.customTime = ''
    }
  }
}
</script>

<style>
.reminder-container { padding: 20px; min-height: 100vh; background: #f5f5f5; }
.header { text-align: center; margin-bottom: 30px; }
.title { font-size: 24px; font-weight: bold; color: #333; }
.reminder-list { background: white; border-radius: 12px; overflow: hidden; margin-bottom: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
.reminder-item { display: flex; justify-content: space-between; align-items: center; padding: 20px; border-bottom: 1px solid #f0f0f0; }
.reminder-item:last-child { border-bottom: none; }
.reminder-info { display: flex; flex-direction: column; }
.reminder-time { font-size: 18px; font-weight: bold; color: #333; margin-bottom: 4px; }
.label-row { display: flex; align-items: center; gap: 8px; }
.reminder-label { font-size: 14px; color: #666; }
.custom-tag { font-size: 10px; color: #007AFF; background: #E3F2FD; padding: 2px 4px; border-radius: 4px; }
.custom-reminder { background: white; padding: 20px; border-radius: 12px; margin-bottom: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
.section-title { font-size: 18px; font-weight: bold; margin-bottom: 15px; display: block; }
.time-picker { display: flex; gap: 10px; align-items: center; }
.time-input { flex: 1; padding: 12px; border: 1px solid #ddd; border-radius: 8px; text-align: center; background: white; }
.add-btn { padding: 12px 20px; background: #007AFF; color: white; border: none; border-radius: 8px; font-size: 14px; }
.add-btn:disabled { background: #ccc; }
.reminder-tips { background: #E3F2FD; padding: 15px; border-radius: 12px; border-left: 4px solid #2196F3; }
.tips-title { font-size: 16px; font-weight: bold; color: #1976D2; display: block; margin-bottom: 8px; }
.tips-content { font-size: 14px; color: #424242; line-height: 1.4; }
</style>