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
        :class="{active: selectedGame===tab.id}" 
        @click="openGame(tab.id)"
      >
        <span class="tab-icon">{{ tab.icon }}</span>
        <span class="tab-text">{{ tab.name }}</span>
      </button>
    </div>

    <!-- عرض اللعبة المختارة - تظهر بملء الشاشة -->
    <div v-if="gameOpened" class="game-fullscreen">
      <!-- Chicken Road -->
      <div v-if="selectedGame==='chicken'" class="casino-card">
        <div class="casino-card-header">
          <h2>🐔 CHICKEN ROAD</h2>
          <div class="casino-glow"></div>
        </div>

        <div class="game-scene">
          <div class="chicken-container">
            <div class="chicken-glow"></div>
            <div class="chicken" :class="{ walking: chickenStarted }">🐔</div>
          </div>

          <div v-if="chickenStarted" class="road-container">
            <div class="road">
              <div
                v-for="(step, i) in chickenSteps"
                :key="i"
                class="step"
                :class="{
                  active: i === chickenPosition,
                  passed: i < chickenPosition,
                  'danger': step.multiplier >= 3,
                  'warning': step.multiplier >= 2 && step.multiplier < 3
                }"
              >
                <span class="step-multiplier">{{ step.multiplier.toFixed(1) }}x</span>
                <div v-if="i === chickenPosition" class="chicken-icon">🐔</div>
                <div v-if="i < chickenPosition" class="step-check">✓</div>
              </div>
            </div>
          </div>
        </div>

        <div class="casino-controls">
          <div v-if="!chickenStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="chickenBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button 
              @click="startChicken"
              class="casino-button"
              :disabled="!chickenBet || chickenBet <= 0"
            >
              <span>ابدأ الرهان</span>
              <i class="fas fa-dice"></i>
            </button>
          </div>

          <div v-if="chickenStarted" class="game-panel">
            <div class="profit-meter">
              <div class="profit-label">الربح المحتمل</div>
              <div class="profit-value">{{ chickenProfit.toFixed(2) }} USDT</div>
            </div>
            <div class="action-row">
              <button @click="chickenNext" class="action-btn danger-btn" :disabled="chickenGameOver">
                <i class="fas fa-arrow-right"></i>
                تقدم
              </button>
              <button @click="chickenCashOut" class="action-btn success-btn" :disabled="chickenGameOver">
                <i class="fas fa-hand-holding-usd"></i>
                سحب
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Dice -->
      <div v-if="selectedGame==='dice'" class="casino-card">
        <div class="casino-card-header">
          <h2>🎲 DICE</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="dice-container" :class="{ rolling: diceRolling }">
            <div class="dice-3d">
              <div class="dice-face front">{{ diceRoll || '?' }}</div>
              <div class="dice-face back">{{ diceRoll || '?' }}</div>
              <div class="dice-face right">{{ diceRoll || '?' }}</div>
              <div class="dice-face left">{{ diceRoll || '?' }}</div>
              <div class="dice-face top">{{ diceRoll || '?' }}</div>
              <div class="dice-face bottom">{{ diceRoll || '?' }}</div>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div class="slider-container">
            <label>نسبة الفوز <span class="slider-value">{{ diceChance }}%</span></label>
            <input 
              type="range" 
              v-model.number="diceChance" 
              min="1" 
              max="20" 
              step="1"
              class="casino-slider"
              :disabled="diceStarted"
            >
          </div>
          
          <div class="multiplier-box">
            <span class="multiplier-label">المضاعف</span>
            <span class="multiplier-value">{{ (95 / diceChance).toFixed(2) }}x</span>
          </div>
          
          <div v-if="!diceStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="diceBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startDice" class="casino-button" :disabled="!diceBet || diceBet <= 0">
              <span>رمي النرد</span>
              <i class="fas fa-dice"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Mines -->
      <div v-if="selectedGame==='mines'" class="casino-card">
        <div class="casino-card-header">
          <h2>💣 MINES</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="mines-container">
            <div class="mines-header">
              <span>الألغام: {{ minesCount }}</span>
              <span>|</span>
              <span>المكشوف: {{ minesRevealed }}</span>
            </div>
            <div v-if="minesStarted" class="mines-grid">
              <button 
                v-for="(cell, index) in minesCells" 
                :key="index"
                class="mine-cell"
                :class="{
                  'revealed': cell.revealed,
                  'mine': cell.mine && cell.revealed,
                  'safe': !cell.mine && cell.revealed,
                }"
                @click="revealMine(index)"
                :disabled="cell.revealed || minesGameOver"
              >
                <span v-if="!cell.revealed">❓</span>
                <span v-else-if="cell.mine">💣</span>
                <span v-else>💎</span>
              </button>
            </div>
            <div v-else class="mines-placeholder">
              <span>ابدأ اللعبة لرؤية الحقل</span>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div class="slider-container">
            <label>عدد الألغام <span class="slider-value">{{ minesCount }}</span></label>
            <input 
              type="range" 
              v-model.number="minesCount" 
              min="8" 
              max="20" 
              step="1"
              class="casino-slider"
              :disabled="minesStarted"
            >
          </div>
          
          <div v-if="!minesStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="minesBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startMines" class="casino-button" :disabled="!minesBet || minesBet <= 0">
              <span>ابدأ اللعبة</span>
              <i class="fas fa-bomb"></i>
            </button>
          </div>
          
          <div v-if="minesStarted && !minesGameOver" class="action-row">
            <button @click="minesCashOut" class="action-btn success-btn">
              <i class="fas fa-hand-holding-usd"></i>
              سحب ({{ minesProfit.toFixed(2) }} USDT)
            </button>
          </div>
        </div>
      </div>

      <!-- Crash -->
      <div v-if="selectedGame==='crash'" class="casino-card">
        <div class="casino-card-header">
          <h2>🚀 CRASH</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="crash-container">
            <div class="multiplier-display" :class="{ 'crashed': crashCrashed }">
              {{ crashMultiplier.toFixed(2) }}x
            </div>
            <div class="rocket-animation">
              <div class="rocket" :class="{ 'launched': crashStarted && !crashCrashed, 'exploded': crashCrashed }">🚀</div>
              <div class="smoke-effect" v-if="crashStarted && !crashCrashed"></div>
            </div>
            <div class="progress-track">
              <div class="progress-fill" :style="{ width: crashProgress + '%' }"></div>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!crashStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="crashBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div class="chip-input">
              <span class="chip-icon">🎯</span>
              <input
                type="number"
                v-model.number="crashAutoCashout"
                placeholder="السحب عند"
                class="casino-input"
                step="0.1"
                min="1.1"
                max="5"
              />
              <span class="chip-currency">x</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startCrash" class="casino-button" :disabled="!crashBet || crashBet <= 0">
              <span>إطلاق الصاروخ</span>
              <i class="fas fa-rocket"></i>
            </button>
          </div>
          
          <div v-if="crashStarted && !crashCrashed" class="action-row">
            <button @click="crashCashOut" class="action-btn success-btn">
              <i class="fas fa-hand-holding-usd"></i>
              سحب ({{ (crashBet * crashMultiplier).toFixed(2) }} USDT)
            </button>
          </div>
        </div>
      </div>

      <!-- Limbo -->
      <div v-if="selectedGame==='limbo'" class="casino-card">
        <div class="casino-card-header">
          <h2>🎯 LIMBO</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="limbo-container">
            <div class="target-line" :style="{ height: (limboTarget * 10) + 'px' }">
              <span class="target-label">{{ limboTarget }}x</span>
            </div>
            <div class="result-ball" :class="{ jumping: limboStarted }">
              {{ limboResult > 0 ? limboResult.toFixed(2) + 'x' : '?' }}
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!limboStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="limboBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div class="chip-input">
              <span class="chip-icon">🎯</span>
              <input
                type="number"
                v-model.number="limboTarget"
                placeholder="المضاعف"
                class="casino-input"
                step="0.1"
                min="1.1"
                max="10"
              />
              <span class="chip-currency">x</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startLimbo" class="casino-button" :disabled="!limboBet || limboBet <= 0">
              <span>ابدأ</span>
              <i class="fas fa-crosshairs"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Blackjack -->
      <div v-if="selectedGame==='blackjack'" class="casino-card">
        <div class="casino-card-header">
          <h2>🃏 BLACKJACK</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="blackjack-table">
            <div class="dealer-area">
              <div class="area-label">الموزع</div>
              <div class="cards-row">
                <div v-for="(card, i) in dealerHand" :key="i" class="casino-card-small" :class="{ 'card-back': i === 1 && !dealerRevealed }">
                  {{ i === 1 && !dealerRevealed ? '?' : card }}
                </div>
              </div>
            </div>
            <div class="player-area">
              <div class="area-label">أنت <span class="score-badge">{{ playerScore }}</span></div>
              <div class="cards-row">
                <div v-for="card in playerHand" :key="card" class="casino-card-small">{{ card }}</div>
              </div>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!blackjackStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="blackjackBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startBlackjack" class="casino-button" :disabled="!blackjackBet || blackjackBet <= 0">
              <span>توزيع</span>
              <i class="fas fa-credit-card"></i>
            </button>
          </div>
          
          <div v-if="blackjackStarted && !blackjackGameOver" class="action-row">
            <button @click="blackjackHit" class="action-btn warning-btn">ضرب 🃏</button>
            <button @click="blackjackStand" class="action-btn success-btn">وقف ✋</button>
          </div>
        </div>
      </div>

      <!-- Slot Machine -->
      <div v-if="selectedGame==='slot'" class="casino-card">
        <div class="casino-card-header">
          <h2>🎰 SLOT MACHINE</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="slot-machine-container">
            <div class="slot-reels" :class="{ spinning: slotSpinning }">
              <div class="reel">{{ slotReels[0] }}</div>
              <div class="reel">{{ slotReels[1] }}</div>
              <div class="reel">{{ slotReels[2] }}</div>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!slotStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="slotBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startSlot" class="casino-button" :disabled="!slotBet || slotBet <= 0">
              <span>لف البكرات</span>
              <i class="fas fa-sliders-h"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Coinflip -->
      <div v-if="selectedGame==='coinflip'" class="casino-card">
        <div class="casino-card-header">
          <h2>🪙 COINFLIP</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="coin-container">
            <div class="coin-3d" :class="{ flipping: coinflipFlipping }">
              <div class="coin-face front">🪙</div>
              <div class="coin-face back">📝</div>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!coinflipStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="coinflipBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div class="choice-row">
              <button 
                @click="coinflipChoice = 'صورة'" 
                class="choice-btn-small" 
                :class="{active: coinflipChoice === 'صورة'}"
              >
                🪙 صورة
              </button>
              <button 
                @click="coinflipChoice = 'كتابة'" 
                class="choice-btn-small" 
                :class="{active: coinflipChoice === 'كتابة'}"
              >
                📝 كتابة
              </button>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startCoinflip" class="casino-button" :disabled="!coinflipBet || coinflipBet <= 0 || !coinflipChoice">
              <span>اقلب العملة</span>
              <i class="fas fa-coins"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Wheel Spin -->
      <div v-if="selectedGame==='wheel'" class="casino-card">
        <div class="casino-card-header">
          <h2>🎡 WHEEL</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="wheel-container">
            <div class="wheel-outer" :style="{ transform: `rotate(${wheelRotation}deg)` }">
              <div v-for="(mult, index) in wheelMultipliers" :key="index" 
                   class="wheel-segment-casino" 
                   :style="{ transform: `rotate(${index * 45}deg)` }">
                <span class="segment-value-casino">{{ mult }}x</span>
              </div>
            </div>
            <div class="wheel-pointer">▼</div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!wheelStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="wheelBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startWheel" class="casino-button" :disabled="!wheelBet || wheelBet <= 0">
              <span>دور العجلة</span>
              <i class="fas fa-circle-notch"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Keno -->
      <div v-if="selectedGame==='keno'" class="casino-card">
        <div class="casino-card-header">
          <h2>🎱 KENO</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="keno-container">
            <div class="keno-grid">
              <div 
                v-for="n in 40" 
                :key="n"
                class="keno-ball"
                :class="{
                  'selected': kenoSelected.includes(n),
                  'drawn': kenoDrawn.includes(n)
                }"
                @click="toggleKenoNumber(n)"
              >
                {{ n }}
              </div>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!kenoStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="kenoBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startKeno" class="casino-button" :disabled="!kenoBet || kenoBet <= 0">
              <span>ابدأ</span>
              <i class="fas fa-dice-d6"></i>
            </button>
          </div>
          
          <div v-if="kenoStarted && kenoSelected.length > 0 && kenoDrawn.length === 0" class="action-row">
            <button @click="drawKeno" class="action-btn gold-btn">
              <i class="fas fa-dice"></i> سحب ({{ kenoSelected.length }})
            </button>
          </div>
        </div>
      </div>

      <!-- Bowling -->
      <div v-if="selectedGame==='bowling'" class="casino-card">
        <div class="casino-card-header">
          <h2>🎳 BOWLING</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="bowling-container">
            <div class="pins-setup">
              <div v-for="pin in 10" :key="pin" 
                   class="bowling-pin" 
                   :class="{ knocked: bowlingKnocked.includes(pin) }">
                🎳
              </div>
            </div>
            <div class="ball-track">
              <div class="bowling-ball-casino" :class="{ rolling: bowlingRolling }">⚫</div>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!bowlingStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="bowlingBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startBowling" class="casino-button" :disabled="!bowlingBet || bowlingBet <= 0">
              <span>ارم الكرة</span>
              <i class="fas fa-bowling-ball"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Puzzle -->
      <div v-if="selectedGame==='puzzle'" class="casino-card">
        <div class="casino-card-header">
          <h2>🧩 PUZZLE</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="puzzle-container">
            <div class="puzzle-grid-casino">
              <div 
                v-for="(piece, index) in puzzlePieces" 
                :key="index"
                class="puzzle-tile"
                :class="{ empty: piece === null }"
                @click="movePuzzle(index)"
              >
                {{ piece !== null ? piece : '' }}
              </div>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!puzzleStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="puzzleBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startPuzzle" class="casino-button" :disabled="!puzzleBet || puzzleBet <= 0">
              <span>ابدأ اللغز</span>
              <i class="fas fa-puzzle-piece"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Target Shot -->
      <div v-if="selectedGame==='target'" class="casino-card">
        <div class="casino-card-header">
          <h2>🎯 TARGET</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="target-container-casino">
            <div class="target-casino">
              <div class="target-ring bullseye" @click="shootTarget(10)">
                <span class="ring-value">10</span>
              </div>
              <div class="target-ring middle" @click="shootTarget(5)">
                <span class="ring-value">5</span>
              </div>
              <div class="target-ring outer" @click="shootTarget(2)">
                <span class="ring-value">2</span>
              </div>
            </div>
            <div class="target-score">النقاط: {{ targetScore }} / 30</div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!targetStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="targetBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startTarget" class="casino-button" :disabled="!targetBet || targetBet <= 0">
              <span>ابدأ التصويب</span>
              <i class="fas fa-bullseye"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Lucky Number -->
      <div v-if="selectedGame==='lucky'" class="casino-card">
        <div class="casino-card-header">
          <h2>🎲 LUCKY NUMBER</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="lucky-container">
            <div class="number-drum" :class="{ spinning: luckyStarted }">
              {{ luckyDrawn || '?' }}
            </div>
            <div class="chosen-number">رقمك: {{ luckyNumber || '--' }}</div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!luckyStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="luckyBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div class="chip-input">
              <span class="chip-icon">#</span>
              <input
                type="number"
                v-model.number="luckyNumber"
                placeholder="رقم الحظ"
                class="casino-input"
                min="1"
                max="100"
              />
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startLucky" class="casino-button" :disabled="!luckyBet || luckyBet <= 0 || !luckyNumber">
              <span>جرب حظك</span>
              <i class="fas fa-star"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- Mystery Box -->
      <div v-if="selectedGame==='mystery'" class="casino-card">
        <div class="casino-card-header">
          <h2>🎁 MYSTERY BOX</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="mystery-container">
            <div class="mystery-box" @click="openMysteryBox" :class="{ opened: mysteryOpened, shaking: !mysteryOpened }">
              <span v-if="!mysteryOpened">🎁</span>
              <span v-else class="box-content">{{ mysteryPrize }}</span>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!mysteryStarted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="mysteryBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
              />
              <span class="chip-currency">USDT</span>
            </div>
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            <button @click="startMystery" class="casino-button" :disabled="!mysteryBet || mysteryBet <= 0">
              <span>افتح الصندوق</span>
              <i class="fas fa-gift"></i>
            </button>
          </div>
        </div>
      </div>

      <!-- High/Low (Trading Game) -->
      <div v-if="selectedGame==='highlow'" class="casino-card">
        <div class="casino-card-header">
          <h2>📊 HIGH / LOW</h2>
          <div class="casino-glow"></div>
        </div>
        
        <div class="game-scene">
          <div class="trading-container">
            <!-- الرسم البياني -->
            <canvas ref="tradingChart" width="400" height="200" class="trading-chart"></canvas>
            
            <!-- السهم المتحرك -->
            <div class="arrow-container" :class="{ 'arrow-moving': isArrowMoving }">
              <div class="arrow-line"></div>
              <div class="arrow-head" :style="{ left: arrowPosition + '%' }">▶</div>
            </div>
            
            <!-- نقاط البداية والنهاية -->
            <div class="price-markers">
              <div class="start-marker">بداية ${{ startPrice.toFixed(2) }}</div>
              <div class="end-marker" v-if="!isArrowMoving">نهاية ${{ endPrice.toFixed(2) }}</div>
            </div>
            
            <!-- المؤقت -->
            <div v-if="isArrowMoving" class="trading-timer">
              <div class="timer-progress" :style="{ width: timerProgress + '%' }"></div>
              <span>{{ remainingTime }}s</span>
            </div>
          </div>
        </div>
        
        <div class="casino-controls">
          <div v-if="!isArrowMoving && !tradeCompleted" class="bet-panel">
            <div class="chip-input">
              <span class="chip-icon">💰</span>
              <input
                type="number"
                v-model.number="highLowBet"
                placeholder="0.00"
                class="casino-input"
                @input="clearGameError"
                :disabled="isBetLocked"
              />
              <span class="chip-currency">USDT</span>
            </div>
            
            <div v-if="gameError" class="casino-error">{{ gameError }}</div>
            
            <div class="trading-buttons">
              <button 
                @click="placeTrade('high')" 
                class="trading-btn high-btn"
                :disabled="!highLowBet || highLowBet <= 0 || isBetLocked"
              >
                <span class="btn-icon">📈</span>
                <span class="btn-text">HIGH</span>
                <span class="btn-multiplier">x1.8</span>
              </button>
              <button 
                @click="placeTrade('low')" 
                class="trading-btn low-btn"
                :disabled="!highLowBet || highLowBet <= 0 || isBetLocked"
              >
                <span class="btn-icon">📉</span>
                <span class="btn-text">LOW</span>
                <span class="btn-multiplier">x1.8</span>
              </button>
            </div>
          </div>
          
          <div v-if="tradeCompleted" class="trade-result" :class="{ 'win-result': highLowWon, 'lose-result': !highLowWon }">
            <div class="result-icon">{{ highLowWon ? '🎉' : '😢' }}</div>
            <div class="result-text">{{ highLowResultMessage }}</div>
            <div class="result-profit" v-if="highLowWon">+{{ highLowProfit.toFixed(2) }} USDT</div>
          </div>
        </div>
      </div>
    </div>
  </div>    
</template>    
    
<script>    
import { auth, db } from "../firebase";    
import { doc, getDoc, updateDoc } from "firebase/firestore";    
    
export default {    
  name: "Games",    
    
  data() {    
    return {    
      gameOpened: false,
      selectedGame: "chicken",
      showResultMessage: false,
      resultType: '',
      resultIcon: '',
      resultText: '',
      resultTimeout: null,
      
      gamesList: [
        { id: 'chicken', name: 'Chicken Road', icon: '🐔' },
        { id: 'dice', name: 'Dice', icon: '🎲' },
        { id: 'mines', name: 'Mines', icon: '💣' },
        { id: 'crash', name: 'Crash', icon: '🚀' },
        { id: 'limbo', name: 'Limbo', icon: '🎯' },
        { id: 'blackjack', name: 'Blackjack', icon: '🃏' },
        { id: 'slot', name: 'Slot', icon: '🎰' },
        { id: 'coinflip', name: 'Coinflip', icon: '🪙' },
        { id: 'wheel', name: 'Wheel', icon: '🎡' },
        { id: 'keno', name: 'Keno', icon: '🎱' },
        { id: 'bowling', name: 'Bowling', icon: '🎳' },
        { id: 'puzzle', name: 'Puzzle', icon: '🧩' },
        { id: 'target', name: 'Target', icon: '🎯' },
        { id: 'lucky', name: 'Lucky Number', icon: '🎮' },
        { id: 'mystery', name: 'Mystery Box', icon: '🎁' },
        { id: 'highlow', name: 'High/Low', icon: '📊' }
      ],
      balance: 0,    
      gameError: "",    
    
      /* Chicken Road */
      chickenBet: null,
      chickenStarted: false,
      chickenGameOver: false,
      chickenPosition: 0,
      chickenSteps: [
        { multiplier: 0.1 },
        { multiplier: 0.2 },
        { multiplier: 0.3 },
        { multiplier: 0.5 },
        { multiplier: 0.8 },
        { multiplier: 1.2 },
        { multiplier: 1.5 },
        { multiplier: 2.0 },
        { multiplier: 2.5 },
        { multiplier: 3.0 }
      ],
      
      /* Dice */
      diceBet: null,
      diceStarted: false,
      diceRolling: false,
      diceChance: 10,
      diceRoll: null,
      diceResultText: "",
      diceWon: false,
      
      /* Mines */
      minesBet: null,
      minesStarted: false,
      minesCount: 12,
      minesCells: [],
      minesGameOver: false,
      minesRevealed: 0,
      
      /* Crash */
      crashBet: null,
      crashStarted: false,
      crashMultiplier: 1.0,
      crashCrashed: false,
      crashProgress: 0,
      crashAutoCashout: 2.0,
      crashInterval: null,
      
      /* Limbo */
      limboBet: null,
      limboStarted: false,
      limboTarget: 2.0,
      limboResult: 0,
      limboWon: false,
      
      /* Blackjack */
      blackjackBet: null,
      blackjackStarted: false,
      blackjackGameOver: false,
      blackjackWon: false,
      blackjackResult: "",
      dealerHand: [],
      playerHand: [],
      dealerRevealed: false,
      deck: [],
      
      /* Slot Machine */
      slotBet: null,
      slotStarted: false,
      slotSpinning: false,
      slotReels: ['🍒', '🍒', '🍒'],
      slotWon: false,
      slotResult: "",
      slotSymbols: ['🍒', '🍋', '🍊', '7️⃣', '💎', '🔔'],
      
      /* Coinflip */
      coinflipBet: null,
      coinflipStarted: false,
      coinflipChoice: "",
      coinflipResult: "",
      coinflipWon: false,
      coinflipFlipping: false,
      
      /* Wheel */
      wheelBet: null,
      wheelStarted: false,
      wheelRotation: 0,
      wheelResult: "",
      wheelMultipliers: [0.2, 0.5, 1, 0.3, 0.8, 1.5, 2, 0.1],
      
      /* Keno */
      kenoBet: null,
      kenoStarted: false,
      kenoSelected: [],
      kenoDrawn: [],
      kenoMatches: 0,
      kenoMultiplier: 0,
      
      /* Bowling */
      bowlingBet: null,
      bowlingStarted: false,
      bowlingKnocked: [],
      bowlingRolling: false,
      bowlingResult: "",
      
      /* Puzzle */
      puzzleBet: null,
      puzzleStarted: false,
      puzzlePieces: [],
      puzzleSolved: false,
      
      /* Target */
      targetBet: null,
      targetStarted: false,
      targetResult: "",
      targetScore: 0,
      
      /* Lucky Number */
      luckyBet: null,
      luckyStarted: false,
      luckyNumber: null,
      luckyDrawn: null,
      luckyWon: false,
      
      /* Mystery Box */
      mysteryBet: null,
      mysteryStarted: false,
      mysteryOpened: false,
      mysteryPrize: "",
      mysteryResult: "",
      
      /* High/Low (Trading Game) */
      highLowBet: null,
      highLowChoice: null,
      isArrowMoving: false,
      isBetLocked: false,
      tradeCompleted: false,
      highLowWon: false,
      highLowResultMessage: "",
      highLowProfit: 0,
      
      // Chart data
      chartData: [],
      chartCanvas: null,
      chartContext: null,
      animationFrame: null,
      
      // Arrow position (0-100%)
      arrowPosition: 0,
      
      // Timer
      totalTime: 5,
      remainingTime: 5,
      timerProgress: 100,
      timerInterval: null,
      
      // Price tracking
      startPrice: 100,
      currentPrice: 100,
      endPrice: 100,
      
      // RNG control (45% win rate for player)
      winProbability: 0.45
    };    
  },    
    
  computed: {    
    chickenProfit() {    
      if (!this.chickenBet) return 0;    
      return this.chickenBet * this.chickenSteps[this.chickenPosition].multiplier;    
    },
    
    minesProfit() {
      if (!this.minesBet) return 0;
      let multiplier = 0.1 + (this.minesRevealed * 0.2);
      return this.minesBet * Math.min(multiplier, 3);
    },
    
    playerScore() {
      return this.calculateHand(this.playerHand);
    }
  },    
    
  async created() {    
    const user = auth.currentUser;    
    if (!user) return;    
    const snap = await getDoc(doc(db, "users", user.uid));    
    if (snap.exists()) {    
      this.balance = Number(snap.data().balance || 0);    
    }    
  },    
    
  mounted() {
    // Initialize chart when highlow game is opened
    this.$watch('selectedGame', (newVal) => {
      if (newVal === 'highlow' && this.gameOpened) {
        this.$nextTick(() => {
          this.initChart();
        });
      }
    });
  },
  
  beforeDestroy() {
    this.cleanupHighLow();
  },
    
  methods: {    
    openGame(gameId) {
      this.selectedGame = gameId;
      this.gameOpened = true;
      this.gameError = "";
      
      // Initialize chart for highlow game
      if (gameId === 'highlow') {
        this.$nextTick(() => {
          this.initChart();
          this.resetHighLow();
        });
      }
    },
    
    closeGame() {
      this.gameOpened = false;
      // إيقاف جميع الألعاب عند الرجوع
      this.chickenStarted = false;
      this.chickenGameOver = false;
      this.diceStarted = false;
      this.minesStarted = false;
      if (this.crashInterval) clearInterval(this.crashInterval);
      this.crashStarted = false;
      this.limboStarted = false;
      this.blackjackStarted = false;
      this.slotStarted = false;
      this.coinflipStarted = false;
      this.wheelStarted = false;
      this.kenoStarted = false;
      this.bowlingStarted = false;
      this.puzzleStarted = false;
      this.targetStarted = false;
      this.luckyStarted = false;
      this.mysteryStarted = false;
      
      // Cleanup highlow
      this.cleanupHighLow();
    },
    
    clearGameError() {
      this.gameError = "";
    },
    
    async updateBalance(newBalance) {
      this.balance = newBalance;
      await updateDoc(doc(db, "users", auth.currentUser.uid), {
        balance: this.balance,
      });
    },
    
    showResult(message, isWin) {
      if (this.resultTimeout) clearTimeout(this.resultTimeout);
      
      this.resultText = message;
      this.resultType = isWin ? 'win-message' : 'lose-message';
      this.resultIcon = isWin ? 'fas fa-trophy' : 'fas fa-skull';
      this.showResultMessage = true;
      
      this.resultTimeout = setTimeout(() => {
        this.showResultMessage = false;
      }, 2000);
    },
    
    /* ===== High/Low Trading Game ===== */
    initChart() {
      this.chartCanvas = this.$refs.tradingChart;
      if (!this.chartCanvas) return;
      
      this.chartContext = this.chartCanvas.getContext('2d');
      this.generateChartData();
      this.drawChart();
    },
    
    generateChartData() {
      // Generate random chart data (50 points)
      this.chartData = [];
      let value = 100;
      for (let i = 0; i < 50; i++) {
        // Random walk
        value += (Math.random() - 0.5) * 4;
        this.chartData.push(value);
      }
      this.startPrice = this.chartData[this.chartData.length - 1];
      this.currentPrice = this.startPrice;
    },
    
    drawChart() {
      if (!this.chartContext || !this.chartCanvas) return;
      
      const ctx = this.chartContext;
      const canvas = this.chartCanvas;
      const width = canvas.width;
      const height = canvas.height;
      
      // Clear canvas
      ctx.clearRect(0, 0, width, height);
      
      // Draw grid
      ctx.strokeStyle = 'rgba(255, 215, 0, 0.2)';
      ctx.lineWidth = 1;
      
      // Horizontal grid lines
      for (let i = 0; i <= 4; i++) {
        const y = height * i / 4;
        ctx.beginPath();
        ctx.moveTo(0, y);
        ctx.lineTo(width, y);
        ctx.strokeStyle = 'rgba(255, 215, 0, 0.1)';
        ctx.stroke();
      }
      
      // Draw chart line
      if (this.chartData.length < 2) return;
      
      const minPrice = Math.min(...this.chartData);
      const maxPrice = Math.max(...this.chartData);
      const priceRange = maxPrice - minPrice || 1;
      
      ctx.beginPath();
      ctx.lineWidth = 2;
      
      for (let i = 0; i < this.chartData.length; i++) {
        const x = (i / (this.chartData.length - 1)) * width;
        const y = height - ((this.chartData[i] - minPrice) / priceRange) * height * 0.8 - height * 0.1;
        
        if (i === 0) {
          ctx.moveTo(x, y);
        } else {
          // Determine color based on price movement
          if (this.chartData[i] > this.chartData[i-1]) {
            ctx.strokeStyle = '#4caf50';
          } else {
            ctx.strokeStyle = '#f44336';
          }
          ctx.lineTo(x, y);
        }
      }
      ctx.stroke();
      
      // Fill area under the line
      ctx.lineTo(width, height);
      ctx.lineTo(0, height);
      ctx.closePath();
      ctx.globalAlpha = 0.1;
      ctx.fillStyle = '#ffd700';
      ctx.fill();
      ctx.globalAlpha = 1;
    },
    
    updateChart() {
      // Add new random point to chart
      const lastValue = this.chartData[this.chartData.length - 1];
      const change = (Math.random() - 0.5) * 6;
      const newValue = lastValue + change;
      
      this.chartData.push(newValue);
      this.chartData.shift();
      
      this.currentPrice = newValue;
      this.drawChart();
    },
    
    resetHighLow() {
      this.highLowBet = null;
      this.highLowChoice = null;
      this.isArrowMoving = false;
      this.isBetLocked = false;
      this.tradeCompleted = false;
      this.arrowPosition = 0;
      this.remainingTime = this.totalTime;
      this.timerProgress = 100;
      this.startPrice = 100;
      this.currentPrice = 100;
      this.generateChartData();
      this.drawChart();
    },
    
    cleanupHighLow() {
      if (this.timerInterval) {
        clearInterval(this.timerInterval);
        this.timerInterval = null;
      }
      if (this.animationFrame) {
        cancelAnimationFrame(this.animationFrame);
        this.animationFrame = null;
      }
      this.isArrowMoving = false;
    },
    
    async placeTrade(choice) {
      // Prevent spam
      if (this.isBetLocked || this.isArrowMoving) {
        this.gameError = "الرجاء الانتظار حتى انتهاء الجولة الحالية";
        return;
      }
      
      if (!this.highLowBet || this.highLowBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      
      if (this.highLowBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.highLowChoice = choice;
      this.isBetLocked = true;
      
      // خصم الرهان
      this.balance -= this.highLowBet;
      await this.updateBalance(this.balance);
      
      // بدء حركة السهم
      this.startTrading();
    },
    
    startTrading() {
      this.isArrowMoving = true;
      this.tradeCompleted = false;
      this.arrowPosition = 0;
      this.remainingTime = this.totalTime;
      this.timerProgress = 100;
      
      // تحديد النتيجة مسبقاً باستخدام RNG (45% فوز للاعب)
      const random = Math.random();
      const willPriceGoUp = random < this.winProbability ? this.highLowChoice === 'high' : this.highLowChoice !== 'high';
      
      // حساب السعر النهائي بناءً على النتيجة
      const startPrice = this.startPrice;
      let finalPrice;
      
      if (willPriceGoUp) {
        // السعر يرتفع (فوز إذا اختار high)
        finalPrice = startPrice * (1 + (Math.random() * 0.15 + 0.05));
      } else {
        // السعر ينخفض (فوز إذا اختار low)
        finalPrice = startPrice * (1 - (Math.random() * 0.15 + 0.05));
      }
      
      this.endPrice = finalPrice;
      
      // حركة السهم والمؤشر
      const startTime = Date.now();
      const duration = this.totalTime * 1000;
        
      const animate = () => {
        const elapsed = Date.now() - startTime;
        const progress = Math.min(elapsed / duration, 1);
        
        // تحديث موضع السهم (0% إلى 100%)
        this.arrowPosition = progress * 100;
        
        // تحديث السعر الحالي (يتحرك بشكل عشوائي)
        const currentProgressPrice = startPrice + (finalPrice - startPrice) * progress;
        const randomFactor = (Math.random() - 0.5) * 2;
        this.currentPrice = currentProgressPrice + randomFactor;
        
        // تحديث الرسم البياني
        this.updateChart();
        
        if (progress < 1) {
          this.animationFrame = requestAnimationFrame(animate);
        } else {
          // انتهت الحركة
          this.currentPrice = finalPrice;
          this.finishTrade();
        }
      };
      
      this.animationFrame = requestAnimationFrame(animate);
      
      // تحديث المؤقت
      this.timerInterval = setInterval(() => {
        if (this.remainingTime > 0) {
          this.remainingTime -= 1;
          this.timerProgress = (this.remainingTime / this.totalTime) * 100;
        }
      }, 1000);
    },
    
    async finishTrade() {
      // إيقاف المؤقت والأنيميشن
      if (this.timerInterval) {
        clearInterval(this.timerInterval);
        this.timerInterval = null;
      }
      if (this.animationFrame) {
        cancelAnimationFrame(this.animationFrame);
        this.animationFrame = null;
      }
      
      this.isArrowMoving = false;
      
      // تحديد نتيجة التداول
      const priceIncreased = this.endPrice > this.startPrice;
      
      if ((this.highLowChoice === 'high' && priceIncreased) || 
          (this.highLowChoice === 'low' && !priceIncreased)) {
        // فوز
        this.highLowWon = true;
        this.highLowProfit = this.highLowBet * 1.8;
        this.balance += this.highLowProfit;
        await this.updateBalance(this.balance);
        this.highLowResultMessage = `فوز! السعر ${priceIncreased ? 'ارتفع' : 'انخفض'} إلى $${this.endPrice.toFixed(2)}`;
        this.showResult(`🎉 ربحت ${this.highLowProfit.toFixed(2)} USDT`, true);
      } else {
        // خسارة
        this.highLowWon = false;
        this.highLowProfit = 0;
        this.highLowResultMessage = `خسارة! السعر ${priceIncreased ? 'ارتفع' : 'انخفض'} إلى $${this.endPrice.toFixed(2)}`;
        this.showResult(`😢 خسرت الرهان`, false);
      }
      
      this.tradeCompleted = true;
      
      // فتح الرهان مرة أخرى بعد 3 ثواني
      setTimeout(() => {
        this.isBetLocked = false;
        this.resetHighLow();
      }, 3000);
    },
    
    /* ===== Chicken Road ===== */
    async startChicken() {
      if (!this.chickenBet || this.chickenBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.chickenBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.chickenBet;
      await this.updateBalance(this.balance);
      
      this.chickenStarted = true;
      this.chickenGameOver = false;
      this.chickenPosition = 0;
    },
    
    chickenNext() {
      if (this.chickenGameOver) return;
      
      const loseChance = 0.7 + (this.chickenPosition * 0.03);
      if (Math.random() < loseChance) {
        this.chickenGameOver = true;
        this.showResult("💥 خسرت الرهان!", false);
        setTimeout(() => {
          this.chickenStarted = false;
        }, 2000);
        return;
      }
      
      if (this.chickenPosition < this.chickenSteps.length - 1) {
        this.chickenPosition++;
      } else {
        this.chickenCashOut();
      }
    },
    
    async chickenCashOut() {
      if (this.chickenGameOver) return;
      
      const profit = this.chickenProfit;
      this.balance += profit;
      await this.updateBalance(this.balance);
      this.showResult(`🎉 ربحت ${profit.toFixed(2)} USDT`, true);
      this.chickenGameOver = true;
      setTimeout(() => {
        this.chickenStarted = false;
      }, 2000);
    },
    
    /* ===== Dice ===== */
    async startDice() {
      if (!this.diceBet || this.diceBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.diceBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.diceBet;
      await this.updateBalance(this.balance);
      
      this.diceStarted = true;
      this.diceRolling = true;
      
      let rolls = 0;
      const interval = setInterval(() => {
        this.diceRoll = Math.floor(Math.random() * 100) + 1;
        rolls++;
        if (rolls >= 15) {
          clearInterval(interval);
          this.diceRolling = false;
          this.checkDiceResult();
        }
      }, 100);
    },
    
    async checkDiceResult() {
      this.diceWon = this.diceRoll <= this.diceChance;
      const multiplier = 95 / this.diceChance;
      const profit = this.diceBet * multiplier;
      
      if (this.diceWon) {
        this.balance += profit;
        await this.updateBalance(this.balance);
        this.showResult(`🎉 ربحت ${profit.toFixed(2)} USDT`, true);
      } else {
        this.showResult(`😢 خسرت! الرقم كان ${this.diceRoll}`, false);
      }
      
      setTimeout(() => {
        this.diceStarted = false;
      }, 2000);
    },
    
    /* ===== Mines ===== */
    async startMines() {
      if (!this.minesBet || this.minesBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.minesBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.minesBet;
      await this.updateBalance(this.balance);
      
      this.minesStarted = true;
      this.minesGameOver = false;
      this.minesRevealed = 0;
      
      this.minesCells = [];
      for (let i = 0; i < 25; i++) {
        this.minesCells.push({ revealed: false, mine: false });
      }
      
      let minesPlaced = 0;
      while (minesPlaced < this.minesCount) {
        const index = Math.floor(Math.random() * 25);
        if (!this.minesCells[index].mine) {
          this.minesCells[index].mine = true;
          minesPlaced++;
        }
      }
    },
    
    revealMine(index) {
      if (this.minesGameOver || this.minesCells[index].revealed) return;
      
      this.minesCells[index].revealed = true;
      
      if (this.minesCells[index].mine) {
        this.minesGameOver = true;
        this.showResult("💥 انفجر لغم! خسرت الرهان", false);
        setTimeout(() => {
          this.minesStarted = false;
        }, 2000);
      } else {
        this.minesRevealed++;
      }
    },
    
    async minesCashOut() {
      if (this.minesGameOver) return;
      
      const profit = this.minesProfit;
      this.balance += profit;
      await this.updateBalance(this.balance);
      this.showResult(`🎉 ربحت ${profit.toFixed(2)} USDT`, true);
      this.minesGameOver = true;
      setTimeout(() => {
        this.minesStarted = false;
      }, 2000);
    },
    
    /* ===== Crash ===== */
    async startCrash() {
      if (!this.crashBet || this.crashBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.crashBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.crashBet;
      await this.updateBalance(this.balance);
      
      this.crashStarted = true;
      this.crashCrashed = false;
      this.crashMultiplier = 1.0;
      this.crashProgress = 0;
      
      const crashPoint = 1.1 + (Math.random() * 4);
      
      this.crashInterval = setInterval(() => {
        if (this.crashCrashed) return;
        
        this.crashMultiplier += 0.02;
        this.crashProgress = ((this.crashMultiplier - 1) / 5) * 100;
        
        if (this.crashMultiplier >= crashPoint) {
          this.crashCrashed = true;
          clearInterval(this.crashInterval);
          this.showResult("💥 انفجر الصاروخ!", false);
          setTimeout(() => {
            this.crashStarted = false;
          }, 2000);
        }
        
        if (this.crashMultiplier >= this.crashAutoCashout && !this.crashCrashed) {
          this.crashCashOut();
        }
      }, 200);
    },
    
    async crashCashOut() {
      if (this.crashCrashed) return;
      
      clearInterval(this.crashInterval);
      const profit = this.crashBet * this.crashMultiplier;
      this.balance += profit;
      await this.updateBalance(this.balance);
      this.showResult(`🎉 سحبت وربحت ${profit.toFixed(2)} USDT`, true);
      this.crashCrashed = true;
      setTimeout(() => {
        this.crashStarted = false;
      }, 2000);
    },
    
    /* ===== Limbo ===== */
    async startLimbo() {
      if (!this.limboBet || this.limboBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.limboBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.limboBet;
      await this.updateBalance(this.balance);
      
      this.limboStarted = true;
      
      setTimeout(() => {
        this.limboResult = 0.1 + (Math.random() * 5);
        this.limboWon = this.limboResult >= this.limboTarget;
        
        if (this.limboWon) {
          const profit = this.limboBet * this.limboResult;
          this.balance += profit;
          this.updateBalance(this.balance);
          this.showResult(`🎉 فوز! المضاعف ${this.limboResult.toFixed(2)}x`, true);
        } else {
          this.showResult(`😢 خسارة! المضاعف ${this.limboResult.toFixed(2)}x`, false);
        }
        
        setTimeout(() => {
          this.limboStarted = false;
        }, 2000);
      }, 1500);
    },
    
    /* ===== Blackjack ===== */
    createDeck() {
      const suits = ['♥', '♦', '♣', '♠'];
      const values = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];
      const deck = [];
      for (let suit of suits) {
        for (let value of values) {
          deck.push(value + suit);
        }
      }
      return this.shuffle(deck);
    },
    
    shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    },
    
    getCardValue(card) {
      const value = card.slice(0, -1);
      if (value === 'A') return 11;
      if (['K', 'Q', 'J'].includes(value)) return 10;
      return parseInt(value);
    },
    
    calculateHand(hand) {
      let sum = 0;
      let aces = 0;
      for (let card of hand) {
        const value = this.getCardValue(card);
        if (value === 11) aces++;
        sum += value;
      }
      while (sum > 21 && aces > 0) {
        sum -= 10;
        aces--;
      }
      return sum;
    },
    
    async startBlackjack() {
      if (!this.blackjackBet || this.blackjackBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.blackjackBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.blackjackBet;
      await this.updateBalance(this.balance);
      
      this.blackjackStarted = true;
      this.blackjackGameOver = false;
      this.dealerRevealed = false;
      this.deck = this.createDeck();
      
      this.playerHand = [this.deck.pop(), this.deck.pop()];
      this.dealerHand = [this.deck.pop(), this.deck.pop()];
    },
    
    blackjackHit() {
      if (this.blackjackGameOver) return;
      
      this.playerHand.push(this.deck.pop());
      if (this.calculateHand(this.playerHand) > 21) {
        this.blackjackStand();
      }
    },
    
    async blackjackStand() {
      this.dealerRevealed = true;
      
      setTimeout(async () => {
        while (this.calculateHand(this.dealerHand) < 17) {
          this.dealerHand.push(this.deck.pop());
        }
        
        const playerScore = this.calculateHand(this.playerHand);
        const dealerScore = this.calculateHand(this.dealerHand);
        
        this.blackjackGameOver = true;
        
        if (playerScore > 21) {
          this.showResult("😢 خسرت! نقاطك تجاوزت 21", false);
        } else if (dealerScore > 21) {
          const profit = this.blackjackBet * 2;
          this.balance += profit;
          await this.updateBalance(this.balance);
          this.showResult(`🎉 فوز! ربحت ${profit.toFixed(2)} USDT`, true);
        } else if (playerScore > dealerScore) {
          const profit = this.blackjackBet * 2;
          this.balance += profit;
          await this.updateBalance(this.balance);
          this.showResult(`🎉 فوز! ${playerScore} ضد ${dealerScore}`, true);
        } else if (playerScore < dealerScore) {
          this.showResult(`😢 خسارة! ${playerScore} ضد ${dealerScore}`, false);
        } else {
          this.balance += this.blackjackBet;
          await this.updateBalance(this.balance);
          this.showResult(`🤝 تعادل! استرداد الرهان`, true);
        }
      }, 1000);
    },
    
    /* ===== Slot Machine ===== */
    async startSlot() {
      if (!this.slotBet || this.slotBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.slotBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.slotBet;
      await this.updateBalance(this.balance);
      
      this.slotStarted = true;
      this.slotSpinning = true;
      
      let spins = 0;
      const spinInterval = setInterval(() => {
        this.slotReels = [
          this.slotSymbols[Math.floor(Math.random() * this.slotSymbols.length)],
          this.slotSymbols[Math.floor(Math.random() * this.slotSymbols.length)],
          this.slotSymbols[Math.floor(Math.random() * this.slotSymbols.length)]
        ];
        spins++;
        
        if (spins >= 20) {
          clearInterval(spinInterval);
          this.slotSpinning = false;
          this.checkSlotWin();
        }
      }, 150);
    },
    
    async checkSlotWin() {
      const [a, b, c] = this.slotReels;
      this.slotWon = a === b && b === c;
      
      if (this.slotWon) {
        let multiplier = 0.5;
        if (a === '7️⃣') multiplier = 3;
        else if (a === '💎') multiplier = 2;
        else if (a === '🔔') multiplier = 1.5;
        else multiplier = 1;
        
        const profit = this.slotBet * multiplier;
        this.balance += profit;
        await this.updateBalance(this.balance);
        this.showResult(`🎉 ربحت ${profit.toFixed(2)} USDT`, true);
      } else {
        this.showResult("😢 للأسف لا فوز", false);
      }
      
      setTimeout(() => {
        this.slotStarted = false;
      }, 2000);
    },
    
    /* ===== Coinflip ===== */
    async startCoinflip() {
      if (!this.coinflipBet || this.coinflipBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (!this.coinflipChoice) {
        this.gameError = "اختر صورة أو كتابة";
        return;
      }
      if (this.coinflipBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.coinflipBet;
      await this.updateBalance(this.balance);
      
      this.coinflipStarted = true;
      this.coinflipFlipping = true;
      
      setTimeout(() => {
        this.coinflipResult = Math.random() < 0.5 ? "صورة" : "كتابة";
        this.coinflipWon = this.coinflipResult === this.coinflipChoice;
        this.coinflipFlipping = false;
        
        if (this.coinflipWon) {
          const profit = this.coinflipBet * 1.8;
          this.balance += profit;
          this.updateBalance(this.balance);
          this.showResult(`🎉 فوز! ربحت ${profit.toFixed(2)} USDT`, true);
        } else {
          this.showResult(`😢 خسارة! كانت النتيجة ${this.coinflipResult}`, false);
        }
        
        setTimeout(() => {
          this.coinflipStarted = false;
        }, 2000);
      }, 2000);
    },
    
    /* ===== Wheel Spin ===== */
    async startWheel() {
      if (!this.wheelBet || this.wheelBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.wheelBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.wheelBet;
      await this.updateBalance(this.balance);
      
      this.wheelStarted = true;
      const spins = 10 + Math.floor(Math.random() * 10);
      const targetIndex = Math.floor(Math.random() * 8);
      const targetRotation = 360 * spins + (targetIndex * 45);
      
      this.wheelRotation = 0;
      const interval = setInterval(() => {
        this.wheelRotation += 15;
        if (this.wheelRotation >= targetRotation) {
          clearInterval(interval);
          
          setTimeout(async () => {
            const multiplier = this.wheelMultipliers[targetIndex];
            const profit = this.wheelBet * multiplier;
            this.balance += profit;
            await this.updateBalance(this.balance);
            
            if (multiplier >= 1) {
              this.showResult(`🎉 ربحت ${profit.toFixed(2)} USDT`, true);
            } else {
              this.showResult(`😢 خسرت! المضاعف ${multiplier}x`, false);
            }
            
            setTimeout(() => {
              this.wheelStarted = false;
            }, 2000);
          }, 500);
        }
      }, 20);
    },
    
    /* ===== Keno ===== */
    async startKeno() {
      if (!this.kenoBet || this.kenoBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.kenoBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.kenoBet;
      await this.updateBalance(this.balance);
      
      this.kenoStarted = true;
      this.kenoSelected = [];
      this.kenoDrawn = [];
      this.kenoMatches = 0;
    },
    
    toggleKenoNumber(n) {
      if (this.kenoDrawn.length > 0) return;
      
      const index = this.kenoSelected.indexOf(n);
      if (index === -1) {
        if (this.kenoSelected.length < 4) {
          this.kenoSelected.push(n);
        }
      } else {
        this.kenoSelected.splice(index, 1);
      }
    },
    
    async drawKeno() {
      if (this.kenoSelected.length === 0) return;
      
      this.kenoDrawn = [];
      const numbers = Array.from({ length: 40 }, (_, i) => i + 1);
      
      let drawn = 0;
      const interval = setInterval(() => {
        if (drawn < 20) {
          const index = Math.floor(Math.random() * numbers.length);
          this.kenoDrawn.push(numbers[index]);
          numbers.splice(index, 1);
          drawn++;
        } else {
          clearInterval(interval);
          this.calculateKenoResult();
        }
      }, 200);
    },
    
    async calculateKenoResult() {
      this.kenoMatches = this.kenoSelected.filter(n => this.kenoDrawn.includes(n)).length;
      
      const multipliers = [0, 0.1, 0.5, 1, 2];
      this.kenoMultiplier = multipliers[this.kenoMatches] || 0;
      
      if (this.kenoMultiplier > 0) {
        const profit = this.kenoBet * this.kenoMultiplier;
        this.balance += profit;
        await this.updateBalance(this.balance);
        this.showResult(`🎉 ${this.kenoMatches} تطابق! ربحت ${profit.toFixed(2)} USDT`, true);
      } else {
        this.showResult(`😢 ${this.kenoMatches} تطابق. لا فوز`, false);
      }
    },
    
    /* ===== Bowling ===== */
    async startBowling() {
      if (!this.bowlingBet || this.bowlingBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.bowlingBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.bowlingBet;
      await this.updateBalance(this.balance);
      
      this.bowlingStarted = true;
      this.bowlingRolling = true;
      this.bowlingKnocked = [];
      
      setTimeout(() => {
        const knocked = Math.floor(Math.random() * 9);
        
        let knockInterval = setInterval(() => {
          if (this.bowlingKnocked.length < knocked) {
            const pin = Math.floor(Math.random() * 10) + 1;
            if (!this.bowlingKnocked.includes(pin)) {
              this.bowlingKnocked.push(pin);
            }
          } else {
            clearInterval(knockInterval);
            this.bowlingRolling = false;
            this.calculateBowlingResult(knocked);
          }
        }, 100);
      }, 1500);
    },
    
    async calculateBowlingResult(knocked) {
      let multiplier = 0.1;
      if (knocked === 10) multiplier = 2;
      else if (knocked >= 8) multiplier = 1.5;
      else if (knocked >= 5) multiplier = 1;
      else if (knocked >= 3) multiplier = 0.5;
      
      const profit = this.bowlingBet * multiplier;
      this.balance += profit;
      await this.updateBalance(this.balance);
      
      if (knocked === 10) {
        this.showResult(`🎳 سترايك! ربحت ${profit.toFixed(2)} USDT`, true);
      } else {
        this.showResult(`🎳 ضربت ${knocked} قوارير وخسرت`, false);
      }
      
      setTimeout(() => {
        this.bowlingStarted = false;
      }, 2000);
    },
    
    /* ===== Puzzle ===== */
    async startPuzzle() {
      if (!this.puzzleBet || this.puzzleBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.puzzleBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.puzzleBet;
      await this.updateBalance(this.balance);
      
      this.puzzleStarted = true;
      this.puzzleSolved = false;
      
      let pieces = [1, 2, 3, 4, 5, 6, 7, 8, null];
      pieces = this.shuffle(pieces);
      this.puzzlePieces = pieces;
    },
    
    movePuzzle(index) {
      if (this.puzzleSolved) return;
      
      const emptyIndex = this.puzzlePieces.indexOf(null);
      const validMoves = [
        emptyIndex - 1, emptyIndex + 1,
        emptyIndex - 3, emptyIndex + 3
      ];
      
      if (validMoves.includes(index)) {
        this.puzzlePieces[emptyIndex] = this.puzzlePieces[index];
        this.puzzlePieces[index] = null;
      }
      
      if (JSON.stringify(this.puzzlePieces) === JSON.stringify([1, 2, 3, 4, 5, 6, 7, 8, null])) {
        this.puzzleSolved = true;
        const profit = this.puzzleBet * 2;
        this.balance += profit;
        this.updateBalance(this.balance);
        this.showResult(`🧩 أحسنت! ربحت ${profit.toFixed(2)} USDT`, true);
        
        setTimeout(() => {
          this.puzzleStarted = false;
        }, 2000);
      }
    },
    
    /* ===== Target Shot ===== */
    async startTarget() {
      if (!this.targetBet || this.targetBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.targetBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.targetBet;
      await this.updateBalance(this.balance);
      
      this.targetStarted = true;
      this.targetScore = 0;
    },
    
    async shootTarget(score) {
      const hit = Math.random() < 0.3;
      
      if (hit) {
        this.targetScore += score;
      }
      
      if (this.targetScore >= 30) {
        const multiplier = 1.5;
        const profit = this.targetBet * multiplier;
        this.balance += profit;
        await this.updateBalance(this.balance);
        this.showResult(`🎉 فوز! ربحت ${profit.toFixed(2)} USDT`, true);
        
        setTimeout(() => {
          this.targetStarted = false;
        }, 2000);
      } else if (!hit) {
        this.showResult(`😢 أخطأت الهدف`, false);
      }
    },
    
    /* ===== Lucky Number ===== */
    async startLucky() {
      if (!this.luckyBet || this.luckyBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (!this.luckyNumber || this.luckyNumber < 1 || this.luckyNumber > 100) {
        this.gameError = "أدخل رقم بين 1 و 100";
        return;
      }
      if (this.luckyBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.luckyBet;
      await this.updateBalance(this.balance);
      
      this.luckyStarted = true;
      
      let rolls = 0;
      const interval = setInterval(() => {
        this.luckyDrawn = Math.floor(Math.random() * 100) + 1;
        rolls++;
        
        if (rolls >= 20) {
          clearInterval(interval);
          this.luckyWon = this.luckyDrawn === this.luckyNumber;
          
          if (this.luckyWon) {
            const profit = this.luckyBet * 10;
            this.balance += profit;
            this.updateBalance(this.balance);
            this.showResult(`🎉 رقم محظوظ! ربحت ${profit.toFixed(2)} USDT`, true);
          } else {
            this.showResult(`😢 الرقم المسحوب ${this.luckyDrawn}`, false);
          }
          
          setTimeout(() => {
            this.luckyStarted = false;
          }, 2000);
        }
      }, 100);
    },
    
    /* ===== Mystery Box ===== */
    async startMystery() {
      if (!this.mysteryBet || this.mysteryBet <= 0) {
        this.gameError = "الرجاء إدخال مبلغ الرهان";
        return;
      }
      if (this.mysteryBet > this.balance) {
        this.gameError = "الرصيد غير كافي";
        return;
      }
      
      this.gameError = "";
      this.balance -= this.mysteryBet;
      await this.updateBalance(this.balance);
      
      this.mysteryStarted = true;
      this.mysteryOpened = false;
    },
    
    async openMysteryBox() {
      if (this.mysteryOpened) return;
      
      this.mysteryOpened = true;
      
      setTimeout(async () => {
        const prizes = [0.1, 0.2, 0.3, 0.5, 0.8, 1, 1.2, 1.5];
        const weights = [0.25, 0.2, 0.15, 0.12, 0.1, 0.08, 0.05, 0.05];
        
        let random = Math.random();
        let cumulative = 0;
        let multiplier = 0.1;
        
        for (let i = 0; i < weights.length; i++) {
          cumulative += weights[i];
          if (random < cumulative) {
            multiplier = prizes[i];
            break;
          }
        }
        
        const profit = this.mysteryBet * multiplier;
        this.balance += profit;
        await this.updateBalance(this.balance);
        
        if (multiplier >= 1) {
          this.mysteryPrize = `${profit.toFixed(2)} USDT`;
          this.showResult(`🎉 ربحت ${profit.toFixed(2)} USDT!`, true);
        } else {
          this.mysteryPrize = `${profit.toFixed(2)} USDT`;
          this.showResult(`😢 خسرت جزء من الرهان`, false);
        }
      }, 1000);
    }
  }
};    
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

/* ===== الكروت الرئيسية ===== */
.casino-card {
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

.casino-card::before {
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

.casino-card-header {
  position: relative;
  margin-bottom: 25px;
  text-align: center;
}

.casino-card-header h2 {
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

.casino-glow {
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

/* ===== مشهد اللعبة ===== */
.game-scene {
  background: rgba(10, 15, 30, 0.7);
  border-radius: 30px;
  padding: 25px;
  margin-bottom: 25px;
  border: 1px solid rgba(255, 215, 0, 0.2);
  box-shadow: inset 0 0 30px rgba(0, 0, 0, 0.5);
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* ===== عناصر التحكم ===== */
.casino-controls {
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

.chip-icon {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 18px;
  z-index: 2;
}

.casino-input {
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

.casino-input:focus {
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

.casino-button {
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

.casino-button:hover:not(:disabled) {
  transform: translateY(-5px);
  box-shadow: 0 15px 35px rgba(255, 215, 0, 0.6), inset 0 2px 10px rgba(255, 255, 255, 0.7);
}

.casino-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  filter: grayscale(0.5);
}

.casino-error {
  color: #f44336;
  font-size: 14px;
  background: rgba(244, 67, 54, 0.1);
  padding: 8px 20px;
  border-radius: 50px;
  border: 1px solid #f44336;
}

/* ===== عناصر اللعبة المشتركة ===== */
.action-row {
  display: flex;
  gap: 15px;
  justify-content: center;
  width: 100%;
}

.action-btn {
  padding: 12px 25px;
  border-radius: 50px;
  border: none;
  font-weight: 700;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
  justify-content: center;
}

.success-btn {
  background: linear-gradient(135deg, #4caf50, #45a049);
  color: white;
  box-shadow: 0 5px 15px rgba(76, 175, 80, 0.3);
}

.danger-btn {
  background: linear-gradient(135deg, #f44336, #d32f2f);
  color: white;
  box-shadow: 0 5px 15px rgba(244, 67, 54, 0.3);
}

.warning-btn {
  background: linear-gradient(135deg, #ff9800, #f57c00);
  color: white;
  box-shadow: 0 5px 15px rgba(255, 152, 0, 0.3);
}

.gold-btn {
  background: linear-gradient(135deg, #ffd700, #ffc107);
  color: #0a0f1e;
  box-shadow: 0 5px 15px rgba(255, 215, 0, 0.3);
}

.profit-meter {
  background: linear-gradient(145deg, #1a1f30, #0f1422);
  padding: 20px;
  border-radius: 20px;
  border: 1px solid rgba(255, 215, 0, 0.3);
  text-align: center;
}

.profit-label {
  color: #8a8f9c;
  font-size: 14px;
  margin-bottom: 5px;
}

.profit-value {
  color: #ffd700;
  font-size: 28px;
  font-weight: 800;
  text-shadow: 0 0 15px #ffd700;
}

.slider-container {
  width: 100%;
  text-align: center;
}

.slider-container label {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  color: #ffd700;
  font-weight: 600;
}

.slider-value {
  background: rgba(255, 215, 0, 0.2);
  padding: 5px 15px;
  border-radius: 50px;
  border: 1px solid #ffd700;
}

.casino-slider {
  width: 100%;
  height: 8px;
  border-radius: 10px;
  background: linear-gradient(90deg, #1a1f30, #ffd700);
  outline: none;
  -webkit-appearance: none;
}

.casino-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 25px;
  height: 25px;
  border-radius: 50%;
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  cursor: pointer;
  box-shadow: 0 0 20px #ffd700;
  border: 2px solid white;
}

.multiplier-box {
  background: rgba(0, 0, 0, 0.3);
  padding: 10px;
  border-radius: 50px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: 1px solid rgba(255, 215, 0, 0.3);
}

.multiplier-label {
  color: #8a8f9c;
  font-size: 14px;
}

.multiplier-value {
  color: #ffd700;
  font-size: 20px;
  font-weight: 700;
  text-shadow: 0 0 10px #ffd700;
}

/* ===== Chicken Road ===== */
.chicken-container {
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  margin: 20px 0;
}

.chicken {
  font-size: 90px;
  filter: drop-shadow(0 0 25px #ffd700);
  animation: float 3s infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-15px); }
}

.chicken.walking {
  animation: walkCycle 0.5s infinite;
}

@keyframes walkCycle {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-8px) rotate(-5deg); }
  75% { transform: translateX(8px) rotate(5deg); }
}

.road-container {
  background: linear-gradient(145deg, #1a1f30, #0f1422);
  border-radius: 30px;
  padding: 20px;
  margin-top: 20px;
}

.road {
  display: flex;
  gap: 8px;
  justify-content: center;
  flex-wrap: wrap;
}

.step {
  flex: 1;
  min-width: 45px;
  background: linear-gradient(145deg, #252b3d, #1a1f30);
  border-radius: 15px;
  padding: 15px 5px;
  position: relative;
  border: 1px solid rgba(255, 215, 0, 0.2);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.step.active {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  transform: scale(1.1);
  box-shadow: 0 10px 30px rgba(255, 215, 0, 0.5);
  z-index: 2;
}

.step.passed {
  border-color: #4caf50;
  opacity: 0.7;
}

.step.danger {
  border-color: #f44336;
}

.step.warning {
  border-color: #ff9800;
}

.step-multiplier {
  font-weight: 700;
  font-size: 14px;
  color: #ffd700;
}

.step.active .step-multiplier {
  color: #0a0f1e;
}

.chicken-icon {
  font-size: 24px;
  margin-top: 8px;
  animation: bounce 0.5s infinite;
}

/* ===== Dice 3D ===== */
.dice-container {
  perspective: 1000px;
  width: 120px;
  height: 120px;
  margin: 20px auto;
}

.dice-3d {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  animation: diceIdle 3s infinite;
}

@keyframes diceIdle {
  0%, 100% { transform: rotateX(0) rotateY(0); }
  25% { transform: rotateX(10deg) rotateY(10deg); }
  75% { transform: rotateX(-10deg) rotateY(-10deg); }
}

.rolling .dice-3d {
  animation: diceRoll 0.5s infinite linear !important;
}

@keyframes diceRoll {
  0% { transform: rotateX(0) rotateY(0); }
  100% { transform: rotateX(360deg) rotateY(360deg); }
}

.dice-face {
  position: absolute;
  width: 100%;
  height: 100%;
  background: linear-gradient(145deg, #ffd700, #ffed4a);
  border: 3px solid #0a0f1e;
  border-radius: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40px;
  font-weight: 800;
  color: #0a0f1e;
  backface-visibility: hidden;
}

.front { transform: translateZ(60px); }
.back { transform: rotateY(180deg) translateZ(60px); }
.right { transform: rotateY(90deg) translateZ(60px); }
.left { transform: rotateY(-90deg) translateZ(60px); }
.top { transform: rotateX(90deg) translateZ(60px); }
.bottom { transform: rotateX(-90deg) translateZ(60px); }

/* ===== Mines ===== */
.mines-container {
  width: 100%;
}

.mines-header {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-bottom: 20px;
  color: #ffd700;
  font-weight: 600;
}

.mines-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 8px;
}

.mine-cell {
  aspect-ratio: 1;
  background: linear-gradient(145deg, #252b3d, #1a1f30);
  border: 1px solid rgba(255, 215, 0, 0.3);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  color: #ffd700;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.3);
}

.mine-cell:hover:not(:disabled) {
  transform: scale(1.05);
  border-color: #ffd700;
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.3);
}

.mine-cell.revealed {
  background: #1a1f30;
}

.mine-cell.mine {
  background: linear-gradient(145deg, #f44336, #d32f2f);
  color: white;
  animation: explode 0.5s;
}

@keyframes explode {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.5); opacity: 0.5; }
}

.mine-cell.safe {
  background: linear-gradient(145deg, #4caf50, #45a049);
  color: white;
}

.mines-placeholder {
  height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #8a8f9c;
  font-size: 18px;
}

/* ===== Crash ===== */
.crash-container {
  text-align: center;
  width: 100%;
}

.multiplier-display {
  font-size: 70px;
  font-weight: 800;
  color: #ffd700;
  text-shadow: 0 0 30px #ffd700;
  margin-bottom: 20px;
  transition: all 0.3s;
}

.multiplier-display.crashed {
  color: #f44336;
  text-shadow: 0 0 30px #f44336;
  animation: crash 0.5s;
}

@keyframes crash {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-20px); }
  75% { transform: translateX(20px); }
}

.rocket-animation {
  height: 100px;
  position: relative;
  margin: 20px 0;
}

.rocket {
  font-size: 50px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  transition: all 0.3s;
  filter: drop-shadow(0 0 20px #ffd700);
}

.rocket.launched {
  animation: flyUp 1s infinite;
}

@keyframes flyUp {
  0% { transform: translateX(-50%) translateY(0); }
  50% { transform: translateX(-50%) translateY(-30px); }
  100% { transform: translateX(-50%) translateY(0); }
}

.rocket.exploded {
  transform: translateX(-50%) rotate(180deg) translateY(30px);
  opacity: 0.3;
  filter: drop-shadow(0 0 20px #f44336);
}

.smoke-effect {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 20px;
  height: 20px;
  background: radial-gradient(circle, rgba(255,255,255,0.8) 0%, transparent 70%);
  border-radius: 50%;
  animation: smoke 1s infinite;
}

@keyframes smoke {
  0% { transform: translateX(-50%) scale(0.5); opacity: 0.8; }
  100% { transform: translateX(-50%) scale(3); opacity: 0; }
}

.progress-track {
  width: 100%;
  height: 8px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #4caf50, #ffd700, #f44336);
  transition: width 0.2s;
}

/* ===== Limbo ===== */
.limbo-container {
  position: relative;
  height: 200px;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
}

.target-line {
  position: absolute;
  bottom: 0;
  width: 4px;
  background: linear-gradient(to top, #f44336, #ffd700);
  border-radius: 4px;
  transition: height 0.3s;
}

.target-label {
  position: absolute;
  top: -25px;
  left: -15px;
  background: rgba(0, 0, 0, 0.7);
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  color: #ffd700;
  border: 1px solid #ffd700;
}

.result-ball {
  width: 60px;
  height: 60px;
  background: radial-gradient(circle at 30% 30%, #ffd700, #b8860b);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  color: #0a0f1e;
  box-shadow: 0 0 30px #ffd700;
  position: relative;
  z-index: 2;
}

.result-ball.jumping {
  animation: jumpBall 1s infinite;
}

@keyframes jumpBall {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-50px); }
}

/* ===== Blackjack ===== */
.blackjack-table {
  background: linear-gradient(145deg, #0a5c0a, #0a4a0a);
  padding: 25px;
  border-radius: 30px;
  border: 2px solid #ffd700;
  box-shadow: inset 0 0 30px rgba(0, 0, 0, 0.5);
}

.dealer-area, .player-area {
  margin: 20px 0;
}

.area-label {
  color: #ffd700;
  font-size: 16px;
  margin-bottom: 10px;
  font-weight: 600;
}

.cards-row {
  display: flex;
  gap: 10px;
  justify-content: center;
  flex-wrap: wrap;
}

.casino-card-small {
  width: 60px;
  height: 90px;
  background: linear-gradient(145deg, #ffffff, #f0f0f0);
  border: 2px solid #ffd700;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  color: #0a0f1e;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  animation: dealCard 0.3s;
}

@keyframes dealCard {
  0% { transform: translateY(-50px) rotate(180deg); opacity: 0; }
  100% { transform: translateY(0) rotate(0); opacity: 1; }
}

.casino-card-small.card-back {
  background: linear-gradient(135deg, #ffd700, #b8860b);
  color: transparent;
  position: relative;
}

.casino-card-small.card-back::after {
  content: '?';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 24px;
}

.score-badge {
  background: #ffd700;
  color: #0a0f1e;
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 14px;
  margin-right: 10px;
}

/* ===== Slot Machine ===== */
.slot-machine-container {
  background: linear-gradient(145deg, #1a1f30, #0f1422);
  padding: 25px;
  border-radius: 30px;
  border: 3px solid #ffd700;
}

.slot-reels {
  display: flex;
  gap: 15px;
  justify-content: center;
  perspective: 500px;
}

.reel {
  width: 80px;
  height: 80px;
  background: linear-gradient(145deg, #ffffff, #f0f0f0);
  border: 3px solid #ffd700;
  border-radius: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.5);
}

.spinning .reel {
  animation: spinReel 0.1s infinite;
}

@keyframes spinReel {
  0% { transform: translateY(0); }
  100% { transform: translateY(30px); }
}

/* ===== Coinflip ===== */
.coin-container {
  perspective: 1000px;
  width: 120px;
  height: 120px;
  margin: 20px auto;
}

.coin-3d {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  animation: coinIdle 3s infinite;
}

@keyframes coinIdle {
  0%, 100% { transform: rotateY(0); }
  50% { transform: rotateY(30deg); }
}

.coin-3d.flipping {
  animation: flipCoin 0.2s linear infinite !important;
}

@keyframes flipCoin {
  0% { transform: rotateY(0); }
  100% { transform: rotateY(360deg); }
}

.coin-face {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: radial-gradient(circle at 30% 30%, #ffd700, #b8860b);
  border: 3px solid #0a0f1e;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 50px;
  backface-visibility: hidden;
  box-shadow: 0 0 30px #ffd700;
}

.coin-face.front {
  transform: translateZ(10px);
}

.coin-face.back {
  transform: rotateY(180deg) translateZ(10px);
}

.choice-row {
  display: flex;
  gap: 10px;
  width: 100%;
  max-width: 280px;
}

.choice-btn-small {
  flex: 1;
  padding: 12px;
  border-radius: 50px;
  background: linear-gradient(145deg, #1e2333, #131826);
  color: white;
  border: 1px solid rgba(255, 215, 0, 0.3);
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
}

.choice-btn-small.active {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  border: none;
  box-shadow: 0 0 20px #ffd700;
}

/* ===== Wheel ===== */
.wheel-container {
  position: relative;
  height: 250px;
  display: flex;
  justify-content: center;
  margin: 20px 0;
}

.wheel-outer {
  width: 220px;
  height: 220px;
  border-radius: 50%;
  background: linear-gradient(145deg, #1e2333, #131826);
  border: 4px solid #ffd700;
  position: relative;
  transition: transform 3s cubic-bezier(0.25, 0.1, 0.15, 1);
  box-shadow: 0 0 40px rgba(255, 215, 0, 0.3);
}

.wheel-segment-casino {
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
}

.segment-value-casino {
  transform: rotate(45deg);
  font-size: 16px;
  font-weight: 700;
  color: #ffd700;
}

.wheel-pointer {
  position: absolute;
  top: -10px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 30px;
  color: #ffd700;
  filter: drop-shadow(0 0 10px #ffd700);
}

/* ===== Keno ===== */
.keno-container {
  width: 100%;
  overflow-x: auto;
}

.keno-grid {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 5px;
  min-width: 280px;
}

.keno-ball {
  aspect-ratio: 1;
  background: linear-gradient(145deg, #1e2333, #131826);
  border: 1px solid rgba(255, 215, 0, 0.3);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
}

.keno-ball:hover {
  transform: scale(1.1);
  border-color: #ffd700;
}

.keno-ball.selected {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  border-color: #ffd700;
  box-shadow: 0 0 20px #ffd700;
}

.keno-ball.drawn {
  background: linear-gradient(145deg, #4caf50, #45a049);
  color: white;
  border-color: #4caf50;
}

/* ===== Bowling ===== */
.bowling-container {
  width: 100%;
}

.pins-setup {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 10px;
  margin-bottom: 30px;
}

.bowling-pin {
  font-size: 30px;
  text-align: center;
  transition: all 0.5s;
  filter: drop-shadow(0 0 10px #ffd700);
}

.bowling-pin.knocked {
  transform: rotate(90deg) scale(0.5);
  opacity: 0;
  filter: blur(2px);
}

.ball-track {
  position: relative;
  height: 50px;
}

.bowling-ball-casino {
  font-size: 40px;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  transition: all 0.3s;
}

.bowling-ball-casino.rolling {
  animation: rollToPins 1s ease;
}

@keyframes rollToPins {
  0% { transform: translateX(-50%) translateY(0); }
  100% { transform: translateX(100px) translateY(-30px); }
}

/* ===== Puzzle ===== */
.puzzle-container {
  width: 100%;
}

.puzzle-grid-casino {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 5px;
  background: linear-gradient(145deg, #1e2333, #131826);
  padding: 15px;
  border-radius: 20px;
}

.puzzle-tile {
  aspect-ratio: 1;
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  font-weight: 700;
  color: #0a0f1e;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.3);
}

.puzzle-tile.empty {
  background: linear-gradient(145deg, #1e2333, #131826);
  border: 2px dashed #ffd700;
}

.puzzle-tile:hover:not(.empty) {
  transform: scale(1.05);
  box-shadow: 0 8px 20px rgba(255, 215, 0, 0.4);
}

/* ===== Target ===== */
.target-container-casino {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.target-casino {
  position: relative;
  width: 200px;
  height: 200px;
  margin: 20px auto;
}

.target-ring {
  position: absolute;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s;
  border: 2px solid #ffd700;
}

.target-ring:hover {
  transform: scale(1.1);
  box-shadow: 0 0 30px rgba(255, 215, 0, 0.5);
}

.ring-value {
  font-size: 18px;
  font-weight: 700;
  color: white;
  text-shadow: 0 0 10px currentColor;
}

.target-ring.bullseye {
  width: 60px;
  height: 60px;
  background: radial-gradient(circle, #f44336, #d32f2f);
  top: 70px;
  left: 70px;
}

.target-ring.middle {
  width: 100px;
  height: 100px;
  background: radial-gradient(circle, #4caf50, #45a049);
  top: 50px;
  left: 50px;
}

.target-ring.outer {
  width: 140px;
  height: 140px;
  background: radial-gradient(circle, #1e2333, #131826);
  top: 30px;
  left: 30px;
}

.target-score {
  margin-top: 20px;
  padding: 10px 25px;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 50px;
  border: 1px solid #ffd700;
  color: #ffd700;
  font-weight: 600;
}

/* ===== Lucky Number ===== */
.lucky-container {
  text-align: center;
}

.number-drum {
  width: 120px;
  height: 120px;
  margin: 20px auto;
  background: linear-gradient(145deg, #1e2333, #131826);
  border: 3px solid #ffd700;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40px;
  font-weight: 800;
  color: #ffd700;
  box-shadow: 0 0 40px rgba(255, 215, 0, 0.3);
}

.number-drum.spinning {
  animation: spinDrum 0.1s linear infinite;
}

@keyframes spinDrum {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.chosen-number {
  margin-top: 15px;
  color: #8a8f9c;
  font-size: 18px;
}

/* ===== Mystery Box ===== */
.mystery-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 200px;
}

.mystery-box {
  width: 150px;
  height: 150px;
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 60px;
  cursor: pointer;
  transition: all 0.5s;
  box-shadow: 0 15px 40px rgba(255, 215, 0, 0.4);
  border: 3px solid rgba(255, 255, 255, 0.3);
}

.mystery-box.shaking {
  animation: shake 0.5s infinite;
}

@keyframes shake {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(10deg); }
  75% { transform: rotate(-10deg); }
}

.mystery-box.opened {
  background: linear-gradient(145deg, #1e2333, #131826);
  border: 2px solid #ffd700;
  font-size: 24px;
  color: #ffd700;
  animation: openBox 0.5s;
}

@keyframes openBox {
  0% { transform: scale(1) rotate(0); }
  50% { transform: scale(1.2) rotate(180deg); }
  100% { transform: scale(1) rotate(360deg); }
}

.box-content {
  font-size: 20px;
  font-weight: 700;
}

/* ===== High/Low Trading Game ===== */
.trading-container {
  width: 100%;
  position: relative;
}

.trading-chart {
  width: 100%;
  height: 200px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 15px;
  border: 1px solid rgba(255, 215, 0, 0.2);
}

.arrow-container {
  position: relative;
  height: 40px;
  margin: 20px 0 10px;
}

.arrow-line {
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, #4caf50, #ffd700, #f44336);
  transform: translateY(-50%);
}

.arrow-head {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  color: #ffd700;
  font-size: 24px;
  text-shadow: 0 0 10px #ffd700;
  transition: left 0.1s linear;
}

.arrow-moving .arrow-head {
  animation: arrowPulse 0.5s infinite;
}

@keyframes arrowPulse {
  0%, 100% { transform: translate(-50%, -50%) scale(1); }
  50% { transform: translate(-50%, -50%) scale(1.2); }
}

.price-markers {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
  font-size: 12px;
  color: #8a8f9c;
}

.start-marker, .end-marker {
  background: rgba(0, 0, 0, 0.5);
  padding: 4px 8px;
  border-radius: 4px;
  border: 1px solid rgba(255, 215, 0, 0.3);
}

.end-marker {
  color: #ffd700;
}

.trading-timer {
  position: relative;
  height: 30px;
  margin-top: 15px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 15px;
  overflow: hidden;
  border: 1px solid rgba(255, 215, 0, 0.3);
}

.timer-progress {
  height: 100%;
  background: linear-gradient(90deg, #4caf50, #ffd700);
  transition: width 1s linear;
}

.trading-timer span {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-weight: 700;
  text-shadow: 0 0 5px black;
}

.trading-buttons {
  display: flex;
  gap: 15px;
  width: 100%;
  max-width: 280px;
}

.trading-btn {
  flex: 1;
  padding: 15px;
  border: none;
  border-radius: 15px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
  background: linear-gradient(145deg, #1e2333, #131826);
  border: 1px solid rgba(255, 215, 0, 0.3);
  color: white;
}

.trading-btn:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(255, 215, 0, 0.3);
}

.trading-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.high-btn {
  border-color: #4caf50;
}

.high-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #4caf50, #45a049);
  box-shadow: 0 8px 20px rgba(76, 175, 80, 0.3);
}

.low-btn {
  border-color: #f44336;
}

.low-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #f44336, #d32f2f);
  box-shadow: 0 8px 20px rgba(244, 67, 54, 0.3);
}

.btn-icon {
  font-size: 24px;
}

.btn-text {
  font-size: 16px;
  font-weight: 700;
}

.btn-multiplier {
  font-size: 14px;
  color: #ffd700;
}

.trade-result {
  text-align: center;
  padding: 20px;
  border-radius: 20px;
  animation: slideUp 0.5s ease;
}

.win-result {
  background: linear-gradient(145deg, #1a2f1a, #0f1f0f);
  border: 1px solid #4caf50;
}

.lose-result {
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
  .game-page {
    padding: 10px;
  }

  .tabs button {
    padding: 10px 15px;
    font-size: 12px;
  }

  .casino-card {
    padding: 20px;
  }

  .casino-card-header h2 {
    font-size: 22px;
  }

  .trading-chart {
    height: 150px;
  }

  .trading-buttons {
    flex-direction: column;
    gap: 10px;
  }

  .result-message {
    font-size: 14px;
    padding: 12px 20px;
    min-width: 240px;
  }

  .action-row {
    flex-direction: column;
  }

  .action-btn {
    width: 100%;
  }

  .arrow-head {
    font-size: 20px;
  }
}
</style>
