<template>
  <div class="game-page">
    <!-- الرصيد -->
    <div class="top-bar">
      <div class="balance-gold">
        <i class="fas fa-coins"></i>
        <span>رصيدك: <strong>{{ balance.toFixed(2) }} USDT</strong></span>
      </div>
    </div>

    <!-- رسالة الفوز/الخسارة - تظهر فقط بعد توقف العجلة -->
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
      <!-- لعبة عجلة الحظ -->
      <div v-if="selectedGame === 'wheel-of-fortune'" class="wheel-card">
        <div class="wheel-header">
          <h2>🎡 WHEEL OF FORTUNE</h2>
          <div class="header-glow"></div>
        </div>

        <div class="wheel-content">
          <!-- العجلة مع المضاعفات داخلها -->
          <div class="wheel-wrapper">
            <div class="pointer">▼</div>
            <div class="wheel-container">
              <svg 
                class="wheel-svg" 
                viewBox="0 0 200 200"
                :style="{ transform: `rotate(${wheelRotation}deg)` }"
              >
                <!-- رسم القطاعات الدائرية -->
                <g v-for="(segment, index) in wheelSegments" :key="index">
                  <path
                    :d="getSegmentPath(index)"
                    :fill="getSegmentColor(segment.value)"
                    stroke="white"
                    stroke-width="1"
                  />
                  <text
                    :x="getTextX(index)"
                    :y="getTextY(index)"
                    text-anchor="middle"
                    dominant-baseline="middle"
                    :fill="getTextColor(segment.value)"
                    font-size="14"
                    font-weight="bold"
                  >
                    {{ segment.value }}x
                  </text>
                </g>
                
                <!-- الدائرة الداخلية -->
                <circle cx="100" cy="100" r="25" fill="#222" stroke="gold" stroke-width="3" />
              </svg>
            </div>
          </div>

          <!-- حقل الرهان -->
          <div class="bet-section">
            <div class="bet-input-wrapper">
              <span class="bet-icon">💰</span>
              <input
                type="number"
                v-model.number="betAmount"
                placeholder="أدخل مبلغ الرهان"
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
              <span v-if="!isSpinning">🎡 ابدأ الدوران</span>
              <span v-else>جاري الدوران...</span>
            </button>
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
      selectedGame: "wheel-of-fortune",
      showResultMessage: false,
      resultType: '',
      resultIcon: '',
      resultText: '',
      resultTimeout: null,
      
      gamesList: [
        { id: 'wheel-of-fortune', name: 'Wheel of Fortune', icon: '🎡' }
      ],
      
      balance: 0,
      gameError: "",
      
      // بيانات عجلة الحظ
      wheelRotation: 0,
      isSpinning: false,
      betAmount: null,
      
      // أجزاء العجلة (8 أجزاء) - المضاعفات المطلوبة
      // تم تبديل القيم: جعلنا 2x في المكان الأول بدلاً من 0x
      wheelSegments: [
        { value: 2 },     // قطاع 0 - أخضر (ربح) - 0-45° (هذا هو القطاع الذي سيتوقف عليه السهم)
        { value: 0.5 },   // قطاع 1 - برتقالي (خسارة نصف) - 45-90°
        { value: 1 },     // قطاع 2 - برتقالي (تعادل) - 90-135°
        { value: 1.5 },   // قطاع 3 - أخضر (ربح) - 135-180°
        { value: 0 },     // قطاع 4 - أحمر (خسارة) - 180-225°
        { value: 3 },     // قطاع 5 - أخضر (ربح) - 225-270°
        { value: 5 },     // قطاع 6 - أخضر (ربح) - 270-315°
        { value: 10 }     // قطاع 7 - ذهبي (ربح كبير) - 315-360°
      ],
      
      lastResult: null,
      
      // أصوات اللعبة
      spinSound: null,
      winSound: null,
      loseSound: null,
      clickSound: null
    }
  },
  
  computed: {
    segmentAngle() {
      return 360 / this.wheelSegments.length // 45 درجة
    },
    
    canSpin() {
      return this.betAmount && 
             this.betAmount > 0 && 
             this.betAmount <= this.balance && 
             !this.isSpinning
    }
  },
  
  async created() {
    const user = auth.currentUser
    if (!user) return
    
    const snap = await getDoc(doc(db, "users", user.uid))
    if (snap.exists()) {
      this.balance = Number(snap.data().balance || 0)
    }
    
    // تهيئة الأصوات
    this.initSounds()
  },
  
  methods: {
    initSounds() {
      try {
        // صوت الدوران
        this.spinSound = new Audio('data:audio/wav;base64,UklGRlwAAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YVQAAACAgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f38=')
        
        // صوت الفوز
        this.winSound = new Audio('data:audio/wav;base64,UklGRlwAAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YVQAAACAgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f38=')
        
        // صوت الخسارة
        this.loseSound = new Audio('data:audio/wav;base64,UklGRlwAAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YVQAAACAgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f38=')
        
        // صوت النقر
        this.clickSound = new Audio('data:audio/wav;base64,UklGRlwAAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YVQAAACAgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f38=')
        
        // ضبط مستوى الصوت
        this.spinSound.volume = 0.3
        this.winSound.volume = 0.5
        this.loseSound.volume = 0.4
        this.clickSound.volume = 0.2
        
      } catch (e) {
        console.log('الصوت غير مدعوم في هذا المتصفح')
      }
    },
    
    playSound(sound) {
      if (sound) {
        sound.currentTime = 0
        sound.play().catch(e => console.log('تعذر تشغيل الصوت'))
      }
    },
    
    getSegmentColor(value) {
      if (value === 0) return '#d32f2f' // أحمر (خسارة)
      if (value === 0.5 || value === 1) return '#fb8c00' // برتقالي
      if (value >= 1.5 && value <= 5) return '#388e3c' // أخضر
      if (value === 10) return '#ffd700' // ذهبي
      return '#388e3c'
    },
    
    getTextColor(value) {
      if (value === 0 || value === 0.5 || value === 1) return 'white'
      if (value === 10) return '#222'
      return 'white'
    },
    
    getSegmentPath(index) {
      const centerX = 100
      const centerY = 100
      const radius = 85
      // SVG: الزاوية 0 على اليمين، 90 للأسفل
      const startAngle = (index * this.segmentAngle) * Math.PI / 180
      const endAngle = ((index + 1) * this.segmentAngle) * Math.PI / 180
      
      const x1 = centerX + radius * Math.cos(startAngle)
      const y1 = centerY + radius * Math.sin(startAngle)
      const x2 = centerX + radius * Math.cos(endAngle)
      const y2 = centerY + radius * Math.sin(endAngle)
      
      return `M ${centerX} ${centerY} L ${x1} ${y1} A ${radius} ${radius} 0 0 1 ${x2} ${y2} Z`
    },
    
    getTextX(index) {
      const centerX = 100
      const radius = 60
      // منتصف القطاع
      const angle = ((index + 0.5) * this.segmentAngle) * Math.PI / 180
      return centerX + radius * Math.cos(angle)
    },
    
    getTextY(index) {
      const centerY = 100
      const radius = 60
      const angle = ((index + 0.5) * this.segmentAngle) * Math.PI / 180
      return centerY + radius * Math.sin(angle)
    },
    
    openGame(gameId) {
      this.selectedGame = gameId
      this.gameOpened = true
      this.gameError = ""
      this.resetGame()
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
    
    resetGame() {
      this.wheelRotation = 0
      this.isSpinning = false
      this.lastResult = null
      this.gameError = ''
    },
    
    async spinWheel() {
      if (!this.canSpin) return
      
      // إخفاء أي رسالة سابقة
      this.showResultMessage = false
      
      // تشغيل صوت النقر
      this.playSound(this.clickSound)
      
      this.gameError = ''
      this.isSpinning = true
      
      // خصم الرهان
      this.balance -= this.betAmount
      await this.updateBalance(this.balance)
      
      // تشغيل صوت الدوران
      this.playSound(this.spinSound)
      
      // نختار القطاع 0 (الذي يحتوي على 2x) - لكننا سنعتبره خسارة
      const winningIndex = 0
      const winningSegment = this.wheelSegments[winningIndex]
      
      // منتصف القطاع 0 هو 22.5 درجة
      // السهم في الأعلى (زاوية 90 درجة في نظام SVG)
      // الزاوية المطلوبة لجعل منتصف القطاع 0 تحت السهم = 90 - 22.5 = 67.5 درجة
      const requiredAngle = 67.5
      
      // عدد دورات عشوائي (15-25 دورة) لتبدو طبيعية
      const spins = 15 + Math.floor(Math.random() * 10)
      
      // الزاوية المستهدفة: (360 * عدد الدورات) + الزاوية المطلوبة
      const targetRotation = (360 * spins) + requiredAngle
      
      const start = 0
      const duration = 3500
      const startTime = performance.now()
      
      const animate = (time) => {
        const elapsed = time - startTime
        const progress = Math.min(elapsed / duration, 1)
        
        // منحنى التباطؤ الطبيعي (يبدأ سريعًا ثم يبطئ)
        const easeOut = 1 - Math.pow(1 - progress, 3)
        
        this.wheelRotation = start + ((targetRotation - start) * easeOut)
        
        if (progress < 1) {
          requestAnimationFrame(animate)
        } else {
          // التأكد من الزاوية النهائية مضبوطة
          this.wheelRotation = targetRotation
          
          setTimeout(() => {
            this.finishSpin(winningIndex, winningSegment)
          }, 200)
        }
      }
      
      requestAnimationFrame(animate)
    },
    
    async finishSpin(winningIndex, winningSegment) {
      this.isSpinning = false
      
      // هنا نغير المنطق: بغض النظر عن قيمة القطاع، نعتبره خسارة
      // نستخدم قيمة 0 بدلاً من قيمة القطاع الفعلية
      const multiplier = 0 // دائمًا 0 (خسارة)
      const winAmount = this.betAmount * multiplier // 0
      
      // دائمًا خسارة - لا يوجد ربح أبداً
      this.showResult(`😢 خسرت الرهان`, false)
      this.playSound(this.loseSound)
      
      // حفظ النتيجة الأخيرة
      this.lastResult = {
        segmentIndex: winningIndex,
        multiplier: multiplier,
        isWin: false,
        winAmount: winAmount,
        message: 'خسارة! خسرت كل الرهان'
      }
      
      // العجلة باقية في مكانها
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

/* ===== بطاقة عجلة الحظ ===== */
.wheel-card {
  background: rgba(20, 25, 40, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 40px;
  padding: 25px;
  width: 100%;
  max-width: 550px;
  margin: 0 auto;
  border: 1px solid rgba(255, 215, 0, 0.3);
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.5), inset 0 0 50px rgba(255, 215, 0, 0.1);
  position: relative;
  overflow: hidden;
}

.wheel-header {
  position: relative;
  margin-bottom: 25px;
  text-align: center;
}

.wheel-header h2 {
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

/* ===== العجلة ===== */
.wheel-wrapper {
  position: relative;
  width: 300px;
  height: 300px;
  margin: 20px auto;
}

@media (max-width: 480px) {
  .wheel-wrapper {
    width: 250px;
    height: 250px;
  }
}

.pointer {
  position: absolute;
  top: -15px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 50px;
  color: gold;
  z-index: 10;
  text-shadow: 0 0 20px gold, 0 0 40px rgba(255, 215, 0, 0.5);
  animation: pointerGlow 1.5s infinite;
}

@keyframes pointerGlow {
  0%, 100% { text-shadow: 0 0 15px gold; }
  50% { text-shadow: 0 0 30px gold; }
}

.wheel-container {
  width: 100%;
  height: 100%;
  position: relative;
}

.wheel-svg {
  width: 100%;
  height: 100%;
  transition: transform 3.5s cubic-bezier(0.1, 0.9, 0.2, 1);
  filter: drop-shadow(0 0 15px rgba(255, 215, 0, 0.3));
}

/* ===== قسم الرهان ===== */
.bet-section {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
  margin-top: 30px;
  padding: 20px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 30px;
  border: 1px solid rgba(255, 215, 0, 0.2);
}

.bet-input-wrapper {
  position: relative;
  width: 100%;
  max-width: 300px;
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
  max-width: 300px;
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
  max-width: 300px;
  text-align: center;
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

  .wheel-card {
    padding: 15px;
  }

  .wheel-header h2 {
    font-size: 22px;
  }

  .pointer {
    font-size: 40px;
    top: -10px;
  }

  .bet-section {
    padding: 15px;
  }
}
</style>
