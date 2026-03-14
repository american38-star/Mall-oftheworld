<template>
  <div class="game-page">
    <!-- الرصيد -->
    <div class="top-bar">
      <div class="balance-gold">
        <i class="fas fa-coins"></i>
        <span>رصيدك: <strong>{{ balance.toFixed(2) }} USDT</strong></span>
      </div>
    </div>

    <!-- رسالة الفوز/الخسارة -->
    <transition name="slide-fade">
      <div v-if="showResultMessage" class="result-message" :class="resultType">
        <i :class="resultIcon"></i>
        <span>{{ resultText }}</span>
      </div>
    </transition>

    <!-- زر الرجوع عندما تكون اللعبة مفتوحة -->
    <div v-if="gameOpened" class="back-button-container">
      <button @click="closeGame" class="back-button">
        <i class="fas fa-arrow-right"></i>
        رجوع إلى الألعاب
      </button>
    </div>

    <!-- التبويبات - تظهر فقط عندما لا تكون اللعبة مفتوحة -->
    <div v-if="!gameOpened" class="tabs">
      <button 
        v-for="tab in gamesList" 
        :key="tab.id"
        :class="{active: selectedGame === tab.id}" 
        @click="openGame(tab.id)"
      >
        <span class="tab-icon">{{ tab.icon }}</span>
        <span class="tab-text">{{ tab.name }}</span>
      </button>
    </div>

    <!-- عرض اللعبة المختارة - تظهر بملء الشاشة -->
    <div v-if="gameOpened" class="game-fullscreen">
      <!-- Şanslı Rulet (لعبة الروليت) -->
      <div v-if="selectedGame === 'rulet'" class="rulet-card">
        <div class="rulet-header">
          <h2>🎰 ŞANSLI RULET</h2>
          <div class="header-glow"></div>
        </div>

        <div class="rulet-content">
          <!-- عجلة الروليت -->
          <div class="wheel-container">
            <div class="wheel" :style="{ transform: `rotate(${wheelRotation}deg)` }">
              <div v-for="(number, index) in ruletNumbers" 
                   :key="index"
                   class="wheel-number"
                   :style="{ transform: `rotate(${index * 9.73}deg)` }"
                   :class="getNumberColorClass(number)">
                <span>{{ number }}</span>
              </div>
            </div>
            <div class="wheel-pointer">▼</div>
          </div>

          <!-- لوحة الرهان -->
          <div class="betting-board">
            <div class="bet-type-selector">
              <button 
                v-for="type in betTypes" 
                :key="type.value"
                @click="selectedBetType = type.value"
                :class="{ active: selectedBetType === type.value }"
                class="bet-type-btn"
              >
                {{ type.name }}
                <span class="multiplier">x{{ type.multiplier }}</span>
              </button>
            </div>

            <div class="number-buttons" v-if="selectedBetType === 'number'">
              <button 
                v-for="n in 36" 
                :key="n"
                @click="selectNumber(n)"
                :class="['number-btn', getNumberColorClass(n), { active: selectedNumber === n }]"
              >
                {{ n }}
              </button>
            </div>

            <div class="color-buttons" v-if="selectedBetType === 'color'">
              <button 
                @click="selectColor('red')"
                :class="['color-btn', 'red-btn', { active: selectedColor === 'red' }]"
              >
                <span class="color-dot red"></span>
                أحمر
              </button>
              <button 
                @click="selectColor('black')"
                :class="['color-btn', 'black-btn', { active: selectedColor === 'black' }]"
              >
                <span class="color-dot black"></span>
                أسود
              </button>
            </div>

            <div class="parity-buttons" v-if="selectedBetType === 'parity'">
              <button 
                @click="selectParity('even')"
                :class="['parity-btn', { active: selectedParity === 'even' }]"
              >
                زوجي
              </button>
              <button 
                @click="selectParity('odd')"
                :class="['parity-btn', { active: selectedParity === 'odd' }]"
              >
                فردي
              </button>
            </div>

            <div class="half-buttons" v-if="selectedBetType === 'half'">
              <button 
                @click="selectHalf('1-18')"
                :class="['half-btn', { active: selectedHalf === '1-18' }]"
              >
                1 - 18
              </button>
              <button 
                @click="selectHalf('19-36')"
                :class="['half-btn', { active: selectedHalf === '19-36' }]"
              >
                19 - 36
              </button>
            </div>

            <!-- حقل الرهان -->
            <div class="bet-section">
              <div class="bet-input-wrapper">
                <span class="bet-icon">💰</span>
                <input
                  type="number"
                  v-model.number="betAmount"
                  placeholder="مبلغ الرهان"
                  class="bet-input"
                  min="0.01"
                  step="0.01"
                  :disabled="isSpinning"
                />
                <span class="bet-currency">USDT</span>
              </div>

              <div v-if="betAmount > balance" class="error-message">
                ❌ الرصيد غير كافي
              </div>

              <div v-if="gameError" class="error-message">
                ❌ {{ gameError }}
              </div>

              <button 
                @click="spinWheel"
                class="spin-btn"
                :disabled="!canSpin || isSpinning"
              >
                <span v-if="!isSpinning">🎰 دير العجلة</span>
                <span v-else>جاري الدوران...</span>
              </button>
            </div>

            <!-- نتيجة الجولة -->
            <div v-if="lastResult" class="last-result" :class="{ 'win': lastResult.isWin, 'lose': !lastResult.isWin }">
              <div class="result-number" :class="getNumberColorClass(lastResult.number)">
                {{ lastResult.number }}
              </div>
              <div class="result-details">
                <span class="result-text">{{ lastResult.message }}</span>
                <span v-if="lastResult.isWin" class="result-amount">+{{ lastResult.winAmount.toFixed(2) }} USDT</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { auth, db } from "../firebase"
import { doc, getDoc, updateDoc } from "firebase/firestore"

export default {
  name: "Games",
  
  data() {
    return {
      gameOpened: false,
      selectedGame: "rulet",
      showResultMessage: false,
      resultType: '',
      resultIcon: '',
      resultText: '',
      resultTimeout: null,
      
      gamesList: [
        { id: 'rulet', name: 'Şanslı Rulet', icon: '🎰' }
      ],
      
      balance: 0,
      gameError: "",
      
      // بيانات لعبة الروليت
      ruletNumbers: Array.from({ length: 37 }, (_, i) => i), // 0-36
      wheelRotation: 0,
      isSpinning: false,
      betAmount: null,
      
      // أنواع الرهان
      betTypes: [
        { name: 'رقم محدد', value: 'number', multiplier: 36 },
        { name: 'لون', value: 'color', multiplier: 2 },
        { name: 'زوجي/فردي', value: 'parity', multiplier: 2 },
        { name: 'نصف', value: 'half', multiplier: 2 }
      ],
      
      selectedBetType: 'number',
      selectedNumber: null,
      selectedColor: null,
      selectedParity: null,
      selectedHalf: null,
      
      lastResult: null,
      
      // أرقام حمراء في الروليت
      redNumbers: [1, 3, 5, 7, 9, 12, 14, 16, 18, 19, 21, 23, 25, 27, 30, 32, 34, 36]
    }
  },
  
  computed: {
    canSpin() {
      if (!this.betAmount || this.betAmount <= 0 || this.betAmount > this.balance) return false
      if (this.isSpinning) return false
      
      switch(this.selectedBetType) {
        case 'number':
          return this.selectedNumber !== null
        case 'color':
          return this.selectedColor !== null
        case 'parity':
          return this.selectedParity !== null
        case 'half':
          return this.selectedHalf !== null
        default:
          return false
      }
    },
    
    currentMultiplier() {
      const betType = this.betTypes.find(t => t.value === this.selectedBetType)
      return betType ? betType.multiplier : 1
    }
  },
  
  async created() {
    const user = auth.currentUser
    if (!user) return
    
    const snap = await getDoc(doc(db, "users", user.uid))
    if (snap.exists()) {
      this.balance = Number(snap.data().balance || 0)
    }
  },
  
  methods: {
    openGame(gameId) {
      this.selectedGame = gameId
      this.gameOpened = true
      this.gameError = ""
    },
    
    closeGame() {
      this.gameOpened = false
    },
    
    async updateBalance(newBalance) {
      this.balance = newBalance
      const user = auth.currentUser
      if (user) {
        await updateDoc(doc(db, "users", user.uid), {
          balance: this.balance
        })
      }
    },
    
    showResult(message, isWin) {
      if (this.resultTimeout) clearTimeout(this.resultTimeout)
      
      this.resultText = message
      this.resultType = isWin ? 'win-message' : 'lose-message'
      this.resultIcon = isWin ? 'fas fa-trophy' : 'fas fa-skull'
      this.showResultMessage = true
      
      this.resultTimeout = setTimeout(() => {
        this.showResultMessage = false
      }, 2000)
    },
    
    // دوال لعبة الروليت
    getNumberColorClass(number) {
      if (number === 0) return 'green'
      return this.redNumbers.includes(number) ? 'red' : 'black'
    },
    
    selectNumber(number) {
      this.selectedNumber = number
      this.selectedBetType = 'number'
    },
    
    selectColor(color) {
      this.selectedColor = color
      this.selectedBetType = 'color'
    },
    
    selectParity(parity) {
      this.selectedParity = parity
      this.selectedBetType = 'parity'
    },
    
    selectHalf(half) {
      this.selectedHalf = half
      this.selectedBetType = 'half'
    },
    
    async spinWheel() {
      if (!this.canSpin) return
      
      this.gameError = ''
      this.isSpinning = true
      
      // خصم الرهان
      this.balance -= this.betAmount
      await this.updateBalance(this.balance)
      
      // رقم الفائز العشوائي
      const winningNumber = Math.floor(Math.random() * 37)
      
      // دوران العجلة
      const spins = 5 + Math.floor(Math.random() * 5)
      const targetRotation = 360 * spins + (winningNumber * 9.73)
      
      // أنيميشن الدوران
      const startTime = Date.now()
      const duration = 3000
      
      const animate = () => {
        const elapsed = Date.now() - startTime
        const progress = Math.min(elapsed / duration, 1)
        
        // منحنى التباطؤ
        const easeProgress = 1 - Math.pow(1 - progress, 3)
        this.wheelRotation = targetRotation * easeProgress
        
        if (progress < 1) {
          requestAnimationFrame(animate)
        } else {
          this.finishSpin(winningNumber)
        }
      }
      
      requestAnimationFrame(animate)
    },
    
    async finishSpin(winningNumber) {
      this.isSpinning = false
      
      // التحقق من الفوز
      let isWin = false
      let winAmount = 0
      let message = ''
      
      switch(this.selectedBetType) {
        case 'number':
          isWin = winningNumber === this.selectedNumber
          winAmount = isWin ? this.betAmount * 36 : 0
          message = isWin ? `🎉 الفائز: ${winningNumber}` : `😢 الفائز: ${winningNumber}`
          break
          
        case 'color':
          const winningColor = this.getNumberColorClass(winningNumber)
          isWin = winningColor === this.selectedColor
          winAmount = isWin ? this.betAmount * 2 : 0
          message = isWin ? `🎉 الفائز: ${winningNumber} ${winningColor === 'red' ? 'أحمر' : 'أسود'}` : `😢 الفائز: ${winningNumber} ${winningColor === 'red' ? 'أحمر' : 'أسود'}`
          break
          
        case 'parity':
          if (winningNumber === 0) {
            isWin = false
          } else {
            const isEven = winningNumber % 2 === 0
            isWin = (this.selectedParity === 'even' && isEven) || (this.selectedParity === 'odd' && !isEven)
          }
          winAmount = isWin ? this.betAmount * 2 : 0
          message = isWin ? `🎉 الفائز: ${winningNumber}` : `😢 الفائز: ${winningNumber}`
          break
          
        case 'half':
          if (winningNumber === 0) {
            isWin = false
          } else {
            const isLow = winningNumber <= 18
            isWin = (this.selectedHalf === '1-18' && isLow) || (this.selectedHalf === '19-36' && !isLow)
          }
          winAmount = isWin ? this.betAmount * 2 : 0
          message = isWin ? `🎉 الفائز: ${winningNumber}` : `😢 الفائز: ${winningNumber}`
          break
      }
      
      // تحديث الرصيد في حالة الفوز
      if (isWin) {
        this.balance += winAmount
        await this.updateBalance(this.balance)
        this.showResult(`🎉 ربحت ${winAmount.toFixed(2)} USDT`, true)
      } else {
        this.showResult(`😢 خسرت الرهان`, false)
      }
      
      // حفظ النتيجة الأخيرة
      this.lastResult = {
        number: winningNumber,
        isWin,
        winAmount,
        message
      }
      
      // إعادة تعيين الرهان بعد 3 ثواني
      setTimeout(() => {
        this.betAmount = null
        this.selectedNumber = null
        this.selectedColor = null
        this.selectedParity = null
        this.selectedHalf = null
        this.selectedBetType = 'number'
      }, 3000)
    }
  }
}
</script>

<style scoped>
/* ===== التصميم العام ===== */
.game-page {
  background: linear-gradient(135deg, #0a0f1e 0%, #1a1f2f 100%);
  min-height: 100vh;
  color: #ffffff;
  padding: 15px;
  text-align: center;
  direction: rtl;
  font-family: 'Montserrat', 'Cairo', sans-serif;
  position: relative;
  overflow-x: hidden;
}

/* ===== الشريط العلوي ===== */
.top-bar {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  z-index: 10;
  position: relative;
}

.balance-gold {
  background: linear-gradient(145deg, #1e2333, #131826);
  padding: 12px 30px;
  border-radius: 100px;
  border: 1px solid #ffd700;
  box-shadow: 0 5px 20px rgba(255, 215, 0, 0.2), inset 0 0 10px rgba(255, 215, 0, 0.1);
  display: flex;
  align-items: center;
  gap: 12px;
  backdrop-filter: blur(10px);
}

.balance-gold i {
  color: #ffd700;
  font-size: 22px;
  filter: drop-shadow(0 0 8px #ffd700);
}

.balance-gold strong {
  color: #ffd700;
  font-size: 20px;
  margin-right: 5px;
  text-shadow: 0 0 15px rgba(255, 215, 0, 0.5);
}

/* ===== رسالة النتيجة ===== */
.result-message {
  position: fixed;
  top: 80px;
  left: 50%;
  transform: translateX(-50%);
  padding: 15px 30px;
  border-radius: 50px;
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 18px;
  font-weight: 700;
  z-index: 9999;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
  border: 1px solid;
  backdrop-filter: blur(10px);
  min-width: 280px;
  justify-content: center;
}

.win-message {
  background: linear-gradient(145deg, #1a2f1a, #0f1f0f);
  border-color: #4caf50;
  box-shadow: 0 0 30px rgba(76, 175, 80, 0.3);
  color: #4caf50;
}

.win-message i {
  color: #4caf50;
  filter: drop-shadow(0 0 8px #4caf50);
}

.lose-message {
  background: linear-gradient(145deg, #2f1a1a, #1f0f0f);
  border-color: #f44336;
  box-shadow: 0 0 30px rgba(244, 67, 54, 0.3);
  color: #f44336;
}

.lose-message i {
  color: #f44336;
  filter: drop-shadow(0 0 8px #f44336);
}

.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateX(-50%) translateY(-20px);
  opacity: 0;
}

/* ===== زر الرجوع ===== */
.back-button-container {
  margin-bottom: 20px;
  text-align: right;
}

.back-button {
  background: linear-gradient(145deg, #1e2333, #131826);
  color: #ffd700;
  border: 1px solid #ffd700;
  padding: 10px 25px;
  border-radius: 50px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.back-button:hover {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  transform: translateX(-5px);
  box-shadow: 0 8px 25px rgba(255, 215, 0, 0.4);
}

/* ===== التبويبات ===== */
.tabs {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
  margin-bottom: 25px;
  max-height: 200px;
  overflow-y: auto;
  padding: 15px;
  background: rgba(30, 35, 51, 0.7);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  border: 1px solid rgba(255, 215, 0, 0.2);
  scrollbar-width: thin;
  scrollbar-color: #ffd700 #1e2333;
}

.tabs button {
  padding: 12px 20px;
  border-radius: 50px;
  background: linear-gradient(145deg, #1e2333, #131826);
  color: #ffffff;
  border: 1px solid rgba(255, 215, 0, 0.3);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.tabs button:hover {
  border-color: #ffd700;
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(255, 215, 0, 0.3);
}

.tabs .active {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  border: none;
  box-shadow: 0 5px 25px rgba(255, 215, 0, 0.5);
}

/* ===== وضع ملء الشاشة ===== */
.game-fullscreen {
  min-height: calc(100vh - 150px);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  padding: 10px 0;
}

/* ===== بطاقة الروليت ===== */
.rulet-card {
  background: rgba(20, 25, 40, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 40px;
  padding: 25px;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  border: 1px solid rgba(255, 215, 0, 0.3);
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.5), inset 0 0 50px rgba(255, 215, 0, 0.1);
  position: relative;
  overflow: hidden;
}

.rulet-header {
  position: relative;
  margin-bottom: 25px;
  text-align: center;
}

.rulet-header h2 {
  font-size: 28px;
  font-weight: 800;
  background: linear-gradient(135deg, #ffd700, #ffed4a, #ffd700);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  position: relative;
  z-index: 1;
  letter-spacing: 2px;
}

.header-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 200px;
  height: 200px;
  background: radial-gradient(circle, rgba(255, 215, 0, 0.2) 0%, transparent 70%);
  filter: blur(40px);
  z-index: 0;
}

/* ===== عجلة الروليت ===== */
.wheel-container {
  position: relative;
  width: 300px;
  height: 300px;
  margin: 20px auto;
}

.wheel {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: linear-gradient(145deg, #2a2f3a, #1a1f2a);
  border: 4px solid #ffd700;
  position: relative;
  transition: transform 3s cubic-bezier(0.25, 0.1, 0.15, 1);
  box-shadow: 0 0 40px rgba(255, 215, 0, 0.3);
  overflow: hidden;
}

.wheel-number {
  position: absolute;
  width: 50%;
  height: 50%;
  transform-origin: bottom right;
  left: 50%;
  top: 50%;
  margin-left: -50%;
  margin-top: -50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  font-weight: 700;
  color: white;
  text-shadow: 0 0 5px black;
}

.wheel-number.red {
  color: #f44336;
}

.wheel-number.black {
  color: #ffffff;
}

.wheel-number.green {
  color: #4caf50;
}

.wheel-pointer {
  position: absolute;
  top: -15px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 40px;
  color: #ffd700;
  filter: drop-shadow(0 0 10px #ffd700);
  z-index: 10;
  animation: pointerPulse 1.5s infinite;
}

@keyframes pointerPulse {
  0%, 100% { transform: translateX(-50%) scale(1); }
  50% { transform: translateX(-50%) scale(1.1); }
}

/* ===== لوحة الرهان ===== */
.betting-board {
  background: rgba(0, 0, 0, 0.3);
  border-radius: 30px;
  padding: 20px;
  border: 1px solid rgba(255, 215, 0, 0.2);
}

.bet-type-selector {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
  margin-bottom: 20px;
}

.bet-type-btn {
  padding: 10px 20px;
  border-radius: 50px;
  background: linear-gradient(145deg, #1e2333, #131826);
  color: white;
  border: 1px solid rgba(255, 215, 0, 0.3);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 100px;
}

.bet-type-btn .multiplier {
  font-size: 12px;
  color: #ffd700;
  margin-top: 5px;
}

.bet-type-btn.active {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  border: none;
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.5);
}

.bet-type-btn.active .multiplier {
  color: #0a0f1e;
}

/* ===== أزرار الأرقام ===== */
.number-buttons {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 8px;
  margin-bottom: 20px;
}

.number-btn {
  aspect-ratio: 1;
  border-radius: 10px;
  border: 1px solid rgba(255, 215, 0, 0.3);
  background: linear-gradient(145deg, #1e2333, #131826);
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 14px;
}

.number-btn.red {
  background: linear-gradient(145deg, #f44336, #d32f2f);
}

.number-btn.black {
  background: linear-gradient(145deg, #333333, #222222);
}

.number-btn.active {
  transform: scale(1.1);
  box-shadow: 0 0 20px #ffd700;
  border-color: #ffd700;
  z-index: 2;
}

/* ===== أزرار الألوان ===== */
.color-buttons {
  display: flex;
  gap: 15px;
  justify-content: center;
  margin-bottom: 20px;
}

.color-btn {
  flex: 1;
  padding: 15px;
  border-radius: 15px;
  border: none;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  max-width: 150px;
}

.color-btn.red-btn {
  background: linear-gradient(135deg, #f44336, #d32f2f);
  color: white;
}

.color-btn.black-btn {
  background: linear-gradient(135deg, #333333, #222222);
  color: white;
}

.color-btn.active {
  transform: scale(1.05);
  box-shadow: 0 0 20px #ffd700;
}

.color-dot {
  width: 20px;
  height: 20px;
  border-radius: 50%;
}

.color-dot.red {
  background: #f44336;
  box-shadow: 0 0 10px #f44336;
}

.color-dot.black {
  background: #000000;
  box-shadow: 0 0 10px #000000;
}

/* ===== أزرار الزوجي/الفردي ===== */
.parity-buttons {
  display: flex;
  gap: 15px;
  justify-content: center;
  margin-bottom: 20px;
}

.parity-btn {
  flex: 1;
  padding: 15px;
  border-radius: 15px;
  border: none;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s;
  background: linear-gradient(145deg, #1e2333, #131826);
  color: white;
  border: 1px solid rgba(255, 215, 0, 0.3);
  max-width: 150px;
}

.parity-btn.active {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  border: none;
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.5);
}

/* ===== أزرار النصف ===== */
.half-buttons {
  display: flex;
  gap: 15px;
  justify-content: center;
  margin-bottom: 20px;
}

.half-btn {
  flex: 1;
  padding: 15px;
  border-radius: 15px;
  border: none;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s;
  background: linear-gradient(145deg, #1e2333, #131826);
  color: white;
  border: 1px solid rgba(255, 215, 0, 0.3);
  max-width: 150px;
}

.half-btn.active {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  border: none;
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.5);
}

/* ===== قسم الرهان ===== */
.bet-section {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid rgba(255, 215, 0, 0.2);
}

.bet-input-wrapper {
  position: relative;
  width: 100%;
  max-width: 280px;
}

.bet-icon {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 18px;
  z-index: 2;
  color: #ffd700;
}

.bet-input {
  width: 100%;
  padding: 15px 45px 15px 70px;
  border-radius: 50px;
  background: linear-gradient(145deg, #1e2333, #131826);
  border: 2px solid rgba(255, 215, 0, 0.3);
  color: white;
  font-size: 16px;
  text-align: center;
  transition: all 0.3s;
}

.bet-input:focus {
  outline: none;
  border-color: #ffd700;
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.2);
}

.bet-input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.bet-currency {
  position: absolute;
  left: 15px;
  top: 50%;
  transform: translateY(-50%);
  color: #ffd700;
  font-weight: 600;
  font-size: 14px;
}

.spin-btn {
  background: linear-gradient(135deg, #ffd700, #ffed4a, #ffd700);
  color: #0a0f1e;
  border: none;
  padding: 15px 35px;
  border-radius: 50px;
  font-weight: 700;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.3s;
  width: 100%;
  max-width: 280px;
  box-shadow: 0 10px 25px rgba(255, 215, 0, 0.4);
}

.spin-btn:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 15px 35px rgba(255, 215, 0, 0.6);
}

.spin-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.error-message {
  color: #f44336;
  font-size: 14px;
  padding: 10px 20px;
  background: rgba(244, 67, 54, 0.1);
  border-radius: 50px;
  border: 1px solid #f44336;
  width: 100%;
  max-width: 280px;
  text-align: center;
}

/* ===== نتيجة الجولة ===== */
.last-result {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 15px;
  border-radius: 20px;
  margin-top: 20px;
  animation: slideUp 0.5s;
}

.last-result.win {
  background: rgba(76, 175, 80, 0.15);
  border: 1px solid #4caf50;
}

.last-result.lose {
  background: rgba(244, 67, 54, 0.15);
  border: 1px solid #f44336;
}

.result-number {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: 700;
  color: white;
}

.result-number.red {
  background: #f44336;
}

.result-number.black {
  background: #222222;
}

.result-number.green {
  background: #4caf50;
}

.result-details {
  flex: 1;
  text-align: right;
}

.result-text {
  display: block;
  font-size: 14px;
  color: #8a8f9c;
  margin-bottom: 5px;
}

.result-amount {
  font-size: 18px;
  font-weight: 700;
  color: #4caf50;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* ===== تحسينات الجوال ===== */
@media (max-width: 480px) {
  .game-page {
    padding: 10px;
  }

  .tabs button {
    padding: 10px 15px;
    font-size: 12px;
  }

  .rulet-card {
    padding: 15px;
  }

  .rulet-header h2 {
    font-size: 22px;
  }

  .wheel-container {
    width: 250px;
    height: 250px;
  }

  .wheel-number {
    font-size: 12px;
  }

  .number-buttons {
    grid-template-columns: repeat(4, 1fr);
  }

  .bet-type-selector {
    flex-direction: column;
    align-items: stretch;
  }

  .bet-type-btn {
    width: 100%;
  }

  .color-buttons,
  .parity-buttons,
  .half-buttons {
    flex-direction: column;
    align-items: center;
  }

  .color-btn,
  .parity-btn,
  .half-btn {
    width: 100%;
    max-width: none;
  }

  .result-number {
    width: 40px;
    height: 40px;
    font-size: 16px;
  }
}
</style>
