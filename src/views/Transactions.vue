<template>
  <div class="transactions-wrapper">
    <h2 class="title">المعاملات</h2>

    <div v-if="loading" class="loading">جاري التحميل...</div>

    <div v-else>
      <div v-if="transactions.length === 0" class="empty">
        <p>لا توجد معاملات</p>
        <p class="uid-info">UID الحالي: {{ currentUserId }}</p>
        <button @click="createTestTransaction" class="test-btn">
          إنشاء معاملة تجريبية
        </button>
      </div>

      <div v-else>
        <p class="count-info">عدد المعاملات: {{ transactions.length }}</p>
        
        <div
          v-for="tx in transactions"
          :key="tx.id"
          class="tx-card"
        >
          <div class="row">
            <span class="label">المعرف</span>
            <span class="value">{{ tx.id }}</span>
          </div>

          <div class="row">
            <span class="label">النوع</span>
            <span class="value">{{ typeLabel(tx.type) }}</span>
          </div>

          <div class="row">
            <span class="label">المبلغ</span>
            <span class="value">{{ tx.amount }} USDT</span>
          </div>

          <div class="row">
            <span class="label">الحالة</span>
            <span :class="['status', tx.status]">
              {{ statusLabel(tx.status) }}
            </span>
          </div>

          <div class="row">
            <span class="label">التاريخ</span>
            <span class="value">{{ formatDate(tx.createdAt) }}</span>
          </div>

          <div v-if="tx.userId" class="row">
            <span class="label">User ID</span>
            <span class="value uid">{{ tx.userId }}</span>
          </div>

          <div
            v-if="tx.status === 'rejected' && tx.reason"
            class="reject-box"
          >
            <strong>سبب الرفض:</strong>
            <div>{{ tx.reason }}</div>
          </div>

          <div
            v-if="tx.adminMessage"
            class="admin-box"
          >
            <strong>رسالة الإدارة:</strong>
            <div>{{ tx.adminMessage }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { auth, db } from "../firebase";
import {
  collection,
  query,
  where,
  orderBy,
  getDocs,
  addDoc,
  serverTimestamp,
  onSnapshot
} from "firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";

export default {
  name: "Transactions",

  data() {
    return {
      loading: true,
      transactions: [],
      currentUserId: "",
      unsubscribe: null
    };
  },

  created() {
    this.loadTransactions();
  },

  beforeUnmount() {
    // تنظيف الاشتراك عند الخروج من الصفحة
    if (this.unsubscribe) {
      this.unsubscribe();
    }
  },

  methods: {
    async loadTransactions() {
      // الاستماع لتغير حالة المصادقة
      onAuthStateChanged(auth, async (user) => {
        if (!user) {
          console.log("❌ لا يوجد مستخدم مسجل دخول");
          this.loading = false;
          this.currentUserId = "";
          return;
        }

        this.currentUserId = user.uid;
        console.log("✅ مستخدم مسجل دخول:", {
          uid: user.uid,
          email: user.email
        });

        // تنظيف الاشتراك السابق إذا كان موجوداً
        if (this.unsubscribe) {
          this.unsubscribe();
        }

        try {
          // إنشاء استعلام بسيط وصحيح
          const q = query(
            collection(db, "transactions"),
            where("userId", "==", user.uid),
            orderBy("createdAt", "desc")
          );

          // استخدام onSnapshot للاستماع للتحديثات الفورية
          this.unsubscribe = onSnapshot(
            q,
            (snapshot) => {
              console.log("📥 تم استقبال بيانات من Firestore");
              
              if (snapshot.empty) {
                console.log("📭 الاستعلام عاد فارغاً، لا توجد معاملات لهذا المستخدم");
                this.transactions = [];
              } else {
                const transactionsData = [];
                snapshot.forEach((doc) => {
                  transactionsData.push({
                    id: doc.id,
                    ...doc.data()
                  });
                });
                
                console.log(`✅ تم تحميل ${transactionsData.length} معاملة`);
                console.log("📋 تفاصيل المعاملات:", transactionsData);
                
                this.transactions = transactionsData;
              }
              
              this.loading = false;
            },
            (error) => {
              console.error("❌ خطأ في الاستماع للمعاملات:", error);
              console.error("تفاصيل الخطأ:", error.code, error.message);
              
              // إذا كان الخطأ بسبب مشكلة في القواعد أو الاستعلام
              if (error.code === 'permission-denied') {
                alert("خطأ في الصلاحيات! تحقق من قواعد Firebase.");
              } else if (error.code === 'failed-precondition') {
                console.log("⚠️ تحتاج إلى إنشاء فهرس في Firebase Console");
                alert("تحتاج إلى إنشاء فهرس في Firebase Console. تحقق من رسائل الخطأ في Console.");
              }
              
              this.loading = false;
            }
          );

        } catch (error) {
          console.error("❌ خطأ في تحميل المعاملات:", error);
          console.error("Stack trace:", error.stack);
          this.loading = false;
        }
      });
    },

    // دالة لإنشاء معاملة تجريبية
    async createTestTransaction() {
      try {
        const user = auth.currentUser;
        if (!user) {
          alert("يجب تسجيل الدخول أولاً");
          return;
        }

        const transactionData = {
          userId: user.uid,
          type: "deposit",
          amount: 100 + Math.floor(Math.random() * 900), // مبلغ عشوائي
          status: "pending",
          createdAt: serverTimestamp(),
          reason: "",
          adminMessage: "تم إنشاؤها للاختبار"
        };

        console.log("📝 جاري إنشاء معاملة تجريبية:", transactionData);

        const docRef = await addDoc(collection(db, "transactions"), transactionData);
        
        console.log("✅ تم إنشاء معاملة جديدة:", docRef.id);
        alert(`تم إنشاء معاملة تجريبية بنجاح! الرقم: ${docRef.id}`);
        
        // سيتم تحديث القائمة تلقائياً عبر onSnapshot

      } catch (error) {
        console.error("❌ خطأ في إنشاء المعاملة:", error);
        alert("خطأ: " + error.message);
      }
    },

    typeLabel(type) {
      const types = {
        recharge: "تعبئة رصيد",
        withdraw: "سحب رصيد",
        deposit: "إيداع",
        vip: "VIP"
      };
      return types[type] || type;
    },

    statusLabel(status) {
      const statuses = {
        pending: "قيد الانتظار",
        approved: "موافق",
        rejected: "مرفوض"
      };
      return statuses[status] || status;
    },

    formatDate(ts) {
      if (!ts) return "غير متوفر";
      
      try {
        let date;
        if (ts.toDate) {
          date = ts.toDate();
        } else if (ts.seconds) {
          date = new Date(ts.seconds * 1000);
        } else {
          date = new Date(ts);
        }
        
        return date.toLocaleString("ar-EG", {
          year: 'numeric',
          month: 'long',
          day: 'numeric',
          hour: '2-digit',
          minute: '2-digit'
        });
      } catch (error) {
        console.error("خطأ في تنسيق التاريخ:", error, ts);
        return "تاريخ غير صالح";
      }
    },
  },
};
</script>

<style scoped>
.transactions-wrapper {
  padding: 16px;
  min-height: 100vh;
  direction: rtl;
  background: linear-gradient(#0d6efd, #6bb4ff);
}

.title {
  text-align: center;
  color: white;
  margin-bottom: 16px;
  font-size: 24px;
}

.loading,
.empty {
  text-align: center;
  color: white;
  margin-top: 40px;
  font-size: 18px;
}

.empty {
  background: rgba(255, 255, 255, 0.1);
  padding: 30px;
  border-radius: 16px;
  margin-top: 30px;
}

.uid-info {
  font-size: 12px;
  opacity: 0.8;
  margin: 10px 0;
  direction: ltr;
  word-break: break-all;
  background: rgba(0, 0, 0, 0.2);
  padding: 8px;
  border-radius: 8px;
}

.count-info {
  color: white;
  text-align: center;
  margin-bottom: 15px;
  font-weight: bold;
}

.test-btn {
  background: #4CAF50;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  margin-top: 15px;
  font-size: 14px;
}

.test-btn:hover {
  background: #45a049;
}

.tx-card {
  background: #ffffffee;
  padding: 18px;
  border-radius: 16px;
  margin-bottom: 16px;
  color: #000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  padding: 4px 0;
  border-bottom: 1px solid #f0f0f0;
}

.row:last-child {
  border-bottom: none;
}

.label {
  font-size: 14px;
  color: #666;
}

.value {
  font-weight: bold;
  color: #333;
}

.value.uid {
  font-size: 11px;
  color: #888;
  direction: ltr;
  word-break: break-all;
  max-width: 60%;
  text-align: left;
}

.status {
  font-weight: bold;
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 13px;
}

.status.pending {
  color: #ff9800;
  background-color: #fff3e0;
  border: 1px solid #ff9800;
}

.status.approved {
  color: #2e7d32;
  background-color: #e8f5e9;
  border: 1px solid #2e7d32;
}

.status.rejected {
  color: #d32f2f;
  background-color: #ffebee;
  border: 1px solid #d32f2f;
}

.reject-box {
  background: #ffe5e5;
  padding: 12px;
  border-radius: 10px;
  margin-top: 10px;
  color: #b00020;
  font-size: 13px;
  border-right: 4px solid #b00020;
}

.admin-box {
  background: #e3f2fd;
  padding: 12px;
  border-radius: 10px;
  margin-top: 10px;
  font-size: 13px;
  border-right: 4px solid #2196F3;
}
</style>
