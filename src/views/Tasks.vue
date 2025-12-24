<template>
  <div class="game-page">
    <!-- العنوان -->
    <div class="game-header">
      <h1>PLINKO</h1>
      <div class="subtitle">BIGAMLINE</div>
    </div>

    <!-- الرصيد -->
    <div class="balance-container">
      <div class="balance-label">BALANCE</div>
      <div class="balance-amount">{{ balance.toFixed(2) }} FUN</div>
    </div>

    <!-- لوحة المضاعفات -->
    <div class="multipliers-board">
      <div 
        v-for="(multiplier, index) in multipliers" 
        :key="index"
        :class="['multiplier-item', getMultiplierClass(multiplier)]"
      >
        x{{ multiplier }}
        <div class="risk-label">{{ getRiskLevel(multiplier) }}</div>
      </div>
    </div>

    <!-- لوحة البلينكو -->
    <div class="plinko-container">
      <div class="plinko-board" ref="plinkoBoard">
        <!-- المسارات والأقراص -->
        <div class="pin-container">
          <div v-for="row in 8" :key="row" class="pin-row" :style="{ top: row * 40 + 'px' }">
            <div 
              v-for="pin in row + 2" 
              :key="pin"
              class="pin"
              :style="{ left: (pin * 40) + 'px' }"
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
      </div>
    </div>

    <!-- خيارات الرهان -->
    <div class="bet-controls">
      <div class="bet-amount">
        <div class="bet-label">Min Bet 1.00 FUN</div>
        <div class="bet-input-container">
          <button class="bet-btn decrement" @click="decrementBet">-</button>
          <input 
            type="number" 
            v-model.number="betAmount" 
            min="1" 
            max="1000"
            class="bet-input"
          />
          <button class="bet-btn increment" @click="incrementBet">+</button>
        </div>
        <div class="bet-label">Max Bet 1000.00 FUN</div>
      </div>

      <!-- مستويات الخطورة -->
      <div class="risk-levels">
        <button 
          v-for="level in riskLevels" 
          :key="level.id"
          :class="['risk-btn', { active: selectedRisk === level.id }]"
          @click="selectRisk(level.id)"
        >
          {{ level.label }}
          <div class="risk-info">{{ level.info }}</div>
        </button>
      </div>

      <!-- زر اللعب -->
      <div class="play-section">
        <button 
          class="play-btn"
          :disabled="isPlaying || balance < betAmount"
          @click="startGame"
        >
          <div class="play-icon">⬇️</div>
          <div class="play-text">DROP</div>
        </button>
      </div>

      <!-- النتائج -->
      <div v-if="result.show" class="result-popup" :class="result.type">
        <div class="result-icon">{{ result.icon }}</div>
        <div class="result-text">{{ result.message }}</div>
        <div class="result-multiplier">Multiplier: x{{ result.multiplier }}</div>
        <div class="result-win">Win: {{ result.win.toFixed(2) }} FUN</div>
        <button class="close-result" @click="closeResult">✕</button>
      </div>
    </div>

    <!-- الإحصائيات -->
    <div class="stats">
      <div class="stat-item">
        <div class="stat-label">Total Bets</div>
        <div class="stat-value">{{ stats.totalBets }}</div>
      </div>
      <div class="stat-item">
        <div class="stat-label">Total Wins</div>
        <div class="stat-value">{{ stats.totalWins }}</div>
      </div>
      <div class="stat-item">
        <div class="stat-label">Win Rate</div>
        <div class="stat-value">{{ stats.winRate }}%</div>
      </div>
      <div class="stat-item">
        <div class="stat-label">Highest Win</div>
        <div class="stat-value">{{ stats.highestWin.toFixed(2) }}</div>
      </div>
    </div>

    <!-- إعادة التعيين (للأغراض التعليمية) -->
    <div class="reset-section">
      <button class="reset-btn" @click="resetGame">
        🔄 Reset Game (Educational Only)
      </button>
      <div class="educational-note">
        ⚠️ This is an educational demo only. No real money involved.
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "PlinkoGame",
  
  data() {
    return {
      // بيانات اللعبة
      balance: 1000.00,
      betAmount: 10,
      selectedRisk: 'normal',
      ball: {
        active: false,
        x: 200,
        y: 0
      },
      dropPosition: 200,
      ballColor: '#FF2D55',
      isPlaying: false,
      
      // المضاعفات (مطابقة للصورة)
      multipliers: [29, 4, 1.5, 0.3, 0.2, 0.3, 1.5, 4, 29],
      
      // مستويات الخطورة
      riskLevels: [
        { id: 'high', label: 'High', info: 'More extreme results' },
        { id: 'normal', label: 'Normal', info: 'Balanced gameplay' },
        { id: 'low', label: 'Low', info: 'More consistent' }
      ],
      
      // النتائج
      result: {
        show: false,
        type: '',
        icon: '',
        message: '',
        multiplier: 0,
        win: 0
      },
      
      // الإحصائيات
      stats: {
        totalBets: 0,
        totalWins: 0,
        winRate: 0,
        highestWin: 0
      },
      
      // متغيرات البلينكو
      gameInterval: null,
      currentMultiplier: 1,
      pinRows: 8
    }
  },
  
  computed: {
    boardWidth() {
      return 400; // عرض لوحة البلينكو
    },
    
    slotCount() {
      return this.multipliers.length;
    },
    
    slotWidth() {
      return this.boardWidth / this.slotCount;
    }
  },
  
  methods: {
    // تغيير قيمة الرهان
    incrementBet() {
      if (this.betAmount < 1000) {
        this.betAmount = Math.min(this.betAmount + 10, 1000);
      }
    },
    
    decrementBet() {
      if (this.betAmount > 1) {
        this.betAmount = Math.max(this.betAmount - 10, 1);
      }
    },
    
    // اختيار مستوى الخطورة
    selectRisk(risk) {
      this.selectedRisk = risk;
      // تغيير لون الكرة حسب الخطورة
      switch(risk) {
        case 'high':
          this.ballColor = '#FF2D55'; // أحمر
          break;
        case 'normal':
          this.ballColor = '#22C55E'; // أخضر
          break;
        case 'low':
          this.ballColor = '#FACC15'; // أصفر
          break;
      }
    },
    
    // بدء اللعبة
    startGame() {
      if (this.isPlaying || this.balance < this.betAmount) return;
      
      // خصم الرهان
      this.balance -= this.betAmount;
      this.stats.totalBets++;
      
      // إعداد الكرة
      this.isPlaying = true;
      this.ball.active = true;
      this.ball.x = this.dropPosition;
      this.ball.y = 0;
      
      // بدء الحركة
      this.startBallAnimation();
    },
    
    // حركة الكرة
    startBallAnimation() {
      let x = this.dropPosition;
      let y = 0;
      const gravity = 0.5;
      let velocityY = 2;
      let velocityX = (Math.random() - 0.5) * 4; // حركة أفقية عشوائية
      
      // تأثير مستوى الخطورة على الحركة
      switch(this.selectedRisk) {
        case 'high':
          velocityX *= 1.5;
          break;
        case 'low':
          velocityX *= 0.5;
          break;
      }
      
      this.gameInterval = setInterval(() => {
        // تطبيق الجاذبية
        velocityY += gravity;
        y += velocityY;
        
        // حركة أفقية مع ارتداد من الجوانب
        x += velocityX;
        if (x <= 20 || x >= this.boardWidth - 20) {
          velocityX *= -0.8; // ارتداد مع فقدان طاقة
        }
        
        // تحديث موقع الكرة
        this.ball.x = x;
        this.ball.y = y;
        
        // التحقق من وصول الكرة للأسفل
        if (y >= 320) {
          this.endGame(x);
          clearInterval(this.gameInterval);
        }
      }, 16); // ~60 فريم في الثانية
    },
    
    // نهاية اللعبة وحساب النتيجة
    endGame(finalX) {
      this.isPlaying = false;
      
      // تحديد السلة التي سقطت فيها الكرة
      const slotIndex = Math.min(
        this.slotCount - 1,
        Math.max(0, Math.floor(finalX / this.slotWidth))
      );
      
      // الحصول على المضاعف
      let baseMultiplier = this.multipliers[slotIndex];
      
      // تطبيق تأثير مستوى الخطورة
      switch(this.selectedRisk) {
        case 'high':
          // عالي الخطورة - تباين أكبر
          if (Math.random() > 0.7) {
            baseMultiplier *= 1.5;
          } else if (Math.random() < 0.3) {
            baseMultiplier *= 0.7;
          }
          break;
        case 'low':
          // منخفض الخطورة - تباين أقل
          baseMultiplier *= (0.9 + Math.random() * 0.2);
          break;
      }
      
      this.currentMultiplier = Math.max(0.1, baseMultiplier);
      const winAmount = this.betAmount * this.currentMultiplier;
      
      // تحديث الرصيد
      this.balance += winAmount;
      
      // تحديث الإحصائيات
      if (winAmount > this.betAmount) {
        this.stats.totalWins++;
        this.stats.highestWin = Math.max(this.stats.highestWin, winAmount);
      }
      this.stats.winRate = this.stats.totalBets > 0 
        ? Math.round((this.stats.totalWins / this.stats.totalBets) * 100) 
        : 0;
      
      // عرض النتيجة
      this.showResult(winAmount);
    },
    
    // عرض نتيجة اللعبة
    showResult(winAmount) {
      let type, icon, message;
      
      if (winAmount > this.betAmount) {
        type = 'win';
        icon = '🎉';
        message = 'WIN!';
      } else if (winAmount === this.betAmount) {
        type = 'draw';
        icon = '🤝';
        message = 'DRAW';
      } else {
        type = 'loss';
        icon = '😢';
        message = 'LOSS';
      }
      
      this.result = {
        show: true,
        type,
        icon,
        message,
        multiplier: this.currentMultiplier.toFixed(2),
        win: winAmount
      };
      
      // إخفاء النتيجة بعد 5 ثواني
      setTimeout(() => {
        if (this.result.show) {
          this.closeResult();
        }
      }, 5000);
    },
    
    // إغلاق نافذة النتيجة
    closeResult() {
      this.result.show = false;
    },
    
    // تصنيف المضاعفات حسب القيمة
    getMultiplierClass(multiplier) {
      if (multiplier >= 10) return 'multiplier-high';
      if (multiplier >= 1) return 'multiplier-medium';
      return 'multiplier-low';
    },
    
    // مستوى الخطورة للمضاعف
    getRiskLevel(multiplier) {
      if (multiplier >= 10) return 'Risk Level: High';
      if (multiplier >= 1) return 'Risk Level: Normal';
      return 'Risk Level: Low';
    },
    
    // إعادة تعيين اللعبة (للأغراض التعليمية)
    resetGame() {
      this.balance = 1000;
      this.betAmount = 10;
      this.selectedRisk = 'normal';
      this.ballColor = '#22C55E';
      this.isPlaying = false;
      
      this.result = {
        show: false,
        type: '',
        icon: '',
        message: '',
        multiplier: 0,
        win: 0
      };
      
      this.stats = {
        totalBets: 0,
        totalWins: 0,
        winRate: 0,
        highestWin: 0
      };
      
      if (this.gameInterval) {
        clearInterval(this.gameInterval);
      }
    },
    
    // تغيير موقع الإسقاط
    updateDropPosition(event) {
      if (this.isPlaying) return;
      
      const board = this.$refs.plinkoBoard;
      if (!board) return;
      
      const rect = board.getBoundingClientRect();
      const x = event.clientX - rect.left;
      
      if (x >= 20 && x <= rect.width - 20) {
        this.dropPosition = x;
      }
    }
  },
  
  mounted() {
    // إضافة مستمع لحركة الماوس على لوحة البلينكو
    const board = this.$refs.plinkoBoard;
    if (board) {
      board.addEventListener('mousemove', this.updateDropPosition);
      board.addEventListener('click', (e) => {
        if (!this.isPlaying) {
          this.updateDropPosition(e);
        }
      });
    }
  },
  
  beforeUnmount() {
    // تنظيف المستمعات عند إلغاء التثبيت
    const board = this.$refs.plinkoBoard;
    if (board) {
      board.removeEventListener('mousemove', this.updateDropPosition);
    }
    
    if (this.gameInterval) {
      clearInterval(this.gameInterval);
    }
  }
}
</script>

<style scoped>
.game-page {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  min-height: 100vh;
  color: white;
  padding: 20px;
  font-family: 'Arial', sans-serif;
  max-width: 800px;
  margin: 0 auto;
}

.game-header {
  text-align: center;
  margin-bottom: 20px;
}

.game-header h1 {
  font-size: 48px;
  font-weight: bold;
  background: linear-gradient(45deg, #FF2D55, #22C55E);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin: 0;
  text-transform: uppercase;
  letter-spacing: 2px;
}

.subtitle {
  font-size: 20px;
  color: #94a3b8;
  letter-spacing: 4px;
  margin-top: 5px;
}

.balance-container {
  background: rgba(0, 0, 0, 0.5);
  border: 2px solid #22C55E;
  border-radius: 15px;
  padding: 15px 30px;
  text-align: center;
  margin: 20px auto;
  max-width: 300px;
  backdrop-filter: blur(10px);
}

.balance-label {
  font-size: 14px;
  color: #94a3b8;
  margin-bottom: 5px;
}

.balance-amount {
  font-size: 32px;
  font-weight: bold;
  color: #22C55E;
}

.multipliers-board {
  display: grid;
  grid-template-columns: repeat(9, 1fr);
  gap: 8px;
  margin: 30px 0;
  padding: 0 10px;
}

.multiplier-item {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  padding: 15px 5px;
  text-align: center;
  font-weight: bold;
  font-size: 20px;
  transition: transform 0.3s ease;
  position: relative;
  overflow: hidden;
}

.multiplier-item:hover {
  transform: translateY(-5px);
}

.multiplier-high {
  background: linear-gradient(135deg, #dc2626, #ef4444);
  color: white;
}

.multiplier-medium {
  background: linear-gradient(135deg, #22C55E, #4ade80);
  color: black;
}

.multiplier-low {
  background: linear-gradient(135deg, #facc15, #fde047);
  color: black;
}

.risk-label {
  font-size: 10px;
  opacity: 0.8;
  margin-top: 5px;
}

.plinko-container {
  position: relative;
  margin: 40px 0;
}

.plinko-board {
  background: rgba(15, 23, 42, 0.8);
  height: 350px;
  border-radius: 20px;
  border: 3px solid #334155;
  position: relative;
  overflow: hidden;
  cursor: pointer;
}

.pin-container {
  position: relative;
  height: 100%;
}

.pin-row {
  position: absolute;
  display: flex;
  justify-content: center;
  width: 100%;
  left: 0;
}

.pin {
  position: absolute;
  width: 8px;
  height: 8px;
  background: white;
  border-radius: 50%;
  margin: 0 10px;
}

.ball {
  position: absolute;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #FF2D55;
  box-shadow: 0 0 10px rgba(255, 45, 85, 0.7);
  transition: all 0.1s ease;
  z-index: 10;
}

.drop-line {
  position: absolute;
  top: 0;
  width: 3px;
  height: 100%;
  background: rgba(255, 255, 255, 0.3);
}

.bet-controls {
  background: rgba(30, 41, 59, 0.8);
  border-radius: 20px;
  padding: 20px;
  margin: 20px 0;
}

.bet-amount {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.bet-label {
  font-size: 14px;
  color: #94a3b8;
  flex: 1;
  text-align: center;
}

.bet-input-container {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 2;
  justify-content: center;
}

.bet-input {
  width: 150px;
  padding: 15px;
  font-size: 24px;
  text-align: center;
  background: rgba(0, 0, 0, 0.5);
  border: 2px solid #22C55E;
  border-radius: 10px;
  color: white;
  font-weight: bold;
}

.bet-btn {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  border: none;
  background: #334155;
  color: white;
  font-size: 24px;
  cursor: pointer;
  transition: background 0.3s ease;
}

.bet-btn:hover {
  background: #475569;
}

.risk-levels {
  display: flex;
  gap: 10px;
  margin: 20px 0;
  justify-content: center;
}

.risk-btn {
  flex: 1;
  padding: 15px;
  border: none;
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.1);
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  text-transform: uppercase;
  font-weight: bold;
}

.risk-btn.active {
  transform: scale(1.05);
}

.risk-btn:nth-child(1).active {
  background: linear-gradient(135deg, #dc2626, #ef4444);
}

.risk-btn:nth-child(2).active {
  background: linear-gradient(135deg, #22C55E, #4ade80);
}

.risk-btn:nth-child(3).active {
  background: linear-gradient(135deg, #facc15, #fde047);
}

.risk-info {
  font-size: 12px;
  opacity: 0.7;
  margin-top: 5px;
  font-weight: normal;
  text-transform: none;
}

.play-section {
  text-align: center;
  margin: 30px 0;
}

.play-btn {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  border: none;
  background: linear-gradient(135deg, #22C55E, #16a34a);
  color: white;
  cursor: pointer;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
}

.play-btn:hover:not(:disabled) {
  transform: scale(1.1);
  box-shadow: 0 0 30px rgba(34, 197, 94, 0.5);
}

.play-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.play-icon {
  font-size: 32px;
  margin-bottom: 5px;
}

.play-text {
  font-size: 18px;
  font-weight: bold;
}

.result-popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(0, 0, 0, 0.95);
  border-radius: 20px;
  padding: 30px;
  text-align: center;
  min-width: 300px;
  z-index: 1000;
  border: 3px solid;
  backdrop-filter: blur(10px);
  animation: popup 0.5s ease;
}

@keyframes popup {
  0% { transform: translate(-50%, -50%) scale(0.5); opacity: 0; }
  100% { transform: translate(-50%, -50%) scale(1); opacity: 1; }
}

.result-popup.win {
  border-color: #22C55E;
}

.result-popup.loss {
  border-color: #dc2626;
}

.result-popup.draw {
  border-color: #facc15;
}

.result-icon {
  font-size: 60px;
  margin-bottom: 15px;
}

.result-text {
  font-size: 36px;
  font-weight: bold;
  margin-bottom: 10px;
}

.result-multiplier {
  font-size: 24px;
  color: #94a3b8;
  margin-bottom: 5px;
}

.result-win {
  font-size: 28px;
  font-weight: bold;
  color: #22C55E;
}

.close-result {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  color: white;
  font-size: 20px;
  cursor: pointer;
  padding: 5px 10px;
}

.stats {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 15px;
  margin: 30px 0;
}

.stat-item {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 10px;
  padding: 15px;
  text-align: center;
}

.stat-label {
  font-size: 12px;
  color: #94a3b8;
  margin-bottom: 5px;
}

.stat-value {
  font-size: 24px;
  font-weight: bold;
  color: white;
}

.reset-section {
  text-align: center;
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.reset-btn {
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  color: white;
  border: none;
  padding: 15px 30px;
  border-radius: 10px;
  font-size: 16px;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.reset-btn:hover {
  transform: scale(1.05);
}

.educational-note {
  font-size: 12px;
  color: #94a3b8;
  margin-top: 10px;
  font-style: italic;
}

/* تصميم متجاوب */
@media (max-width: 768px) {
  .game-header h1 {
    font-size: 36px;
  }
  
  .multipliers-board {
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
  }
  
  .stat-item {
    padding: 10px;
  }
  
  .stat-value {
    font-size: 18px;
  }
  
  .bet-amount {
    flex-direction: column;
    gap: 15px;
  }
  
  .risk-levels {
    flex-direction: column;
  }
  
  .stats {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
