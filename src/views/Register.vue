<template>
  <div class="container">
    <div class="card">
      <h2 class="title">إنشاء حساب</h2>

      <!-- البريد الإلكتروني -->
      <label class="label">البريد الإلكتروني</label>
      <input
        type="email"
        v-model="email"
        placeholder="البريد الإلكتروني"
        class="input"
      />

      <!-- رقم الهاتف (إضافة جديدة) -->
      <label class="label">رقم الهاتف (اختياري)</label>
      <input
        type="tel"
        v-model="phoneNumber"
        placeholder="رقم الهاتف"
        class="input"
        @input="validatePhoneNumber"
      />
      <span v-if="phoneError" class="error-message">{{ phoneError }}</span>

      <!-- كلمة المرور -->
      <label class="label">كلمة المرور</label>
      <div class="input-box">
        <input
          :type="showPassword ? 'text' : 'password'"
          v-model="password"
          placeholder="كلمة المرور"
          class="input"
        />
        <span class="toggle" @click="togglePassword">
          {{ showPassword ? "إخفاء" : "إظهار" }}
        </span>
      </div>

      <!-- كود الإحالة -->
      <label class="label">كود الإحالة</label>
      <input
        type="text"
        v-model="inviteCode"
        placeholder="كود الإحالة"
        class="input"
      />

      <!-- زر إنشاء الحساب -->
      <button class="btn" @click="registerUser" :disabled="loading">
        <span v-if="!loading">إنشاء حساب</span>
        <span v-else class="loader"></span>
      </button>

      <p class="link">
        لديك حساب؟
        <router-link to="/login">تسجيل الدخول</router-link>
      </p>
    </div>
  </div>
</template>

<script>
import {
  getAuth,
  createUserWithEmailAndPassword,
  browserLocalPersistence,
} from "firebase/auth";

import { db } from "../firebase";
import router from "../router";

import {
  doc,
  setDoc,
  collection,
  query,
  where,
  getDocs,
  serverTimestamp,
  getDoc,
} from "firebase/firestore";

export default {
  name: "RegisterPage",

  data() {
    return {
      email: "",
      phoneNumber: "",
      password: "",
      inviteCode: "",
      showPassword: false,
      loading: false,
      phoneError: "",
    };
  },

  created() {
    const ref = this.$route.query.ref;
    if (ref) {
      this.inviteCode = String(ref).trim();
    }
  },

  methods: {
    togglePassword() {
      this.showPassword = !this.showPassword;
    },

    validateEmail(email) {
      const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return re.test(email);
    },

    validatePassword(password) {
      return password.length >= 6;
    },

    validatePhoneNumber() {
      if (!this.phoneNumber) {
        this.phoneError = "";
        return true;
      }
      
      // التحقق من رقم الهاتف (أرقام فقط، 10-15 رقم)
      const phoneRegex = /^[0-9]{10,15}$/;
      if (!phoneRegex.test(this.phoneNumber.replace(/[^0-9]/g, ''))) {
        this.phoneError = "رقم الهاتف يجب أن يكون بين 10 و 15 رقم";
        return false;
      }
      
      this.phoneError = "";
      return true;
    },

    async registerUser() {
      if (!this.email || !this.password) {
        alert("يرجى تعبئة البريد الإلكتروني وكلمة المرور");
        return;
      }

      if (!this.validateEmail(this.email)) {
        alert("البريد الإلكتروني غير صالح");
        return;
      }

      if (!this.validatePassword(this.password)) {
        alert("كلمة المرور يجب أن تكون 6 أحرف على الأقل");
        return;
      }

      if (this.phoneNumber && !this.validatePhoneNumber()) {
        alert("رقم الهاتف غير صالح");
        return;
      }

      this.loading = true;

      try {
        const auth = getAuth();
        await auth.setPersistence(browserLocalPersistence);

        const userCredential = await createUserWithEmailAndPassword(
          auth,
          this.email.trim(),
          this.password
        );

        const user = userCredential.user;

        let inviterUID = null;
        let level2 = null;
        let level3 = null;

        if (this.inviteCode) {
          const enteredCode = String(this.inviteCode).trim();

          if (enteredCode === user.uid.substring(0, 6)) {
            alert("لا يمكنك استخدام كود الإحالة الخاص بك");
          } else {
            const q = query(
              collection(db, "users"),
              where("referralCode", "==", enteredCode)
            );
            const result = await getDocs(q);

            if (!result.empty) {
              const inviterDoc = result.docs[0];
              inviterUID = inviterDoc.id;

              const inviterData = inviterDoc.data();
              if (inviterData.invitedBy) {
                level2 = inviterData.invitedBy;
              }

              if (level2) {
                const docLevel2 = await getDoc(doc(db, "users", level2));
                if (docLevel2.exists()) {
                  const level2Data = docLevel2.data();
                  if (level2Data.invitedBy) {
                    level3 = level2Data.invitedBy;
                  }
                }
              }
            }
          }
        }

        // تنظيف رقم الهاتف من الرموز غير الرقمية
        const cleanPhoneNumber = this.phoneNumber ? this.phoneNumber.replace(/[^0-9]/g, '') : "";

        await setDoc(doc(db, "users", user.uid), {
          uid: user.uid,
          email: this.email.trim(),
          phoneNumber: cleanPhoneNumber || null, // إضافة رقم الهاتف
          referralCode: user.uid.substring(0, 6),
          invitedBy: inviterUID || null,
          level2: level2 || null,
          level3: level3 || null,
          balance: 0,
          vipLevel: 0,
          blocked: false,
          createdAt: serverTimestamp(),
        });

        router.push("/home");
      } catch (err) {
        console.error("Register error:", err);
        
        // رسائل خطأ مفهومة
        if (err.code === 'auth/email-already-in-use') {
          alert("البريد الإلكتروني مستخدم بالفعل");
        } else if (err.code === 'auth/invalid-email') {
          alert("البريد الإلكتروني غير صالح");
        } else if (err.code === 'auth/weak-password') {
          alert("كلمة المرور ضعيفة - يجب أن تكون 6 أحرف على الأقل");
        } else {
          alert("خطأ: " + (err.message || String(err)));
        }
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px;
  min-height: 100vh;
  background: #0A0C10;
  direction: rtl;
}

/* كرت إنشاء الحساب - رمادي غامق */
.card {
  background: #11151C;
  width: 90%;
  max-width: 380px;
  padding: 35px 25px;
  border-radius: 20px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5), 0 0 0 1px rgba(212, 175, 55, 0.1);
  border: 1px solid rgba(212, 175, 55, 0.2);
  position: relative;
}

/* تأثير الحدود الذهبية */
.card::before {
  content: '';
  position: absolute;
  top: -2px;
  left: -2px;
  right: -2px;
  bottom: -2px;
  background: linear-gradient(135deg, #D4AF37, transparent, #C5A028);
  border-radius: 22px;
  z-index: -1;
  opacity: 0.3;
}

/* العنوان - ذهبي فاخر */
.title {
  text-align: center;
  margin-bottom: 25px;
  font-weight: 700;
  font-size: 28px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: 0 2px 10px rgba(212, 175, 55, 0.3);
}

/* تسميات الحقول */
.label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #D4AF37;
  font-size: 14px;
  letter-spacing: 0.5px;
}

/* حقول الإدخال */
.input {
  width: 100%;
  padding: 14px 12px;
  margin-bottom: 20px;
  border-radius: 12px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  background: #1A1F2A;
  color: #ffffff;
  font-size: 15px;
  transition: all 0.3s ease;
  box-sizing: border-box;
}

.input:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 0 3px rgba(212, 175, 55, 0.1);
  background: #1E2430;
}

.input::placeholder {
  color: rgba(255, 255, 255, 0.3);
}

/* حقل كلمة المرور مع زر الإظهار/الإخفاء */
.input-box {
  position: relative;
  width: 100%;
}

.toggle {
  position: absolute;
  left: 15px;
  top: 14px;
  color: #D4AF37;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.3s ease;
  padding: 2px 8px;
  border-radius: 6px;
  background: rgba(212, 175, 55, 0.1);
}

.toggle:hover {
  background: rgba(212, 175, 55, 0.2);
  color: #F6E27A;
}

/* رسالة خطأ الهاتف */
.error-message {
  color: #ff6b6b;
  font-size: 12px;
  margin-top: -15px;
  margin-bottom: 10px;
  text-align: right;
  display: block;
}

/* زر إنشاء الحساب - ذهبي متدرج */
.btn {
  width: 100%;
  padding: 14px;
  border: none;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border-radius: 12px;
  font-size: 18px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  margin: 10px 0 15px;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #B8962E, #D4AF37, #B8962E);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(212, 175, 55, 0.4);
}

.btn:active:not(:disabled) {
  transform: translateY(0);
}

.btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  filter: grayscale(20%);
}

/* رابط تسجيل الدخول */
.link {
  text-align: center;
  margin-top: 20px;
  color: rgba(255, 255, 255, 0.6);
  font-size: 15px;
}

.link a {
  color: #D4AF37;
  text-decoration: none;
  font-weight: 600;
  margin-right: 5px;
  transition: all 0.3s ease;
  border-bottom: 1px solid transparent;
}

.link a:hover {
  color: #F6E27A;
  border-bottom-color: #D4AF37;
}

/* Loader - ذهبي */
.loader {
  width: 22px;
  height: 22px;
  border: 3px solid #0A0C10;
  border-top: 3px solid #D4AF37;
  border-radius: 50%;
  display: inline-block;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* تأثيرات إضافية للحقول */
.input:-webkit-autofill,
.input:-webkit-autofill:hover,
.input:-webkit-autofill:focus {
  -webkit-text-fill-color: #ffffff;
  -webkit-box-shadow: 0 0 0px 1000px #1A1F2A inset;
  transition: background-color 5000s ease-in-out 0s;
}

/* أيقونة كود الإحالة */
.input[placeholder="كود الإحالة"] {
  padding-right: 40px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%23D4AF37' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2'%3E%3C/path%3E%3Ccircle cx='12' cy='7' r='4'%3E%3C/circle%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 12px center;
  background-size: 18px;
}

/* أيقونة رقم الهاتف */
.input[placeholder="رقم الهاتف"] {
  padding-right: 40px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%23D4AF37' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Crect x='5' y='2' width='14' height='20' rx='2' ry='2'%3E%3C/rect%3E%3Cline x1='12' y1='18' x2='12.01' y2='18'%3E%3C/line%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 12px center;
  background-size: 18px;
}

/* تحسين للهواتف */
@media (max-width: 480px) {
  .container {
    padding: 20px;
  }
  
  .card {
    width: 95%;
    padding: 25px 15px;
  }
  
  .title {
    font-size: 24px;
  }
  
  .input {
    padding: 12px 10px;
    font-size: 14px;
  }
  
  .toggle {
    top: 12px;
    font-size: 13px;
  }
  
  .btn {
    padding: 12px;
    font-size: 16px;
  }
}
</style>
