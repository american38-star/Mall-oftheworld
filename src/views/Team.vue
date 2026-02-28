<template>
  <div class="team-page">
    <!-- حالة التحميل -->
    <div v-if="loading" class="loading-box">جاري تحميل بيانات الفريق...</div>

    <!-- حالة الخطأ -->
    <div v-if="error" class="error-box">
      حدث خطأ أثناء جلب بيانات الفريق:<br />
      <strong>{{ error }}</strong>
    </div>

    <!-- كود الدعوة -->
    <div class="invite-section" v-if="!loading && !error">
      <h2>فريقك</h2>

      <div class="ref-box">
        <label>كود الإحالة:</label>
        <div class="ref-code">{{ referralCode || "غير متوفر" }}</div>
        <button @click="copyText(referralCode)">نسخ</button>
      </div>

      <div class="ref-box">
        <label>رابط الدعوة:</label>
        <div class="ref-code">{{ inviteLink || "غير متوفر" }}</div>
        <button @click="copyText(inviteLink)">نسخ</button>
      </div>

      <!-- أزرار المشاركة - واتساب وتليجرام -->
      <div class="share-buttons">
        <button class="share-btn whatsapp" @click="shareViaWhatsApp">
          <i class="fab fa-whatsapp"></i>
          مشاركة عبر واتساب
        </button>
        <button class="share-btn telegram" @click="shareViaTelegram">
          <i class="fab fa-telegram"></i>
          مشاركة عبر تليجرام
        </button>
      </div>
    </div>

    <!-- إحصائيات الفريق -->
    <div class="team-stats-box" v-if="!loading && !error">
      <div class="stat-row">
        <div class="stat-item">
          <div class="stat-title">انسحاب الفريق</div>
          <div class="stat-value">${{ teamStats.withdraw }}</div>
        </div>

        <div class="stat-item">
          <div class="stat-title">إعادة شحن الفريق</div>
          <div class="stat-value">${{ teamStats.recharge }}</div>
        </div>

        <div class="stat-item">
          <div class="stat-title">حجم الفريق</div>
          <div class="stat-value">{{ teamStats.totalMembers }}</div>
        </div>
      </div>

      <div class="stat-row">
        <div class="stat-item">
          <div class="stat-title">الانسحاب الأول</div>
          <div class="stat-value">{{ teamStats.firstWithdraw }}</div>
        </div>

        <div class="stat-item">
          <div class="stat-title">الشحن لأول مرة</div>
          <div class="stat-value">{{ teamStats.firstRecharge }}</div>
        </div>

        <div class="stat-item">
          <div class="stat-title">فريق جديد</div>
          <div class="stat-value">{{ teamStats.newMembers }}</div>
        </div>
      </div>
    </div>

    <!-- المستويات -->
    <div class="levels-container" v-if="!loading && !error">
      <div class="level-card level1">
        <div class="lvl-header">مستوى 1</div>
        <div class="lvl-body">
          <div>عدد الإحالات: <strong>{{ stats.l1.count }}</strong></div>
          <div>العمولة: <strong>6%</strong></div>
          <div>الدخل: <strong>{{ stats.l1.earnings.toFixed(2) }} USDT</strong></div>
        </div>
      </div>

      <div class="level-card level2">
        <div class="lvl-header">مستوى 2</div>
        <div class="lvl-body">
          <div>عدد الإحالات: <strong>{{ stats.l2.count }}</strong></div>
          <div>العمولة: <strong>2%</strong></div>
          <div>الدخل: <strong>{{ stats.l2.earnings.toFixed(2) }} USDT</strong></div>
        </div>
      </div>

      <div class="level-card level3">
        <div class="lvl-header">مستوى 3</div>
        <div class="lvl-body">
          <div>عدد الإحالات: <strong>{{ stats.l3.count }}</strong></div>
          <div>العمولة: <strong>1%</strong></div>
          <div>الدخل: <strong>{{ stats.l3.earnings.toFixed(2) }} USDT</strong></div>
        </div>
      </div>
    </div>

    <button class="btn-back" @click="$router.push('/home')">عودة</button>
  </div>
</template>

<script>
import { auth, db } from "../firebase";
import {
  doc,
  getDoc,
  collection,
  query,
  where,
  getDocs
} from "firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";

export default {
  name: "Team",
  data() {
    return {
      referralCode: "",
      inviteLink: "",
      loading: true,
      error: null,

      userIdFieldInLogs: "userId",
      txIdFieldInLogs: "txid",

      teamStats: {
        withdraw: "0.00",
        recharge: "0.00",
        totalMembers: 0,
        newMembers: 0,
        firstRecharge: 0,
        firstWithdraw: 0,
      },

      stats: {
        l1: { count: 0, earnings: 0 },
        l2: { count: 0, earnings: 0 },
        l3: { count: 0, earnings: 0 },
      },
    };
  },

  created() {
    onAuthStateChanged(auth, async (user) => {
      if (!user) {
        this.loading = false;
        return;
      }

      try {
        const uid = user.uid;
        const udoc = await getDoc(doc(db, "users", uid));
        if (udoc.exists()) {
          const data = udoc.data();
          this.referralCode = data.referralCode || uid.substring(0, 6);
          this.inviteLink = `${window.location.origin}/register?ref=${this.referralCode}`;
        } else {
          this.referralCode = uid.substring(0, 6);
          this.inviteLink = `${window.location.origin}/register?ref=${this.referralCode}`;
        }

        await this.loadTeamLevels(uid);
        await this.loadTeamStats(uid);
      } catch (err) {
        console.error("Team load error:", err);
        this.error = err.message || String(err);
      } finally {
        this.loading = false;
      }
    });
  },

  methods: {
    copyText(text) {
      if (!text) {
        alert("لا يوجد شيء للنسخ");
        return;
      }
      navigator.clipboard
        .writeText(text)
        .then(() => alert("تم النسخ"))
        .catch(() => alert("فشل النسخ — انسخ يدويًا"));
    },

    // دالة مشاركة عبر واتساب
    shareViaWhatsApp() {
      const message = `انضم إلى فريقي في Mall of the World باستخدام رابط الدعوة الخاص بي: ${this.inviteLink}`;
      const whatsappUrl = `https://wa.me/?text=${encodeURIComponent(message)}`;
      window.open(whatsappUrl, '_blank');
    },

    // دالة مشاركة عبر تليجرام
    shareViaTelegram() {
      const message = `انضم إلى فريقي في Mall of the World باستخدام رابط الدعوة الخاص بي: ${this.inviteLink}`;
      const telegramUrl = `https://t.me/share/url?url=${encodeURIComponent(this.inviteLink)}&text=${encodeURIComponent(message)}`;
      window.open(telegramUrl, '_blank');
    },

    chunkArray(arr, size = 10) {
      const res = [];
      for (let i = 0; i < arr.length; i += size) {
        res.push(arr.slice(i, i + size));
      }
      return res;
    },

    async loadTeamLevels(uid) {
      try {
        const usersRef = collection(db, "users");

        // L1
        const q1 = query(usersRef, where("invitedBy", "==", uid));
        const s1 = await getDocs(q1);
        const level1Ids = s1.docs.map((d) => d.id);
        this.stats.l1.count = level1Ids.length;

        // L2
        let level2Ids = [];
        if (level1Ids.length) {
          const chunks = this.chunkArray(level1Ids, 10);
          for (const ch of chunks) {
            const q2 = query(usersRef, where("invitedBy", "in", ch));
            const s2 = await getDocs(q2);
            level2Ids = level2Ids.concat(s2.docs.map((d) => d.id));
          }
        }
        this.stats.l2.count = level2Ids.length;

        // L3
        let level3Ids = [];
        if (level2Ids.length) {
          const chunks2 = this.chunkArray(level2Ids, 10);
          for (const ch of chunks2) {
            const q3 = query(usersRef, where("invitedBy", "in", ch));
            const s3 = await getDocs(q3);
            level3Ids = level3Ids.concat(s3.docs.map((d) => d.id));
          }
        }
        this.stats.l3.count = level3Ids.length;

        const allIds = [...level1Ids, ...level2Ids, ...level3Ids];
        const uniqueIds = Array.from(new Set(allIds));
        this.teamStats.totalMembers = uniqueIds.length;
        this.teamStats.newMembers = level1Ids.length;

        await this.loadReferralRewards(uid);
      } catch (err) {
        console.error("loadTeamLevels error:", err);
        throw err;
      }
    },

    async loadReferralRewards(uid) {
      try {
        const rewardsRef = collection(db, "referral_rewards");
        const calc = async (level) => {
          const q = query(rewardsRef, where("receiver", "==", uid), where("level", "==", level));
          const s = await getDocs(q);
          return s.docs.reduce((sum, d) => sum + Number(d.data().amount || 0), 0);
        };

        this.stats.l1.earnings = await calc(1);
        this.stats.l2.earnings = await calc(2);
        this.stats.l3.earnings = await calc(3);
      } catch (err) {
        console.warn("loadReferralRewards warning:", err);
        this.stats.l1.earnings = 0;
        this.stats.l2.earnings = 0;
        this.stats.l3.earnings = 0;
      }
    },

    async loadTeamStats(uid) {
      try {
        const usersRef = collection(db, "users");

        const q1 = query(usersRef, where("invitedBy", "==", uid));
        const s1 = await getDocs(q1);
        const level1Members = s1.docs.map((d) => d.id);

        let level2Members = [];
        if (level1Members.length) {
          for (const ch of this.chunkArray(level1Members, 10)) {
            const q2 = query(usersRef, where("invitedBy", "in", ch));
            const s2 = await getDocs(q2);
            level2Members = level2Members.concat(s2.docs.map((d) => d.id));
          }
        }

        let level3Members = [];
        if (level2Members.length) {
          for (const ch of this.chunkArray(level2Members, 10)) {
            const q3 = query(usersRef, where("invitedBy", "in", ch));
            const s3 = await getDocs(q3);
            level3Members = level3Members.concat(s3.docs.map((d) => d.id));
          }
        }

        const all = [...level1Members, ...level2Members, ...level3Members];
        const membersUnique = Array.from(new Set(all));

        let withdrawSum = 0;
        let rechargeSum = 0;
        let firstWithdrawCount = 0;
        let firstRechargeCount = 0;

        const seenRechargeTx = new Set();
        const seenWithdrawTx = new Set();

        const uidField = this.userIdFieldInLogs;
        const txField = this.txIdFieldInLogs;

        for (const memberId of membersUnique) {
          // withdraw_logs
          const withdrawQ = query(collection(db, "withdraw_logs"), where(uidField, "==", memberId));
          const wSnap = await getDocs(withdrawQ);

          let memberHadWithdraw = false;
          wSnap.forEach((d) => {
            const data = d.data() || {};
            if (data.type && String(data.type).toLowerCase() !== "approved") return;
            if (data.status && String(data.status).toLowerCase() !== "approved") return;

            const key = (data[txField] && String(data[txField])) || d.id;
            if (seenWithdrawTx.has(key)) return;
            seenWithdrawTx.add(key);

            const amt = Number(data.amount || 0);
            if (!isNaN(amt) && amt !== 0) {
              withdrawSum += amt;
              memberHadWithdraw = true;
            }
          });
          if (memberHadWithdraw) firstWithdrawCount++;

          // recharge_logs
          const rechargeQ = query(collection(db, "recharge_logs"), where(uidField, "==", memberId));
          const rSnap = await getDocs(rechargeQ);

          let memberHadRecharge = false;
          rSnap.forEach((d) => {
            const data = d.data() || {};
            if (data.type && String(data.type).toLowerCase() !== "approved") return;
            if (data.status && String(data.status).toLowerCase() !== "approved") return;

            const key = (data[txField] && String(data[txField])) || d.id;
            if (seenRechargeTx.has(key)) return;
            seenRechargeTx.add(key);

            const amt = Number(data.amount || 0);
            if (!isNaN(amt) && amt !== 0) {
              rechargeSum += amt;
              memberHadRecharge = true;
            }
          });
          if (memberHadRecharge) firstRechargeCount++;
        }

        this.teamStats.withdraw = parseFloat(withdrawSum || 0).toFixed(2);
        this.teamStats.recharge = parseFloat(rechargeSum || 0).toFixed(2);
        this.teamStats.firstWithdraw = firstWithdrawCount;
        this.teamStats.firstRecharge = firstRechargeCount;
        this.teamStats.totalMembers = membersUnique.length;
      } catch (err) {
        console.error("loadTeamStats error:", err);
        this.error = err.message || String(err);
      }
    },
  },
};
</script>

<style scoped>
.team-page {
  direction: rtl;
  padding: 12px;
  background: #0A0C10;
  min-height: 100vh;
  padding-bottom: 100px; /* مساحة للشريط السفلي */
}

.invite-section {
  background: #11151C;
  padding: 15px;
  border-radius: 12px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  margin-bottom: 20px;
  text-align: center;
}

.invite-section h2 {
  color: #D4AF37;
  margin-bottom: 15px;
}

.ref-box {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 8px 0;
}

.ref-box label {
  color: #D4AF37;
  margin-left: 10px;
}

.ref-code {
  background: #1A1F2A;
  padding: 8px 12px;
  border-radius: 8px;
  margin: 0 8px;
  flex: 1;
  word-break: break-all;
  color: #fff;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.ref-box button {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border: none;
  padding: 8px 15px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
}

/* أزرار المشاركة */
.share-buttons {
  display: flex;
  gap: 10px;
  margin-top: 15px;
  justify-content: center;
}

.share-btn {
  flex: 1;
  padding: 10px;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: all 0.3s ease;
}

.share-btn.whatsapp {
  background: #25D366;
  color: white;
}

.share-btn.telegram {
  background: #0088cc;
  color: white;
}

.share-btn:hover {
  transform: translateY(-2px);
  filter: brightness(1.1);
}

.share-btn i {
  font-size: 18px;
}

.team-stats-box {
  background: #11151C;
  padding: 15px;
  border-radius: 14px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  margin-bottom: 20px;
}

.stat-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
  gap: 10px;
}

.stat-item {
  text-align: center;
  flex: 1;
  background: #1A1F2A;
  padding: 10px 5px;
  border-radius: 10px;
  border: 1px solid rgba(212, 175, 55, 0.1);
}

.stat-title {
  color: rgba(255, 255, 255, 0.6);
  font-size: 13px;
  margin-bottom: 4px;
}

.stat-value {
  font-size: 15px;
  font-weight: bold;
  color: #D4AF37;
}

.levels-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 12px;
  margin-bottom: 20px;
}

.level-card {
  border-radius: 14px;
  color: white;
  padding: 12px;
  min-height: 140px;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.level1 {
  background: linear-gradient(135deg, #D4AF37, #C5A028);
}

.level2 {
  background: linear-gradient(135deg, #C5A028, #B8962E);
}

.level3 {
  background: linear-gradient(135deg, #B8962E, #A47C1E);
}

.lvl-header {
  font-weight: bold;
  font-size: 16px;
  margin-bottom: 10px;
  color: #0A0C10;
  border-bottom: 1px solid rgba(0,0,0,0.1);
  padding-bottom: 5px;
}

.lvl-body {
  font-size: 13px;
  color: #0A0C10;
}

.lvl-body div {
  margin: 5px 0;
}

.lvl-body strong {
  color: #0A0C10;
}

.btn-back {
  margin-top: 20px;
  width: 100%;
  padding: 12px;
  border-radius: 12px;
  border: 2px solid #D4AF37;
  background: transparent;
  color: #D4AF37;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-back:hover {
  background: #D4AF37;
  color: #0A0C10;
}

.loading-box,
.error-box {
  background: #11151C;
  margin: 16px;
  padding: 14px;
  border-radius: 12px;
  text-align: center;
  border: 1px solid rgba(212, 175, 55, 0.2);
  color: #fff;
}

.error-box {
  color: #ff6b6b;
}

/* تحسينات للهواتف */
@media (max-width: 480px) {
  .share-buttons {
    flex-direction: column;
    gap: 8px;
  }
  
  .share-btn {
    width: 100%;
  }
}
</style>
