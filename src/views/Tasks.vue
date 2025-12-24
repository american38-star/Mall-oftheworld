<template>
  <div class="game-page">
    <!-- الهيدر العلوي -->
    <div class="header">
      <div class="logo">d-lkn9.vercel.app</div>
      <div class="header-menu">
        <button class="menu-btn">حسابي</button>
        <button class="menu-btn vip">VIP</button>
        <button class="menu-btn">الفريق</button>
        <button class="menu-btn">المهام</button>
        <button class="menu-btn active">الرئيسية</button>
      </div>
    </div>

    <!-- محتوى الصفحة -->
    <div class="content">
      <!-- العمود الأيمن: لوحة المضاعفات -->
      <div class="multipliers-panel">
        <h1>PLINKO</h1>
        <div class="subtitle">BIGAMLINE</div>
        
        <!-- رصيد اللاعب -->
        <div class="balance-card">
          <div class="balance-label">BALANCE</div>
          <div class="balance-amount">FUN {{ balance.toFixed(2) }}</div>
        </div>

        <!-- المضاعفات -->
        <div class="multipliers-grid">
          <div v-for="(mult, index) in multipliers" :key="index" class="mult-card">
            <div class="mult-value">x{{ mult.value }}</div>
            <div class="mult-risk">Risk Level: {{ mult.risk }}</div>
          </div>
        </div>
      </div>

      <!-- العمود الأيسر: لوحة اللعبة -->
      <div class="game-panel">
        <!-- لوحة البلينكو -->
        <div class="plinko-board" ref="board">
          <!-- المسامير -->
          <div class="pins-container">
            <div v-for="row in 8" :key="row" class="pin-row">
              <div 
                v-for="pin in getPinsInRow(row)" 
                :key="pin.id"
                class="pin"
                :style="{
                  left: pin.left + 'px',
                  top: (row * 40) + 'px'
                }"
              ></div>
            </div>
          </div>

          <!-- الكرة -->
          <div 
            v-if="ball.active"
            class="ball"
            :style="{
              top: ball.y + 'px',
              left: ball.x + 'px',
              backgroundColor: ballColor
            }"
          ></div>

          <!-- خط الإسقاط -->
          <div class="drop-line" :style="{ left: dropPosition + 'px' }"></div>

          <!-- السلات النهائية -->
          <div class="slots-container">
            <div 
              v-for="(slot, index) in slots" 
              :key="index"
              class="slot"
              :style="{
                left: (index * 44) + 'px',
                backgroundColor: slot.color
              }"
            >
              <div class="slot-text">x{{ slot.multiplier }}</div>
            </div>
          </div>
        </div>

        <!-- عناصر التحكم -->
        <div class="controls">
          <!-- خيارات الرهان -->
          <div class="bet-controls">
            <div class="bet-info">
              <div class="min-bet">Min Bet 1.00 FUN</div>
              <div class="max-bet">Max Bet 1000.00 FUN</div>
            </div>
            
            <div class="bet-input-group">
              <button class="bet-btn minus" @click="decrementBet">-</button>
              <input 
                type="number" 
                v-model.number="betAmount" 
                class="bet-input"
                min="1"
                max="1000"
                step="10"
              />
              <button class="bet-btn plus" @click="incrementBet">+</button>
            </div>

            <div class="more-options">
              <button class="more-btn">More</button>
            </div>
          </div>

          <!-- مستويات الخطورة -->
          <div class="risk-controls">
            <button 
              v-for="level in riskLevels" 
              :key="level.id"
              :class="['risk-btn', level.class, { active: selectedRisk === level.id }]"
              @click="selectRisk(level.id)"
            >
              {{ level.label }}
            </button>
          </div>

          <!-- زر الإسقاط -->
          <div class="drop-section">
            <button 
              class="drop-btn"
              :disabled="isPlaying || betAmount > balance"
              @click="dropBall"
            >
              <div class="drop-icon">⬇️</div>
              <div class="drop-text">DROP</div>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- نتيجة الجولة -->
    <div v-if="showResult" class="result-overlay">
      <div class="result-modal" :class="resultClass">
        <div class="result-icon">{{ resultIcon }}</div>
        <div class="result-title">{{ resultTitle }}</div>
        <div class="result-multiplier">Multiplier: x{{ currentMultiplier.toFixed(2) }}</div>
        <div class="result-amount">{{ resultAmount.toFixed(2) }} FUN</div>
        <button class="close-btn" @click="closeResult">✕</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PlinkoGame',
  
  data() {
    return {
      // بيانات اللعبة
      balance: 1000.00,
      betAmount: 10,
      selectedRisk: 'normal',
      
      // الكرة
      ball: {
        active: false,
        x: 200,
        y: 20
      },
      ballColor: '#00ff88',
      dropPosition: 200,
      isPlaying: false,
      
      // المضاعفات كما في الصورة
      multipliers: [
        { value: 1.5, risk: 'Normal' },
        { value: 4, risk: 'Normal' },
        { value: 29, risk: 'High' },
        { value: 0.3, risk: 'Low' },
        { value: 0.2, risk: 'Low' },
        { value: 0.3, risk: 'Low' },
        { value: 29, risk: 'High' },
        { value: 4, risk: 'Normal' },
        { value: 1.5, risk: 'Normal' }
      ],
      
      // السلات النهائية
      slots: [
        { multiplier: 29, color: '#ff4444' },
        { multiplier: 4, color: '#44ff44' },
        { multiplier: 1.5, color: '#44ff44' },
        { multiplier: 0.3, color: '#ffff44' },
        { multiplier: 0.2, color: '#ffff44' },
        { multiplier: 0.3, color: '#ffff44' },
        { multiplier: 1.5, color: '#44ff44' },
        { multiplier: 4, color: '#44ff44' },
        { multiplier: 29, color: '#ff4444' }
      ],
      
      // مستويات الخطورة
      riskLevels: [
        { id: 'high', label: 'High', class: 'risk-high' },
        { id: 'normal', label: 'Normal', class: 'risk-normal' },
        { id: 'low', label: 'Low', class: 'risk-low' }
      ],
      
      // النتيجة
      showResult: false,
      currentMultiplier: 1,
      resultAmount: 0,
      resultTitle: '',
      resultIcon: '',
      resultClass: '',
      
      // الحركة
      animationInterval: null,
      velocityX: 0,
      velocityY: 0
    }
  },
  
  computed: {
    boardWidth() {
      return 400
    }
  },
  
  methods: {
    // حساب عدد المسامير في كل صف
    getPinsInRow(row) {
      const pins = []
      const pinCount = row + 3
      const spacing = this.boardWidth / (pinCount + 1)
      
      for (let i = 1; i <= pinCount; i++) {
        pins.push({
          id: `${row}-${i}`,
          left: spacing * i
        })
      }
      
      return pins
    },
    
    // تعديل الرهان
    incrementBet() {
      if (this.betAmount < 1000) {
        this.betAmount += 10
        if (this.betAmount > 1000) this.betAmount = 1000
      }
    },
    
    decrementBet() {
      if (this.betAmount > 1) {
        this.betAmount -= 10
        if (this.betAmount < 1) this.betAmount = 1
      }
    },
    
    // اختيار مستوى الخطورة
    selectRisk(level) {
      this.selectedRisk = level
      
      // تغيير لون الكرة حسب الخطورة
      switch(level) {
        case 'high':
          this.ballColor = '#ff4444'
          break
        case 'normal':
          this.ballColor = '#00ff88'
          break
        case 'low':
          this.ballColor = '#ffff44'
          break
      }
    },
    
    // إسقاط الكرة
    async dropBall() {
      if (this.isPlaying || this.betAmount > this.balance) return
      
      // خصم الرهان
      this.balance -= this.betAmount
      this.isPlaying = true
      
      // إعداد الكرة
      this.ball.active = true
      this.ball.x = this.dropPosition
      this.ball.y = 20
      
      // سرعة ابتدائية عشوائية
      this.velocityX = (Math.random() - 0.5) * 8
      this.velocityY = 2
      
      // تأثير مستوى الخطورة على الحركة
      switch(this.selectedRisk) {
        case 'high':
          this.velocityX *= 2
          break
        case 'low':
          this.velocityX *= 0.5
          break
      }
      
      // بدء الحركة
      this.startAnimation()
    },
    
    // حركة الكرة
    startAnimation() {
      const gravity = 0.3
      const friction = 0.98
      
      this.animationInterval = setInterval(() => {
        // تطبيق الجاذبية
        this.velocityY += gravity
        
        // تحديث الموضع
        this.ball.x += this.velocityX
        this.ball.y += this.velocityY
        
        // تطبيق الاحتكاك
        this.velocityX *= friction
        
        // ارتداد من الجوانب
        if (this.ball.x <= 10 || this.ball.x >= this.boardWidth - 10) {
          this.velocityX *= -0.8
          this.ball.x = Math.max(10, Math.min(this.boardWidth - 10, this.ball.x))
        }
        
        // التحقق من الوصول للأسفل
        if (this.ball.y >= 340) {
          this.endAnimation()
        }
      }, 16)
    },
    
    // نهاية الحركة
    endAnimation() {
      clearInterval(this.animationInterval)
      this.isPlaying = false
      
      // حساب السلة النهائية
      const slotIndex = Math.min(
        8,
        Math.max(0, Math.floor((this.ball.x - 20) / 40))
      )
      
      // المضاعف النهائي
      this.currentMultiplier = this.slots[slotIndex].multiplier
      
      // تأثير مستوى الخطورة
      if (this.selectedRisk === 'high') {
        if (Math.random() > 0.5) {
          this.currentMultiplier *= 1.5
        } else {
          this.currentMultiplier *= 0.7
        }
      } else if (this.selectedRisk === 'low') {
        this.currentMultiplier *= (0.8 + Math.random() * 0.4)
      }
      
      this.currentMultiplier = Math.max(0.1, this.currentMultiplier)
      
      // حساب المكسب
      this.resultAmount = this.betAmount * this.currentMultiplier
      this.balance += this.resultAmount
      
      // عرض النتيجة
      this.showGameResult()
    },
    
    // عرض نتيجة الجولة
    showGameResult() {
      if (this.resultAmount > this.betAmount) {
        this.resultTitle = 'WIN!'
        this.resultIcon = '🎉'
        this.resultClass = 'win'
      } else if (this.resultAmount === this.betAmount) {
        this.resultTitle = 'BREAK EVEN'
        this.resultIcon = '🤝'
        this.resultClass = 'draw'
      } else {
        this.resultTitle = 'LOSS'
        this.resultIcon = '😢'
        this.resultClass = 'loss'
      }
      
      this.showResult = true
      
      // إخفاء النتيجة بعد 3 ثواني
      setTimeout(() => {
        this.closeResult()
      }, 3000)
    },
    
    // إغلاق نافذة النتيجة
    closeResult() {
      this.showResult = false
      this.ball.active = false
    },
    
    // تحديث موقع الإسقاط
    updateDropPosition(event) {
      if (this.isPlaying) return
      
      const board = this.$refs.board
      if (!board) return
      
      const rect = board.getBoundingClientRect()
      const x = event.clientX - rect.left
      
      if (x >= 30 && x <= rect.width - 30) {
        this.dropPosition = x
      }
    }
  },
  
  mounted() {
    // إضافة مستمع لحركة الماوس على اللوحة
    const board = this.$refs.board
    if (board) {
      board.addEventListener('mousemove', this.updateDropPosition)
      board.addEventListener('click', this.updateDropPosition)
    }
  },
  
  beforeUnmount() {
    // تنظيف المستمعات
    const board = this.$refs.board
    if (board) {
      board.removeEventListener('mousemove', this.updateDropPosition)
      board.removeEventListener('click', this.updateDropPosition)
    }
    
    if (this.animationInterval) {
      clearInterval(this.animationInterval)
    }
  }
}
</script>

<style scoped>
.game-page {
  background: linear-gradient(135deg, #0a0a0a 0%, #1a1a1a 100%);
  min-height: 100vh;
  color: white;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* الهيدر */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 30px;
  background: rgba(0, 0, 0, 0.8);
  border-bottom: 1px solid #333;
}

.logo {
  font-size: 20px;
  font-weight: bold;
  color: #00ff88;
}

.header-menu {
  display: flex;
  gap: 10px;
}

.menu-btn {
  padding: 8px 20px;
  background: transparent;
  border: 1px solid #444;
  color: #ccc;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s;
}

.menu-btn:hover {
  background: #222;
  color: white;
}

.menu-btn.active {
  background: #00ff88;
  color: black;
  border-color: #00ff88;
}

.menu-btn.vip {
  background: linear-gradient(45deg, #ffd700, #ffaa00);
  color: black;
  border: none;
}

/* المحتوى الرئيسي */
.content {
  display: flex;
  padding: 20px;
  gap: 30px;
  max-width: 1400px;
  margin: 0 auto;
}

/* لوحة المضاعفات (اليمين) */
.multipliers-panel {
  flex: 0 0 300px;
  background: rgba(20, 20, 20, 0.9);
  border-radius: 15px;
  padding: 25px;
  border: 1px solid #333;
}

.multipliers-panel h1 {
  font-size: 42px;
  font-weight: bold;
  background: linear-gradient(45deg, #00ff88, #0088ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin: 0 0 5px 0;
  text-align: center;
}

.subtitle {
  font-size: 18px;
  color: #888;
  text-align: center;
  margin-bottom: 30px;
  letter-spacing: 2px;
}

.balance-card {
  background: linear-gradient(135deg, #1a1a1a, #2a2a2a);
  border: 2px solid #00ff88;
  border-radius: 12px;
  padding: 20px;
  text-align: center;
  margin-bottom: 30px;
}

.balance-label {
  font-size: 14px;
  color: #888;
  margin-bottom: 8px;
}

.balance-amount {
  font-size: 32px;
  font-weight: bold;
  color: #00ff88;
}

.multipliers-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.mult-card {
  background: rgba(40, 40, 40, 0.9);
  border-radius: 10px;
  padding: 15px;
  text-align: center;
  border: 1px solid #444;
  transition: transform 0.3s;
}

.mult-card:hover {
  transform: translateY(-5px);
  border-color: #00ff88;
}

.mult-value {
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 5px;
}

.mult-card:nth-child(3) .mult-value,
.mult-card:nth-child(7) .mult-value {
  color: #ff4444;
}

.mult-card:nth-child(4) .mult-value,
.mult-card:nth-child(5) .mult-value,
.mult-card:nth-child(6) .mult-value {
  color: #ffff44;
}

.mult-card:nth-child(1) .mult-value,
.mult-card:nth-child(2) .mult-value,
.mult-card:nth-child(8) .mult-value,
.mult-card:nth-child(9) .mult-value {
  color: #00ff88;
}

.mult-risk {
  font-size: 12px;
  color: #888;
}

/* لوحة اللعبة (اليسار) */
.game-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 30px;
}

/* لوحة البلينكو */
.plinko-board {
  flex: 1;
  background: rgba(10, 10, 10, 0.9);
  border-radius: 15px;
  padding: 30px;
  position: relative;
  border: 2px solid #333;
  min-height: 500px;
}

.pins-container {
  position: relative;
  height: 300px;
}

.pin-row {
  position: absolute;
  width: 100%;
}

.pin {
  position: absolute;
  width: 6px;
  height: 6px;
  background: white;
  border-radius: 50%;
  transform: translate(-50%, -50%);
}

.ball {
  position: absolute;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #00ff88;
  transform: translate(-50%, -50%);
  box-shadow: 0 0 20px currentColor;
  z-index: 10;
  transition: left 0.1s, top 0.1s;
}

.drop-line {
  position: absolute;
  top: 0;
  width: 2px;
  height: 100%;
  background: rgba(0, 255, 136, 0.3);
  z-index: 5;
}

.slots-container {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  justify-content: space-between;
  padding: 0 20px;
}

.slot {
  width: 40px;
  height: 60px;
  border-radius: 8px 8px 0 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  color: black;
}

.slot-text {
  font-size: 14px;
  font-weight: bold;
}

/* عناصر التحكم */
.controls {
  background: rgba(20, 20, 20, 0.9);
  border-radius: 15px;
  padding: 25px;
  border: 1px solid #333;
}

.bet-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  gap: 20px;
}

.bet-info {
  display: flex;
  flex-direction: column;
  gap: 5px;
  flex: 1;
}

.min-bet,
.max-bet {
  font-size: 14px;
  color: #888;
  text-align: center;
}

.bet-input-group {
  display: flex;
  align-items: center;
  gap: 15px;
  flex: 2;
  justify-content: center;
}

.bet-input {
  width: 150px;
  padding: 15px;
  font-size: 28px;
  text-align: center;
  background: rgba(0, 0, 0, 0.7);
  border: 2px solid #00ff88;
  border-radius: 10px;
  color: white;
  font-weight: bold;
}

.bet-btn {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  border: 2px solid #444;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  font-size: 24px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s;
}

.bet-btn:hover {
  border-color: #00ff88;
  background: rgba(0, 255, 136, 0.1);
}

.more-options {
  flex: 1;
  text-align: right;
}

.more-btn {
  padding: 12px 30px;
  background: rgba(0, 255, 136, 0.1);
  border: 1px solid #00ff88;
  color: #00ff88;
  border-radius: 8px;
  cursor: pointer;
}

.risk-controls {
  display: flex;
  gap: 15px;
  margin-bottom: 25px;
  justify-content: center;
}

.risk-btn {
  flex: 1;
  padding: 18px 0;
  border: 2px solid #444;
  background: rgba(0, 0, 0, 0.7);
  color: #ccc;
  border-radius: 10px;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s;
}

.risk-btn.active {
  transform: scale(1.05);
}

.risk-high.active {
  background: linear-gradient(135deg, #ff4444, #ff0000);
  border-color: #ff4444;
  color: white;
}

.risk-normal.active {
  background: linear-gradient(135deg, #00ff88, #008844);
  border-color: #00ff88;
  color: black;
}

.risk-low.active {
  background: linear-gradient(135deg, #ffff44, #ffaa00);
  border-color: #ffff44;
  color: black;
}

.drop-section {
  text-align: center;
}

.drop-btn {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  border: none;
  background: linear-gradient(135deg, #00ff88, #008844);
  color: black;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
}

.drop-btn:hover:not(:disabled) {
  transform: scale(1.1);
  box-shadow: 0 0 40px rgba(0, 255, 136, 0.5);
}

.drop-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.drop-icon {
  font-size: 40px;
  margin-bottom: 8px;
}

.drop-text {
  font-size: 20px;
  font-weight: bold;
}

/* نافذة النتيجة */
.result-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.result-modal {
  background: rgba(30, 30, 30, 0.95);
  border-radius: 20px;
  padding: 40px;
  text-align: center;
  min-width: 300px;
  border: 3px solid;
  position: relative;
  animation: popIn 0.5s;
}

@keyframes popIn {
  0% { transform: scale(0.5); opacity: 0; }
  100% { transform: scale(1); opacity: 1; }
}

.result-modal.win {
  border-color: #00ff88;
}

.result-modal.loss {
  border-color: #ff4444;
}

.result-modal.draw {
  border-color: #ffff44;
}

.result-icon {
  font-size: 60px;
  margin-bottom: 20px;
}

.result-title {
  font-size: 36px;
  font-weight: bold;
  margin-bottom: 10px;
}

.result-multiplier {
  font-size: 20px;
  color: #888;
  margin-bottom: 10px;
}

.result-amount {
  font-size: 32px;
  font-weight: bold;
  color: #00ff88;
}

.close-btn {
  position: absolute;
  top: 15px;
  right: 15px;
  background: none;
  border: none;
  color: #888;
  font-size: 24px;
  cursor: pointer;
  padding: 5px;
}

.close-btn:hover {
  color: white;
}

/* تصميم متجاوب */
@media (max-width: 1200px) {
  .content {
    flex-direction: column;
  }
  
  .multipliers-panel {
    flex: none;
    width: 100%;
  }
  
  .bet-controls {
    flex-direction: column;
  }
  
  .multipliers-grid {
    grid-template-columns: repeat(5, 1fr);
  }
}

@media (max-width: 768px) {
  .header {
    flex-direction: column;
    gap: 15px;
  }
  
  .header-menu {
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .multipliers-grid {
    grid-template-columns: repeat(3, 1fr);
  }
  
  .bet-input {
    width: 120px;
    font-size: 24px;
  }
  
  .drop-btn {
    width: 100px;
    height: 100px;
  }
}
</style>
