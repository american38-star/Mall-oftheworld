<!-- src/components/games/MegaBoomingFruits.vue -->
<template>
  <div class="mega-booming-fruits">
    <!-- رأس اللعبة -->
    <div class="game-header">
      <div class="game-title-section">
        <h1 class="game-title">🍒 MEGA BOOMING FRUITS 🍒</h1>
        <div class="game-subtitle">X50 MAX WIN</div>
      </div>
      <div class="game-stats-header">
        <div class="stat-box">
          <span class="stat-label">LAST WIN</span>
          <span class="stat-value" :class="{ 'highlight': lastWin > 0 }">${{ lastWin.toFixed(2) }}</span>
        </div>
        <div class="stat-box">
          <span class="stat-label">TOTAL BETS</span>
          <span class="stat-value">${{ totalBets.toFixed(2) }}</span>
        </div>
        <div class="stat-box">
          <span class="stat-label">RTP</span>
          <span class="stat-value">97.5%</span>
        </div>
      </div>
    </div>

    <!-- شريط الروليت -->
    <div class="roulette-section">
      <div class="roulette-container" :class="{ spinning: isSpinning }">
        <div class="roulette-track" ref="rouletteTrack">
          <div 
            v-for="(segment, index) in rouletteSegments" 
            :key="index"
            class="roulette-segment"
            :class="getSegmentClass(segment)"
          >
            <span class="segment-icon">{{ getFruitIcon(segment) }}</span>
            <span class="segment-value">{{ segment.value }}x</span>
          </div>
          <!-- تكرار للحركة المستمرة -->
          <div 
            v-for="(segment, index) in rouletteSegments" 
            :key="'dup-' + index"
            class="roulette-segment"
            :class="getSegmentClass(segment)"
          >
            <span class="segment-icon">{{ getFruitIcon(segment) }}</span>
            <span class="segment-value">{{ segment.value }}x</span>
          </div>
        </div>
        <div class="roulette-indicator"></div>
      </div>
      
      <!-- نتيجة الجولة -->
      <transition name="pop">
        <div v-if="showResult" class="result-display" :class="resultClass">
          <span class="result-icon">{{ resultIcon }}</span>
          <div class="result-details">
            <span class="result-text">{{ resultText }}</span>
            <span class="result-amount" v-if="resultAmount > 0">+${{ resultAmount.toFixed(2) }}</span>
          </div>
        </div>
      </transition>
    </div>

    <!-- لوحة الرهان -->
    <div class="betting-panel">
      <div class="balance-info">
        <div class="current-balance">
          <i class="fas fa-coins"></i>
          <span>${{ balance.toFixed(2) }}</span>
        </div>
        <div class="bet-multiplier" v-if="currentMultiplier > 1">
          <span class="multiplier">{{ currentMultiplier.toFixed(2) }}x</span>
          <span class="potential-win">${{ (betAmount * currentMultiplier).toFixed(2) }}</span>
        </div>
      </div>

      <!-- خيارات الرهان -->
      <div class="bet-options">
        <div class="bet-amount-section">
          <label>💰 BET AMOUNT</label>
          <div class="amount-controls">
            <button @click="decreaseBet" :disabled="isSpinning || betAmount <= minBet">-</button>
            <input 
              type="number" 
              v-model.number="betAmount" 
              :min="minBet" 
              :max="balance"
              :disabled="isSpinning"
              step="0.1"
            >
            <button @click="increaseBet" :disabled="isSpinning || betAmount >= balance">+</button>
          </div>
        </div>

        <div class="quick-bets">
          <button 
            v-for="amount in quickBetAmounts" 
            :key="amount"
            @click="setBetAmount(amount)"
            :disabled="isSpinning || amount > balance"
            class="quick-bet-btn"
          >
            ${{ amount }}
          </button>
          <button 
            @click="setBetAmount(balance)"
            :disabled="isSpinning"
            class="quick-bet-btn max"
          >
            MAX
          </button>
        </div>
      </div>

      <!-- أزرار التحكم -->
      <div class="action-buttons">
        <button 
          class="spin-btn" 
          @click="startSpin"
          :disabled="isSpinning || betAmount > balance || betAmount < minBet"
        >
          <span v-if="!isSpinning">🎰 SPIN</span>
          <span v-else><i class="fas fa-spinner fa-spin"></i> SPINNING...</span>
        </button>
        
        <button 
          class="auto-btn" 
          @click="toggleAutoPlay"
          :class="{ active: autoPlayActive }"
          :disabled="isSpinning || betAmount > balance"
        >
          <i class="fas fa-play-circle"></i>
          AUTO
        </button>
      </div>

      <!-- إعدادات التشغيل التلقائي -->
      <transition name="slide">
        <div v-if="autoPlayActive" class="auto-settings">
          <div class="auto-controls">
            <div class="auto-spins">
              <label>SPINS</label>
              <div class="auto-spins-control">
                <button @click="decreaseAutoSpins" :disabled="autoSpins <= 1">-</button>
                <input type="number" v-model.number="autoSpins" min="1" max="100">
                <button @click="increaseAutoSpins" :disabled="autoSpins >= 100">+</button>
              </div>
            </div>
            <div class="stop-conditions">
              <label>STOP ON</label>
              <div class="condition-checkbox">
                <label>
                  <input type="checkbox" v-model="stopOnWin"> WIN
                </label>
                <label>
                  <input type="checkbox" v-model="stopOnMultiplier"> 10x+
                </label>
              </div>
            </div>
            <button class="stop-auto-btn" @click="stopAutoPlay">
              <i class="fas fa-stop"></i> STOP
            </button>
          </div>
        </div>
      </transition>
    </div>

    <!-- سجل النتائج -->
    <div class="history-section">
      <h3>📊 LAST 10 SPINS</h3>
      <div class="history-list">
        <div 
          v-for="(item, index) in spinHistory" 
          :key="index"
          class="history-item"
          :class="{ win: item.won }"
        >
          <span class="history-icon">{{ getFruitIcon(item.segment) }}</span>
          <span class="history-multiplier">{{ item.multiplier }}x</span>
          <span class="history-bet">${{ item.betAmount.toFixed(2) }}</span>
          <span class="history-result" :class="{ 'win-text': item.won }">
            {{ item.won ? '+' : '-' }}${{ Math.abs(item.profit).toFixed(2) }}
          </span>
        </div>
      </div>
    </div>

    <!-- جدول المدفوعات -->
    <div class="paytable-section">
      <h3>📋 PAYTABLE</h3>
      <div class="paytable-grid">
        <div v-for="segment in rouletteSegments" :key="segment.id" class="paytable-item">
          <div class="paytable-icon" :class="getSegmentClass(segment)">
            {{ getFruitIcon(segment) }}
          </div>
          <div class="paytable-details">
            <span class="paytable-name">{{ segment.name }}</span>
            <span class="paytable-multiplier">{{ segment.value }}x</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "MegaBoomingFruits",
  
  static: {
    meta: {
      name: 'Mega Booming Fruits',
      icon: '🍒',
      description: 'احصل على مضاعفات تصل إلى 50x'
    }
  },

  props: {
    balance: {
      type: Number,
      required: true,
      default: 0
    }
  },

  emits: ['update-balance', 'show-result'],

  data() {
    return {
      // إعدادات اللعبة
      minBet: 0.1,
      maxBet: 1000,
      
      // الرهان الحالي
      betAmount: 1,
      
      // حالة اللعبة
      isSpinning: false,
      showResult: false,
      currentMultiplier: 1,
      
      // نتائج
      lastWin: 0,
      totalBets: 0,
      spinHistory: [],
      
      // التشغيل التلقائي
      autoPlayActive: false,
      autoSpins: 10,
      autoSpinsRemaining: 0,
      stopOnWin: false,
      stopOnMultiplier: false,
      
      // نتيجة الجولة
      resultText: '',
      resultIcon: '',
      resultClass: '',
      resultAmount: 0,
      
      // قطاعات الروليت (مثل العاب الكازينو الحقيقية)
      rouletteSegments: [
        { id: 1, name: 'CHERRY', icon: '🍒', value: 2, color: 'red', probability: 0.2 },
        { id: 2, name: 'LEMON', icon: '🍋', value: 3, color: 'yellow', probability: 0.15 },
        { id: 3, name: 'ORANGE', icon: '🍊', value: 4, color: 'orange', probability: 0.12 },
        { id: 4, name: 'PLUM', icon: '🍑', value: 5, color: 'purple', probability: 0.1 },
        { id: 5, name: 'GRAPES', icon: '🍇', value: 6, color: 'purple', probability: 0.08 },
        { id: 6, name: 'WATERMELON', icon: '🍉', value: 8, color: 'green', probability: 0.06 },
        { id: 7, name: 'BELL', icon: '🔔', value: 10, color: 'gold', probability: 0.05 },
        { id: 8, name: 'BAR', icon: '💎', value: 15, color: 'blue', probability: 0.03 },
        { id: 9, name: 'SEVEN', icon: '7️⃣', value: 20, color: 'red', probability: 0.02 },
        { id: 10, name: 'STAR', icon: '⭐', value: 30, color: 'gold', probability: 0.01 },
        { id: 11, name: 'MEGA', icon: '💥', value: 40, color: 'rainbow', probability: 0.005 },
        { id: 12, name: 'BOOM', icon: '🔥', value: 50, color: 'rainbow', probability: 0.002 }
      ],
      
      quickBetAmounts: [1, 5, 10, 25, 50, 100],
      autoSpinTimer: null
    }
  },

  computed: {
    // حساب إجمالي الاحتمالات
    totalProbability() {
      return this.rouletteSegments.reduce((sum, seg) => sum + seg.probability, 0)
    }
  },

  watch: {
    // مراقبة الرهان
    betAmount: {
      handler(newVal) {
        if (newVal > this.balance) {
          this.betAmount = this.balance
        }
        if (newVal < this.minBet) {
          this.betAmount = this.minBet
        }
      },
      immediate: true
    },

    // مراقبة التشغيل التلقائي
    autoPlayActive(active) {
      if (active) {
        this.autoSpinsRemaining = this.autoSpins
        this.startAutoSpin()
      }
    }
  },

  methods: {
    // زيادة الرهان
    increaseBet() {
      const newAmount = this.betAmount + 1
      if (newAmount <= this.balance) {
        this.betAmount = newAmount
      }
    },

    // تقليل الرهان
    decreaseBet() {
      const newAmount = this.betAmount - 1
      if (newAmount >= this.minBet) {
        this.betAmount = newAmount
      }
    },

    // تعيين مبلغ الرهان
    setBetAmount(amount) {
      if (amount <= this.balance) {
        this.betAmount = amount
      }
    },

    // بدء الدوران
    async startSpin() {
      if (this.isSpinning || this.betAmount > this.balance) return

      this.isSpinning = true
      this.showResult = false
      
      // خصم الرهان
      const newBalance = this.balance - this.betAmount
      this.$emit('update-balance', newBalance, 'Mega Booming Fruits', this.betAmount, false)
      
      this.totalBets += this.betAmount

      // تحديد النتيجة بناءً على الاحتمالات
      const result = this.getRandomSegment()
      
      // محاكاة دوران الروليت
      await this.animateRoulette(result)
      
      // حساب الربح
      const winAmount = this.betAmount * result.value
      const won = result.value > 1
      
      if (won) {
        const newBalanceWithWin = newBalance + winAmount
        this.$emit('update-balance', newBalanceWithWin, 'Mega Booming Fruits', this.betAmount, true)
        this.lastWin = winAmount
        this.resultAmount = winAmount
        this.resultText = `YOU WON ${result.value}X!`
        this.resultIcon = this.getFruitIcon(result)
        this.resultClass = 'win'
        
        this.$emit('show-result', `🎉 MEGA WIN! ${result.value}x = $${winAmount.toFixed(2)}`, true)
      } else {
        this.lastWin = 0
        this.resultAmount = 0
        this.resultText = 'TRY AGAIN!'
        this.resultIcon = '😢'
        this.resultClass = 'lose'
        
        this.$emit('show-result', `😢 Better luck next time!`, false)
      }

      // إضافة إلى السجل
      this.spinHistory.unshift({
        segment: result,
        multiplier: result.value,
        betAmount: this.betAmount,
        profit: won ? winAmount : -this.betAmount,
        won: won
      })

      // الاحتفاظ بآخر 10 جولات فقط
      if (this.spinHistory.length > 10) {
        this.spinHistory.pop()
      }

      this.showResult = true
      this.isSpinning = false

      // التحقق من التشغيل التلقائي
      this.checkAutoPlay()
    },

    // الحصول على قطاع عشوائي بناءً على الاحتمالات
    getRandomSegment() {
      const random = Math.random()
      let cumulativeProbability = 0
      
      for (const segment of this.rouletteSegments) {
        cumulativeProbability += segment.probability
        if (random < cumulativeProbability) {
          return segment
        }
      }
      
      return this.rouletteSegments[0]
    },

    // محاكاة دوران الروليت
    async animateRoulette(targetSegment) {
      const track = this.$refs.rouletteTrack
      if (!track) return

      const segmentWidth = 120 // عرض القطاع بالبكسل
      const spins = 20 // عدد الدورات
      const targetIndex = this.rouletteSegments.findIndex(s => s.id === targetSegment.id)
      
      // حساب المسافة للوصول إلى القطاع المطلوب
      const targetPosition = (targetIndex + this.rouletteSegments.length * 3) * segmentWidth
      
      // تحريك الروليت
      track.style.transition = 'transform 3s cubic-bezier(0.25, 0.1, 0.15, 1)'
      track.style.transform = `translateX(-${targetPosition}px)`
      
      // انتظار انتهاء الحركة
      await new Promise(resolve => setTimeout(resolve, 3000))
      
      // إعادة التعيين للحركة التالية
      track.style.transition = 'none'
      track.style.transform = `translateX(-${targetIndex * segmentWidth}px)`
    },

    // الحصول على أيقونة الفاكهة
    getFruitIcon(segment) {
      return segment.icon
    },

    // الحصول على كلاس القطاع
    getSegmentClass(segment) {
      return `segment-${segment.color}`
    },

    // تبديل التشغيل التلقائي
    toggleAutoPlay() {
      if (this.autoPlayActive) {
        this.stopAutoPlay()
      } else {
        this.autoPlayActive = true
      }
    },

    // بدء التشغيل التلقائي
    startAutoSpin() {
      if (this.autoSpinsRemaining > 0 && this.autoPlayActive) {
        this.startSpin()
      } else {
        this.stopAutoPlay()
      }
    },

    // التحقق من التشغيل التلقائي
    checkAutoPlay() {
      if (!this.autoPlayActive) return

      this.autoSpinsRemaining--

      // التحقق من شروط الإيقاف
      if (this.stopOnWin && this.lastWin > 0) {
        this.stopAutoPlay()
        this.$emit('show-result', '🛑 Auto play stopped: Win detected', false)
        return
      }

      if (this.stopOnMultiplier && this.currentMultiplier >= 10) {
        this.stopAutoPlay()
        this.$emit('show-result', '🛑 Auto play stopped: 10x+ multiplier', false)
        return
      }

      if (this.autoSpinsRemaining > 0 && this.autoPlayActive) {
        setTimeout(() => {
          this.startSpin()
        }, 500)
      } else {
        this.stopAutoPlay()
      }
    },

    // إيقاف التشغيل التلقائي
    stopAutoPlay() {
      this.autoPlayActive = false
      this.autoSpinsRemaining = 0
      if (this.autoSpinTimer) {
        clearTimeout(this.autoSpinTimer)
      }
    },

    // زيادة عدد الدورات التلقائية
    increaseAutoSpins() {
      if (this.autoSpins < 100) {
        this.autoSpins++
      }
    },

    // تقليل عدد الدورات التلقائية
    decreaseAutoSpins() {
      if (this.autoSpins > 1) {
        this.autoSpins--
      }
    }
  },

  beforeUnmount() {
    if (this.autoSpinTimer) {
      clearTimeout(this.autoSpinTimer)
    }
  }
}
</script>

<style scoped>
.mega-booming-fruits {
  max-width: 1200px;
  margin: 0 auto;
  background: linear-gradient(135deg, #1a1f2e 0%, #2a2f3f 100%);
  border-radius: 20px;
  padding: 25px;
  color: #ffffff;
  font-family: 'Montserrat', 'Arial', sans-serif;
}

/* رأس اللعبة */
.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 2px solid #ffd700;
}

.game-title-section {
  text-align: left;
}

.game-title {
  font-size: 28px;
  font-weight: 800;
  color: #ffd700;
  text-transform: uppercase;
  margin: 0 0 5px 0;
  text-shadow: 0 0 10px rgba(255, 215, 0, 0.3);
  animation: glow 2s ease-in-out infinite;
}

@keyframes glow {
  0%, 100% { text-shadow: 0 0 10px rgba(255, 215, 0, 0.3); }
  50% { text-shadow: 0 0 20px rgba(255, 215, 0, 0.6); }
}

.game-subtitle {
  font-size: 14px;
  color: #a0a0a0;
  letter-spacing: 2px;
}

.game-stats-header {
  display: flex;
  gap: 15px;
}

.stat-box {
  background: #2a2f42;
  padding: 10px 15px;
  border-radius: 8px;
  text-align: center;
  min-width: 100px;
}

.stat-label {
  display: block;
  font-size: 12px;
  color: #a0a0a0;
  margin-bottom: 5px;
}

.stat-value {
  display: block;
  font-size: 18px;
  font-weight: 700;
  color: #ffd700;
}

.stat-value.highlight {
  animation: pulse 1s ease;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}

/* قسم الروليت */
.roulette-section {
  position: relative;
  margin-bottom: 30px;
  padding: 20px 0;
}

.roulette-container {
  width: 100%;
  overflow: hidden;
  border-radius: 10px;
  background: #1e2335;
  padding: 15px 0;
  position: relative;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.5);
}

.roulette-track {
  display: flex;
  transition: transform 3s cubic-bezier(0.25, 0.1, 0.15, 1);
  will-change: transform;
}

.roulette-segment {
  min-width: 120px;
  height: 100px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 0 5px;
  border-radius: 10px;
  background: #2a2f42;
  border: 2px solid #3a4055;
  transition: all 0.3s ease;
}

.segment-red {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
}

.segment-yellow {
  background: linear-gradient(135deg, #f1c40f, #f39c12);
}

.segment-orange {
  background: linear-gradient(135deg, #e67e22, #d35400);
}

.segment-purple {
  background: linear-gradient(135deg, #9b59b6, #8e44ad);
}

.segment-green {
  background: linear-gradient(135deg, #27ae60, #2ecc71);
}

.segment-gold {
  background: linear-gradient(135deg, #ffd700, #f1c40f);
}

.segment-blue {
  background: linear-gradient(135deg, #3498db, #2980b9);
}

.segment-rainbow {
  background: linear-gradient(135deg, #ffd700, #e74c3c, #9b59b6, #3498db);
  animation: rainbow 3s linear infinite;
}

@keyframes rainbow {
  0% { filter: hue-rotate(0deg); }
  100% { filter: hue-rotate(360deg); }
}

.segment-icon {
  font-size: 30px;
  margin-bottom: 5px;
}

.segment-value {
  font-size: 20px;
  font-weight: 700;
  color: #ffffff;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.roulette-indicator {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 50%;
  width: 4px;
  background: #ffd700;
  transform: translateX(-50%);
  box-shadow: 0 0 20px #ffd700;
  z-index: 2;
}

/* عرض النتيجة */
.result-display {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(0, 0, 0, 0.9);
  padding: 20px 40px;
  border-radius: 50px;
  display: flex;
  align-items: center;
  gap: 20px;
  z-index: 10;
  border: 2px solid;
  backdrop-filter: blur(10px);
  animation: popIn 0.3s ease;
}

@keyframes popIn {
  0% { transform: translate(-50%, -50%) scale(0); }
  80% { transform: translate(-50%, -50%) scale(1.1); }
  100% { transform: translate(-50%, -50%) scale(1); }
}

.result-display.win {
  border-color: #2ecc71;
  box-shadow: 0 0 30px rgba(46, 204, 113, 0.5);
}

.result-display.lose {
  border-color: #e74c3c;
  box-shadow: 0 0 30px rgba(231, 76, 60, 0.5);
}

.result-icon {
  font-size: 40px;
}

.result-details {
  text-align: center;
}

.result-text {
  display: block;
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 5px;
}

.result-amount {
  display: block;
  font-size: 28px;
  font-weight: 800;
  color: #2ecc71;
}

.pop-enter-active, .pop-leave-active {
  transition: all 0.3s ease;
}

.pop-enter-from, .pop-leave-to {
  opacity: 0;
  transform: translate(-50%, -50%) scale(0);
}

/* لوحة الرهان */
.betting-panel {
  background: #2a2f42;
  border-radius: 15px;
  padding: 20px;
  margin-bottom: 30px;
  border: 1px solid #3a4055;
}

.balance-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #3a4055;
}

.current-balance {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 20px;
  font-weight: 600;
}

.current-balance i {
  color: #ffd700;
  font-size: 24px;
}

.bet-multiplier {
  text-align: right;
}

.multiplier {
  display: block;
  font-size: 24px;
  font-weight: 800;
  color: #ffd700;
}

.potential-win {
  display: block;
  font-size: 14px;
  color: #a0a0a0;
}

.bet-options {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 20px;
}

.bet-amount-section label {
  display: block;
  font-size: 12px;
  color: #a0a0a0;
  margin-bottom: 8px;
  letter-spacing: 1px;
}

.amount-controls {
  display: flex;
  align-items: center;
  gap: 10px;
}

.amount-controls button {
  width: 40px;
  height: 40px;
  border: none;
  border-radius: 8px;
  background: #1e2335;
  color: #ffd700;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid #3a4055;
}

.amount-controls button:hover:not(:disabled) {
  background: #ffd700;
  color: #1e2335;
  transform: scale(1.1);
}

.amount-controls button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.amount-controls input {
  flex: 1;
  height: 40px;
  border: none;
  border-radius: 8px;
  background: #1e2335;
  color: #ffd700;
  font-size: 18px;
  text-align: center;
  border: 1px solid #3a4055;
  transition: all 0.3s ease;
}

.amount-controls input:focus {
  outline: none;
  border-color: #ffd700;
}

.quick-bets {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.quick-bet-btn {
  padding: 10px;
  border: none;
  border-radius: 6px;
  background: #1e2335;
  color: #ffd700;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid #3a4055;
}

.quick-bet-btn:hover:not(:disabled) {
  background: #ffd700;
  color: #1e2335;
  transform: translateY(-2px);
}

.quick-bet-btn.max {
  background: #ffd700;
  color: #1e2335;
  font-weight: 700;
}

.quick-bet-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.action-buttons {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 15px;
}

.spin-btn {
  padding: 15px;
  border: none;
  border-radius: 10px;
  background: linear-gradient(135deg, #ffd700, #ffa500);
  color: #1e2335;
  font-size: 20px;
  font-weight: 800;
  cursor: pointer;
  transition: all 0.3s ease;
  text-transform: uppercase;
  letter-spacing: 2px;
}

.spin-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(255, 215, 0, 0.3);
}

.spin-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.auto-btn {
  padding: 15px;
  border: none;
  border-radius: 10px;
  background: #1e2335;
  color: #ffd700;
  font-size: 18px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid #ffd700;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.auto-btn.active {
  background: #ffd700;
  color: #1e2335;
}

.auto-btn:hover:not(:disabled) {
  background: #ffd700;
  color: #1e2335;
}

.auto-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* إعدادات التشغيل التلقائي */
.auto-settings {
  margin-top: 20px;
  padding: 20px;
  background: #1e2335;
  border-radius: 10px;
  border: 1px solid #3a4055;
}

.auto-controls {
  display: grid;
  grid-template-columns: 1fr 1fr auto;
  gap: 20px;
  align-items: center;
}

.auto-spins label {
  display: block;
  font-size: 12px;
  color: #a0a0a0;
  margin-bottom: 5px;
}

.auto-spins-control {
  display: flex;
  align-items: center;
  gap: 5px;
}

.auto-spins-control button {
  width: 30px;
  height: 30px;
  border: none;
  border-radius: 5px;
  background: #2a2f42;
  color: #ffd700;
  cursor: pointer;
  transition: all 0.3s ease;
}

.auto-spins-control button:hover {
  background: #ffd700;
  color: #1e2335;
}

.auto-spins-control input {
  width: 60px;
  height: 30px;
  border: none;
  border-radius: 5px;
  background: #2a2f42;
  color: #ffd700;
  text-align: center;
  border: 1px solid #3a4055;
}

.stop-conditions label {
  display: block;
  font-size: 12px;
  color: #a0a0a0;
  margin-bottom: 5px;
}

.condition-checkbox {
  display: flex;
  gap: 15px;
}

.condition-checkbox label {
  display: flex;
  align-items: center;
  gap: 5px;
  color: #ffffff;
  font-size: 14px;
  cursor: pointer;
}

.condition-checkbox input {
  cursor: pointer;
}

.stop-auto-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  background: #e74c3c;
  color: white;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
}

.stop-auto-btn:hover {
  background: #c0392b;
  transform: scale(1.05);
}

.slide-enter-active, .slide-leave-active {
  transition: all 0.3s ease;
}

.slide-enter-from, .slide-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

/* سجل النتائج */
.history-section {
  margin-bottom: 30px;
}

.history-section h3 {
  color: #ffd700;
  margin-bottom: 15px;
  font-size: 18px;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.history-item {
  display: grid;
  grid-template-columns: 50px 60px 1fr 100px;
  align-items: center;
  padding: 10px 15px;
  background: #2a2f42;
  border-radius: 8px;
  border-left: 4px solid #e74c3c;
  transition: all 0.3s ease;
}

.history-item.win {
  border-left-color: #2ecc71;
  background: rgba(46, 204, 113, 0.1);
}

.history-item:hover {
  transform: translateX(-5px);
}

.history-icon {
  font-size: 24px;
}

.history-multiplier {
  font-size: 18px;
  font-weight: 700;
  color: #ffd700;
}

.history-bet {
  color: #a0a0a0;
}

.history-result {
  font-weight: 700;
  text-align: right;
}

.history-result.win-text {
  color: #2ecc71;
}

/* جدول المدفوعات */
.paytable-section {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid #3a4055;
}

.paytable-section h3 {
  color: #ffd700;
  margin-bottom: 20px;
  font-size: 18px;
}

.paytable-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 15px;
}

.paytable-item {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 10px;
  background: #2a2f42;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.paytable-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(255, 215, 0, 0.2);
}

.paytable-icon {
  width: 50px;
  height: 50px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
}

.paytable-details {
  flex: 1;
}

.paytable-name {
  display: block;
  font-size: 14px;
  color: #a0a0a0;
  margin-bottom: 3px;
}

.paytable-multiplier {
  display: block;
  font-size: 18px;
  font-weight: 700;
  color: #ffd700;
}

/* تحسينات الجوال */
@media (max-width: 768px) {
  .mega-booming-fruits {
    padding: 15px;
  }

  .game-header {
    flex-direction: column;
    gap: 15px;
    text-align: center;
  }

  .game-title-section {
    text-align: center;
  }

  .game-title {
    font-size: 22px;
  }

  .game-stats-header {
    flex-wrap: wrap;
    justify-content: center;
  }

  .bet-options {
    grid-template-columns: 1fr;
  }

  .auto-controls {
    grid-template-columns: 1fr;
    gap: 15px;
  }

  .condition-checkbox {
    justify-content: center;
  }

  .history-item {
    grid-template-columns: 40px 50px 1fr 80px;
    font-size: 14px;
  }

  .paytable-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .game-title {
    font-size: 18px;
  }

  .stat-box {
    min-width: 70px;
    padding: 8px;
  }

  .stat-value {
    font-size: 14px;
  }

  .action-buttons {
    grid-template-columns: 1fr;
  }

  .paytable-grid {
    grid-template-columns: 1fr;
  }

  .result-display {
    padding: 15px 25px;
  }

  .result-text {
    font-size: 18px;
  }

  .result-amount {
    font-size: 20px;
  }
}
</style>
