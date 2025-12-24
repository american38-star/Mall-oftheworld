<template>
  <div class="game-page">

    <!-- الرصيد -->
    <div class="top-bar">
      <div class="balance">رصيدك: {{ balance.toFixed(2) }} USDT</div>
    </div>

    <!-- التبويبات -->
    <div class="tabs">
      <button :class="{active: game==='chicken'}" @click="switchGame('chicken')">
        🐔 Chicken Road
      </button>
      <button :class="{active: game==='plinko'}" @click="switchGame('plinko')">
        🔴 Plinko
      </button>
    </div>

    <!-- ================= CHICKEN ROAD ================= -->
    <div v-if="game==='chicken'" class="card">

      <h2>🐔 Chicken Road</h2>

      <div v-if="!started" class="bet-box">
        <input type="number" v-model.number="bet" placeholder="مبلغ الرهان USDT" />
        <button @click="startChicken">ابدأ</button>
      </div>

      <div v-if="started" class="road">
        <div
          v-for="(step,i) in steps"
          :key="i"
          class="step"
          :class="{active:i===position}"
        >
          x{{ step.multiplier }}
          <div v-if="i===position" class="icon">🐔</div>
        </div>
      </div>

      <div v-if="started" class="controls">
        <div class="profit">الربح: {{ currentProfit.toFixed(2) }} USDT</div>
        <button @click="goNext">تقدم</button>
        <button @click="cashOutChicken">سحب</button>
      </div>
    </div>

    <!-- ================= PLINKO ================= -->
    <div v-if="game==='plinko'" class="card">

      <h2>🔴 Plinko</h2>

      <div class="bet-box">
        <input type="number" v-model.number="plinkoBet" placeholder="مبلغ الرهان USDT" />
        <button :disabled="ball.active" @click="startPlinko">PLAY</button>
      </div>

      <!-- اللوحة -->
      <div class="plinko-board">
        <div
          v-for="(row,r) in rows"
          :key="r"
          class="row"
        >
          <span v-for="d in row" :key="d" class="dot"></span>
        </div>

        <div
          v-if="ball.active"
          class="ball"
          :style="{ top: ball.y+'px', left: ball.x+'px' }"
        ></div>
      </div>

      <!-- المضاعفات -->
      <div class="multipliers">
        <span
          v-for="(m,i) in plinkoMultipliers"
          :key="i"
        >
          x{{ m }}
        </span>
      </div>
    </div>

    <div v-if="result" class="result">{{ result }}</div>

  </div>
</template>

<script>
import { auth, db } from "../firebase";
import { doc, getDoc, updateDoc } from "firebase/firestore";

export default {
  name: "Games",

  data() {
    return {
      game: "chicken",
      balance: 0,
      result: "",

      /* ===== Chicken ===== */
      bet: null,
      started: false,
      position: 0,
      steps: [
        { multiplier: 1 },
        { multiplier: 1.3 },
        { multiplier: 1.7 },
        { multiplier: 2.5 },
        { multiplier: 4 },
      ],

      /* ===== Plinko ===== */
      plinkoBet: null,
      rows: Array.from({ length: 8 }, (_, i) => i + 3),
      plinkoMultipliers: [0.3, 0.5, 1, 1.5, 3, 5, 10],
      ball: {
        x: 140,
        y: 0,
        active: false,
      },
    };
  },

  computed: {
    currentProfit() {
      return this.bet
        ? this.bet * this.steps[this.position].multiplier
        : 0;
    },
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
    switchGame(g) {
      this.result = "";
      this.started = false;
      this.ball.active = false;
      this.game = g;
    },

    /* ===== Chicken ===== */
    async startChicken() {
      if (!this.bet || this.bet <= 0 || this.bet > this.balance) return;

      this.balance -= this.bet;
      await updateDoc(doc(db, "users", auth.currentUser.uid), {
        balance: this.balance,
      });

      this.started = true;
      this.position = 0;
      this.result = "";
    },

    goNext() {
      const loseChance = 0.55 + this.position * 0.08;
      if (Math.random() < loseChance) {
        this.result = "💥 خسرت";
        this.started = false;
        return;
      }

      if (this.position < this.steps.length - 1) {
        this.position++;
      } else {
        this.cashOutChicken();
      }
    },

    async cashOutChicken() {
      const profit = this.currentProfit;
      this.balance += profit;

      await updateDoc(doc(db, "users", auth.currentUser.uid), {
        balance: this.balance,
      });

      this.result = `🎉 ربحت ${profit.toFixed(2)} USDT`;
      this.started = false;
    },

    /* ===== Plinko ===== */
    async startPlinko() {
      if (!this.plinkoBet || this.plinkoBet <= 0 || this.plinkoBet > this.balance)
        return;

      this.balance -= this.plinkoBet;
      await updateDoc(doc(db, "users", auth.currentUser.uid), {
        balance: this.balance,
      });

      this.ball = { x: 140, y: 0, active: true };
      this.dropBall();
    },

    dropBall() {
      const interval = setInterval(async () => {
        this.ball.y += 10;
        this.ball.x += Math.random() > 0.5 ? 8 : -8;

        if (this.ball.y >= 260) {
          clearInterval(interval);
          this.ball.active = false;

          const index = Math.floor(
            Math.random() * this.plinkoMultipliers.length
          );

          const win = this.plinkoBet * this.plinkoMultipliers[index];
          this.balance += win;

          await updateDoc(doc(db, "users", auth.currentUser.uid), {
            balance: this.balance,
          });

          this.result = `🎯 ربحت ${win.toFixed(2)} USDT`;
        }
      }, 40);
    },
  },
};
</script>

<style scoped>
.game-page {
  background: #0f172a;
  min-height: 100vh;
  color: white;
  padding: 15px;
  text-align: center;
}

.top-bar {
  font-weight: bold;
  margin-bottom: 10px;
}

.tabs {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 15px;
}

.tabs button {
  padding: 10px 14px;
  border-radius: 10px;
  background: #1e293b;
  color: white;
  border: none;
}

.tabs .active {
  background: #22c55e;
}

.card {
  background: #020617;
  border-radius: 14px;
  padding: 15px;
  max-width: 420px;
  margin: auto;
}

.bet-box input {
  width: 100%;
  padding: 10px;
  border-radius: 10px;
  margin-bottom: 8px;
  border: none;
}

.bet-box button {
  width: 100%;
  padding: 10px;
  border-radius: 10px;
  background: #22c55e;
  border: none;
  color: black;
}

.road {
  display: flex;
  justify-content: space-between;
  margin: 15px 0;
}

.step {
  width: 18%;
  background: #1e293b;
  border-radius: 10px;
  padding: 8px;
}

.step.active {
  background: #22c55e;
  color: black;
}

.icon {
  font-size: 22px;
}

.controls button {
  width: 48%;
  padding: 10px;
  margin: 4px;
  border-radius: 10px;
  border: none;
}

.plinko-board {
  position: relative;
  height: 300px;
  margin: 15px auto;
}

.row {
  display: flex;
  justify-content: center;
}

.dot {
  width: 6px;
  height: 6px;
  background: white;
  border-radius: 50%;
  margin: 8px;
}

.ball {
  position: absolute;
  width: 14px;
  height: 14px;
  background: red;
  border-radius: 50%;
}

.multipliers span {
  margin: 3px;
  padding: 5px 8px;
  background: #1e293b;
  border-radius: 8px;
}

.result {
  margin-top: 15px;
  font-size: 18px;
  font-weight: bold;
}
</style>
