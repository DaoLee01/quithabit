<template>
  <view class="container">
    <view class="nav-tabs">
      <view 
        v-for="tab in tabs" 
        :key="tab.id"
        :class="['nav-tab', activeTab === tab.id ? 'nav-tab-active' : '']"
        @click="switchTab(tab.id)"
      >
        <text class="nav-tab-text">{{ tab.id === 'custom' ? customHabitName : tab.name }}</text>
      </view>
    </view>

    <view class="reward-section">
      <view class="stars-display">
        <view v-for="star in currentStars" :key="star.id" :class="['star', `star-${star.type}`]">⭐</view>
      </view>
      <text class="reward-text">{{ rewardMessage }}</text>
    </view>

    <view class="habits-container">
      
      <!-- 1. 戒除习惯 -->
      <view v-if="activeTab === 'quit'" class="habit-section">
        <view class="header">
          <text class="days-text">{{ quitHabit.streakDays }}天</text>
          <text class="subtitle">已坚持戒除</text>
        </view>
        <view class="motivation-card"><text class="motivation-text">{{ motivationQuote }}</text></view>
        <view class="progress-section">
          <progress :percent="quitHabit.progressPercent" show-info stroke-width="6" />
          <text class="progress-text">本月进度: {{ quitHabit.currentMonthSuccess }}/{{ quitHabit.targetDays }} ({{ quitHabit.targetLabel }})</text>
        </view>
        <view class="action-buttons">
          <button class="success-btn" @click="recordHabitSuccess('quit')">✅ 今日成功</button>
          <view class="btn-row">
            <button class="struggle-btn half-btn" @click="openModal('struggle')">😔 遇到困难</button>
            <button class="history-btn half-btn" @click="openHistory('struggle')">📜 心路历程</button>
          </view>
        </view>
      </view>

      <!-- 2. 健身习惯 -->
      <view v-if="activeTab === 'fitness'" class="habit-section">
        <view class="header">
          <text class="days-text">{{ fitnessHabit.totalDays }}天</text>
          <text class="subtitle">累计健身</text>
        </view>
        <view class="motivation-card"><text class="motivation-text">{{ motivationQuote }}</text></view>
        <view class="progress-section">
          <progress :percent="fitnessHabit.progressPercent" show-info stroke-width="6" />
          <text class="progress-text">本月进度: {{ fitnessHabit.currentMonthSuccess }}/{{ fitnessHabit.targetDays }} ({{ fitnessHabit.targetLabel }})</text>
        </view>
        <view class="fitness-types">
          <text class="section-title">今日运动类型</text>
          <view class="type-buttons">
            <button v-for="type in fitnessTypes" :key="type.id" :class="['type-btn', selectedFitnessType === type.id ? 'type-btn-active' : '']" @click="selectFitnessType(type.id)">{{ type.name }}</button>
          </view>
        </view>
        <view class="action-buttons">
          <button class="success-btn" @click="recordHabitSuccess('fitness')">🏃 完成锻炼</button>
          <button class="info-btn" @click="showFitnessTips">📚 浑元形意太极门建议</button>
        </view>
      </view>

      <!-- 3. 读书习惯 -->
      <view v-if="activeTab === 'reading'" class="habit-section">
        <view class="header">
          <text class="days-text">{{ readingHabit.totalDays }}天</text>
          <text class="subtitle">累计阅读</text>
        </view>
        <view class="motivation-card"><text class="motivation-text">书中自有黄金屋，坚持阅读 📚</text></view>
        <view class="progress-section">
          <progress :percent="readingHabit.progressPercent" show-info stroke-width="6" />
          <text class="progress-text">本月进度: {{ readingHabit.currentMonthSuccess }}/{{ readingHabit.targetDays }} ({{ readingHabit.targetLabel }})</text>
        </view>
        <view class="reading-input">
          <text class="section-title">今日阅读内容</text>
          <input v-model="readingContent" class="reading-input-field" placeholder="在此输入书名..." placeholder-style="color:#999" :cursor-spacing="20" />
          <view class="reading-time">
            <text>阅读时长:</text>
            <input v-model="readingMinutes" class="time-input" type="number" placeholder="分钟" placeholder-style="color:#999" />
          </view>
        </view>
        <view class="action-buttons">
          <button class="success-btn" @click="recordHabitSuccess('reading')">📖 完成阅读</button>
          <view class="btn-row">
            <button class="info-btn half-btn" @click="showReadingRecommendations">📚 推荐/抽签</button>
            <button class="history-btn half-btn" @click="openModal('addBook')">➕ 贡献书单</button>
          </view>
          <button class="history-btn full-btn-margin" @click="openHistory('reading')">📜 阅读历史 (点击可修改)</button>
        </view>
      </view>

      <!-- 4. 背单词习惯 (增加复制、笔记) -->
      <view v-if="activeTab === 'words'" class="habit-section">
        <view class="header">
          <text class="days-text">{{ wordsHabit.totalDays }}天</text>
          <text class="subtitle">累计背单词</text>
        </view>
        <view class="daily-word-card" @click="toggleWordMeaning">
          <view class="card-header">
            <text class="section-title">✨ 实用口语/场景词</text>
            <view class="card-icons">
                <text class="copy-icon" @click.stop="copyWord">📋</text>
                <text class="refresh-icon" @click.stop="loadDailyWord">🔄</text>
            </view>
          </view>
          <text class="word-text">{{ currentDailyWord.word }}</text>
          <text class="word-pronunciation">{{ currentDailyWord.pronunciation }}</text>
          <view v-if="showWordMeaning" class="word-reveal">
            <text class="word-meaning">{{ currentDailyWord.meaning }}</text>
            <text class="word-example">{{ currentDailyWord.example }}</text>
          </view>
          <text v-else class="tap-hint">(点击查看释义与例句)</text>
        </view>
        <view class="progress-section">
          <progress :percent="wordsHabit.progressPercent" show-info stroke-width="6" />
          <text class="progress-text">本月进度: {{ wordsHabit.currentMonthSuccess }}/{{ wordsHabit.targetDays }} ({{ wordsHabit.targetLabel }})</text>
        </view>
        <view class="words-input">
          <text class="section-title">今日学习情况</text>
          <view class="words-count">
            <text>今日单词量:</text>
            <input v-model="wordsCount" class="count-input" type="number" placeholder="个" />
          </view>
          <!-- 这里改为点击保存到笔记 -->
          <view class="btn-row" style="margin-top:10px">
             <button class="history-btn half-btn" @click="openModal('wordNote')">📝 记录/保存笔记</button>
             <button class="history-btn half-btn" @click="openHistory('words')">📜 单词历史</button>
          </view>
        </view>
        <view class="action-buttons">
          <button class="success-btn" @click="recordHabitSuccess('words')">📝 完成学习打卡</button>
          <button class="info-btn" @click="showWordsTips">💡 学习技巧</button>
        </view>
      </view>

      <!-- 5. 自定义习惯 -->
      <view v-if="activeTab === 'custom'" class="habit-section">
        <view class="header">
          <text class="days-text">{{ customHabit.totalDays }}天</text>
          <text class="subtitle">累计{{ customHabitName }}</text>
        </view>
        <view class="motivation-card"><text class="motivation-text">{{ motivationQuote }}</text></view>
        <view class="progress-section">
          <progress :percent="customHabit.progressPercent" show-info stroke-width="6" />
          <text class="progress-text">本月进度: {{ customHabit.currentMonthSuccess }}/{{ customHabit.targetDays }} ({{ customHabit.targetLabel }})</text>
        </view>
        <view class="action-buttons">
          <button class="success-btn" @click="recordHabitSuccess('custom')">✅ 完成打卡</button>
          <button class="info-btn" @click="showCustomTips">💡 习惯养成小贴士</button>
        </view>
      </view>
    </view>

    <!-- 弹窗组件区域 -->
    <view v-if="showInputModal" class="modal-mask top-layer">
      <view class="modal-content">
        <text class="modal-title">{{ modalTitle }}</text>
        
        <textarea v-if="['struggle', 'editHistory', 'wordNote'].includes(modalType)" v-model="inputText" 
            :placeholder="modalType === 'wordNote' ? '记录今天学到的词汇或心得...' : '记录下你的感受...'" 
            class="modal-textarea"></textarea>
        
        <view v-if="modalType === 'addBook'" class="add-book-form">
          <input v-model="newBookTitle" class="modal-input" placeholder="书名 (如：《平凡的世界》)" />
          <input v-model="newBookDesc" class="modal-input" placeholder="一句话推荐理由" />
        </view>

        <view class="modal-buttons">
          <button class="modal-cancel" @click="showInputModal = false">取消</button>
          <button class="modal-confirm" @click="handleModalConfirm">保存</button>
        </view>
      </view>
    </view>
    
    <view v-if="showHistoryModal" class="modal-mask">
      <view class="modal-content history-content">
        <text class="modal-title">{{ historyTitle }}</text>
        <scroll-view scroll-y="true" class="history-list">
          <view v-if="historyList.length === 0" class="empty-history">暂无记录，加油！</view>
          <view v-else v-for="(item, index) in historyList" :key="index" class="history-item" @click="onHistoryItemClick(index)">
            <view class="history-row-top">
                <text class="history-date">{{ item.date }}</text>
                <text v-if="activeTab === 'reading' || activeTab === 'words'" class="edit-icon">✎ 修改</text>
            </view>
            <text class="history-note">{{ item.content }}</text>
            <text v-if="item.extra" class="history-extra">{{ item.extra }}</text>
          </view>
        </scroll-view>
        <button class="modal-confirm full-width" @click="showHistoryModal = false">关闭</button>
      </view>
    </view>
  </view>
</template>

<script>
import { wordDatabase } from '@/common/wordData.js'
import { defaultBooks } from '@/common/bookData.js'

export default {
  data() {
    return {
      activeTab: 'quit',
      tabs: [{id:'quit',name:'戒除'},{id:'fitness',name:'健身'},{id:'reading',name:'读书'},{id:'words',name:'单词'},{id:'custom',name:'自定义'}],
      motivationQuote: '',
      
      showInputModal: false, showHistoryModal: false, modalType: '', modalTitle: '', inputText: '', newBookTitle: '', newBookDesc: '', 
      historyTitle: '', historyList: [], editingHistoryIndex: -1,
      currentStars: [], rewardMessage: '',
      
      quitHabit: { streakDays: 0, totalDays: 0, currentMonthSuccess: 0, progressPercent: 0, targetDays: 30, targetLabel: '全月' },
      fitnessHabit: { streakDays: 0, totalDays: 0, currentMonthSuccess: 0, progressPercent: 0, targetDays: 30, targetLabel: '全月' },
      readingHabit: { streakDays: 0, totalDays: 0, currentMonthSuccess: 0, progressPercent: 0, targetDays: 30, targetLabel: '全月' },
      wordsHabit: { streakDays: 0, totalDays: 0, currentMonthSuccess: 0, progressPercent: 0, targetDays: 30, targetLabel: '全月' },
      customHabit: { streakDays: 0, totalDays: 0, currentMonthSuccess: 0, progressPercent: 0, targetDays: 30, targetLabel: '全月' },
      
      customHabitName: '自律打卡',
      fitnessTypes: [{id:'running',name:'跑步'},{id:'gym',name:'健身房'},{id:'yoga',name:'瑜伽'},{id:'home',name:'居家训练'},{id:'other',name:'闪电五连鞭'}],
      selectedFitnessType: '', readingContent: '', readingMinutes: '', userCustomBooks: [], wordsCount: '', wordsMemo: '', showWordMeaning: false, currentDailyWord: {},
      
      normalQuotes: ["每一次坚持都让你更强大 💪", "进步不是直线，偶尔的挫折是正常的 🌱", "你正在建立更好的自己 ✨", "一天一天来，你会成功的 🌟", "你的决心值得赞赏 🌈"],
      maQuotes: [
		"你在健身房练死劲，不好用！",
        "小朋友，你两个手来折我一个手指头，折不动。",
        "二百多斤的英国大力士都握不动我这一个手指！",
        "他啪就站起来了，很快啊！一个左正蹬 一个右鞭tei，一个左刺拳。",
        "我全部防出去了啊。传统功夫点到为止。",
        "我大意了啊，没有闪。",
        "小伙子你不讲武德，你不懂。",
        "他说他是乱打的。他可不是乱打的啊，蒸蹬 鞭tei 左刺拳，训练有素！",
        "年轻人不讲武德，来骗，来偷袭！我69岁的老同志。这好吗？这不好。",
        "年轻人，耗子尾汁。",
        "武林要以和为贵，要讲武德。 不要搞窝里斗。"
		]
    }
  },
  onLoad() { this.loadSettings(); this.loadAllHabitsData(); this.updateMotivation(); this.updateRewardSystem(); this.loadDailyWord(); this.loadUserBooks() },
  onShow() { this.loadSettings(); this.loadAllHabitsData(); this.updateRewardSystem() },
  methods: {
    loadSettings() { const settings = uni.getStorageSync('appSettings') || {}; if (settings.customHabitName) this.customHabitName = settings.customHabitName },
    switchTab(tabId) { this.activeTab = tabId; this.updateMotivation(); this.updateRewardSystem() },
    updateMotivation() {
      if (this.activeTab === 'fitness') this.motivationQuote = this.maQuotes[Math.floor(Math.random() * this.maQuotes.length)]
      else this.motivationQuote = this.normalQuotes[Math.floor(Math.random() * this.normalQuotes.length)]
    },
    updateRewardSystem() {
      const currentHabitKey = `${this.activeTab}Habit`
      const currentHabitData = this[currentHabitKey]
      let daysForStars = this.activeTab === 'quit' ? (currentHabitData.streakDays || 0) : (currentHabitData.totalDays || 0)
      this.currentStars = []
      let remainingDays = daysForStars
      const levels = [365, 90, 30, 7, 1]
      const types = ['platinum', 'red', 'orange', 'purple', 'green']
      levels.forEach((lvl, i) => {
        const count = Math.floor(remainingDays / lvl)
        remainingDays %= lvl
        for(let k=0; k<count; k++) this.currentStars.push({id: this.currentStars.length, type: types[i]})
      })
      if(daysForStars === 0) this.rewardMessage = this.activeTab === 'quit' ? "⚠️ 破戒归零！重新开始！" : "💪 开始积累你的成就吧！"
      else this.rewardMessage = `已坚持 ${daysForStars} 天，继续保持！`
    },
    
    loadAllHabitsData() { ['quit', 'fitness', 'reading', 'words', 'custom'].forEach(t => this.loadHabitData(t)) },
    
    // --- V3.5 核心：数据去重与清洗 ---
    loadHabitData(habitType) {
      try {
        const record = uni.getStorageSync(`${habitType}Record`) || {}
        const habitData = this[`${habitType}Habit`]
        
        // 1. 去重 + 清洗
        const rawHistory = record.history || []
        const uniqueHistory = [...new Set(rawHistory)] // 去重关键
        const now = new Date()
        const bufferTime = now.getTime() + 86400000 
        const validHistory = uniqueHistory.filter(d => new Date(d).getTime() <= bufferTime)
        
        let streak = 0
        let total = validHistory.length // 正确的累计天数
        
        if (habitType === 'quit') {
            const sorted = [...validHistory].sort((a,b) => new Date(b) - new Date(a))
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
        } else {
            streak = record.streakDays || 0
        }
        
        habitData.streakDays = streak
        habitData.totalDays = total
        
        // 2. 进度条分母计算 (剩余天数逻辑)
        const currentYear = now.getFullYear()
        const currentMonth = now.getMonth()
        const daysInMonth = new Date(currentYear, currentMonth + 1, 0).getDate()
        
        if (!record.startDate) { record.startDate = new Date().toDateString(); uni.setStorageSync(`${habitType}Record`, record) }
        const startDate = new Date(record.startDate)
        
        let targetDays = daysInMonth
        let targetLabel = '全月'
        // 如果是本月开始的，分母 = 月底 - 开始日 + 1
        if (startDate.getFullYear() === currentYear && startDate.getMonth() === currentMonth) {
          targetDays = daysInMonth - startDate.getDate() + 1
          targetLabel = '剩余挑战'
        }
        
        const currentMonthSuccessCount = validHistory.filter(dateStr => {
          const d = new Date(dateStr)
          return d.getFullYear() === currentYear && d.getMonth() === currentMonth
        }).length
        
        habitData.targetDays = targetDays
        habitData.targetLabel = targetLabel
        habitData.currentMonthSuccess = currentMonthSuccessCount
        habitData.progressPercent = targetDays > 0 ? Math.round((currentMonthSuccessCount / targetDays) * 100) : 0
        
      } catch (e) { console.error(e) }
    },
    
    recordHabitSuccess(habitType) {
        let record = uni.getStorageSync(`${habitType}Record`) || {}
        if (!record.history) record.history = []
        
        const now = new Date()
        const bufferTime = now.getTime() + 86400000 
        const futureRecords = record.history.filter(d => new Date(d).getTime() > bufferTime)
        
        if (futureRecords.length > 0) {
            uni.showModal({
                title: '时间线冲突',
                content: `检测到 ${futureRecords.length} 条未来记录。需要清理。`,
                confirmText: '清理并打卡',
                cancelText: '取消',
                confirmColor: '#FF3B30',
                success: (res) => {
                    if (res.confirm) {
                        record.history = record.history.filter(d => new Date(d).getTime() <= bufferTime)
                        uni.setStorageSync(`${habitType}Record`, record)
                        const today = new Date().toDateString()
                        if (record.history.includes(today)) {
                            uni.showToast({ title: '清理完成，今天已打卡', icon: 'none' })
                            this.loadHabitData(habitType); return
                        }
                        this._proceedToRecord(habitType, record)
                    }
                }
            })
            return
        }
        
        const today = new Date().toDateString()
        if (record.history.includes(today)) { uni.showToast({ title: '今天已打卡', icon: 'none' }); return }
        
        if (record.lastSuccess) {
            const last = new Date(record.lastSuccess); last.setHours(0,0,0,0)
            const curr = new Date(today); curr.setHours(0,0,0,0)
            if (curr < last) { uni.showToast({ title: '无法在历史时间补签', icon: 'none' }); return }
        }
        
        if (habitType === 'reading' && !this.readingContent.trim()) {
            uni.showModal({
                title: '提示', content: '您还没填写书名，确定要直接打卡吗？', confirmText: '直接打卡', cancelText: '去填写',
                success: (res) => { if (res.confirm) { this._proceedToRecord(habitType, record) } }
            }); return
        }
        this._proceedToRecord(habitType, record)
    },

    _proceedToRecord(habitType, record) {
      const today = new Date().toDateString()
      if (!record.startDate) record.startDate = today
      
      const lastDate = record.lastSuccess ? new Date(record.lastSuccess) : null
      const todayObj = new Date()
      let isConsecutive = false
      if (lastDate) {
         lastDate.setHours(0,0,0,0); todayObj.setHours(0,0,0,0);
         const diffDays = Math.floor((todayObj - lastDate) / (1000*60*60*24))
         if (diffDays === 1) isConsecutive = true
      }
      
      if (isConsecutive) record.streakDays = (record.streakDays || 0) + 1
      else record.streakDays = 1
      if (record.streakDays > (record.bestStreak || 0)) record.bestStreak = record.streakDays
      
      record.history.push(today)
      // 使用 Set 去重后再计算长度，防止重复
      record.history = [...new Set(record.history)]
      record.totalDays = record.history.length
      record.lastSuccess = today
      
      if (habitType === 'reading') {
        const hist = uni.getStorageSync('readingHistory') || []
        hist.unshift({ date: new Date().toLocaleString(), content: this.readingContent || '未记录书名', extra: this.readingMinutes ? `${this.readingMinutes}分钟` : '' })
        uni.setStorageSync('readingHistory', hist)
      }
      
      uni.setStorageSync(`${habitType}Record`, record)
      this.loadHabitData(habitType)
      this.showSuccessMessage(habitType)
      this.updateRewardSystem()
      
      if (habitType === 'reading') { this.readingContent = ''; this.readingMinutes = '' }
      else if (habitType === 'words') { this.wordsCount = ''; this.wordsMemo = '' }
    },
    
    // 弹窗相关
    openModal(type) {
      this.modalType = type; this.showInputModal = true
      if (type === 'struggle') { this.modalTitle = '记录感受与挑战'; this.inputText = '' }
      else if (type === 'addBook') { this.modalTitle = '添加想读/推荐的书'; this.newBookTitle = ''; this.newBookDesc = '' }
      else if (type === 'wordNote') { this.modalTitle = '单词/心得笔记'; this.inputText = '' }
    },
    handleModalConfirm() {
      if (this.modalType === 'struggle') this.saveStruggleRecord()
      else if (this.modalType === 'addBook') this.saveNewBook()
      else if (this.modalType === 'editHistory') this.saveHistoryEdit()
      else if (this.modalType === 'wordNote') this.saveWordNote()
    },
    saveStruggleRecord() {
      if (!this.inputText.trim()) return
      const list = uni.getStorageSync('struggleHistory') || []
      list.unshift({ date: new Date().toLocaleString(), content: this.inputText })
      uni.setStorageSync('struggleHistory', list)
      this.showInputModal = false; uni.showToast({ title: '记录已保存', icon: 'success' })
    },
    saveWordNote() {
      if (!this.inputText.trim()) return
      const list = uni.getStorageSync('wordHistory') || [] // 独立的单词历史
      list.unshift({ date: new Date().toLocaleString(), content: this.inputText })
      uni.setStorageSync('wordHistory', list)
      this.showInputModal = false; uni.showToast({ title: '笔记已保存', icon: 'success' })
    },
    saveNewBook() {
      if (!this.newBookTitle.trim()) { uni.showToast({ title: '书名不能为空', icon: 'none' }); return }
      const newBook = { title: `《${this.newBookTitle}》`, desc: this.newBookDesc || '用户自选好书', isFixed: false }
      this.userCustomBooks.push(newBook)
      uni.setStorageSync('userCustomBooks', this.userCustomBooks)
      this.showInputModal = false; uni.showToast({ title: '添加成功', icon: 'success' })
    },
    openHistory(type) {
        this.showHistoryModal = true
        if (type === 'struggle') { this.historyTitle = '📜 戒除心路历程'; this.historyList = uni.getStorageSync('struggleHistory') || [] }
        else if (type === 'reading') { this.historyTitle = '📜 阅读历史'; this.historyList = uni.getStorageSync('readingHistory') || [] }
        else if (type === 'words') { this.historyTitle = '📜 单词笔记'; this.historyList = uni.getStorageSync('wordHistory') || [] }
    },
    onHistoryItemClick(index) {
        if (this.activeTab !== 'reading' && this.activeTab !== 'words') return
        this.editingHistoryIndex = index
        this.inputText = this.historyList[index].content
        this.modalType = 'editHistory'; this.modalTitle = '修改记录'; this.showInputModal = true
    },
    saveHistoryEdit() {
        if (!this.inputText.trim()) { uni.showToast({ title: '内容不能为空', icon: 'none' }); return }
        this.historyList[this.editingHistoryIndex].content = this.inputText
        // 根据不同tab存回不同key
        let storageKey = 'readingHistory'
        if (this.activeTab === 'words') storageKey = 'wordHistory'
        uni.setStorageSync(storageKey, this.historyList)
        this.showInputModal = false; uni.showToast({ title: '修改成功', icon: 'success' })
    },
    
    // 单词复制
    copyWord() {
        const text = `${this.currentDailyWord.word} ${this.currentDailyWord.pronunciation}\n${this.currentDailyWord.meaning}\n例句: ${this.currentDailyWord.example}`
        uni.setClipboardData({ data: text, success: () => uni.showToast({ title: '单词已复制', icon: 'none' }) })
    },
    
    loadDailyWord() { this.currentDailyWord = wordDatabase[Math.floor(Math.random() * wordDatabase.length)]; this.showWordMeaning = false },
    toggleWordMeaning() { this.showWordMeaning = !this.showWordMeaning },
    loadUserBooks() { this.userCustomBooks = uni.getStorageSync('userCustomBooks') || [] },
    showReadingRecommendations() {
      const fixedBooks = defaultBooks.filter(b => b.isFixed)
      const randomPool = [ ...defaultBooks.filter(b => !b.isFixed), ...this.userCustomBooks ]
      let randomPick = { title: "暂无更多推荐", desc: "快去贡献你的书单吧" }
      if (randomPool.length > 0) randomPick = randomPool[Math.floor(Math.random() * randomPool.length)]
      let content = `【斗数宝典】\n`; fixedBooks.forEach(b => { content += `${b.title}\n${b.desc}\n\n` })
      content += `【今日随机 / 用户推荐】\n${randomPick.title}\n- ${randomPick.desc}`
      uni.showModal({ title: '📚 书单推荐', content: content, showCancel: true, cancelText: '关闭', confirmText: '复制书单', success: (res) => { if (res.confirm) uni.setClipboardData({ data: content, success: () => uni.showToast({ title: '已复制', icon: 'none' }) }) } })
    },
    selectFitnessType(t) { this.selectedFitnessType = t },
    showSuccessMessage(t) { uni.showToast({ title: '坚持就是胜利！', icon: 'success' }) },
    showFitnessTips() { uni.showModal({ title: '浑元形意太极门', content: '接化发要练好，耗子尾汁！', showCancel: false }) },
    showCustomTips() { uni.showModal({ title: '习惯养成', content: '一个行为重复21天会成为习惯，重复90天会成为稳定的生活方式。', showCancel: false }) },
    showWordsTips() { uni.showModal({ title: '技巧', content: '多看例句，结合场景记忆', showCancel: false }) }
  }
}
</script>

<style>
/* 样式复用 V3.4 (已含输入框修复和层级修复) */
.container { padding: 20px; min-height: 100vh; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding-bottom: 20px; }
.nav-tabs { display: flex; background: rgba(255, 255, 255, 0.9); border-radius: 12px; padding: 4px; margin-bottom: 20px; }
.nav-tab { flex: 1; text-align: center; padding: 10px; border-radius: 8px; transition: all 0.3s ease; }
.nav-tab-active { background: #007AFF; }
.nav-tab-text { font-size: 14px; font-weight: bold; color: #333; }
.nav-tab-active .nav-tab-text { color: white; }
.reward-section { background: rgba(255, 255, 255, 0.95); padding: 15px; border-radius: 12px; margin-bottom: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); text-align: center; }
.stars-display { display: flex; justify-content: center; gap: 8px; margin-bottom: 10px; flex-wrap: wrap; }
.star { font-size: 20px; animation: glow 2s infinite; }
.star-green { color: #4CD964; text-shadow: 0 0 8px #4CD964; filter: hue-rotate(0deg); }
.star-purple { color: #AF52DE; text-shadow: 0 0 8px #AF52DE; filter: hue-rotate(200deg); }
.star-orange { color: #FF9500; text-shadow: 0 0 10px #FF9500; filter: hue-rotate(30deg); font-size: 22px; }
.star-red { color: #FF3B30; text-shadow: 0 0 10px #FF3B30; filter: hue-rotate(300deg); font-size: 24px; }
.star-platinum { color: #E5E4E2; text-shadow: 0 0 15px #FFFFFF; filter: brightness(1.5); font-size: 26px; }
@keyframes glow { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.1); } }
.reward-text { font-size: 14px; color: #333; font-weight: bold; text-align: center; }
.habit-section { padding: 10px 0; }
.header { margin: 20px 0 15px 0; text-align: center; }
.days-text { font-size: 48px; font-weight: bold; color: white; text-shadow: 0 2px 4px rgba(0,0,0,0.3); }
.subtitle { font-size: 16px; color: rgba(255,255,255,0.8); display: block; margin-top: 8px; }
.motivation-card, .progress-section, .fitness-types, .reading-input, .words-input { background: rgba(255,255,255,0.95); padding: 15px; border-radius: 12px; margin: 15px 0; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
.motivation-text { font-size: 14px; color: #333; line-height: 1.4; text-align: center; }
.progress-text { display: block; text-align: center; margin-top: 10px; color: #666; font-size: 13px; }
.daily-word-card { background: linear-gradient(120deg, #fdfbfb 0%, #ebedee 100%); padding: 15px; border-radius: 12px; margin: 15px 0; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
.card-header { display: flex; justify-content: space-between; align-items: center; }
.card-icons { display: flex; gap: 10px; }
.word-text { font-size: 24px; font-weight: bold; color: #007AFF; margin: 10px 0 5px; display: block; }
.word-pronunciation { font-size: 14px; color: #666; font-family: monospace; display: block; margin-bottom: 10px;}
.word-reveal { animation: fadeIn 0.5s; }
.word-meaning { font-weight: bold; color: #333; display: block; margin-bottom: 5px; }
.word-example { font-style: italic; color: #555; font-size: 14px; }
.tap-hint { font-size: 12px; color: #999; margin-top: 5px; display: block; text-align: center;}
.action-buttons { margin-top: 20px; }
.success-btn { background: #4CD964; color: white; width: 100%; padding: 12px; border-radius: 10px; font-weight: bold; margin-bottom: 10px; }
.btn-row { display: flex; gap: 10px; margin-bottom: 10px; }
.half-btn { flex: 1; padding: 10px; border-radius: 8px; font-size: 13px; border: none; font-weight: bold; }
.struggle-btn { background: #FF9500; color: white; }
.info-btn { background: #5AC8FA; color: white; }
.history-btn { background: #E5E5EA; color: #333; }
.full-btn-margin { width: 100%; padding: 10px; border-radius: 8px; font-weight: bold; }
.type-buttons { display: flex; gap: 8px; flex-wrap: wrap; }
.type-btn { flex: 1; min-width: 60px; padding: 8px; background: white; border: 1px solid #ddd; border-radius: 6px; font-size: 12px; }
.type-btn-active { background: #007AFF; color: white; border-color: #007AFF; }
.reading-input-field, .words-memo, .modal-input, .modal-textarea { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 6px; margin-bottom: 10px; font-size: 14px; box-sizing: border-box; background-color: #f9f9f9; color: #333; z-index: 10; height: 44px; }
.words-memo, .modal-textarea { height: 80px; }
.reading-time, .words-count { display: flex; align-items: center; gap: 10px; }
.time-input, .count-input { width: 80px; padding: 8px; border: 1px solid #ddd; border-radius: 6px; text-align: center; box-sizing: border-box; background: #fff; height: 40px; }
.modal-mask { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center; z-index: 1000; }
.modal-mask.top-layer { z-index: 2000; }
.modal-content { background: white; width: 80%; max-width: 400px; padding: 25px; border-radius: 15px; display: flex; flex-direction: column; }
.modal-title { font-size: 18px; font-weight: bold; text-align: center; margin-bottom: 15px; }
.add-book-form { margin-bottom: 20px; }
.modal-buttons { display: flex; gap: 10px; }
.modal-cancel { background: #f8f9fa; color: #666; flex: 1; padding: 10px; border-radius: 8px; }
.modal-confirm { background: #007AFF; color: white; flex: 1; padding: 10px; border-radius: 8px; }
.history-content { max-height: 70vh; }
.history-list { max-height: 300px; margin-bottom: 15px; }
.history-item { background: #f9f9f9; padding: 10px; border-radius: 8px; margin-bottom: 8px; border-left: 3px solid #007AFF; }
.history-date { font-size: 12px; color: #999; margin-bottom: 4px; display: block; }
.history-note { font-size: 14px; color: #333; line-height: 1.4; }
.history-extra { font-size: 12px; color: #007AFF; margin-top: 4px; font-weight: bold; display: block; }
.empty-history { text-align: center; color: #999; padding: 20px; }
.history-row-top { display: flex; justify-content: space-between; align-items: center; margin-bottom: 4px; }
.edit-icon { font-size: 12px; color: #007AFF; background: #f0f7ff; padding: 2px 6px; border-radius: 4px; }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>