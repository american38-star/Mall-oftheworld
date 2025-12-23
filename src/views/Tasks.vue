<template>
  <div class="game-page">

    <h2 class="title">🐔 Chicken Road</h2>
    <p class="sub">غامر خطوة بخطوة – لا يوجد ربح مضمون</p>

    <div class="balance">رصيدك: {{ balance.toFixed(2) }} USDT</div>

    <!-- إدخال الرهان -->
    <div v-if="!started" class="bet-box">
      <input
        type="number"
        v-model.number="bet"
        placeholder="أدخل مبلغ USDT"
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

      <button
        class="cashout"
        @click="cashOut"
        :disabled="position === 0"
      >
        سحب الأرباح
      </button>
    </div>

    <div v-if="result" class="result">
      {{ result }}
    </div>

  </div>
</template>

<script>
import { auth, db } from "../firebase";
import { doc, getDoc, updateDoc } from "firebase/firestore";

export default {
  name: "ChickenRoad",

  data() {
    return {
      balance: 0,
      bet: 0,
      started: false,
      position: 0,
      result: "",

      // ✅ أول خطوة بدون ربح
      steps: [
        { multiplier: 1.0 },  // لا ربح (إلغاء الربح المضمون)
        { multiplier: 1.2 },
        { multiplier: 1.5 },
        { multiplier: 2.0 },
        { multiplier: 3.0 },
        { multiplier: 5.0 },
      ],

      // ✅ نسبة الفوز 5% فقط
      winChance: 0.05,
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
      if (this.bet <= 0 || this.bet > this.balance) return;

      const user = auth.currentUser;
      if (!user) return;

      // خصم الرهان فورًا
      this.balance -= this.bet;
      await updateDoc(doc(db, "users", user.uid), {
        balance: this.balance,
      });

      this.started = true;
      this.position = 0;
      this.result = "";
    },

    goNext() {
      const roll = Math.random();

      // ❌ خسارة بنسبة 95%
      if (roll > this.winChance) {
        this.result = "💥 خسرت الرهان";
        this.started = false;
        return;
      }

      // تقدم خطوة واحدة فقط
      if (this.position < this.steps.length - 1) {
        this.position++;
      } else {
        // وصل للنهاية = فوز إجباري
        this.cashOut();
      }
    },

    async cashOut() {
      const user = auth.currentUser;
      if (!user) return;

      const profit = this.currentProfit;

      this.balance += profit;

      await updateDoc(doc(db, "users", user.uid), {
        balance: this.balance,
      });

      this.result = `🎉 ربحت ${profit.toFixed(2)} USDT`;
      this.started = false;
    },
  },
};
</script>

<style scoped>
.game-page {
  direction: rtl;
  padding: 20px;
  min-height: 100vh;
  background: #111;
  color: #fff;
  text-align: center;
}

.title {
  font-size: 24px;
  margin-bottom: 5px;
}

.sub {
  color: #bbb;
  margin-bottom: 10px;
}

.balance {
  margin-bottom: 15px;
  font-weight: bold;
}

.bet-box input {
  width: 80%;
  padding: 10px;
  border-radius: 10px;
  margin-bottom: 10px;
  border: none;
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
  width: 15%;
  background: #333;
  border-radius: 12px;
  padding: 10px;
}

.step.active {
  background: #0d6efd;
}

.chicken {
  font-size: 28px;
}

.controls button {
  width: 45%;
  padding: 12px;
  border-radius: 12px;
  margin: 5px;
  border: none;
}

.forward {
  background: #28a745;
  color: white;
}

.cashout {
  background: #ffc107;
  color: black;
}

.result {
  margin-top: 20px;
  font-size: 20px;
  font-weight: bold;
}
</style>
