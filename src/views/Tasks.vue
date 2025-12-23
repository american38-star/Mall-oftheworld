<template>
  <div class="games-page">

    <!-- العنوان -->
    <h2 class="title">🎮 الألعاب</h2>
    <p class="sub">العب واربح USDT بنظام آمن</p>

    <!-- لعبة 1 -->
    <div class="game-card">
      <h3>📦 الصندوق الآمن</h3>
      <p>اربح بين 1% و 3% ربح ثابت</p>
      <input v-model.number="boxAmount" type="number" placeholder="المبلغ USDT" />
      <button @click="playSafeBox">العب</button>
    </div>

    <!-- لعبة 2 -->
    <div class="game-card">
      <h3>🎲 الحظ المحدود</h3>
      <p>فرصة ربح 2x (نسبة فوز 40%)</p>
      <input v-model.number="luckAmount" type="number" placeholder="المبلغ USDT" />
      <button @click="playLuck">جرب حظك</button>
    </div>

    <!-- لعبة 3 -->
    <div class="game-card">
      <h3>📈 الاستثمار الذكي</h3>
      <p>ربح يومي 1.5% (قفل 24 ساعة)</p>
      <input v-model.number="investAmount" type="number" placeholder="المبلغ USDT" />
      <button @click="startInvestment">استثمر</button>
    </div>

  </div>
</template>

<script>
import { auth, db } from "../firebase";
import {
  doc,
  getDoc,
  runTransaction,
  serverTimestamp
} from "firebase/firestore";

export default {
  name: "Games",

  data() {
    return {
      boxAmount: 0,
      luckAmount: 0,
      investAmount: 0,
    };
  },

  methods: {
    // 🟢 لعبة الصندوق الآمن
    async playSafeBox() {
      const user = auth.currentUser;
      if (!user || this.boxAmount <= 0) return alert("مبلغ غير صالح");

      const percent = Math.random() * (0.03 - 0.01) + 0.01; // 1% - 3%
      const profit = this.boxAmount * percent;

      await this.updateBalance(user.uid, this.boxAmount, profit);
      alert(`✔ ربحت ${profit.toFixed(2)} USDT`);
    },

    // 🟡 لعبة الحظ
    async playLuck() {
      const user = auth.currentUser;
      if (!user || this.luckAmount <= 0) return alert("مبلغ غير صالح");

      const win = Math.random() < 0.4; // 40% فوز
      const profit = win ? this.luckAmount : -this.luckAmount;

      await this.updateBalance(user.uid, this.luckAmount, profit);
      alert(win ? "🎉 ربحت!" : "❌ خسرت");
    },

    // 🔵 الاستثمار الذكي
    async startInvestment() {
      const user = auth.currentUser;
      if (!user || this.investAmount <= 0) return alert("مبلغ غير صالح");

      const userRef = doc(db, "users", user.uid);

      await runTransaction(db, async (tx) => {
        const snap = await tx.get(userRef);
        const balance = Number(snap.data().balance || 0);

        if (balance < this.investAmount) {
          throw "رصيد غير كافي";
        }

        tx.update(userRef, {
          balance: balance - this.investAmount,
        });

        tx.set(doc(db, "users", user.uid, "investments", Date.now().toString()), {
          amount: this.investAmount,
          profit: this.investAmount * 0.015,
          unlockAt: Date.now() + 86400000,
          createdAt: serverTimestamp(),
        });
      });

      alert("✔ تم بدء الاستثمار");
    },

    // 🔒 تحديث الرصيد الآمن
    async updateBalance(uid, amount, profit) {
      const userRef = doc(db, "users", uid);

      await runTransaction(db, async (tx) => {
        const snap = await tx.get(userRef);
        const balance = Number(snap.data().balance || 0);

        if (balance < amount) {
          throw "رصيد غير كافي";
        }

        tx.update(userRef, {
          balance: balance + profit,
        });
      });
    },
  },
};
</script>

<style scoped>
.games-page {
  direction: rtl;
  padding: 16px;
  min-height: 100vh;
  background: linear-gradient(#0d6efd, #6bb4ff);
  color: #fff;
}

.title {
  text-align: center;
  font-size: 22px;
  font-weight: bold;
}

.sub {
  text-align: center;
  margin-bottom: 20px;
}

.game-card {
  background: #ffffffcc;
  color: black;
  padding: 16px;
  border-radius: 16px;
  margin-bottom: 16px;
  text-align: center;
}

.game-card input {
  width: 100%;
  padding: 8px;
  margin: 8px 0;
  border-radius: 8px;
  border: 1px solid #ccc;
}

.game-card button {
  width: 100%;
  padding: 10px;
  border-radius: 10px;
  border: none;
  background: #0d6efd;
  color: white;
  font-weight: bold;
}
</style>
