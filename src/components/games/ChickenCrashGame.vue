<template>
  <div class="chicken-game-container">
    <!-- عرض الرصيد الحالي -->
    <div class="current-balance">
      <i class="fas fa-coins"></i>
      <span>الرصيد: <strong>{{ balance.toFixed(2) }} USDT</strong></span>
    </div>

    <!-- رأس اللعبة -->
    <div class="game-header">
      <h2>🐔 CHICKEN ROAD GAME</h2>
      <div class="header-glow"></div>
    </div>

    <!-- محتوى اللعبة -->
    <div class="game-content">
      <div class="chicken-area">
        <div class="chicken" :class="{ walking: gameActive }">🐔</div>
        <div class="road-container">
          <div class="road">
            <div 
              v-for="(step, index) in steps" 
              :key="index"
              class="road-step"
              :class="{ 
                'active': currentStep === index, 
                'passed': index < currentStep,
                'danger': step >= 3
              }"
            >
              <span class="step-multiplier">x{{ step.toFixed(1) }}</span>
              <div v-if="index === currentStep" class="chicken-icon">🐔</div>
              <div v-if="index < currentStep" class="step-check">✓</div>
            </div>
          </div>
        </div>
      </div>

      <!-- منطقة الرهان -->
      <div class="bet-area">
        <div v-if="!gameActive && !gameOver" class="bet-controls">
          <div class="input-wrapper">
            <span class="input-icon">💰</span>
            <input 
              type="number"
              v-model.number="betAmount"
              placeholder="أدخل مبلغ الرهان"
              class="bet-input"
              min="0.01"
              step="0.01"
              :disabled="isBetLocked"
            />
            <span class="input-currency">USDT</span>
          </div>
          
          <div v-if="betAmount > balance" class="error-message">
            ❌ الرصيد غير كافي
          </div>
          
          <div v-if="gameError" class="error-message">
            ❌ {{ gameError }}
          </div>
          
          <button 
            @click="startGame"
            class="start-btn"
            :disabled="!isValidBet || isBetLocked"
          >
            <span>ابدأ الرحلة</span>
            <i class="fas fa-play"></i>
          </button>
        </div>

        <div v-if="gameActive" class="game-controls">
          <div class="multiplier-display">
            <span class="label">المضاعف الحالي</span>
            <span class="value">{{ currentMultiplier.toFixed(2) }}x</span>
          </div>
          
          <div class="potential-profit">
            <span class="label">الربح المحتمل</span>
            <span class="value">{{ (betAmount * currentMultiplier).toFixed(2) }} USDT</span>
          </div>

          <div class="action-buttons">
            <button @click="nextStep" class="next-btn">
              <i class="fas fa-arrow-right"></i>
              تقدم
            </button>
            <button @click="cashOut" class="cashout-btn">
              <i class="fas fa-hand-holding-usd"></i>
              سحب
            </button>
          </div>
        </div>

        <div v-if="gameOver" class="game-result" :class="{ 'win': gameWon, 'lose': !gameWon }">
          <div class="result-icon">{{ gameWon ? '🎉' : '💥' }}</div>
          <div class="result-message">{{ gameMessage }}</div>
          <div class="result-amount" v-if="gameWon">+{{ profitAmount.toFixed(2) }} USDT</div>
          <button @click="resetGame" class="play-again-btn">
            لعب مرة أخرى
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ChickenRoadGame',
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
      betAmount: null,
      gameActive: false,
      gameOver: false,
      gameWon: false,
      gameMessage: '',
      gameError: '',
      profitAmount: 0,
      isBetLocked: false,
      currentStep: 0,
      currentMultiplier: 1.0,
      steps: [1.0, 1.2, 1.5, 2.0, 2.5, 3.0, 4.0, 5.0, 6.0, 8.0, 10.0],
      crashBaseRate: 0.25
    }
  },
  computed: {
    isValidBet() {
      return this.betAmount && 
             this.betAmount > 0 && 
             this.betAmount <= this.balance
    }
  },
  methods: {
    startGame() {
      this.gameError = ''
      
      // التحقق من صحة الرهان
      if (!this.betAmount || this.betAmount <= 0) {
        this.gameError = 'الرجاء إدخال مبلغ الرهان'
        this.$emit('show-result', '❌ مبلغ الرهان غير صحيح', false)
        return
      }
      
      if (this.betAmount > this.balance) {
        this.gameError = 'الرصيد غير كافي'
        this.$emit('show-result', '❌ الرصيد غير كافي', false)
        return
      }

      // قفل الرهان
      this.isBetLocked = true

      // خصم الرهان من الرصيد
      const newBalance = this.balance - this.betAmount
      this.$emit('update-balance', newBalance)

      // بدء اللعبة
      this.gameActive = true
      this.gameOver = false
      this.currentStep = 0
      this.currentMultiplier = this.steps[0]
      
      this.$emit('show-result', '🚀 انطلقت الرحلة!', true)
    },

    nextStep() {
      // حساب فرصة الخسارة (تزيد كلما تقدمت)
      const loseChance = this.crashBaseRate + (this.currentStep * 0.04)
      
      if (Math.random() < loseChance) {
        // خسارة
        this.gameActive = false
        this.gameOver = true
        this.gameWon = false
        this.profitAmount = 0
        this.gameMessage = '💥 انفجر الطريق! خسرت الرهان'
        this.$emit('show-result', this.gameMessage, false)
        this.isBetLocked = false
        return
      }

      // الانتقال للخطوة التالية
      if (this.currentStep < this.steps.length - 1) {
        this.currentStep++
        this.currentMultiplier = this.steps[this.currentStep]
      } else {
        // وصل للنهاية (فوز تلقائي)
        this.cashOut()
      }
    },

    cashOut() {
      // حساب الربح
      this.profitAmount = this.betAmount * this.currentMultiplier
      
      // إضافة الربح للرصيد
      const newBalance = this.balance + this.profitAmount
      this.$emit('update-balance', newBalance)

      // عرض نتيجة الفوز
      this.gameActive = false
      this.gameOver = true
      this.gameWon = true
      this.gameMessage = `🎉 ربحت ${this.profitAmount.toFixed(2)} USDT`
      
      this.$emit('show-result', this.gameMessage, true)
      this.isBetLocked = false
    },

    resetGame() {
      // إعادة تعيين اللعبة
      this.gameActive = false
      this.gameOver = false
      this.betAmount = null
      this.currentStep = 0
      this.currentMultiplier = 1.0
      this.gameError = ''
      this.isBetLocked = false
    }
  }
}
</script>

<style scoped>
.chicken-game-container {
  background: linear-gradient(145deg, #1a1f30, #0f1422);
  border-radius: 30px;
  padding: 25px;
  border: 1px solid rgba(255, 215, 0, 0.3);
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.5);
  max-width: 550px;
  margin: 0 auto;
  position: relative;
  overflow: hidden;
}

.chicken-game-container::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255, 215, 0, 0.1) 0%, transparent 70%);
  animation: rotate 25s linear infinite;
  pointer-events: none;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.current-balance {
  text-align: center;
  margin-bottom: 20px;
  padding: 12px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 50px;
  border: 1px solid rgba(255, 215, 0, 0.2);
  color: #ffd700;
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.current-balance i {
  color: #ffd700;
  font-size: 18px;
}

.current-balance strong {
  color: #ffd700;
  font-size: 18px;
  margin: 0 5px;
}

.game-header {
  position: relative;
  margin-bottom: 25px;
  text-align: center;
}

.game-header h2 {
  font-size: 24px;
  font-weight: 800;
  background: linear-gradient(135deg, #ffd700, #ffed4a, #ffd700);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  position: relative;
  z-index: 1;
  letter-spacing: 1px;
}

.header-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 150px;
  height: 150px;
  background: radial-gradient(circle, rgba(255, 215, 0, 0.2) 0%, transparent 70%);
  filter: blur(30px);
  z-index: 0;
}

.game-content {
  display: flex;
  flex-direction: column;
  gap: 25px;
}

.chicken-area {
  background: rgba(0, 0, 0, 0.3);
  border-radius: 20px;
  padding: 20px;
  border: 1px solid rgba(255, 215, 0, 0.2);
}

.chicken {
  font-size: 90px;
  text-align: center;
  margin-bottom: 20px;
  filter: drop-shadow(0 0 20px #ffd700);
  transition: all 0.3s;
  animation: float 3s infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

.chicken.walking {
  animation: walk 0.5s infinite;
}

@keyframes walk {
  0%, 100% { transform: translateX(0) rotate(0deg); }
  25% { transform: translateX(-5px) rotate(-5deg); }
  75% { transform: translateX(5px) rotate(5deg); }
}

.road-container {
  overflow-x: auto;
  white-space: nowrap;
  padding: 10px 0;
  scrollbar-width: thin;
  scrollbar-color: #ffd700 #1a1f30;
  -webkit-overflow-scrolling: touch;
}

.road-container::-webkit-scrollbar {
  height: 8px;
}

.road-container::-webkit-scrollbar-track {
  background: #1a1f30;
  border-radius: 10px;
}

.road-container::-webkit-scrollbar-thumb {
  background: #ffd700;
  border-radius: 10px;
}

.road {
  display: inline-flex;
  gap: 10px;
  padding: 5px;
  min-width: min-content;
}

.road-step {
  position: relative;
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-width: 80px;
  height: 100px;
  background: linear-gradient(145deg, #252b3d, #1a1f30);
  border-radius: 15px;
  padding: 10px 5px;
  border: 2px solid rgba(255, 215, 0, 0.2);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  transition: all 0.3s ease;
}

.road-step.active {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  transform: scale(1.1);
  box-shadow: 0 10px 30px rgba(255, 215, 0, 0.5);
  z-index: 2;
  border-color: transparent;
}

.road-step.passed {
  border-color: #4caf50;
  opacity: 0.8;
  background: linear-gradient(145deg, #1e2430, #151a24);
}

.road-step.danger {
  border-color: #f44336;
}

.step-multiplier {
  font-weight: 700;
  font-size: 18px;
  color: #ffd700;
  margin-bottom: 5px;
}

.road-step.active .step-multiplier {
  color: #0a0f1e;
}

.chicken-icon {
  font-size: 24px;
  margin-top: 5px;
  animation: bounce 0.5s infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}

.step-check {
  position: absolute;
  top: 5px;
  right: 5px;
  color: #4caf50;
  font-size: 18px;
  font-weight: 700;
  animation: checkPop 0.3s ease;
}

@keyframes checkPop {
  0% { transform: scale(0); }
  80% { transform: scale(1.2); }
  100% { transform: scale(1); }
}

.bet-area {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 20px;
  padding: 20px;
  border: 1px solid rgba(255, 215, 0, 0.2);
}

.bet-controls {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
}

.input-wrapper {
  position: relative;
  width: 100%;
  max-width: 300px;
}

.input-icon {
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

.input-currency {
  position: absolute;
  left: 15px;
  top: 50%;
  transform: translateY(-50%);
  color: #ffd700;
  font-weight: 600;
  font-size: 14px;
}

.start-btn {
  background: linear-gradient(135deg, #ffd700, #ffed4a, #ffd700);
  color: #0a0f1e;
  border: none;
  padding: 15px 35px;
  border-radius: 50px;
  font-weight: 700;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  max-width: 300px;
  justify-content: center;
  box-shadow: 0 10px 25px rgba(255, 215, 0, 0.4);
}

.start-btn:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 15px 35px rgba(255, 215, 0, 0.6);
}

.start-btn:disabled {
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
  max-width: 300px;
  text-align: center;
}

.game-controls {
  display: flex;
  flex-direction: column;
  gap: 20px;
  align-items: center;
}

.multiplier-display, .potential-profit {
  text-align: center;
  padding: 15px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 15px;
  width: 100%;
  max-width: 300px;
  border: 1px solid rgba(255, 215, 0, 0.2);
}

.multiplier-display .label, .potential-profit .label {
  display: block;
  color: #8a8f9c;
  font-size: 14px;
  margin-bottom: 8px;
}

.multiplier-display .value {
  font-size: 42px;
  font-weight: 800;
  color: #ffd700;
  text-shadow: 0 0 20px #ffd700;
}

.potential-profit .value {
  font-size: 24px;
  font-weight: 700;
  color: #4caf50;
  text-shadow: 0 0 10px #4caf50;
}

.action-buttons {
  display: flex;
  gap: 15px;
  width: 100%;
  max-width: 300px;
  justify-content: center;
}

.next-btn, .cashout-btn {
  flex: 1;
  padding: 12px 20px;
  border: none;
  border-radius: 50px;
  font-weight: 700;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.next-btn {
  background: linear-gradient(135deg, #f44336, #d32f2f);
  color: white;
  box-shadow: 0 5px 15px rgba(244, 67, 54, 0.3);
}

.cashout-btn {
  background: linear-gradient(135deg, #4caf50, #45a049);
  color: white;
  box-shadow: 0 5px 15px rgba(76, 175, 80, 0.3);
}

.next-btn:hover, .cashout-btn:hover {
  transform: translateY(-2px);
  filter: brightness(1.1);
}

.game-result {
  text-align: center;
  padding: 25px;
  border-radius: 20px;
  animation: slideUp 0.5s;
}

.game-result.win {
  background: rgba(76, 175, 80, 0.15);
  border: 2px solid #4caf50;
  box-shadow: 0 0 30px rgba(76, 175, 80, 0.2);
}

.game-result.lose {
  background: rgba(244, 67, 54, 0.15);
  border: 2px solid #f44336;
  box-shadow: 0 0 30px rgba(244, 67, 54, 0.2);
}

.result-icon {
  font-size: 60px;
  margin-bottom: 15px;
  animation: popIn 0.5s;
}

@keyframes popIn {
  0% { transform: scale(0); }
  80% { transform: scale(1.2); }
  100% { transform: scale(1); }
}

.result-message {
  font-size: 20px;
  font-weight: 700;
  margin-bottom: 10px;
}

.game-result.win .result-message {
  color: #4caf50;
}

.game-result.lose .result-message {
  color: #f44336;
}

.result-amount {
  font-size: 28px;
  font-weight: 800;
  color: #4caf50;
  margin-bottom: 20px;
  text-shadow: 0 0 15px #4caf50;
}

.play-again-btn {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  border: none;
  padding: 12px 30px;
  border-radius: 50px;
  font-weight: 700;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 5px 15px rgba(255, 215, 0, 0.3);
}

.play-again-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(255, 215, 0, 0.4);
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

@media (max-width: 480px) {
  .chicken-game-container {
    padding: 15px;
  }

  .game-header h2 {
    font-size: 20px;
  }

  .chicken {
    font-size: 70px;
  }

  .road-step {
    min-width: 70px;
    height: 90px;
  }

  .step-multiplier {
    font-size: 16px;
  }

  .multiplier-display .value {
    font-size: 36px;
  }

  .action-buttons {
    flex-direction: column;
  }

  .next-btn, .cashout-btn {
    width: 100%;
  }

  .result-icon {
    font-size: 50px;
  }

  .result-message {
    font-size: 18px;
  }

  .result-amount {
    font-size: 24px;
  }
}
</style>
