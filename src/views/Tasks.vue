<template>    
  <div class="game-page">    
    <!-- الرصيد -->
    <div class="top-bar">
      <div class="balance-gold">
        <i class="fas fa-coins"></i>
        <span>رصيدك: <strong>{{ balance.toFixed(2) }} USDT</strong></span>
      </div>
    </div>

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
      <div v-if="selectedGame==='chicken'" class="card chicken-card fullscreen-card">
        <div class="card-header">
          <h2>🐔 Chicken Road</h2>
          <div class="header-glow"></div>
        </div>

        <div class="chicken-container">
          <div class="chicken-glow"></div>
          <div class="chicken" :class="{ walking: chickenStarted }">🐔</div>
        </div>

        <div v-if="chickenStarted" class="road">
          <div
            v-for="(step, i) in chickenSteps"
            :key="i"
            class="step"
            :class="{
              active: i === chickenPosition,
              passed: i < chickenPosition,
              'gold-step': step.multiplier >= 3,
              'silver-step': step.multiplier >= 2 && step.multiplier < 3
            }"
          >
            <span class="multiplier">x{{ step.multiplier.toFixed(2) }}</span>
            <div v-if="i === chickenPosition" class="chicken-icon">🐔</div>
            <div v-if="i < chickenPosition" class="step-check">✓</div>
          </div>
        </div>

        <div class="bet-section">
          <div v-if="!chickenStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="chickenBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">
              <i class="fas fa-exclamation-circle"></i>
              {{ gameError }}
            </div>
            <button 
              @click="startChicken"
              class="gold-button pulse"
              :disabled="!chickenBet || chickenBet <= 0"
            >
              <i class="fas fa-play"></i>
              ابدأ الآن
            </button>
          </div>

          <div v-if="chickenStarted" class="game-controls">
            <div class="profit-display">
              <span class="profit-label">الربح المحتمل:</span>
              <span class="profit-value">{{ chickenProfit.toFixed(2) }} USDT</span>
            </div>
            <div class="action-buttons">
              <button @click="chickenNext" class="action-btn gold" :disabled="chickenGameOver">
                <i class="fas fa-arrow-right"></i>
                تقدم
              </button>
              <button @click="chickenCashOut" class="action-btn green" :disabled="chickenGameOver">
                <i class="fas fa-hand-holding-usd"></i>
                سحب
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Dice -->
      <div v-if="selectedGame==='dice'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🎲 Dice</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon" :class="{ rolling: diceRolling }">🎲</div>
        
        <div class="game-controls-full">
          <div class="slider-container">
            <label>نسبة الفوز: <span class="highlight">{{ diceChance }}%</span></label>
            <input 
              type="range" 
              v-model.number="diceChance" 
              min="1" 
              max="30" 
              step="1"
              class="gold-slider"
              :disabled="diceStarted"
            >
          </div>
          
          <div class="multiplier-display">
            <span class="glow-text">x{{ (95 / diceChance).toFixed(2) }}</span>
          </div>
          
          <div v-if="!diceStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="diceBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startDice" class="gold-button pulse" :disabled="!diceBet || diceBet <= 0">
              <i class="fas fa-dice"></i> رمي النرد
            </button>
          </div>
          
          <div v-if="diceStarted" class="result-box">
            <div class="dice-result" :class="{ 'win-effect': diceWon, 'lose-effect': !diceWon }">
              {{ diceRoll }}
            </div>
            <div class="result-text" :class="diceWon ? 'win-text' : 'lose-text'">
              {{ diceResultText }}
            </div>
          </div>
        </div>
      </div>

      <!-- Mines -->
      <div v-if="selectedGame==='mines'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>💣 Mines</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon">💣</div>
        
        <div class="game-controls-full">
          <div class="slider-container">
            <label>عدد الألغام: <span class="highlight">{{ minesCount }}</span></label>
            <input 
              type="range" 
              v-model.number="minesCount" 
              min="5" 
              max="15" 
              step="1"
              class="gold-slider"
              :disabled="minesStarted"
            >
          </div>
          
          <div v-if="!minesStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="minesBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startMines" class="gold-button pulse" :disabled="!minesBet || minesBet <= 0">
              <i class="fas fa-bomb"></i> ابدأ
            </button>
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
                'explode': cell.mine && cell.revealed && minesGameOver
              }"
              @click="revealMine(index)"
              :disabled="cell.revealed || minesGameOver"
            >
              <span v-if="!cell.revealed">?</span>
              <span v-else-if="cell.mine">💣</span>
              <span v-else>💎</span>
            </button>
          </div>
          
          <div v-if="minesStarted && !minesGameOver" class="action-buttons">
            <button @click="minesCashOut" class="action-btn green pulse">
              <i class="fas fa-hand-holding-usd"></i> سحب ({{ minesProfit.toFixed(2) }} USDT)
            </button>
          </div>
        </div>
      </div>

      <!-- Crash -->
      <div v-if="selectedGame==='crash'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🚀 Crash</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon" :class="{ flying: crashStarted && !crashCrashed, exploding: crashCrashed }">🚀</div>
        
        <div class="game-controls-full">
          <div v-if="!crashStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="crashBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="crashAutoCashout"
                placeholder="السحب عند"
                class="gold-input"
                step="0.1"
                min="1.2"
                max="10"
              />
              <span class="input-currency">x</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startCrash" class="gold-button pulse" :disabled="!crashBet || crashBet <= 0">
              <i class="fas fa-rocket"></i> إطلاق
            </button>
          </div>
          
          <div v-if="crashStarted" class="crash-display">
            <div class="multiplier-box" :class="{ 'crashed': crashCrashed, 'pulse-text': !crashCrashed }">
              x{{ crashMultiplier.toFixed(2) }}
            </div>
            <div class="rocket-container">
              <div class="rocket" :class="{ 'launched': crashStarted, 'crashed': crashCrashed }">🚀</div>
              <div class="smoke" v-if="crashStarted && !crashCrashed"></div>
            </div>
            <progress class="crash-progress" :value="crashProgress" max="100"></progress>
          </div>
          
          <div v-if="crashStarted && !crashCrashed" class="action-buttons">
            <button @click="crashCashOut" class="action-btn green pulse">
              <i class="fas fa-hand-holding-usd"></i> سحب ({{ (crashBet * crashMultiplier).toFixed(2) }} USDT)
            </button>
          </div>
        </div>
      </div>

      <!-- Limbo -->
      <div v-if="selectedGame==='limbo'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🎯 Limbo</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon" :class="{ jumping: limboStarted }">🎯</div>
        
        <div class="game-controls-full">
          <div v-if="!limboStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="limboBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="limboTarget"
                placeholder="المضاعف المستهدف"
                class="gold-input"
                step="0.1"
                min="1.2"
                max="10"
              />
              <span class="input-currency">x</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startLimbo" class="gold-button pulse" :disabled="!limboBet || limboBet <= 0">
              <i class="fas fa-crosshairs"></i> ابدأ
            </button>
          </div>
          
          <div v-if="limboStarted" class="limbo-display">
            <div class="result-number" :class="{ 'win-effect': limboWon, 'lose-effect': !limboWon }">
              {{ limboResult.toFixed(2) }}x
            </div>
            <div class="result-text" :class="limboWon ? 'win-text' : 'lose-text'">
              {{ limboWon ? '🎉 فوز! مضاعف عالي' : '💥 خسارة' }}
            </div>
          </div>
        </div>
      </div>

      <!-- Blackjack -->
      <div v-if="selectedGame==='blackjack'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🃏 Blackjack</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon">🃏</div>
        
        <div class="game-controls-full">
          <div v-if="!blackjackStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="blackjackBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startBlackjack" class="gold-button pulse" :disabled="!blackjackBet || blackjackBet <= 0">
              <i class="fas fa-credit-card"></i> توزيع
            </button>
          </div>
          
          <div v-if="blackjackStarted" class="blackjack-table">
            <div class="dealer-hand">
              <span class="hand-label">الموزع:</span>
              <div class="cards">
                <span v-for="(card, i) in dealerHand" :key="i" class="card" :class="{ 'card-back': i === 1 && !dealerRevealed }">
                  {{ i === 1 && !dealerRevealed ? '?' : card }}
                </span>
              </div>
            </div>
            <div class="player-hand">
              <span class="hand-label">أنت:</span>
              <div class="cards">
                <span v-for="card in playerHand" :key="card" class="card">{{ card }}</span>
              </div>
              <span class="score">({{ playerScore }})</span>
            </div>
            
            <div v-if="!blackjackGameOver" class="action-buttons">
              <button @click="blackjackHit" class="action-btn gold">ضرب 🃏</button>
              <button @click="blackjackStand" class="action-btn green">وقف ✋</button>
            </div>
            
            <div v-if="blackjackGameOver" class="result-text" :class="blackjackWon ? 'win-text' : 'lose-text'">
              {{ blackjackResult }}
            </div>
          </div>
        </div>
      </div>

      <!-- Slot Machine -->
      <div v-if="selectedGame==='slot'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🎰 Slot Machine</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon">🎰</div>
        
        <div class="game-controls-full">
          <div v-if="!slotStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="slotBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startSlot" class="gold-button pulse" :disabled="!slotBet || slotBet <= 0">
              <i class="fas fa-sliders-h"></i> لعب
            </button>
          </div>
          
          <div v-if="slotStarted" class="slot-machine">
            <div class="slot-row" :class="{ spinning: slotSpinning }">
              <div class="slot-reel">{{ slotReels[0] }}</div>
              <div class="slot-reel">{{ slotReels[1] }}</div>
              <div class="slot-reel">{{ slotReels[2] }}</div>
            </div>
            <div class="result-text" :class="slotWon ? 'win-text' : 'lose-text'">
              {{ slotResult }}
            </div>
          </div>
        </div>
      </div>

      <!-- Coinflip -->
      <div v-if="selectedGame==='coinflip'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🪙 Coinflip</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon" :class="{ flipping: coinflipFlipping }">🪙</div>
        
        <div class="game-controls-full">
          <div v-if="!coinflipStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="coinflipBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div class="choice-buttons">
              <button 
                @click="coinflipChoice = 'صورة'" 
                class="choice-btn" 
                :class="{active: coinflipChoice === 'صورة'}"
              >
                🪙 صورة
              </button>
              <button 
                @click="coinflipChoice = 'كتابة'" 
                class="choice-btn" 
                :class="{active: coinflipChoice === 'كتابة'}"
              >
                📝 كتابة
              </button>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startCoinflip" class="gold-button pulse" :disabled="!coinflipBet || coinflipBet <= 0 || !coinflipChoice">
              <i class="fas fa-coins"></i> اقلب
            </button>
          </div>
          
          <div v-if="coinflipStarted" class="coinflip-result">
            <div class="coin" :class="{ 'flipping': coinflipFlipping, 'show-result': !coinflipFlipping }">
              {{ coinflipFlipping ? '🪙' : coinflipResult }}
            </div>
            <div class="result-text" :class="coinflipWon ? 'win-text' : 'lose-text'">
              {{ coinflipWon ? '🎉 فوز!' : '💥 خسارة' }}
            </div>
          </div>
        </div>
      </div>

      <!-- Wheel Spin -->
      <div v-if="selectedGame==='wheel'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🎯 Wheel Spin</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon">🎡</div>
        
        <div class="game-controls-full">
          <div v-if="!wheelStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="wheelBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startWheel" class="gold-button pulse" :disabled="!wheelBet || wheelBet <= 0">
              <i class="fas fa-circle-notch"></i> دوّر
            </button>
          </div>
          
          <div v-if="wheelStarted" class="wheel-container">
            <div class="wheel-spinner" :style="{ transform: `rotate(${wheelRotation}deg)` }">
              <div v-for="(mult, index) in wheelMultipliers" :key="index" 
                   class="wheel-segment" 
                   :style="{ transform: `rotate(${index * 45}deg)` }">
                <span class="segment-value">{{ mult }}x</span>
              </div>
            </div>
            <div class="wheel-pointer">▼</div>
            <div class="result-text">{{ wheelResult }}</div>
          </div>
        </div>
      </div>

      <!-- Keno -->
      <div v-if="selectedGame==='keno'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🧨 Keno</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon">🎱</div>
        
        <div class="game-controls-full">
          <div v-if="!kenoStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="kenoBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startKeno" class="gold-button pulse" :disabled="!kenoBet || kenoBet <= 0">
              <i class="fas fa-dice-d6"></i> ابدأ
            </button>
          </div>
          
          <div v-if="kenoStarted" class="keno-grid">
            <div 
              v-for="n in 40" 
              :key="n"
              class="keno-cell"
              :class="{
                'selected': kenoSelected.includes(n),
                'drawn': kenoDrawn.includes(n),
                'match': kenoSelected.includes(n) && kenoDrawn.includes(n)
              }"
              @click="toggleKenoNumber(n)"
            >
              {{ n }}
              <div v-if="kenoDrawn.includes(n)" class="draw-effect"></div>
            </div>
          </div>
          
          <div v-if="kenoStarted && kenoSelected.length > 0 && kenoDrawn.length === 0" class="action-buttons">
            <button @click="drawKeno" class="action-btn gold pulse">
              <i class="fas fa-dice"></i> سحب ({{ kenoSelected.length }} أرقام)
            </button>
          </div>
          
          <div v-if="kenoDrawn.length > 0" class="keno-results">
            <div class="matches">التطابقات: {{ kenoMatches }}</div>
            <div class="multiplier">المضاعف: x{{ kenoMultiplier }}</div>
          </div>
        </div>
      </div>

      <!-- Bowling -->
      <div v-if="selectedGame==='bowling'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🎳 Bowling</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon">🎳</div>
        
        <div class="game-controls-full">
          <div v-if="!bowlingStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="bowlingBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startBowling" class="gold-button pulse" :disabled="!bowlingBet || bowlingBet <= 0">
              <i class="fas fa-bowling-ball"></i> ارمِ
            </button>
          </div>
          
          <div v-if="bowlingStarted" class="bowling-alley">
            <div class="pins-row">
              <div v-for="pin in 10" :key="pin" 
                   class="pin" 
                   :class="{ knocked: bowlingKnocked.includes(pin) }">
                <span class="pin-emoji">🎳</span>
              </div>
            </div>
            <div class="bowling-ball-container">
              <div class="bowling-ball" :class="{ rolling: bowlingRolling }">⚫</div>
            </div>
            <div class="result-text">{{ bowlingResult }}</div>
          </div>
        </div>
      </div>

      <!-- Puzzle -->
      <div v-if="selectedGame==='puzzle'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🧩 Puzzle</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon">🧩</div>
        
        <div class="game-controls-full">
          <div v-if="!puzzleStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="puzzleBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startPuzzle" class="gold-button pulse" :disabled="!puzzleBet || puzzleBet <= 0">
              <i class="fas fa-puzzle-piece"></i> ابدأ
            </button>
          </div>
          
          <div v-if="puzzleStarted" class="puzzle-grid">
            <div 
              v-for="(piece, index) in puzzlePieces" 
              :key="index"
              class="puzzle-piece"
              :class="{ 
                empty: piece === null,
                correct: piece === index + 1,
                'win-glow': puzzleSolved
              }"
              @click="movePuzzle(index)"
            >
              {{ piece !== null ? piece : '' }}
            </div>
          </div>
          
          <div v-if="puzzleStarted" class="result-text" :class="puzzleSolved ? 'win-text' : ''">
            {{ puzzleSolved ? '🎉 أحسنت! حل اللغز وربحت!' : 'حرك القطع لترتيب الأرقام من 1 إلى 8' }}
          </div>
        </div>
      </div>

      <!-- Target Shot -->
      <div v-if="selectedGame==='target'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🎯 Target Shot</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon">🎯</div>
        
        <div class="game-controls-full">
          <div v-if="!targetStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="targetBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startTarget" class="gold-button pulse" :disabled="!targetBet || targetBet <= 0">
              <i class="fas fa-bullseye"></i> صوّب
            </button>
          </div>
          
          <div v-if="targetStarted" class="target-container">
            <div class="target">
              <div class="target-circle bullseye" @click="shootTarget(10)">
                <span class="target-value">10</span>
              </div>
              <div class="target-circle middle" @click="shootTarget(5)">
                <span class="target-value">5</span>
              </div>
              <div class="target-circle outer" @click="shootTarget(2)">
                <span class="target-value">2</span>
              </div>
            </div>
            <div class="score-display">النقاط: {{ targetScore }} / 30</div>
            <div class="result-text">{{ targetResult }}</div>
          </div>
        </div>
      </div>

      <!-- Lucky Number -->
      <div v-if="selectedGame==='lucky'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🎮 Lucky Number</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon" :class="{ spinning: luckyStarted }">🎲</div>
        
        <div class="game-controls-full">
          <div v-if="!luckyStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="luckyBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="luckyNumber"
                placeholder="رقم الحظ"
                class="gold-input"
                min="1"
                max="100"
              />
              <span class="input-currency">#</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startLucky" class="gold-button pulse" :disabled="!luckyBet || luckyBet <= 0 || !luckyNumber">
              <i class="fas fa-star"></i> جرب حظك
            </button>
          </div>
          
          <div v-if="luckyStarted" class="lucky-display">
            <div class="drawn-number" :class="{ 'match-effect': luckyWon }">{{ luckyDrawn }}</div>
            <div class="your-number">رقمك: {{ luckyNumber }}</div>
            <div class="result-text" :class="luckyWon ? 'win-text' : 'lose-text'">
              {{ luckyWon ? '🎉 رقم محظوظ! فوز كبير!' : '😢 حظ أوفر المرة الجاية' }}
            </div>
          </div>
        </div>
      </div>

      <!-- Mystery Box -->
      <div v-if="selectedGame==='mystery'" class="card game-card fullscreen-card">
        <div class="card-header">
          <h2>🎁 Mystery Box</h2>
          <div class="header-glow"></div>
        </div>
        
        <div class="game-icon">📦</div>
        
        <div class="game-controls-full">
          <div v-if="!mysteryStarted" class="bet-controls">
            <div class="input-gold-wrapper">
              <input
                type="number"
                v-model.number="mysteryBet"
                placeholder="أدخل المبلغ"
                class="gold-input"
                @input="clearGameError"
              />
              <span class="input-currency">USDT</span>
            </div>
            <div v-if="gameError" class="error-message">{{ gameError }}</div>
            <button @click="startMystery" class="gold-button pulse" :disabled="!mysteryBet || mysteryBet <= 0">
              <i class="fas fa-gift"></i> افتح الصندوق
            </button>
          </div>
          
          <div v-if="mysteryStarted" class="mystery-box-container">
            <div class="box" @click="openMysteryBox" :class="{ opened: mysteryOpened, shaking: !mysteryOpened }">
              <span v-if="!mysteryOpened">🎁</span>
              <span v-else class="prize">{{ mysteryPrize }}</span>
            </div>
            <div class="result-text">{{ mysteryResult }}</div>
          </div>
        </div>
      </div>
    </div>

    <!-- نتيجة اللعبة -->
    <div v-if="result" class="result-popup" :class="result.includes('ربحت') || result.includes('فوز') ? 'win' : 'lose'">
      <i :class="result.includes('ربحت') || result.includes('فوز') ? 'fas fa-trophy' : 'fas fa-skull'"></i>
      <span>{{ result }}</span>
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
        { id: 'mystery', name: 'Mystery Box', icon: '🎁' }
      ],
      balance: 0,    
      result: "",    
      gameError: "",    
    
      /* Chicken Road */
      chickenBet: null,
      chickenStarted: false,
      chickenGameOver: false,
      chickenPosition: 0,
      chickenSteps: [
        { multiplier: 1.0 },
        { multiplier: 1.2 },
        { multiplier: 1.5 },
        { multiplier: 2.0 },
        { multiplier: 2.5 },
        { multiplier: 3.0 },
        { multiplier: 4.0 },
        { multiplier: 5.0 },
        { multiplier: 6.0 },
        { multiplier: 8.0 },
        { multiplier: 10.0 }
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
      minesCount: 8,
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
      wheelMultipliers: [20, 5, 2, 10, 1.5, 3, 50, 1.2],
      
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
      mysteryResult: ""
    };    
  },    
    
  computed: {    
    chickenProfit() {    
      if (!this.chickenBet) return 0;    
      return this.chickenBet * this.chickenSteps[this.chickenPosition].multiplier;    
    },
    
    minesProfit() {
      if (!this.minesBet) return 0;
      let multiplier = 1 + (this.minesRevealed * 0.3);
      return this.minesBet * Math.min(multiplier, 5);
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
    
  methods: {    
    openGame(gameId) {
      this.selectedGame = gameId;
      this.gameOpened = true;
      this.result = "";
      this.gameError = "";
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
      this.result = "";
    },
    
    chickenNext() {
      if (this.chickenGameOver) return;
      
      const loseChance = 0.5 + (this.chickenPosition * 0.05);
      if (Math.random() < loseChance) {
        this.chickenGameOver = true;
        this.result = "💥 للأسف خسرت الرهان";
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
      this.result = `🎉 مبروك! ربحت ${profit.toFixed(2)} USDT`;
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
      
      // تأثير الدوران
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
        this.diceResultText = `🎉 فوز! ربحت ${profit.toFixed(2)} USDT`;
        this.result = this.diceResultText;
      } else {
        this.diceResultText = `😢 خسارة! الرقم كان ${this.diceRoll}`;
        this.result = this.diceResultText;
      }
      
      setTimeout(() => {
        this.diceStarted = false;
      }, 3000);
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
      
      // إنشاء حقل الألغام
      this.minesCells = [];
      for (let i = 0; i < 25; i++) {
        this.minesCells.push({ revealed: false, mine: false });
      }
      
      // وضع الألغام
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
        this.result = "💥 انفجر لغم! خسرت الرهان";
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
      this.result = `🎉 ربحت ${profit.toFixed(2)} USDT`;
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
      
      const crashPoint = 1.2 + (Math.random() * 8); // نقطة الانفجار بين 1.2 و 9.2
      
      this.crashInterval = setInterval(() => {
        if (this.crashCrashed) return;
        
        this.crashMultiplier += 0.03;
        this.crashProgress = ((this.crashMultiplier - 1) / 9) * 100;
        
        if (this.crashMultiplier >= crashPoint) {
          this.crashCrashed = true;
          clearInterval(this.crashInterval);
          this.result = "💥 انفجر الصاروخ!";
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
      this.result = `🎉 سحبت قبل الانفجار! ربحت ${profit.toFixed(2)} USDT`;
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
      
      // تأثير الانتظار
      setTimeout(() => {
        this.limboResult = 1.2 + (Math.random() * 9);
        this.limboWon = this.limboResult >= this.limboTarget;
        
        if (this.limboWon) {
          const profit = this.limboBet * this.limboResult;
          this.balance += profit;
          this.updateBalance(this.balance);
          this.result = `🎉 فوز! المضاعف ${this.limboResult.toFixed(2)}x ربحت ${profit.toFixed(2)} USDT`;
        } else {
          this.result = `😢 خسارة! المضاعف ${this.limboResult.toFixed(2)}x`;
        }
        
        setTimeout(() => {
          this.limboStarted = false;
        }, 3000);
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
      
      // تأثير انتظار
      setTimeout(async () => {
        while (this.calculateHand(this.dealerHand) < 17) {
          this.dealerHand.push(this.deck.pop());
        }
        
        const playerScore = this.calculateHand(this.playerHand);
        const dealerScore = this.calculateHand(this.dealerHand);
        
        this.blackjackGameOver = true;
        
        if (playerScore > 21) {
          this.blackjackWon = false;
          this.blackjackResult = "😢 خسرت! نقاطك تجاوزت 21";
          this.result = this.blackjackResult;
        } else if (dealerScore > 21) {
          this.blackjackWon = true;
          const profit = this.blackjackBet * 2;
          this.balance += profit;
          await this.updateBalance(this.balance);
          this.blackjackResult = `🎉 فوز! الموزع خسر ربحت ${profit.toFixed(2)} USDT`;
          this.result = this.blackjackResult;
        } else if (playerScore > dealerScore) {
          this.blackjackWon = true;
          const profit = this.blackjackBet * 2;
          this.balance += profit;
          await this.updateBalance(this.balance);
          this.blackjackResult = `🎉 فوز! ${playerScore} ضد ${dealerScore} ربحت ${profit.toFixed(2)} USDT`;
          this.result = this.blackjackResult;
        } else if (playerScore < dealerScore) {
          this.blackjackWon = false;
          this.blackjackResult = `😢 خسارة! ${playerScore} ضد ${dealerScore}`;
          this.result = this.blackjackResult;
        } else {
          this.blackjackWon = true;
          this.balance += this.blackjackBet;
          await this.updateBalance(this.balance);
          this.blackjackResult = `🤝 تعادل! استرداد الرهان`;
          this.result = this.blackjackResult;
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
        let multiplier = 1;
        if (a === '7️⃣') multiplier = 8;
        else if (a === '💎') multiplier = 6;
        else if (a === '🔔') multiplier = 4;
        else multiplier = 3;
        
        const profit = this.slotBet * multiplier;
        this.balance += profit;
        await this.updateBalance(this.balance);
        this.slotResult = `🎉 فوز! ربحت ${profit.toFixed(2)} USDT`;
        this.result = this.slotResult;
      } else {
        this.slotResult = "😢 للأسف لا فوز";
        this.result = this.slotResult;
      }
      
      setTimeout(() => {
        this.slotStarted = false;
      }, 3000);
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
          const profit = this.coinflipBet * 1.9;
          this.balance += profit;
          this.updateBalance(this.balance);
          this.result = `🎉 فوز! ربحت ${profit.toFixed(2)} USDT`;
        } else {
          this.result = `😢 خسارة! كانت النتيجة ${this.coinflipResult}`;
        }
        
        setTimeout(() => {
          this.coinflipStarted = false;
        }, 3000);
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
            this.wheelResult = `🎉 وقفت على x${multiplier} ربحت ${profit.toFixed(2)} USDT`;
            this.result = this.wheelResult;
            
            setTimeout(() => {
              this.wheelStarted = false;
            }, 3000);
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
        if (this.kenoSelected.length < 5) {
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
      
      // تأثير السحب التدريجي
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
      
      const multipliers = [0, 1, 2, 5, 10, 20];
      this.kenoMultiplier = multipliers[this.kenoMatches] || 0;
      
      if (this.kenoMultiplier > 0) {
        const profit = this.kenoBet * this.kenoMultiplier;
        this.balance += profit;
        await this.updateBalance(this.balance);
        this.result = `🎉 ${this.kenoMatches} أرقام متطابقة! ربحت ${profit.toFixed(2)} USDT`;
      } else {
        this.result = `😢 ${this.kenoMatches} أرقام متطابقة. لا فوز`;
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
        const knocked = Math.floor(Math.random() * 10);
        
        // تأثير سقوط القوارير
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
      let multiplier = 0.5;
      if (knocked === 10) multiplier = 3;
      else if (knocked >= 8) multiplier = 2;
      else if (knocked >= 5) multiplier = 1.5;
      else if (knocked >= 3) multiplier = 1;
      
      const profit = this.bowlingBet * multiplier;
      this.balance += profit;
      await this.updateBalance(this.balance);
      
      if (knocked === 10) {
        this.bowlingResult = `🎳 سترايك! ربحت ${profit.toFixed(2)} USDT`;
      } else {
        this.bowlingResult = `🎳 ضربت ${knocked} قوارير ربحت ${profit.toFixed(2)} USDT`;
      }
      this.result = this.bowlingResult;
      
      setTimeout(() => {
        this.bowlingStarted = false;
      }, 3000);
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
        const profit = this.puzzleBet * 3;
        this.balance += profit;
        this.updateBalance(this.balance);
        this.result = `🧩 أحسنت! حللت اللغز وربحت ${profit.toFixed(2)} USDT`;
        
        setTimeout(() => {
          this.puzzleStarted = false;
        }, 3000);
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
      this.targetResult = "اضرب الهدف لجمع النقاط";
    },
    
    async shootTarget(score) {
      const hit = Math.random() < 0.4;
      
      if (hit) {
        this.targetScore += score;
        this.targetResult = `🎯 أصابت الهدف! +${score} نقطة`;
      } else {
        this.targetResult = "😢 أخطأت الهدف";
      }
      
      if (this.targetScore >= 30) {
        const multiplier = 1.5 + (this.targetScore / 50);
        const profit = this.targetBet * multiplier;
        this.balance += profit;
        await this.updateBalance(this.balance);
        this.targetResult = `🎉 فوز! مجموع النقاط ${this.targetScore} ربحت ${profit.toFixed(2)} USDT`;
        this.result = this.targetResult;
        
        setTimeout(() => {
          this.targetStarted = false;
        }, 3000);
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
      
      // تأثير السحب
      let rolls = 0;
      const interval = setInterval(() => {
        this.luckyDrawn = Math.floor(Math.random() * 100) + 1;
        rolls++;
        
        if (rolls >= 20) {
          clearInterval(interval);
          this.luckyWon = this.luckyDrawn === this.luckyNumber;
          
          if (this.luckyWon) {
            const profit = this.luckyBet * 25;
            this.balance += profit;
            this.updateBalance(this.balance);
            this.result = `🎉 رقم محظوظ! ربحت ${profit.toFixed(2)} USDT`;
          } else {
            this.result = `😢 الرقم المسحوب ${this.luckyDrawn}`;
          }
          
          setTimeout(() => {
            this.luckyStarted = false;
          }, 3000);
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
      
      // تأثير الانتظار
      setTimeout(async () => {
        const prizes = [0.3, 0.5, 0.8, 1, 1.2, 1.5, 2, 2.5, 3, 5];
        const weights = [0.25, 0.2, 0.15, 0.12, 0.1, 0.08, 0.05, 0.03, 0.01, 0.01];
        
        let random = Math.random();
        let cumulative = 0;
        let multiplier = 0.3;
        
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
          this.mysteryPrize = `💰 ${profit.toFixed(2)} USDT`;
          this.mysteryResult = `🎉 ربحت ${profit.toFixed(2)} USDT!`;
        } else {
          this.mysteryPrize = `😢 ${profit.toFixed(2)} USDT`;
          this.mysteryResult = `😢 خسرت جزء من الرهان`;
        }
        
        this.result = this.mysteryResult;
      }, 1000);
    }
  }
};    
</script>    
    
<style scoped>    
/* الخلفية الرئيسية - أسود فاخر */
.game-page {    
  background: linear-gradient(135deg, #0A0C10 0%, #1A1F2A 100%);
  min-height: 100vh;    
  color: #ffffff;    
  padding: 20px;    
  text-align: center;    
  direction: rtl;
  font-family: 'Cairo', sans-serif;
}    

/* الشريط العلوي */
.top-bar {
  display: flex;
  justify-content: center;
  margin-bottom: 25px;
}

.balance-gold {
  background: linear-gradient(145deg, #11151C, #1A1F2A);
  padding: 12px 25px;
  border-radius: 50px;
  border: 1px solid #D4AF37;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3), inset 0 0 10px rgba(212, 175, 55, 0.2);
  display: flex;
  align-items: center;
  gap: 10px;
  animation: glowPulse 2s infinite;
}

@keyframes glowPulse {
  0%, 100% { box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3), inset 0 0 10px rgba(212, 175, 55, 0.2); }
  50% { box-shadow: 0 5px 30px rgba(212, 175, 55, 0.5), inset 0 0 20px rgba(212, 175, 55, 0.3); }
}

.balance-gold i {
  color: #D4AF37;
  font-size: 20px;
  filter: drop-shadow(0 0 5px #D4AF37);
}

.balance-gold strong {
  color: #D4AF37;
  font-size: 18px;
  margin-right: 5px;
  text-shadow: 0 0 10px rgba(212, 175, 55, 0.5);
}

/* زر الرجوع */
.back-button-container {
  margin-bottom: 20px;
  text-align: right;
}

.back-button {
  background: linear-gradient(145deg, #1A1F2A, #11151C);
  color: #D4AF37;
  border: 1px solid #D4AF37;
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
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  transform: translateX(-5px);
  box-shadow: 0 8px 20px rgba(212, 175, 55, 0.4);
}

.back-button i {
  font-size: 18px;
}

/* التبويبات */
.tabs {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
  margin-bottom: 25px;
  max-height: 200px;
  overflow-y: auto;
  padding: 15px;
  background: linear-gradient(145deg, #11151C, #0A0C10);
  border-radius: 20px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  scrollbar-width: thin;
  scrollbar-color: #D4AF37 #1A1F2A;
}

.tabs::-webkit-scrollbar {
  width: 8px;
}

.tabs::-webkit-scrollbar-track {
  background: #1A1F2A;
  border-radius: 10px;
}

.tabs::-webkit-scrollbar-thumb {
  background: #D4AF37;
  border-radius: 10px;
  box-shadow: inset 0 0 5px #F6E27A;
}

.tabs button {
  padding: 10px 15px;
  border-radius: 50px;
  background: linear-gradient(145deg, #1A1F2A, #11151C);
  color: #ffffff;
  border: 1px solid rgba(212, 175, 55, 0.3);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 5px;
  white-space: nowrap;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.3);
}

.tabs button:hover {
  border-color: #D4AF37;
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(212, 175, 55, 0.3);
}

.tabs .active {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border: none;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.5), inset 0 0 10px rgba(255, 255, 255, 0.5);
  animation: activePulse 2s infinite;
}

@keyframes activePulse {
  0%, 100% { box-shadow: 0 5px 20px rgba(212, 175, 55, 0.5), inset 0 0 10px rgba(255, 255, 255, 0.5); }
  50% { box-shadow: 0 5px 30px rgba(212, 175, 55, 0.8), inset 0 0 20px rgba(255, 255, 255, 0.7); }
}

.tab-icon {
  font-size: 18px;
  filter: drop-shadow(0 0 5px currentColor);
}

.tab-text {
  font-size: 12px;
}

/* وضع ملء الشاشة للألعاب */
.game-fullscreen {
  min-height: calc(100vh - 150px);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  padding: 10px 0;
}

.fullscreen-card {
  width: 100%;
  max-width: 550px;
  margin: 0 auto;
  min-height: auto;
}

/* البطاقات */
.game-container {
  max-width: 500px;
  margin: 0 auto;
  display: block;
  width: 100%;
}

.card {
  background: linear-gradient(145deg, #11151C, #0A0C10);
  border-radius: 30px;
  padding: 20px;
  border: 1px solid rgba(212, 175, 55, 0.3);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.8), inset 0 0 50px rgba(212, 175, 55, 0.1);
  position: relative;
  overflow: hidden;
  backdrop-filter: blur(10px);
  display: block;
  width: 100%;
}

.card::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.1) 0%, transparent 70%);
  animation: rotate 20s linear infinite;
  pointer-events: none;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.card-header {
  position: relative;
  margin-bottom: 15px;
}

.card-header h2 {
  font-size: 24px;
  font-weight: 800;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  position: relative;
  z-index: 1;
  text-shadow: 0 0 20px rgba(212, 175, 55, 0.5);
}

.header-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 150px;
  height: 150px;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.3) 0%, transparent 70%);
  filter: blur(40px);
  z-index: 0;
  animation: glowMove 5s infinite;
}

@keyframes glowMove {
  0%, 100% { transform: translate(-50%, -50%) scale(1); opacity: 0.5; }
  50% { transform: translate(-50%, -50%) scale(1.2); opacity: 0.8; }
}

.game-icon {
  font-size: 80px;
  margin: 20px 0;
  filter: drop-shadow(0 0 20px #D4AF37);
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-15px); }
}

/* ===== Chicken Road Styles ===== */
.chicken-card .chicken-container {
  margin: 20px auto;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
}

.chicken {
  font-size: 100px;
  position: relative;
  z-index: 2;
  filter: drop-shadow(0 0 20px #D4AF37);
  transition: all 0.3s ease;
}

.chicken.walking {
  animation: walk 0.5s infinite;
}

@keyframes walk {
  0%, 100% { transform: translateX(0) rotate(0deg); }
  25% { transform: translateX(-5px) rotate(-5deg); }
  75% { transform: translateX(5px) rotate(5deg); }
}

.chicken-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 150px;
  height: 150px;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.4) 0%, transparent 70%);
  filter: blur(30px);
  border-radius: 50%;
  animation: pulseGlow 2s infinite;
}

@keyframes pulseGlow {
  0%, 100% { transform: translate(-50%, -50%) scale(1); opacity: 0.5; }
  50% { transform: translate(-50%, -50%) scale(1.2); opacity: 0.8; }
}

.road {
  display: flex;
  justify-content: space-between;
  margin: 20px 0;
  background: linear-gradient(145deg, #1A1F2A, #11151C);
  padding: 15px;
  border-radius: 20px;
  gap: 5px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  flex-wrap: wrap;
}

.step {
  flex: 1;
  min-width: 40px;
  background: linear-gradient(145deg, #1E2430, #151A24);
  border-radius: 12px;
  padding: 10px 2px;
  font-size: 12px;
  position: relative;
  min-height: 70px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border: 1px solid rgba(212, 175, 55, 0.2);
  transition: all 0.3s ease;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.3);
}

.step.active {
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  transform: scale(1.05);
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.5), inset 0 0 10px rgba(255, 255, 255, 0.5);
  animation: activeStep 1s infinite;
}

@keyframes activeStep {
  0%, 100% { box-shadow: 0 10px 25px rgba(212, 175, 55, 0.5), inset 0 0 10px rgba(255, 255, 255, 0.5); }
  50% { box-shadow: 0 15px 35px rgba(212, 175, 55, 0.8), inset 0 0 20px rgba(255, 255, 255, 0.7); }
}

.step.passed {
  background: linear-gradient(145deg, #2A2F3A, #1E2430);
  border-color: #22c55e;
}

.gold-step {
  border-color: #D4AF37;
  box-shadow: 0 0 10px rgba(212, 175, 55, 0.3);
}

.silver-step {
  border-color: #C0C0C0;
  box-shadow: 0 0 10px rgba(192, 192, 192, 0.3);
}

.multiplier {
  font-weight: 700;
  font-size: 14px;
}

.step-check {
  position: absolute;
  top: 5px;
  left: 5px;
  color: #22c55e;
  font-size: 16px;
  filter: drop-shadow(0 0 5px #22c55e);
  animation: checkPop 0.3s ease;
}

@keyframes checkPop {
  0% { transform: scale(0); }
  80% { transform: scale(1.2); }
  100% { transform: scale(1); }
}

.chicken-icon {
  font-size: 28px;
  margin-top: 5px;
  filter: drop-shadow(0 5px 10px rgba(0, 0, 0, 0.5));
  animation: iconBounce 0.5s infinite;
}

@keyframes iconBounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}

/* قسم الرهان */
.bet-section {
  margin-top: 20px;
  padding-top: 15px;
  border-top: 1px solid rgba(212, 175, 55, 0.3);
}

.bet-controls {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
  width: 100%;
}

.input-gold-wrapper {
  position: relative;
  width: 100%;
  max-width: 250px;
}

.gold-input {
  width: 100%;
  padding: 12px 20px;
  border-radius: 50px;
  background: linear-gradient(145deg, #1A1F2A, #11151C);
  color: #ffffff;
  border: 2px solid rgba(212, 175, 55, 0.3);
  font-size: 16px;
  text-align: center;
  transition: all 0.3s ease;
  box-shadow: inset 0 2px 5px rgba(0, 0, 0, 0.5);
}

.gold-input:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 20px rgba(212, 175, 55, 0.3), inset 0 2px 5px rgba(0, 0, 0, 0.5);
}

.input-currency {
  position: absolute;
  left: 15px;
  top: 50%;
  transform: translateY(-50%);
  color: #D4AF37;
  font-weight: 600;
  text-shadow: 0 0 5px #D4AF37;
}

.gold-button {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border: none;
  padding: 12px 35px;
  border-radius: 50px;
  font-weight: 700;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.4), inset 0 2px 5px rgba(255, 255, 255, 0.5);
  width: 100%;
  max-width: 250px;
}

.gold-button.pulse {
  animation: buttonPulse 2s infinite;
}

@keyframes buttonPulse {
  0%, 100% { box-shadow: 0 5px 20px rgba(212, 175, 55, 0.4), inset 0 2px 5px rgba(255, 255, 255, 0.5); }
  50% { box-shadow: 0 5px 30px rgba(212, 175, 55, 0.8), inset 0 2px 10px rgba(255, 255, 255, 0.7); }
}

.gold-button:hover:not(:disabled) {
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.6), inset 0 2px 10px rgba(255, 255, 255, 0.7);
}

.gold-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  filter: grayscale(0.5);
}

/* عناصر التحكم أثناء اللعبة */
.game-controls {
  display: flex;
  flex-direction: column;
  gap: 15px;
  width: 100%;
}

.game-controls-full {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 100%;
}

.profit-display {
  background: linear-gradient(145deg, #1A1F2A, #11151C);
  padding: 15px;
  border-radius: 16px;
  border: 1px solid rgba(212, 175, 55, 0.3);
  box-shadow: inset 0 2px 5px rgba(0, 0, 0, 0.5);
}

.profit-label {
  color: rgba(255, 255, 255, 0.7);
  margin-left: 10px;
}

.profit-value {
  color: #D4AF37;
  font-size: 20px;
  font-weight: 800;
  text-shadow: 0 0 10px #D4AF37;
}

.action-buttons {
  display: flex;
  justify-content: center;
  gap: 15px;
  flex-wrap: wrap;
  width: 100%;
}

.action-btn {
  padding: 12px 20px;
  border-radius: 50px;
  border: none;
  font-weight: 700;
  font-size: 15px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  flex: 1;
  min-width: 120px;
}

.action-btn.gold {
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3), inset 0 2px 5px rgba(255, 255, 255, 0.5);
}

.action-btn.green {
  background: linear-gradient(135deg, #22c55e, #16a34a);
  color: white;
  box-shadow: 0 5px 15px rgba(34, 197, 94, 0.3), inset 0 2px 5px rgba(255, 255, 255, 0.5);
}

.action-btn:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(212, 175, 55, 0.4), inset 0 2px 5px rgba(255, 255, 255, 0.5);
}

.action-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* رسائل الخطأ */
.error-message {
  color: #ef4444;
  font-size: 13px;
  display: flex;
  align-items: center;
  gap: 5px;
  background: rgba(239, 68, 68, 0.15);
  padding: 5px 12px;
  border-radius: 20px;
  border: 1px solid #ef4444;
  box-shadow: 0 0 10px rgba(239, 68, 68, 0.3);
}

/* نتيجة اللعبة */
.result-popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: linear-gradient(145deg, #11151C, #0A0C10);
  padding: 20px 40px;
  border-radius: 20px;
  display: flex;
  align-items: center;
  gap: 15px;
  font-size: 20px;
  font-weight: 700;
  z-index: 1000;
  border: 2px solid;
  animation: popIn 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.8);
}

.result-popup.win {
  border-color: #D4AF37;
  box-shadow: 0 0 50px rgba(212, 175, 55, 0.5), inset 0 0 20px rgba(212, 175, 55, 0.3);
}

.result-popup.win i {
  color: #D4AF37;
  filter: drop-shadow(0 0 10px #D4AF37);
}

.result-popup.lose {
  border-color: #ef4444;
  box-shadow: 0 0 50px rgba(239, 68, 68, 0.5), inset 0 0 20px rgba(239, 68, 68, 0.3);
}

.result-popup.lose i {
  color: #ef4444;
  filter: drop-shadow(0 0 10px #ef4444);
}

@keyframes popIn {
  0% {
    opacity: 0;
    transform: translate(-50%, -40%) scale(0.8);
  }
  100% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }
}

/* باقي الألعاب بنفس التنسيقات السابقة ولكن تم اختصارها في هذا الرد للطول */
/* يرجى الرجوع إلى الكود الكامل في الردود السابقة لبقية التنسيقات */

/* تحسينات للجوال */
@media (max-width: 480px) {
  .game-page {
    padding: 10px;
  }

  .tabs {
    padding: 10px;
    gap: 8px;
  }

  .tabs button {
    padding: 8px 12px;
    font-size: 12px;
  }

  .card {
    padding: 15px;
  }

  .card-header h2 {
    font-size: 20px;
  }

  .game-icon {
    font-size: 60px;
  }

  .step {
    padding: 8px 1px;
    font-size: 10px;
    min-height: 60px;
  }

  .multiplier {
    font-size: 12px;
  }

  .result-popup {
    padding: 15px 25px;
    font-size: 16px;
  }

  .input-gold-wrapper {
    width: 100%;
  }

  .gold-button {
    width: 100%;
    justify-content: center;
  }

  .action-btn {
    flex: 1;
    justify-content: center;
    font-size: 14px;
    padding: 10px 15px;
  }

  .slot-reel {
    width: 60px;
    height: 60px;
    font-size: 30px;
  }

  .mines-grid {
    gap: 5px;
  }

  .mine-cell {
    font-size: 16px;
  }

  .keno-cell {
    font-size: 12px;
  }

  .puzzle-piece {
    font-size: 18px;
  }

  .drawn-number {
    font-size: 60px;
  }

  .wheel-spinner {
    width: 200px;
    height: 200px;
  }

  .segment-value {
    font-size: 14px;
  }

  .target {
    width: 200px;
    height: 200px;
  }

  .bullseye {
    width: 60px;
    height: 60px;
    top: 70px;
    left: 70px;
  }

  .middle {
    width: 100px;
    height: 100px;
    top: 50px;
    left: 50px;
  }

  .outer {
    width: 140px;
    height: 140px;
    top: 30px;
    left: 30px;
  }

  .back-button {
    width: 100%;
    justify-content: center;
  }
}
</style>
