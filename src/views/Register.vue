<template>
  <div class="container">
    <div class="card">
      <h2 class="title">إنشاء حساب</h2>

      <!-- اختيار طريقة إنشاء الحساب -->
      <div class="register-type-selector">
        <button 
          class="type-btn" 
          :class="{ active: registerType === 'email' }"
          @click="registerType = 'email'"
        >
          <i class="fas fa-envelope"></i>
          البريد الإلكتروني
        </button>
        <button 
          class="type-btn" 
          :class="{ active: registerType === 'phone' }"
          @click="registerType = 'phone'"
        >
          <i class="fas fa-phone"></i>
          رقم الهاتف
        </button>
      </div>

      <!-- إنشاء حساب بالبريد الإلكتروني -->
      <template v-if="registerType === 'email'">
        <label class="label">البريد الإلكتروني</label>
        <input
          type="email"
          v-model="email"
          placeholder="البريد الإلكتروني"
          class="input"
        />
      </template>

      <!-- إنشاء حساب برقم الهاتف -->
      <template v-if="registerType === 'phone'">
        <label class="label">رقم الهاتف مع رمز الدولة</label>
        <div class="phone-input-container">
          <select v-model="countryCode" class="country-select" @change="validatePhoneNumber">
            <option value="">اختر الرمز</option>
            <option value="+964">🇮🇶 العراق (+964)</option>
            <option value="+966">🇸🇦 السعودية (+966)</option>
            <option value="+971">🇦🇪 الإمارات (+971)</option>
            <option value="+965">🇰🇼 الكويت (+965)</option>
            <option value="+974">🇶🇦 قطر (+974)</option>
            <option value="+973">🇧🇭 البحرين (+973)</option>
            <option value="+968">🇴🇲 عمان (+968)</option>
            <option value="+962">🇯🇴 الأردن (+962)</option>
            <option value="+20">🇪🇬 مصر (+20)</option>
            <option value="+963">🇸🇾 سوريا (+963)</option>
            <option value="+961">🇱🇧 لبنان (+961)</option>
            <option value="+218">🇱🇾 ليبيا (+218)</option>
            <option value="+216">🇹🇳 تونس (+216)</option>
            <option value="+213">🇩🇿 الجزائر (+213)</option>
            <option value="+212">🇲🇦 المغرب (+212)</option>
            <option value="+222">🇲🇷 موريتانيا (+222)</option>
            <option value="+249">🇸🇩 السودان (+249)</option>
            <option value="+967">🇾🇪 اليمن (+967)</option>
            <option value="+970">🇵🇸 فلسطين (+970)</option>
            <option value="+90">🇹🇷 تركيا (+90)</option>
            <option value="+44">🇬🇧 بريطانيا (+44)</option>
            <option value="+1">🇺🇸 أمريكا (+1)</option>
            <option value="+49">🇩🇪 ألمانيا (+49)</option>
            <option value="+33">🇫🇷 فرنسا (+33)</option>
            <option value="+39">🇮🇹 إيطاليا (+39)</option>
            <option value="+34">🇪🇸 إسبانيا (+34)</option>
            <option value="+31">🇳🇱 هولندا (+31)</option>
            <option value="+46">🇸🇪 السويد (+46)</option>
            <option value="+47">🇳🇴 النرويج (+47)</option>
            <option value="+45">🇩🇰 الدنمارك (+45)</option>
            <option value="+358">🇫🇮 فنلندا (+358)</option>
            <option value="+41">🇨🇭 سويسرا (+41)</option>
            <option value="+43">🇦🇹 النمسا (+43)</option>
            <option value="+32">🇧🇪 بلجيكا (+32)</option>
            <option value="+48">🇵🇱 بولندا (+48)</option>
            <option value="+420">🇨🇿 التشيك (+420)</option>
            <option value="+36">🇭🇺 المجر (+36)</option>
            <option value="+40">🇷🇴 رومانيا (+40)</option>
            <option value="+359">🇧🇬 بلغاريا (+359)</option>
            <option value="+30">🇬🇷 اليونان (+30)</option>
            <option value="+351">🇵🇹 البرتغال (+351)</option>
            <option value="+7">🇷🇺 روسيا (+7)</option>
            <option value="+380">🇺🇦 أوكرانيا (+380)</option>
            <option value="+375">🇧🇾 بيلاروسيا (+375)</option>
            <option value="+995">🇬🇪 جورجيا (+995)</option>
            <option value="+994">🇦🇿 أذربيجان (+994)</option>
            <option value="+374">🇦🇲 أرمينيا (+374)</option>
            <option value="+998">🇺🇿 أوزبكستان (+998)</option>
            <option value="+996">🇰🇬 قرغيزستان (+996)</option>
            <option value="+992">🇹🇯 طاجيكستان (+992)</option>
            <option value="+993">🇹🇲 تركمانستان (+993)</option>
            <option value="+86">🇨🇳 الصين (+86)</option>
            <option value="+91">🇮🇳 الهند (+91)</option>
            <option value="+92">🇵🇰 باكستان (+92)</option>
            <option value="+93">🇦🇫 أفغانستان (+93)</option>
            <option value="+94">🇱🇰 سريلانكا (+94)</option>
            <option value="+95">🇲🇲 ميانمار (+95)</option>
            <option value="+66">🇹🇭 تايلاند (+66)</option>
            <option value="+84">🇻🇳 فيتنام (+84)</option>
            <option value="+60">🇲🇾 ماليزيا (+60)</option>
            <option value="+65">🇸🇬 سنغافورة (+65)</option>
            <option value="+62">🇮🇩 إندونيسيا (+62)</option>
            <option value="+63">🇵🇭 الفلبين (+63)</option>
            <option value="+82">🇰🇷 كوريا الجنوبية (+82)</option>
            <option value="+81">🇯🇵 اليابان (+81)</option>
          </select>
          <input
            type="tel"
            v-model="phoneNumber"
            placeholder="رقم الهاتف"
            class="phone-input"
            :disabled="!countryCode"
            @input="validatePhoneNumber"
          />
        </div>
        <span v-if="phoneError" class="error-message">{{ phoneError }}</span>
        <span v-if="!countryCode && phoneNumber" class="error-message">الرجاء اختيار رمز الدولة أولاً</span>
      </template>

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

      <!-- تأكيد كلمة المرور -->
      <label class="label">تأكيد كلمة المرور</label>
      <div class="input-box">
        <input
          :type="showConfirmPassword ? 'text' : 'password'"
          v-model="confirmPassword"
          placeholder="تأكيد كلمة المرور"
          class="input"
        />
        <span class="toggle" @click="toggleConfirmPassword">
          {{ showConfirmPassword ? "إخفاء" : "إظهار" }}
        </span>
      </div>
      <span v-if="passwordError" class="error-message">{{ passwordError }}</span>

      <!-- كود الإحالة -->
      <label class="label">كود الإحالة (اختياري)</label>
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
      registerType: 'email', // 'email' أو 'phone'
      email: "",
      countryCode: "",
      phoneNumber: "",
      fullPhoneNumber: "",
      password: "",
      confirmPassword: "",
      inviteCode: "",
      showPassword: false,
      showConfirmPassword: false,
      loading: false,
      phoneError: "",
      passwordError: "",
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

    toggleConfirmPassword() {
      this.showConfirmPassword = !this.showConfirmPassword;
    },

    validateEmail(email) {
      const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return re.test(email);
    },

    validatePassword(password) {
      return password.length >= 6;
    },

    validatePhoneNumber() {
      // إذا لم يتم إدخال رقم هاتف، نعتبره صحيح (اختياري)
      if (!this.phoneNumber && !this.countryCode) {
        this.phoneError = "";
        return true;
      }

      // التحقق من اختيار رمز الدولة
      if (!this.countryCode) {
        this.phoneError = "الرجاء اختيار رمز الدولة";
        return false;
      }

      // التحقق من إدخال رقم الهاتف
      if (!this.phoneNumber) {
        this.phoneError = "الرجاء إدخال رقم الهاتف";
        return false;
      }

      // تنظيف الرقم من الرموز غير الرقمية
      const cleanPhone = this.phoneNumber.replace(/[^0-9]/g, '');
      
      // التحقق من طول الرقم (يختلف حسب الدولة)
      const phoneLengths = {
        '+964': [10, 11], // العراق: 10-11 رقم
        '+966': [9, 10],  // السعودية: 9-10 أرقام
        '+971': [9, 10],  // الإمارات: 9-10 أرقام
        '+965': [8, 8],   // الكويت: 8 أرقام
        '+974': [8, 8],   // قطر: 8 أرقام
        '+973': [8, 8],   // البحرين: 8 أرقام
        '+968': [8, 8],   // عمان: 8 أرقام
        '+962': [9, 10],  // الأردن: 9-10 أرقام
        '+20': [10, 11],  // مصر: 10-11 رقم
        '+963': [9, 10],  // سوريا: 9-10 أرقام
        '+961': [7, 8],   // لبنان: 7-8 أرقام
        '+218': [9, 10],  // ليبيا: 9-10 أرقام
        '+216': [8, 8],   // تونس: 8 أرقام
        '+213': [9, 10],  // الجزائر: 9-10 أرقام
        '+212': [9, 10],  // المغرب: 9-10 أرقام
        '+222': [7, 8],   // موريتانيا: 7-8 أرقام
        '+249': [9, 10],  // السودان: 9-10 أرقام
        '+967': [9, 10],  // اليمن: 9-10 أرقام
        '+970': [9, 10],  // فلسطين: 9-10 أرقام
        '+90': [10, 11],  // تركيا: 10-11 رقم
        '+44': [10, 11],  // بريطانيا: 10-11 رقم
        '+1': [10, 10],   // أمريكا: 10 أرقام
        '+49': [10, 11],  // ألمانيا: 10-11 رقم
        '+33': [9, 10],   // فرنسا: 9-10 أرقام
        '+39': [9, 10],   // إيطاليا: 9-10 أرقام
        '+34': [9, 10],   // إسبانيا: 9-10 أرقام
        '+31': [9, 10],   // هولندا: 9-10 أرقام
        '+46': [9, 10],   // السويد: 9-10 أرقام
        '+47': [8, 8],    // النرويج: 8 أرقام
        '+45': [8, 8],    // الدنمارك: 8 أرقام
        '+358': [9, 10],  // فنلندا: 9-10 أرقام
        '+41': [9, 10],   // سويسرا: 9-10 أرقام
        '+43': [9, 10],   // النمسا: 9-10 أرقام
        '+32': [8, 9],    // بلجيكا: 8-9 أرقام
        '+48': [9, 9],    // بولندا: 9 أرقام
        '+420': [9, 9],   // التشيك: 9 أرقام
        '+36': [8, 9],    // المجر: 8-9 أرقام
        '+40': [9, 10],   // رومانيا: 9-10 أرقام
        '+359': [8, 9],   // بلغاريا: 8-9 أرقام
        '+30': [10, 10],  // اليونان: 10 أرقام
        '+351': [9, 9],   // البرتغال: 9 أرقام
        '+7': [10, 11],   // روسيا: 10-11 رقم
        '+380': [9, 10],  // أوكرانيا: 9-10 أرقام
        '+375': [9, 9],   // بيلاروسيا: 9 أرقام
        '+995': [9, 9],   // جورجيا: 9 أرقام
        '+994': [9, 9],   // أذربيجان: 9 أرقام
        '+374': [8, 8],   // أرمينيا: 8 أرقام
        '+998': [9, 9],   // أوزبكستان: 9 أرقام
        '+996': [9, 9],   // قرغيزستان: 9 أرقام
        '+992': [9, 9],   // طاجيكستان: 9 أرقام
        '+993': [8, 8],   // تركمانستان: 8 أرقام
        '+86': [11, 11],  // الصين: 11 رقم
        '+91': [10, 10],  // الهند: 10 أرقام
        '+92': [10, 11],  // باكستان: 10-11 رقم
        '+93': [9, 9],    // أفغانستان: 9 أرقام
        '+94': [9, 9],    // سريلانكا: 9 أرقام
        '+95': [7, 9],    // ميانمار: 7-9 أرقام
        '+66': [9, 9],    // تايلاند: 9 أرقام
        '+84': [9, 10],   // فيتنام: 9-10 أرقام
        '+60': [9, 10],   // ماليزيا: 9-10 أرقام
        '+65': [8, 8],    // سنغافورة: 8 أرقام
        '+62': [10, 12],  // إندونيسيا: 10-12 رقم
        '+63': [10, 10],  // الفلبين: 10 أرقام
        '+82': [9, 10],   // كوريا الجنوبية: 9-10 أرقام
        '+81': [10, 11],  // اليابان: 10-11 رقم
      };

      const [minLength, maxLength] = phoneLengths[this.countryCode] || [7, 15];
      
      if (cleanPhone.length < minLength || cleanPhone.length > maxLength) {
        this.phoneError = `رقم الهاتف يجب أن يكون بين ${minLength} و ${maxLength} رقم لهذه الدولة`;
        return false;
      }

      this.fullPhoneNumber = this.countryCode + cleanPhone;
      this.phoneError = "";
      return true;
    },

    validatePasswords() {
      if (!this.password) {
        this.passwordError = "كلمة المرور مطلوبة";
        return false;
      }

      if (!this.validatePassword(this.password)) {
        this.passwordError = "كلمة المرور يجب أن تكون 6 أحرف على الأقل";
        return false;
      }

      if (this.password !== this.confirmPassword) {
        this.passwordError = "كلمة المرور غير متطابقة";
        return false;
      }

      this.passwordError = "";
      return true;
    },

    async checkPhoneExists(phoneNumber) {
      try {
        const q = query(
          collection(db, "users"),
          where("phoneNumber", "==", phoneNumber)
        );
        const querySnapshot = await getDocs(q);
        return !querySnapshot.empty;
      } catch (error) {
        console.error("خطأ في التحقق من رقم الهاتف:", error);
        return false;
      }
    },

    async registerUser() {
      // التحقق من المدخلات حسب نوع التسجيل
      if (this.registerType === 'email') {
        if (!this.email) {
          alert("يرجى تعبئة البريد الإلكتروني");
          return;
        }
        if (!this.validateEmail(this.email)) {
          alert("البريد الإلكتروني غير صالح");
          return;
        }
      } else {
        if (!this.validatePhoneNumber()) {
          alert(this.phoneError || "يرجى التحقق من رقم الهاتف");
          return;
        }
      }

      // التحقق من كلمة المرور
      if (!this.validatePasswords()) {
        alert(this.passwordError);
        return;
      }

      this.loading = true;

      try {
        const auth = getAuth();
        await auth.setPersistence(browserLocalPersistence);

        let userEmail = this.email;

        // إذا كان التسجيل برقم الهاتف، ننشئ بريداً وهمياً
        if (this.registerType === 'phone') {
          // التحقق من عدم وجود الرقم مسبقاً
          const phoneExists = await this.checkPhoneExists(this.fullPhoneNumber);
          if (phoneExists) {
            alert("رقم الهاتف مستخدم بالفعل");
            this.loading = false;
            return;
          }

          // إنشاء بريد وهمي فريد
          const cleanPhone = this.phoneNumber.replace(/[^0-9]/g, '');
          userEmail = `user_${cleanPhone}_${Date.now()}@temp.com`;
        }

        const userCredential = await createUserWithEmailAndPassword(
          auth,
          userEmail,
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

        // حفظ بيانات المستخدم
        const userData = {
          uid: user.uid,
          email: this.registerType === 'email' ? this.email.trim() : null,
          phoneNumber: this.registerType === 'phone' ? this.fullPhoneNumber : null,
          countryCode: this.registerType === 'phone' ? this.countryCode : null,
          phoneNumberOnly: this.registerType === 'phone' ? this.phoneNumber.replace(/[^0-9]/g, '') : null,
          referralCode: user.uid.substring(0, 6),
          invitedBy: inviterUID || null,
          level2: level2 || null,
          level3: level3 || null,
          balance: 0,
          vipLevel: 0,
          blocked: false,
          createdAt: serverTimestamp(),
          registrationMethod: this.registerType,
        };

        await setDoc(doc(db, "users", user.uid), userData);

        // ✅ تخزين رقم الهاتف في displayName للتسجيل برقم الهاتف
        if (this.registerType === 'phone') {
          await updateProfile(user, {
            displayName: this.fullPhoneNumber
          });
        }

        router.push("/home");
      } catch (err) {
        console.error("Register error:", err);
        
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
  max-width: 420px;
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

/* محدد نوع إنشاء الحساب */
.register-type-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 25px;
}

.type-btn {
  flex: 1;
  padding: 12px;
  border-radius: 12px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  background: #1A1F2A;
  color: rgba(255, 255, 255, 0.7);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.type-btn i {
  font-size: 16px;
  color: #D4AF37;
}

.type-btn:hover {
  border-color: #D4AF37;
  color: #D4AF37;
}

.type-btn.active {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border-color: transparent;
}

.type-btn.active i {
  color: #0A0C10;
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

/* حاوية رقم الهاتف */
.phone-input-container {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

/* اختيار رمز الدولة */
.country-select {
  width: 40%;
  padding: 14px 10px;
  border-radius: 12px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  background: #1A1F2A;
  color: #ffffff;
  font-size: 14px;
  transition: all 0.3s ease;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%23D4AF37' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='6 9 12 15 18 9'%3E%3C/polyline%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: left 12px center;
  background-size: 16px;
  padding-left: 35px;
}

.country-select:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 0 3px rgba(212, 175, 55, 0.1);
}

.country-select option {
  background: #1A1F2A;
  color: #ffffff;
  padding: 10px;
}

/* حقل رقم الهاتف */
.phone-input {
  width: 60%;
  padding: 14px 12px;
  border-radius: 12px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  background: #1A1F2A;
  color: #ffffff;
  font-size: 15px;
  transition: all 0.3s ease;
}

.phone-input:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 0 3px rgba(212, 175, 55, 0.1);
  background: #1E2430;
}

.phone-input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.phone-input::placeholder {
  color: rgba(255, 255, 255, 0.3);
}

/* حقول الإدخال العادية */
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
  margin-bottom: 20px;
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

/* رسائل الخطأ */
.error-message {
  color: #ff6b6b;
  font-size: 12px;
  margin-top: -10px;
  margin-bottom: 15px;
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
  
  .register-type-selector {
    flex-direction: column;
  }
  
  .phone-input-container {
    flex-direction: column;
    gap: 10px;
  }
  
  .country-select,
  .phone-input {
    width: 100%;
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
