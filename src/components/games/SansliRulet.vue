<template>
  <div class="rulet-card">
    <div class="rulet-card-header">
      <h2>🎰 ŞANSLI RULET</h2>
      <div class="rulet-glow"></div>
    </div>

    <div class="game-scene">
      <div class="rulet-container">
        <!-- عجلة الروليت مع الصور -->
        <div class="wheel-wrapper">
          <img 
            src="@/assets/images/roulette-wheel.png" 
            alt="Roulette Wheel" 
            class="wheel-image"
            :class="{ spinning: isSpinning }"
            :style="{ transform: `rotate(${wheelRotation}deg)` }"
          />
          <img 
            src="@/assets/images/roulette-ball.png" 
            alt="Roulette Ball" 
            class="ball-image"
            :class="{ bouncing: isSpinning }"
            :style="{ 
              transform: `rotate(${ballRotation}deg) translateX(${ballPosition}px)`,
              opacity: ballVisible ? 1 : 0
            }"
          />
        </div>

        <!-- لوحة الأرقام للرهان -->
        <div class="betting-table" :style="{ backgroundImage: `url(${require('@/assets/images/table/roulette-table-bg.jpg')})` }">
          <!-- صفر -->
          <button 
            class="zero-cell" 
            @click="selectBet('zero')"
            :class="{ 'selected-bet': selectedBetType === 'zero' }"
          >
            0
          </button>

          <!-- شبكة الأرقام 1-36 -->
          <div class="numbers-grid">
            <button 
              v-for="num in 36" 
              :key="num"
              class="number-cell"
              :class="{
                'red': isRed(num),
                'black': !isRed(num),
                'selected-bet': selectedBetType === 'number' && selectedNumber === num
              }"
              @click="selectNumber(num)"
            >
              {{ num }}
            </button>
          </div>

          <!-- صفوف الرهانات الخارجية -->
          <div class="outside-bets">
            <button class="outside-btn dozen" @click="selectBet('1-12')" :class="{ 'selected-bet': selectedBetType === '1-12' }">1-12</button>
            <button class="outside-btn dozen" @click="selectBet('13-24')" :class="{ 'selected-bet': selectedBetType === '13-24' }">13-24</button>
            <button class="outside-btn dozen" @click="selectBet('25-36')" :class="{ 'selected-bet': selectedBetType === '25-36' }">25-36</button>
            
            <button class="outside-btn" @click="selectBet('1-18')" :class="{ 'selected-bet': selectedBetType === '1-18' }">1-18</button>
            <button class="outside-btn" @click="selectBet('even')" :class="{ 'selected-bet': selectedBetType === 'even' }">زوجي</button>
            <button class="outside-btn red-btn" @click="selectBet('red')" :class="{ 'selected-bet': selectedBetType === 'red' }">أحمر</button>
            <button class="outside-btn black-btn" @click="selectBet('black')" :class="{ 'selected-bet': selectedBetType === 'black' }">أسود</button>
            <button class="outside-btn" @click="selectBet('odd')" :class="{ 'selected-bet': selectedBetType === 'odd' }">فردي</button>
            <button class="outside-btn" @click="selectBet('19-36')" :class="{ 'selected-bet': selectedBetType === '19-36' }">19-36</button>
          </div>
        </div>

        <!-- معلومات الرهان الحالي -->
        <div class="current-bet-info">
          <div class="bet-type" v-if="selectedBetType">
            <span class="label">الرهان:</span>
            <span class="value">{{ getBetTypeText() }}</span>
            <span class="multiplier" v-if="getMultiplier()">x{{ getMultiplier() }}</span>
          </div>
          <div class="selected-number" v-if="selectedBetType === 'number' && selectedNumber">
            <span class="label">الرقم:</span>
            <span class="value" :class="{ 'red-text': isRed(selectedNumber), 'black-text': !isRed(selectedNumber) }">
              {{ selectedNumber }}
            </span>
          </div>
        </div>
      </div>
    </div>

    <div class="rulet-controls">
      <div v-if="!isSpinning" class="bet-panel">
        <div class="chip-input">
          <img src="@/assets/images/chips/chip-red.png" alt="chip" class="chip-icon-img" />
          <input
            type="number"
            v-model.number="betAmount"
            placeholder="0.00"
            class="rulet-input"
            @input="clearError"
            min="0.01"
            step="0.01"
          />
          <span class="chip-currency">USDT</span>
        </div>
        
        <div v-if="error" class="rulet-error">{{ error }}</div>
        
        <div class="chip-options">
          <button 
            v-for="chip in chipValues" 
            :key="chip"
            class="chip-option"
            @click="betAmount = chip"
          >
            <img :src="getChipImage(chip)" :alt="chip" class="chip-option-img" />
            <span>{{ chip }}</span>
          </button>
        </div>

        <button 
          @click="spinWheel" 
          class="rulet-button"
          :disabled="!canSpin"
        >
          <img src="@/assets/images/roulette-wheel.png" alt="spin" class="button-icon" />
          <span>دور العجلة</span>
        </button>
      </div>

      <div v-if="isSpinning" class="spinning-status">
        <div class="status-text">جاري الدوران...</div>
        <div class="current-number" v-if="currentNumber">
          الرقم: <strong :class="{ 'red-text': isRed(currentNumber), 'black-text': !isRed(currentNumber) }">
            {{ currentNumber }}
          </strong>
        </div>
      </div>

      <div v-if="result" class="result-display" :class="{ 'win': lastResultWon, 'lose': !lastResultWon }">
        <div class="result-icon">{{ lastResultWon ? '🎉' : '😢' }}</div>
        <div class="result-text">{{ result }}</div>
        <div class="result-profit" v-if="lastResultWon && lastProfit">+{{ lastProfit.toFixed(2) }} USDT</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SansliRulet',
  
  props: {
    initialBalance: {
      type: Number,
      required: true
    }
  },
  
  emits: ['update-balance', 'show-result'],
  
  data() {
    return {
      balance: this.initialBalance,
      betAmount: null,
      selectedBetType: null,
      selectedNumber: null,
      isSpinning: false,
      wheelRotation: 0,
      ballRotation: 0,
      ballPosition: 0,
      ballVisible: true,
      currentNumber: null,
      result: '',
      lastResultWon: false,
      lastProfit: 0,
      error: '',
      
      chipValues: [0.1, 0.5, 1, 5, 10, 25, 50, 100],
      
      // أرقام الروليت الحمراء
      redNumbers: [1, 3, 5, 7, 9, 12, 14, 16, 18, 19, 21, 23, 25, 27, 30, 32, 34, 36]
    }
  },
  
  computed: {
    canSpin() {
      return (
        this.betAmount && 
        this.betAmount > 0 && 
        this.betAmount <= this.balance &&
        this.selectedBetType &&
        !this.isSpinning
      );
    }
  },
  
  methods: {
    isRed(number) {
      return this.redNumbers.includes(number);
    },
    
    getChipImage(value) {
      if (value <= 1) return require('@/assets/images/chips/chip-red.png');
      if (value <= 5) return require('@/assets/images/chips/chip-blue.png');
      if (value <= 25) return require('@/assets/images/chips/chip-green.png');
      return require('@/assets/images/chips/chip-black.png');
    },
    
    selectNumber(num) {
      this.selectedBetType = 'number';
      this.selectedNumber = num;
      this.error = '';
    },
    
    selectBet(type) {
      this.selectedBetType = type;
      this.selectedNumber = null;
      this.error = '';
    },
    
    getBetTypeText() {
      const texts = {
        'number': 'رقم',
        'zero': 'صفر',
        'red': 'أحمر',
        'black': 'أسود',
        'even': 'زوجي',
        'odd': 'فردي',
        '1-18': '1-18',
        '19-36': '19-36',
        '1-12': '1-12',
        '13-24': '13-24',
        '25-36': '25-36'
      };
      return texts[this.selectedBetType] || '';
    },
    
    getMultiplier() {
      const multipliers = {
        'number': 35,
        'zero': 35,
        'red': 2,
        'black': 2,
        'even': 2,
        'odd': 2,
        '1-18': 2,
        '19-36': 2,
        '1-12': 3,
        '13-24': 3,
        '25-36': 3
      };
      return multipliers[this.selectedBetType] || 1;
    },
    
    clearError() {
      this.error = '';
    },
    
    spinWheel() {
      // التحقق من الرهان
      if (!this.betAmount || this.betAmount <= 0) {
        this.error = 'الرجاء إدخال مبلغ الرهان';
        return;
      }
      
      if (this.betAmount > this.balance) {
        this.error = 'الرصيد غير كافي';
        return;
      }
      
      if (!this.selectedBetType) {
        this.error = 'الرجاء اختيار نوع الرهان';
        return;
      }
      
      this.error = '';
      
      // خصم الرهان من الرصيد
      this.balance -= this.betAmount;
      this.$emit('update-balance', this.balance);
      
      // بدء الدوران
      this.isSpinning = true;
      this.result = '';
      this.ballVisible = true;
      this.currentNumber = null;
      
      // تحديد الرقم الفائز عشوائياً
      const winningNumber = Math.floor(Math.random() * 37); // 0-36
      
      // بدء حركة العجلة والكرة
      const spinDuration = 5000; // 5 ثوان
      const startTime = Date.now();
      const startRotation = this.wheelRotation;
      const startBallRotation = this.ballRotation;
      
      // عدد الدورات الكاملة (5-10 دورات)
      const totalWheelSpins = 5 + Math.floor(Math.random() * 5);
      const totalBallSpins = 10 + Math.floor(Math.random() * 10);
      
      // الزاوية النهائية للرقم الفائز (كل رقم بزاوية محددة)
      const targetAngle = (winningNumber * (360 / 37)) + (Math.random() * 10 - 5);
      
      const animate = () => {
        const elapsed = Date.now() - startTime;
        const progress = Math.min(elapsed / spinDuration, 1);
        
        // حركة العجلة (تتباطأ في النهاية)
        const easeOutProgress = 1 - Math.pow(1 - progress, 3);
        
        // دوران العجلة
        this.wheelRotation = startRotation + (totalWheelSpins * 360 * easeOutProgress);
        
        // دوران الكرة (أسرع من العجلة)
        this.ballRotation = startBallRotation + (totalBallSpins * 360 * easeOutProgress * 1.5);
        
        // حركة الكرة (تقترب من المركز في النهاية)
        this.ballPosition = 50 * (1 - progress);
        
        // تحديث الرقم المعروض أثناء الدوران
        const currentAngle = (this.wheelRotation % 360 + 360) % 360;
        const estimatedNumber = Math.floor(currentAngle / (360 / 37)) % 37;
        this.currentNumber = estimatedNumber;
        
        if (progress < 1) {
          requestAnimationFrame(animate);
        } else {
          // انتهى الدوران
          this.finishSpin(winningNumber);
        }
      };
      
      requestAnimationFrame(animate);
    },
    
    finishSpin(winningNumber) {
      this.isSpinning = false;
      this.currentNumber = winningNumber;
      
      // التحقق من الفوز
      let won = false;
      
      switch (this.selectedBetType) {
        case 'number':
          won = (winningNumber === this.selectedNumber);
          break;
        case 'zero':
          won = (winningNumber === 0);
          break;
        case 'red':
          won = (winningNumber !== 0 && this.isRed(winningNumber));
          break;
        case 'black':
          won = (winningNumber !== 0 && !this.isRed(winningNumber));
          break;
        case 'even':
          won = (winningNumber !== 0 && winningNumber % 2 === 0);
          break;
        case 'odd':
          won = (winningNumber !== 0 && winningNumber % 2 === 1);
          break;
        case '1-18':
          won = (winningNumber >= 1 && winningNumber <= 18);
          break;
        case '19-36':
          won = (winningNumber >= 19 && winningNumber <= 36);
          break;
        case '1-12':
          won = (winningNumber >= 1 && winningNumber <= 12);
          break;
        case '13-24':
          won = (winningNumber >= 13 && winningNumber <= 24);
          break;
        case '25-36':
          won = (winningNumber >= 25 && winningNumber <= 36);
          break;
      }
      
      const multiplier = this.getMultiplier();
      const profit = this.betAmount * multiplier;
      
      this.lastResultWon = won;
      this.lastProfit = won ? profit - this.betAmount : 0;
      
      if (won) {
        this.balance += profit;
        this.$emit('update-balance', this.balance);
        this.result = `🎉 فوز! الرقم ${winningNumber} - ربحت ${profit.toFixed(2)} USDT`;
        this.$emit('show-result', `🎉 فوز! ربحت ${profit.toFixed(2)} USDT`, true);
      } else {
        this.result = `😢 خسارة! الرقم ${winningNumber}`;
        this.$emit('show-result', `😢 خسارة! الرقم ${winningNumber}`, false);
      }
      
      // إخفاء الكرة بعد قليل
      setTimeout(() => {
        this.ballVisible = false;
      }, 1000);
    }
  }
}
</script>

<style scoped>
.rulet-card {
  background: rgba(20, 25, 40, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 40px;
  padding: 25px;
  width: 100%;
  max-width: 700px;
  margin: 0 auto;
  border: 1px solid rgba(255, 215, 0, 0.3);
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.5), inset 0 0 50px rgba(255, 215, 0, 0.1);
  position: relative;
  overflow: hidden;
}

.rulet-card-header {
  position: relative;
  margin-bottom: 25px;
  text-align: center;
}

.rulet-card-header h2 {
  font-size: 28px;
  font-weight: 800;
  background: linear-gradient(135deg, #ffd700, #ffed4a, #ffd700);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  position: relative;
  z-index: 1;
  letter-spacing: 2px;
  text-shadow: 0 0 30px rgba(255, 215, 0, 0.3);
}

.rulet-glow {
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

.game-scene {
  background: rgba(10, 15, 30, 0.7);
  border-radius: 30px;
  padding: 25px;
  margin-bottom: 25px;
  border: 1px solid rgba(255, 215, 0, 0.2);
  box-shadow: inset 0 0 30px rgba(0, 0, 0, 0.5);
  min-height: 500px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.rulet-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 25px;
}

.wheel-wrapper {
  position: relative;
  width: 250px;
  height: 250px;
  margin: 0 auto;
}

.wheel-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
  transition: transform 0.1s linear;
  filter: drop-shadow(0 0 20px rgba(255, 215, 0, 0.3));
}

.wheel-image.spinning {
  animation: wheelGlow 1s infinite;
}

@keyframes wheelGlow {
  0%, 100% { filter: drop-shadow(0 0 20px rgba(255, 215, 0, 0.3)); }
  50% { filter: drop-shadow(0 0 40px rgba(255, 215, 0, 0.8)); }
}

.ball-image {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 30px;
  height: 30px;
  object-fit: contain;
  transform-origin: center;
  filter: drop-shadow(0 0 10px gold);
  z-index: 10;
}

.ball-image.bouncing {
  animation: ballBounce 0.3s infinite alternate;
}

@keyframes ballBounce {
  from { transform: translate(-50%, -50%) scale(1); }
  to { transform: translate(-50%, -50%) scale(1.2); }
}

/* طاولة الرهان */
.betting-table {
  background-size: cover;
  background-position: center;
  border-radius: 20px;
  padding: 20px;
  border: 2px solid #ffd700;
  width: 100%;
  max-width: 600px;
}

.zero-cell {
  width: 100%;
  height: 60px;
  background: linear-gradient(145deg, #1e8f3e, #0a5c0a);
  color: white;
  border: 2px solid #ffd700;
  border-radius: 10px;
  font-size: 24px;
  font-weight: 800;
  margin-bottom: 10px;
  cursor: pointer;
  transition: all 0.3s;
}

.zero-cell:hover {
  transform: scale(1.02);
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.5);
}

.numbers-grid {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 5px;
  margin-bottom: 15px;
}

.number-cell {
  aspect-ratio: 1;
  border: 2px solid #ffd700;
  border-radius: 5px;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.number-cell.red {
  background: linear-gradient(145deg, #f44336, #d32f2f);
  color: white;
}

.number-cell.black {
  background: linear-gradient(145deg, #1a1a1a, #000000);
  color: white;
}

.number-cell:hover {
  transform: scale(1.05);
  box-shadow: 0 0 15px rgba(255, 215, 0, 0.5);
  z-index: 2;
}

.outside-bets {
  display: grid;
  grid-template-columns: repeat(9, 1fr);
  gap: 5px;
  margin-top: 10px;
}

.outside-btn {
  padding: 10px 5px;
  background: linear-gradient(145deg, #1e2333, #131826);
  color: white;
  border: 1px solid #ffd700;
  border-radius: 8px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  text-align: center;
}

.outside-btn.dozen {
  background: linear-gradient(145deg, #4a2e1e, #2e1a0f);
}

.outside-btn.red-btn {
  background: linear-gradient(145deg, #f44336, #d32f2f);
}

.outside-btn.black-btn {
  background: linear-gradient(145deg, #1a1a1a, #000000);
}

.outside-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(255, 215, 0, 0.3);
}

.selected-bet {
  box-shadow: 0 0 0 3px #ffd700, 0 0 20px #ffd700;
  transform: scale(1.02);
  z-index: 3;
  position: relative;
}

.current-bet-info {
  display: flex;
  gap: 20px;
  justify-content: center;
  padding: 15px;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 15px;
  border: 1px solid rgba(255, 215, 0, 0.3);
}

.bet-type, .selected-number {
  display: flex;
  align-items: center;
  gap: 8px;
}

.label {
  color: #8a8f9c;
  font-size: 14px;
}

.value {
  color: #ffd700;
  font-size: 16px;
  font-weight: 600;
}

.value.red-text {
  color: #f44336;
}

.value.black-text {
  color: #ffffff;
}

.multiplier {
  background: rgba(255, 215, 0, 0.2);
  padding: 4px 8px;
  border-radius: 20px;
  font-size: 12px;
  color: #ffd700;
  border: 1px solid #ffd700;
}

.rulet-controls {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.bet-panel {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
}

.chip-input {
  position: relative;
  width: 100%;
  max-width: 280px;
}

.chip-icon-img {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  width: 20px;
  height: 20px;
  z-index: 2;
  filter: drop-shadow(0 0 5px #ffd700);
}

.rulet-input {
  width: 100%;
  padding: 15px 45px 15px 60px;
  border-radius: 50px;
  background: linear-gradient(145deg, #1a1f30, #0f1422);
  color: #ffffff;
  border: 2px solid rgba(255, 215, 0, 0.3);
  font-size: 18px;
  text-align: center;
  transition: all 0.3s ease;
  box-shadow: inset 0 5px 10px rgba(0, 0, 0, 0.5);
}

.rulet-input:focus {
  outline: none;
  border-color: #ffd700;
  box-shadow: 0 0 25px rgba(255, 215, 0, 0.3), inset 0 5px 10px rgba(0, 0, 0, 0.5);
}

.chip-currency {
  position: absolute;
  left: 15px;
  top: 50%;
  transform: translateY(-50%);
  color: #ffd700;
  font-weight: 700;
  font-size: 14px;
  text-shadow: 0 0 8px #ffd700;
}

.chip-options {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
  max-width: 280px;
}

.chip-option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  background: transparent;
  border: none;
  color: white;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.3s;
  padding: 5px;
}

.chip-option:hover {
  transform: translateY(-3px);
}

.chip-option-img {
  width: 40px;
  height: 40px;
  filter: drop-shadow(0 3px 5px rgba(0, 0, 0, 0.5));
}

.rulet-button {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  border: none;
  padding: 15px 40px;
  border-radius: 50px;
  font-weight: 700;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  box-shadow: 0 10px 25px rgba(255, 215, 0, 0.4), inset 0 2px 5px rgba(255, 255, 255, 0.5);
  width: 100%;
  max-width: 280px;
}

.rulet-button:hover:not(:disabled) {
  transform: translateY(-5px);
  box-shadow: 0 15px 35px rgba(255, 215, 0, 0.6), inset 0 2px 10px rgba(255, 255, 255, 0.7);
}

.rulet-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  filter: grayscale(0.5);
}

.button-icon {
  width: 24px;
  height: 24px;
}

.rulet-error {
  color: #f44336;
  font-size: 14px;
  background: rgba(244, 67, 54, 0.1);
  padding: 8px 20px;
  border-radius: 50px;
  border: 1px solid #f44336;
}

.spinning-status {
  text-align: center;
  padding: 20px;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 20px;
  border: 1px solid #ffd700;
}

.status-text {
  color: #ffd700;
  font-size: 18px;
  margin-bottom: 10px;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.current-number {
  font-size: 24px;
  color: white;
}

.current-number strong {
  font-size: 32px;
  margin-right: 8px;
}

.result-display {
  text-align: center;
  padding: 20px;
  border-radius: 20px;
  animation: slideUp 0.5s ease;
}

.result-display.win {
  background: linear-gradient(145deg, #1a2f1a, #0f1f0f);
  border: 1px solid #4caf50;
}

.result-display.lose {
  background: linear-gradient(145deg, #2f1a1a, #1f0f0f);
  border: 1px solid #f44336;
}

.result-icon {
  font-size: 48px;
  margin-bottom: 10px;
}

.result-profit {
  font-size: 24px;
  font-weight: 700;
  color: #4caf50;
  margin-top: 10px;
  text-shadow: 0 0 10px #4caf50;
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
  .rulet-card {
    padding: 15px;
  }

  .rulet-card-header h2 {
    font-size: 22px;
  }

  .wheel-wrapper {
    width: 200px;
    height: 200px;
  }

  .numbers-grid {
    grid-template-columns: repeat(6, 1fr);
  }

  .outside-bets {
    grid-template-columns: repeat(3, 1fr);
  }

  .outside-btn {
    font-size: 10px;
    padding: 8px 3px;
  }

  .current-bet-info {
    flex-direction: column;
    gap: 10px;
    align-items: center;
  }

  .chip-options {
    max-width: 100%;
  }
}
</style>
