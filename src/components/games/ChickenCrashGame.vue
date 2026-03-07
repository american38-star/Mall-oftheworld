<template>
  <div class="chicken-game-container">
    <!-- رأس اللعبة -->
    <div class="game-header">
      <h2>🐔 CHICKEN ROAD GAME</h2>
      <div class="header-glow"></div>
    </div>

    <!-- عرض الرصيد الحالي -->
    <div class="current-balance">
      الرصيد الحالي: <strong>{{ balance.toFixed(2) }} USDT</strong>
    </div>

    <!-- محتوى اللعبة -->
    <div class="game-content">
      <div class="chicken-area">
        <div class="chicken" :class="{ walking: gameActive }">🐔</div>
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
              placeholder="مبلغ الرهان"
              class="bet-input"
              min="0.01"
              step="0.01"
            >
            <span class="input-currency">USDT</span>
          </div>
          
          <button 
            @click="startGame" 
            class="start-btn"
            :disabled="!isValidBet"
          >
            <span>ابدأ اللعبة</span>
            <i class="fas fa-play"></i>
          </button>

          <div v-if="betAmount > balance" class="error-message">
            ❌ الرصيد غير كافي
          </div>
        </div>

        <div v-if="gameActive" class="game-controls">
          <div class="multiplier-display">
            <span class="label">المضاعف الحالي</span>
            <span class="value">{{ currentMultiplier.toFixed(2) }}x</span>
          </div>
          
          <div class="potential-profit">
            الربح المحتمل: <strong>{{ (betAmount * currentMultiplier).toFixed(2) }} USDT</strong>
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
      // التحقق من صحة الرهان
      if (!this.isValidBet) {
        this.$emit('show-result', '❌ مبلغ الرهان غير صحيح', false)
        return
      }

      // خصم الرهان من الرصيد (عن طريق emit)
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
        // خسارة 💥
        this.gameActive = false
        this.gameOver = true
        this.gameWon = false
        this.gameMessage = '💥 انفجرت! خسرت الرهان'
        this.$emit('show-result', this.gameMessage, false)
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
      const profit = this.betAmount * this.currentMultiplier
      
      // إضافة الربح للرصيد
      const newBalance = this.balance + profit
      this.$emit('update-balance', newBalance)

      // عرض نتيجة الفوز
      this.gameActive = false
      this.gameOver = true
      this.gameWon = true
      this.gameMessage = `🎉 ربحت ${profit.toFixed(2)} USDT`
      
      this.$emit('show-result', this.gameMessage, true)
    },

    resetGame() {
      // إعادة تعيين اللعبة
      this.gameActive = false
      this.gameOver = false
      this.betAmount = null
      this.currentStep = 0
      this.currentMultiplier = 1.0
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
}

.game-header {
  position: relative;
  margin-bottom: 15px;
  text-align: center;
}

.game-header h2 {
  font-size: 24px;
  font-weight: 800;
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  position: relative;
  z-index: 1;
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

.current-balance {
  text-align: center;
  margin-bottom: 20px;
  padding: 10px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 50px;
  border: 1px solid rgba(255, 215, 0, 0.2);
  color: #ffd700;
}

.current-balance strong {
  font-size: 18px;
  margin-left: 5px;
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
  font-size: 80px;
  text-align: center;
  margin-bottom: 20px;
  filter: drop-shadow(0 0 20px #ffd700);
  transition: all 0.3s;
}

.chicken.walking {
  animation: walk 0.5s infinite;
}

@keyframes walk {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px) rotate(-5deg); }
  75% { transform: translateX(5px) rotate(5deg); }
}

.road {
  display: flex;
  gap: 5px;
  overflow-x: auto;
  padding: 10px 0;
  white-space: nowrap;
  scrollbar-width: thin;
  scrollbar-color: #ffd700 #1a1f30;
}

.road::-webkit-scrollbar {
  height: 6px;
}

.road::-webkit-scrollbar-track {
  background: #1a1f30;
  border-radius: 10px;
}

.road::-webkit-scrollbar-thumb {
  background: #ffd700;
  border-radius: 10px;
}

.road-step {
  min-width: 65px;
  height: 85px;
  background: linear-gradient(145deg, #252b3d, #1a1f30);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(255, 215, 0, 0.2);
  transition: all 0.3s;
}

.road-step.active {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  transform: scale(1.05);
  box-shadow: 0 0 25px rgba(255, 215, 0, 0.5);
  z-index: 2;
}

.road-step.passed {
  border-color: #4caf50;
  opacity: 0.7;
}

.road-step.danger {
  border-color: #f44336;
}

.step-multiplier {
  font-weight: 700;
  color: #ffd700;
}

.road-step.active .step-multiplier {
  color: #0a0f1e;
}

.bet-area {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 20px;
  padding: 20px;
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
  max-width: 280px;
}

.input-icon {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 18px;
  z-index: 2;
}

.bet-input {
  width: 100%;
  padding: 15px 45px 15px 60px;
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
  background: linear-gradient(135deg, #ffd700, #ffed4a);
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
  max-width: 280px;
  justify-content: center;
}

.start-btn:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(255, 215, 0, 0.4);
}

.start-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.error-message {
  color: #f44336;
  font-size: 14px;
  padding: 8px 15px;
  background: rgba(244, 67, 54, 0.1);
  border-radius: 50px;
  border: 1px solid #f44336;
}

.game-controls {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
}

.multiplier-display {
  text-align: center;
  padding: 15px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 15px;
  width: 100%;
}

.multiplier-display .label {
  display: block;
  color: #8a8f9c;
  font-size: 14px;
  margin-bottom: 5px;
}

.multiplier-display .value {
  font-size: 36px;
  font-weight: 800;
  color: #ffd700;
  text-shadow: 0 0 15px #ffd700;
}

.potential-profit {
  text-align: center;
  color: #8a8f9c;
}

.potential-profit strong {
  color: #ffd700;
  font-size: 18px;
}

.action-buttons {
  display: flex;
  gap: 15px;
  width: 100%;
  justify-content: center;
}

.next-btn, .cashout-btn {
  padding: 12px 20px;
  border: none;
  border-radius: 50px;
  font-weight: 700;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.next-btn {
  background: linear-gradient(135deg, #f44336, #d32f2f);
  color: white;
}

.cashout-btn {
  background: linear-gradient(135deg, #4caf50, #45a049);
  color: white;
}

.next-btn:hover, .cashout-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.game-result {
  text-align: center;
  padding: 20px;
  border-radius: 20px;
  animation: slideUp 0.5s;
}

.game-result.win {
  background: rgba(76, 175, 80, 0.15);
  border: 1px solid #4caf50;
}

.game-result.lose {
  background: rgba(244, 67, 54, 0.15);
  border: 1px solid #f44336;
}

.result-icon {
  font-size: 48px;
  margin-bottom: 10px;
}

.result-message {
  font-size: 18px;
  font-weight: 700;
  margin-bottom: 15px;
}

.game-result.win .result-message {
  color: #4caf50;
}

.game-result.lose .result-message {
  color: #f44336;
}

.play-again-btn {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  border: none;
  padding: 12px 25px;
  border-radius: 50px;
  font-weight: 700;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
}

.play-again-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(255, 215, 0, 0.3);
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
    font-size: 60px;
  }

  .road-step {
    min-width: 55px;
    height: 75px;
  }

  .action-buttons {
    flex-direction: column;
  }

  .next-btn, .cashout-btn {
    width: 100%;
  }

  .multiplier-display .value {
    font-size: 28px;
  }
}
</style>
