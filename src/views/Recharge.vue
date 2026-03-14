<template>
  <div class="recharge-page">
    <div class="card">
      <div class="card-header">
        <h2 class="title">
          <i class="fas fa-coins"></i>
          تعبئة الرصيد
          <span class="title-glow">USDT</span>
        </h2>
        <div class="header-glow"></div>
      </div>

      <!-- اختيار الشبكة -->
      <div class="field network-field">
        <label>
          <i class="fas fa-network-wired"></i>
          اختر الشبكة
        </label>
        <div class="network-row">
          <div class="usdt-badge">
            <span>₮</span>
          </div>
          <select v-model="network" class="gold-select">
            <option value="TRC20">TRC20 (Tron)</option>
            <option value="ERC20">ERC20 (Ethereum)</option>
            <option value="BEP20">BEP20 (BSC)</option>
            <option value="SOL">SOL (Solana)</option>
          </select>
        </div>
      </div>

      <!-- QR Code -->
      <div class="qr-box">
        <div class="qr-glow"></div>
        <img :src="getQr(network)" alt="QR Code" class="qr" />
        <div class="qr-label">امسح للدفع</div>
      </div>

      <!-- عنوان المحفظة -->
      <div class="field">
        <label>
          <i class="fas fa-wallet"></i>
          عنوان المحفظة
        </label>
        <div class="address-row">
          <input
            class="gold-input address-input"
            :value="getAddress(network)"
            readonly
            ref="addrInput"
          />
          <button class="copy-btn" @click="copyAddress" :title="copied ? 'تم النسخ!' : 'نسخ'">
            <i class="fas" :class="copied ? 'fa-check' : 'fa-copy'"></i>
          </button>
        </div>
        <div v-if="copied" class="copy-msg">
          <i class="fas fa-check-circle"></i>
          تم نسخ العنوان بنجاح
        </div>
      </div>

      <!-- المبلغ -->
      <div class="field">
        <label>
          <i class="fas fa-calculator"></i>
          المبلغ (USDT)
        </label>
        <div class="amount-input-wrapper">
          <input
            class="gold-input amount-input"
            type="number"
            v-model.number="amount"
            placeholder="0.00"
            min="0"
            step="0.01"
          />
          <span class="input-currency">USDT</span>
        </div>
      </div>

      <!-- TxID -->
      <div class="field">
        <label>
          <i class="fas fa-hashtag"></i>
          معرف التحويل (TxID)
        </label>
        <input
          class="gold-input"
          type="text"
          v-model="txid"
          placeholder="أدخل TxID الخاص بالتحويل"
        />
        <div class="field-hint">
          <i class="fas fa-info-circle"></i>
          يمكنك العثور على TxID في محفظتك بعد إتمام التحويل
        </div>
      </div>

      <!-- زر الإرسال -->
      <button class="gold-button" @click="submit" :disabled="loading">
        <i class="fas" :class="loading ? 'fa-spinner fa-spin' : 'fa-paper-plane'"></i>
        {{ loading ? "جاري الإرسال..." : "إرسال طلب التعبئة" }}
      </button>

      <!-- رسالة -->
      <transition name="fade">
        <div
          v-if="message"
          class="message"
          :class="messageType"
        >
          <i class="fas" :class="messageType === 'error' ? 'fa-exclamation-circle' : 'fa-check-circle'"></i>
          {{ message }}
        </div>
      </transition>

      <!-- معلومات إضافية -->
      <div class="info-panel">
        <div class="info-item">
          <i class="fas fa-clock"></i>
          <span>مدة المعالجة: 5-30 دقيقة</span>
        </div>
        <div class="info-item">
          <i class="fas fa-shield-alt"></i>
          <span>جميع المعاملات آمنة ومشفرة</span>
        </div>
        <div class="info-item">
          <i class="fas fa-headset"></i>
          <span>الدعم الفني متاح 24/7</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { getAuth } from "firebase/auth";
import { collection, addDoc, serverTimestamp } from "firebase/firestore";
import { db } from "../firebase";

export default {
  name: "Recharge",

  data() {
    return {
      network: "TRC20",
      amount: null,
      txid: "",
      copied: false,
      loading: false,
      message: "",
      messageType: "info",

      addresses: {
        TRC20: "TC9YKMxacLkvJLA6JbQVGofnjumTqkaUjD",
        ERC20: "0x17e488f699CC96E4dd05e4E2c869789534E2F634",
        BEP20: "0x17e488f699CC96E4dd05e4E2c869789534E2F634",
        SOL: "GgVi3xNRUQeJrNqt8Hg3eBJx5B8yuyggjNvvHxxKVnTR",
      },

      userEmail: "",
      userId: "",
    };
  },

  created() {
    const auth = getAuth();
    auth.onAuthStateChanged((u) => {
      if (u) {
        this.userEmail = u.email;
        this.userId = u.uid;
      }
    });
  },

  methods: {
    getQr(net) {
      return `/qr/${net}.png`;
    },

    getAddress(net) {
      return this.addresses[net] || "";
    },

    async copyAddress() {
      const text = this.getAddress(this.network);
      if (!text) return;

      try {
        if (navigator.clipboard && navigator.clipboard.writeText) {
          await navigator.clipboard.writeText(text);
        } else {
          const input = this.$refs.addrInput;
          input.removeAttribute("readonly");
          input.select();
          document.execCommand("copy");
          input.setAttribute("readonly", true);
        }

        this.copied = true;
        setTimeout(() => (this.copied = false), 2000);
      } catch (err) {
        alert("فشل النسخ — قم بالنسخ يدوياً.");
      }
    },

    async submit() {
      this.message = "";

      if (!this.userId) {
        this.message = "يرجى تسجيل الدخول أولاً";
        this.messageType = "error";
        return;
      }

      if (!this.amount || this.amount <= 0) {
        this.message = "الرجاء إدخال مبلغ صالح";
        this.messageType = "error";
        return;
      }

      if (!this.txid || this.txid.trim() === "") {
        this.message = "الرجاء إدخال معرف التحويل (TxID)";
        this.messageType = "error";
        return;
      }

      this.loading = true;

      try {
        // حفظ في payments
        await addDoc(collection(db, "payments"), {
          userId: this.userId,
          email: this.userEmail,
          amount: this.amount,
          txid: this.txid,
          network: this.network,
          status: "pending",
          createdAt: serverTimestamp(),
        });

        // حفظ في transactions
        await addDoc(collection(db, "transactions"), {
          userId: this.userId,
          email: this.userEmail,
          type: "recharge",
          amount: this.amount,
          network: this.network,
          txid: this.txid,
          status: "pending",
          reason: "",
          adminMessage: "",
          createdAt: serverTimestamp(),
        });

        this.message = "✅ تم إرسال طلب التعبئة بنجاح. سيتم المراجعة خلال 5-30 دقيقة";
        this.messageType = "success";

        this.amount = null;
        this.txid = "";
      } catch (err) {
        console.error(err);
        this.message = "❌ حدث خطأ أثناء الإرسال. يرجى المحاولة مرة أخرى";
        this.messageType = "error";
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.recharge-page {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: #0A0C10;
  padding: 20px;
  direction: rtl;
  font-family: 'Cairo', sans-serif;
}

/* البطاقة الرئيسية */
.card {
  width: 100%;
  max-width: 480px;
  background: #11151C;
  padding: 30px;
  border-radius: 30px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5), 0 0 0 1px rgba(212, 175, 55, 0.1);
  position: relative;
  overflow: hidden;
}

.card::before {
  content: '';
  position: absolute;
  top: -50%;
  right: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.03) 0%, transparent 70%);
  animation: rotate 30s linear infinite;
  pointer-events: none;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* رأس البطاقة */
.card-header {
  position: relative;
  margin-bottom: 25px;
  text-align: center;
}

.title {
  font-size: 28px;
  font-weight: 800;
  color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  position: relative;
  z-index: 1;
}

.title i {
  color: #D4AF37;
  font-size: 32px;
}

.title-glow {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 900;
}

.header-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 200px;
  height: 200px;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.1) 0%, transparent 70%);
  filter: blur(30px);
  z-index: 0;
}

/* الحقول */
.field {
  margin-bottom: 25px;
}

.field label {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #D4AF37;
  font-size: 15px;
  font-weight: 600;
  margin-bottom: 10px;
}

.field label i {
  font-size: 16px;
}

/* شبكة العملات */
.network-row {
  display: flex;
  align-items: center;
  gap: 15px;
}

.usdt-badge {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  font-weight: 900;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #0A0C10;
  font-size: 20px;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
}

.gold-select {
  flex: 1;
  padding: 14px 20px;
  border-radius: 16px;
  background: #1A1F2A;
  color: #ffffff;
  border: 2px solid rgba(212, 175, 55, 0.3);
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  appearance: none;
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='%23D4AF37'%3e%3cpath d='M7 10l5 5 5-5z'/%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: left 15px center;
  background-size: 20px;
}

.gold-select:hover, .gold-select:focus {
  border-color: #D4AF37;
  outline: none;
  box-shadow: 0 0 20px rgba(212, 175, 55, 0.2);
}

.gold-select option {
  background: #1A1F2A;
  color: #ffffff;
}

/* QR Code */
.qr-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 25px 0;
  position: relative;
}

.qr-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 180px;
  height: 180px;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.2) 0%, transparent 70%);
  filter: blur(20px);
  border-radius: 50%;
}

.qr {
  width: 160px;
  height: 160px;
  border-radius: 20px;
  border: 3px solid #D4AF37;
  padding: 5px;
  background: #ffffff;
  position: relative;
  z-index: 1;
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.3);
}

.qr-label {
  margin-top: 10px;
  color: #D4AF37;
  font-size: 14px;
  font-weight: 600;
}

/* عنوان المحفظة */
.address-row {
  display: flex;
  gap: 10px;
  align-items: center;
}

.gold-input {
  flex: 1;
  padding: 14px 20px;
  border-radius: 16px;
  background: #1A1F2A;
  color: #ffffff;
  border: 2px solid rgba(212, 175, 55, 0.3);
  font-size: 14px;
  transition: all 0.3s ease;
  font-family: monospace;
}

.gold-input:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 20px rgba(212, 175, 55, 0.2);
}

.gold-input::placeholder {
  color: rgba(255, 255, 255, 0.3);
}

.address-input {
  font-size: 13px;
  letter-spacing: 0.5px;
}

.copy-btn {
  width: 50px;
  height: 50px;
  border-radius: 16px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  border: none;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
}

.copy-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 8px 25px rgba(212, 175, 55, 0.4);
}

.copy-msg {
  margin-top: 8px;
  color: #22c55e;
  font-size: 13px;
  display: flex;
  align-items: center;
  gap: 5px;
  background: rgba(34, 197, 94, 0.1);
  padding: 8px 12px;
  border-radius: 12px;
  border: 1px solid #22c55e;
}

.copy-msg i {
  font-size: 14px;
}

/* حقل المبلغ */
.amount-input-wrapper {
  position: relative;
}

.amount-input {
  padding-left: 80px !important;
}

.input-currency {
  position: absolute;
  left: 20px;
  top: 50%;
  transform: translateY(-50%);
  color: #D4AF37;
  font-weight: 700;
  font-size: 15px;
  background: rgba(212, 175, 55, 0.1);
  padding: 4px 12px;
  border-radius: 20px;
  border: 1px solid rgba(212, 175, 55, 0.3);
}

/* تلميحات الحقل */
.field-hint {
  margin-top: 8px;
  color: rgba(255, 255, 255, 0.5);
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 5px;
}

.field-hint i {
  color: #D4AF37;
  font-size: 13px;
}

/* الزر الرئيسي */
.gold-button {
  width: 100%;
  padding: 16px;
  border-radius: 20px;
  border: none;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  font-size: 18px;
  font-weight: 800;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin: 20px 0;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.gold-button:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.4);
}

.gold-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  filter: grayscale(20%);
}

/* الرسائل */
.message {
  margin-top: 15px;
  padding: 15px 20px;
  border-radius: 16px;
  font-size: 14px;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 10px;
  border: 1px solid;
}

.message.success {
  background: rgba(34, 197, 94, 0.1);
  color: #22c55e;
  border-color: #22c55e;
}

.message.error {
  background: rgba(239, 68, 68, 0.1);
  color: #ef4444;
  border-color: #ef4444;
}

.message i {
  font-size: 18px;
}

/* لوحة المعلومات */
.info-panel {
  margin-top: 25px;
  padding: 15px;
  background: #1A1F2A;
  border-radius: 16px;
  border: 1px solid rgba(212, 175, 55, 0.1);
}

.info-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 0;
  color: rgba(255, 255, 255, 0.7);
  font-size: 13px;
  border-bottom: 1px solid rgba(212, 175, 55, 0.1);
}

.info-item:last-child {
  border-bottom: none;
}

.info-item i {
  color: #D4AF37;
  width: 20px;
  font-size: 14px;
}

/* حركة الرسائل */
.fade-enter-active, .fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

/* تحسينات للجوال */
@media (max-width: 480px) {
  .recharge-page {
    padding: 15px;
  }

  .card {
    padding: 20px;
  }

  .title {
    font-size: 24px;
  }

  .title i {
    font-size: 28px;
  }

  .qr {
    width: 140px;
    height: 140px;
  }

  .gold-select {
    padding: 12px 15px;
    font-size: 14px;
  }

  .gold-input {
    padding: 12px 15px;
    font-size: 13px;
  }

  .copy-btn {
    width: 45px;
    height: 45px;
    font-size: 18px;
  }

  .input-currency {
    left: 15px;
    padding: 3px 10px;
    font-size: 13px;
  }

  .gold-button {
    padding: 14px;
    font-size: 16px;
  }

  .info-panel {
    padding: 12px;
  }

  .info-item {
    font-size: 12px;
  }
}

/* تخصيص شريط التمرير */
.address-input::-webkit-scrollbar {
  height: 4px;
}

.address-input::-webkit-scrollbar-track {
  background: #1A1F2A;
  border-radius: 4px;
}

.address-input::-webkit-scrollbar-thumb {
  background: #D4AF37;
  border-radius: 4px;
}

/* إزالة أزرار الزيادة/النقص من input number */
input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type=number] {
  -moz-appearance: textfield;
}
</style>
