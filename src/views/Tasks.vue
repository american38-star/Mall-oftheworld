<template>      
  <div class="game-page">      
    <div class="top-bar">      
      <div class="balance">رصيدك: {{ balance.toFixed(2) }} USDT</div>      
    </div>      

    <div class="tabs">      
      <button :class="{active: game==='chicken'}" @click="switchGame('chicken')">      
        🐔 Chicken Road      
      </button>      
      <button :class="{active: game==='plinko'}" @click="switchGame('plinko')">      
        🔴 Plinko      
      </button>      
    </div>      

    <div v-if="game==='chicken'" class="card">      
      <h2>🐔 Chicken Road</h2>      
      <div v-if="!started" class="bet-box">      
        <input type="number" v-model.number="bet" placeholder="مبلغ الرهان USDT" />      
        <button @click="startChicken">ابدأ</button>      
      </div>      
      <div v-if="started" class="road">      
        <div v-for="(step,i) in steps" :key="i" class="step" :class="{active:i===position}">      
          x{{ step.multiplier.toFixed(2) }}      
          <div v-if="i===position" class="icon">🐔</div>      
        </div>      
      </div>      
      <div v-if="started" class="controls">      
        <div class="profit">الربح: {{ currentProfit.toFixed(2) }} USDT</div>      
        <button @click="goNext">تقدم</button>      
        <button @click="cashOutChicken">سحب</button>      
      </div>      
    </div>      

    <div v-if="game==='plinko'" class="card">      
      <h2>🔴 Plinko</h2>      
      <div class="bet-box">      
        <input type="number" v-model.number="plinkoBet" placeholder="مبلغ الرهان USDT" />      
        <button :disabled="ball.active" @click="startPlinko">PLAY</button>      
      </div>      

      <div class="plinko-board">      
        <div v-for="(row,r) in rows" :key="r" class="row">      
          <span v-for="n in row" :key="n" class="dot"></span>      
        </div>      

        <div v-if="ball.active" class="ball" :style="{ top: ball.y+'px', left: ball.x+'px' }"></div>      
      </div>      

      <div class="multipliers">      
        <span v-for="(m,i) in plinkoMultipliers" :key="i" :class="multiplierClass(m)">      
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
      bet: null,      
      started: false,      
      position: 0,      
      steps: [      
        { multiplier: 1.0 },      
        { multiplier: 1.1 },      
        { multiplier: 1.3 },      
        { multiplier: 1.5 },      
        { multiplier: 2.0 },      
        { multiplier: 3.0 },      
        { multiplier: 5.0 },      
      ],      
      plinkoBet: null,      
      rows: [3,4,5,6,7,8,9,10],      
      plinkoMultipliers: [29, 4, 1.5, 0.3, 0.2, 0.3, 1.5, 4, 29],      
      ball: { x: 150, y: 0, active: false },      
    };      
  },      
  computed: {      
    currentProfit() {      
      if (!this.bet) return 0;      
      return this.bet * this.steps[this.position].multiplier;      
    },      
  },      
  async created() {      
    const user = auth.currentUser;      
    if (!user) return;      
    const snap = await getDoc(doc(db, "users", user.uid));      
    if (snap.exists()) this.balance = Number(snap.data().balance || 0);      
  },      
  methods: {      
    switchGame(g) {      
      this.result = "";      
      this.started = false;      
      this.ball.active = false;      
      this.game = g;      
    },      
    async startChicken() {      
      if (!this.bet || this.bet <= 0 || this.bet > this.balance) return;      
      this.balance -= this.bet;      
      await updateDoc(doc(db, "users", auth.currentUser.uid), { balance: this.balance });      
      this.started = true;      
      this.position = 0;      
    },      
    goNext() {      
      const loseChance = 0.4 + this.position * 0.07;      
      if (Math.random() < loseChance) {      
        this.result = "💥 خسرت";      
        this.started = false;      
        return;      
      }      
      if (this.position < this.steps.length - 1) this.position++;      
      else this.cashOutChicken();      
    },      
    async cashOutChicken() {      
      const profit = this.currentProfit;      
      this.balance += profit;      
      await updateDoc(doc(db, "users", auth.currentUser.uid), { balance: this.balance });      
      this.result = `🎉 ربحت ${profit.toFixed(2)} USDT`;      
      this.started = false;      
    },      
    async startPlinko() {      
      if (!this.plinkoBet || this.plinkoBet <= 0 || this.plinkoBet > this.balance) return;      
      this.balance -= this.plinkoBet;      
      await updateDoc(doc(db, "users", auth.currentUser.uid), { balance: this.balance });      
      this.ball = { x: 150, y: 0, active: true };      
      this.dropBall();      
    },      
    dropBall() {      
      const interval = setInterval(async () => {      
        this.ball.y += 10;      
        this.ball.x += Math.random() > 0.5 ? 12 : -12;      
        if (this.ball.y >= 260) {      
          clearInterval(interval);      
          this.ball.active = false;      
          const index = Math.floor(Math.random() * this.plinkoMultipliers.length);      
          const multiplier = this.plinkoMultipliers[index];      
          const win = this.plinkoBet * multiplier;      
          this.balance += win;      
          await updateDoc(doc(db, "users", auth.currentUser.uid), { balance: this.balance });      
          this.result = `🎯 ربحت ${win.toFixed(2)} USDT`;      
        }      
      }, 40);      
    },      
    multiplierClass(m) {      
      if (m >= 10) return "high";      
      if (m <= 0.3) return "low";      
      return "mid";      
    },      
  },      
};      
</script>      

<style scoped>      
.game-page { background:#0f172a; min-height:100vh; color:white; padding:15px; text-align:center; }      
.tabs { display:flex; justify-content:center; gap:10px; margin-bottom:15px; }      
.tabs button { padding:10px 14px; border-radius:10px; background:#1e293b; color:white; border:none; }      
.tabs .active { background:#22c55e; }      
.card { background:#020617; border-radius:14px; padding:15px; max-width:420px; margin:auto; }      

.plinko-board { position:relative; height:260px; margin:10px auto 0; }      
.row { display:flex; justify-content:center; }      
.dot { width:6px; height:6px; background:white; border-radius:50%; margin:7px; }      
.ball { position:absolute; width:14px; height:14px; background:#ff2d55; border-radius:50%; }      

/* ✅ التعديل المطلوب فقط هنا */
.multipliers {      
  display:flex;      
  justify-content:center;      
  margin-top:-8px; /* رفع المضاعفات تحت آخر صف نقاط */      
}      

.multipliers span {      
  margin:0 3px;      
  padding:3px 5px;      
  border-radius:5px;      
  font-weight:bold;      
  font-size:11px; /* تصغير الأرقام */      
  min-width:34px;      
  text-align:center;      
}      

.high { background:#dc2626; }      
.mid { background:#22c55e; color:black; }      
.low { background:#facc15; color:black; }      
.result { margin-top:15px; font-size:18px; font-weight:bold; }      
</style>
