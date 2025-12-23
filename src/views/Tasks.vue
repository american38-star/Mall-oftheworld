<template>
  <div class="game-card">
    <h3>🎲 الحظ المحدود</h3>
    <p>فرصة ربح ×2 (نسبة فوز 40%)</p>

    <input
      type="number"
      v-model.number="bet"
      placeholder="أدخل المبلغ USDT"
    />

    <button @click="play" :disabled="loading">
      {{ loading ? "جاري اللعب..." : "جرّب حظك" }}
    </button>

    <p v-if="message" :class="resultClass">{{ message }}</p>
  </div>
</template>

<script>
import { auth, db } from "../firebase";
import {
  doc,
  getDoc,
  runTransaction,
  addDoc,
  collection,
  serverTimestamp
} from "firebase/firestore";

export default {
  name: "LimitedLuck",

  data() {
    return {
      bet: 0,
      loading: false,
      message: "",
      resultClass: ""
    };
  },

  methods: {
    async play() {
      if (this.bet <= 0) {
        this.message = "❌ أدخل مبلغ صحيح";
        this.resultClass = "lose";
        return;
      }

      const user = auth.currentUser;
      if (!user) return;

      this.loading = true;
      this.message = "";

      const userRef = doc(db, "users", user.uid);

      try {
        await runTransaction(db, async (tx) => {
          const snap = await tx.get(userRef);
          if (!snap.exists()) throw "User not found";

          const balance = Number(snap.data().balance || 0);

          if (balance < this.bet) {
            throw "رصيدك غير كافٍ";
          }

          // 🎯 منطق الفوز (40%)
          const win = Math.random() < 0.4;
          let profit = 0;

          if (win) {
            profit = this.bet; // ×2 = ربح صافي = نفس المبلغ
            tx.update(userRef, {
              balance: balance + profit
            });
          } else {
            profit = -this.bet;
            tx.update(userRef, {
              balance: balance - this.bet
            });
          }

          // 🧾 تسجيل اللعبة
          await addDoc(collection(db, "games_logs"), {
            uid: user.uid,
            game: "limited_luck",
            bet: this.bet,
            result: win ? "win" : "lose",
            profit,
            createdAt: serverTimestamp()
          });

          this.message = win
            ? `✅ ربحت ${profit} USDT`
            : `❌ خسرت ${this.bet} USDT`;

          this.resultClass = win ? "win" : "lose";
        });
      } catch (e) {
        this.message = typeof e === "string" ? e : "حدث خطأ";
        this.resultClass = "lose";
      }

      this.loading = false;
    }
  }
};
</script>

<style scoped>
.game-card {
  background: #eef5ff;
  padding: 16px;
  border-radius: 16px;
  margin-bottom: 20px;
}

input {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
}

button {
  width: 100%;
  padding: 12px;
  background: #0d6efd;
  color: white;
  border-radius: 12px;
  border: none;
  font-weight: bold;
}

.win {
  color: green;
  font-weight: bold;
}

.lose {
  color: red;
  font-weight: bold;
}
</style>
