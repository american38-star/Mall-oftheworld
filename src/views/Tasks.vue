<template>
  <div class="game-page">

    <h2 class="title">🐔 Chicken Road</h2>
    <p class="sub">غامر خطوة بخطوة واربح USDT (الخطر يزداد)</p>

    <!-- الرصيد -->
    <div class="balance">رصيدك: {{ balance.toFixed(2) }} USDT</div>

    <!-- إدخال الرهان -->
    <div v-if="!started" class="bet-box">
      <input
        type="number"
        v-model.number="bet"
        placeholder="مبلغ الرهان USDT"
      />
      <button @click="startGame" :disabled="bet <= 0 || bet > balance">
        ابدأ اللعب
      </button>
    </div>

    <!-- الطريق -->
    <div v-if="started" class="road">
      <div
        v-for="(step, i) in steps"
        :key="i"
        class="step"
        :class="{ active: i === position }"
      >
        <div class="multiplier">x{{ step.multiplier }}</div>
        <div v-if="i === position" class="chicken">🐔</div>
      </div>
    </div>

    <!-- التحكم -->
    <div v-if="started" class="controls">
      <div class="profit">
        الربح الحالي: {{ currentProfit.toFixed(2) }} USDT
      </div>

      <button class="forward" @click="goNext">
        إلى الأمام
      </button>

      <button class="cashout" @click="cashOut">
        سحب الأرباح
      </button>
    </div>

    <!-- النتيجة -->
    <div v-if="result" class="result" :class="result">
      {{ result === 'win' ? '🎉 ربحت' : '💥 خسرت' }}
    </div>

  </div>
</template>

<script>
import { auth, db } from "../firebase";
import { doc, getDoc, runTransaction } from "firebase/firestore";

export default {
  name: "ChickenRoad",

  data() {
    return {
      balance: 0,
      bet: 0,
      started: false,
      position: 0,
      result: null,

      // 🔥 خسارة أعلى من الربح
      steps: [
        { multiplier: 1.2, loseChance: 0.25 },
        { multiplier: 1.5, loseChance: 0.35 },
        { multiplier: 2.0, loseChance: 0.45 },
        { multiplier: 3.0, loseChance: 0.60 },
        { multiplier: 5.0, loseChance: 0.75 },
      ],
    };
  },

  computed: {
    currentProfit() {
      if (!this.started) return 0;
      return this.bet * this.steps[this.position].multiplier;
    },
  },

  async created() {
    await this.loadBalance();
  },

  methods: {
    async loadBalance() {
      const user = auth.currentUser;
      if (!user) return;

      const ref = doc(db, "users", user.uid);
      const snap = await getDoc(ref);
      if (snap.exists()) {
        this.balance = Number(snap.data().balance || 0);
      }
    },

    async startGame() {
      if (this.bet > this.balance) return;

      const user = auth.currentUser;
      if (!user) return;

      const userRef = doc(db, "users", user.uid);

      // خصم الرهان فورًا
      await runTransaction(db, async (tx) => {
        const snap = await tx.get(userRef);
        const bal = Number(snap.data().balance || 0);
        if (bal < this.bet) throw "رصيد غير كافي";
        tx.update(userRef, { balance: bal - this.bet });
      });

      this.balance -= this.bet;
      this.started = true;
      this.position = 0;
      this.result = null;
    },

    goNext() {
      const step = this.steps[this.position];
      const roll = Math.random();

      // 💥 خسارة
      if (roll < step.loseChance) {
        this.result = "lose";
        this.started = false;
        return;
      }

      // تقدم
      if (this.position < this.steps.length - 1) {
        this.position++;
      }
    },

    async cashOut() {
      const user = auth.currentUser;
      if (!user) return;

      const winAmount = this.currentProfit;
      const userRef = doc(db, "users", user.uid);

      await runTransaction(db, async (tx) => {
        const snap = await tx.get(userRef);
        const bal = Number(snap.data().balance || 0);
        tx.update(userRef, { balance: bal + winAmount });
      });

      this.balance += winAmount;
      this.started = false;
      this.result = "win";
    },
  },
};
</script>

<style scoped>
.game-page {
  direction: rtl;
  padding: 20px;
  min-height: 100vh;
  background: linear-gradient(#111, #333);
  color: #fff;
  text-align: center;
}

.title { font-size: 24px; }
.sub { color: #ccc; margin-bottom: 10px; }
.balance { margin-bottom: 15px; font-weight: bold; }

.bet-box input {
  width: 80%;
  padding: 10px;
  border-radius: 10px;
  border: none;
  margin-bottom: 10px;
}

.bet-box button {
  width: 80%;
  padding: 12px;
  border-radius: 12px;
  background: #0d6efd;
  color: white;
  border: none;
}

.road {
  display: flex;
  justify-content: space-between;
  margin: 20px 0;
}

.step {
  width: 18%;
  background: #222;
  border-radius: 12px;
  padding: 10px;
}

.step.active { background: #0d6efd; }
.chicken { font-size: 28px; }

.controls button {
  width: 45%;
  padding: 12px;
  border-radius: 12px;
  border: none;
  margin: 5px;
}

.forward { background: #28a745; color: white; }
.cashout { background: #ffc107; color: black; }

.result {
  margin-top: 20px;
  font-size: 22px;
  font-weight: bold;
}
</style>
