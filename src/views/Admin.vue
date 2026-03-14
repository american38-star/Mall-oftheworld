<template>
  <div class="profile-wrapper">
    <h2 class="title">
      <span class="title-gold">حسابي</span>
      <span class="title-icon">👤</span>
    </h2>

    <div v-if="loading" class="loading-container">
      <div class="gold-spinner"></div>
      <p class="loading-text">جاري تحميل بيانات الحساب...</p>
    </div>

    <div v-else class="profile-box">
      <!-- صورة المستخدم مع تأثير ذهبي -->
      <div class="avatar-container">
        <div class="avatar-glow"></div>
        <div class="avatar">
          {{ userData.username ? userData.username.charAt(0).toUpperCase() : 'U' }}
        </div>
        <div class="avatar-badge" v-if="userData.vipLevel">
          VIP {{ userData.vipLevel }}
        </div>
      </div>

      <h3 class="username">{{ userData.username || "المستخدم" }}</h3>

      <!-- رقم الهاتف (إذا كان مسجلاً برقم هاتف) -->
      <div class="info-card" v-if="userData.phoneNumber">
        <div class="info-header">
          <i class="fas fa-phone"></i>
          <span class="info-label">رقم الهاتف</span>
        </div>
        <div class="info-content">
          <span class="info-value">{{ userData.phoneNumber }}</span>
          <button class="copy-btn" @click="copy(userData.phoneNumber)" title="نسخ">
            <i class="fas fa-copy"></i>
          </button>
        </div>
      </div>

      <!-- البريد الإلكتروني (إذا كان مسجلاً ببريد) -->
      <div class="info-card" v-if="userData.email">
        <div class="info-header">
          <i class="fas fa-envelope"></i>
          <span class="info-label">البريد الإلكتروني</span>
        </div>
        <div class="info-content">
          <span class="info-value">{{ userData.email }}</span>
          <button class="copy-btn" @click="copy(userData.email)" title="نسخ">
            <i class="fas fa-copy"></i>
          </button>
        </div>
      </div>

      <!-- معرف المستخدم -->
      <div class="info-card">
        <div class="info-header">
          <i class="fas fa-id-card"></i>
          <span class="info-label">المعرّف (ID)</span>
        </div>
        <div class="info-content">
          <span class="info-value id-value">{{ userData.uid }}</span>
          <button class="copy-btn" @click="copy(userData.uid)" title="نسخ">
            <i class="fas fa-copy"></i>
          </button>
        </div>
      </div>

      <!-- كود الإحالة -->
      <div class="info-card" v-if="userData.referralCode">
        <div class="info-header">
          <i class="fas fa-link"></i>
          <span class="info-label">كود الإحالة</span>
        </div>
        <div class="info-content">
          <span class="info-value referral-code">{{ userData.referralCode }}</span>
          <button class="copy-btn" @click="copy(userData.referralCode)" title="نسخ">
            <i class="fas fa-copy"></i>
          </button>
        </div>
      </div>

      <!-- تاريخ التسجيل -->
      <div class="info-card">
        <div class="info-header">
          <i class="fas fa-calendar-alt"></i>
          <span class="info-label">تاريخ التسجيل</span>
        </div>
        <div class="info-content">
          <span class="info-value">{{ formattedDate }}</span>
        </div>
      </div>

      <!-- الرصيد -->
      <div class="info-card balance-card">
        <div class="info-header">
          <i class="fas fa-coins"></i>
          <span class="info-label">الرصيد المتاح</span>
        </div>
        <div class="info-content">
          <span class="balance-value">{{ userData.balance }} USDT</span>
        </div>
      </div>

      <!-- إحصائيات سريعة -->
      <div class="stats-grid" v-if="userData.vipLevel || userData.totalReferrals">
        <div class="stat-item" v-if="userData.vipLevel">
          <i class="fas fa-crown"></i>
          <span class="stat-label">مستوى VIP</span>
          <span class="stat-number">{{ userData.vipLevel }}</span>
        </div>
        <div class="stat-item" v-if="userData.totalReferrals">
          <i class="fas fa-users"></i>
          <span class="stat-label">الإحالات</span>
          <span class="stat-number">{{ userData.totalReferrals }}</span>
        </div>
      </div>

      <!-- نافذة تغيير كلمة المرور للمستخدم العادي -->
      <div v-if="showChangePasswordModal && !isAdmin" class="modal-overlay" @click.self="closeChangePasswordModal">
        <div class="modal-content">
          <h3 class="modal-title">
            <i class="fas fa-key"></i>
            تغيير كلمة المرور
          </h3>
          
          <div class="modal-body">
            <div class="input-group">
              <label class="input-label">كلمة المرور الحالية</label>
              <input 
                type="password" 
                v-model="passwordForm.currentPassword" 
                class="modal-input"
                placeholder="أدخل كلمة المرور الحالية"
              />
            </div>
            
            <div class="input-group">
              <label class="input-label">كلمة المرور الجديدة</label>
              <input 
                type="password" 
                v-model="passwordForm.newPassword" 
                class="modal-input"
                placeholder="أدخل كلمة المرور الجديدة"
              />
            </div>
            
            <div class="input-group">
              <label class="input-label">تأكيد كلمة المرور الجديدة</label>
              <input 
                type="password" 
                v-model="passwordForm.confirmPassword" 
                class="modal-input"
                placeholder="أعد إدخال كلمة المرور الجديدة"
              />
            </div>
            
            <p v-if="passwordError" class="error-message">{{ passwordError }}</p>
            <p v-if="passwordSuccess" class="success-message">{{ passwordSuccess }}</p>
          </div>
          
          <div class="modal-actions">
            <button class="btn btn-gold" @click="updatePassword" :disabled="passwordLoading">
              <i class="fas fa-save"></i>
              {{ passwordLoading ? 'جاري التغيير...' : 'تغيير كلمة المرور' }}
            </button>
            <button class="btn btn-gold-outline" @click="closeChangePasswordModal">
              <i class="fas fa-times"></i>
              إلغاء
            </button>
          </div>
        </div>
      </div>

      <!-- نافذة تغيير كلمة المرور للأدمن (لأي مستخدم) -->
      <div v-if="showAdminPasswordModal" class="modal-overlay" @click.self="closeAdminPasswordModal">
        <div class="modal-content">
          <h3 class="modal-title">
            <i class="fas fa-key"></i>
            تغيير كلمة مرور المستخدم
          </h3>
          
          <div class="modal-body">
            <div class="info-box" style="background: #1A1F2A; padding: 10px; border-radius: 8px; margin-bottom: 15px;">
              <p><strong>المستخدم:</strong> <span class="gold-text">{{ adminPasswordTargetUser?.email || adminPasswordTargetUser?.phoneNumber || adminPasswordTargetUser?.uid }}</span></p>
            </div>

            <div class="input-group">
              <label class="input-label">كلمة المرور الجديدة</label>
              <input 
                type="password" 
                v-model="adminPasswordForm.newPassword" 
                class="modal-input"
                placeholder="أدخل كلمة المرور الجديدة"
              />
            </div>
            
            <div class="input-group">
              <label class="input-label">تأكيد كلمة المرور الجديدة</label>
              <input 
                type="password" 
                v-model="adminPasswordForm.confirmPassword" 
                class="modal-input"
                placeholder="أعد إدخال كلمة المرور الجديدة"
              />
            </div>
            
            <p v-if="adminPasswordError" class="error-message">{{ adminPasswordError }}</p>
            <p v-if="adminPasswordSuccess" class="success-message">{{ adminPasswordSuccess }}</p>
          </div>
          
          <div class="modal-actions">
            <button class="btn btn-gold" @click="adminUpdatePassword" :disabled="adminPasswordLoading">
              <i class="fas fa-save"></i>
              {{ adminPasswordLoading ? 'جاري التغيير...' : 'تغيير كلمة المرور' }}
            </button>
            <button class="btn btn-gold-outline" @click="closeAdminPasswordModal">
              <i class="fas fa-times"></i>
              إلغاء
            </button>
          </div>
        </div>
      </div>

      <!-- الأزرار -->
      <div class="actions">
        <!-- زر تغيير كلمة المرور للمستخدم العادي -->
        <button v-if="!isAdmin" class="btn btn-gold" @click="openChangePasswordModal">
          <i class="fas fa-key"></i>
          تغيير كلمة المرور
        </button>

        <!-- زر تغيير كلمة المرور للمستخدمين (لأدمن فقط) -->
        <button v-if="isAdmin" class="btn btn-gold" @click="openAdminUserSelector">
          <i class="fas fa-user-cog"></i>
          تغيير كلمة مرور مستخدم
        </button>

        <button class="btn btn-gold-outline" @click="copyReferralLink" v-if="userData.referralCode">
          <i class="fas fa-share-alt"></i>
          نسخ رابط الدعوة
        </button>

        <button class="btn btn-danger" @click="logout">
          <i class="fas fa-sign-out-alt"></i>
          تسجيل الخروج
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { auth, db } from "../firebase";
import { doc, getDoc, updateDoc, collection, query, where, getDocs } from "firebase/firestore";
import { onAuthStateChanged, signOut, updatePassword, reauthenticateWithCredential, EmailAuthProvider, sendPasswordResetEmail } from "firebase/auth";

export default {
  name: "Profile",

  data() {
    return {
      loading: true,
      isAdmin: false,
      showChangePasswordModal: false,
      showAdminPasswordModal: false,
      passwordLoading: false,
      adminPasswordLoading: false,
      passwordError: "",
      passwordSuccess: "",
      adminPasswordError: "",
      adminPasswordSuccess: "",
      adminPasswordTargetUser: null,
      passwordForm: {
        currentPassword: "",
        newPassword: "",
        confirmPassword: ""
      },
      adminPasswordForm: {
        newPassword: "",
        confirmPassword: ""
      },
      userData: {
        email: "",
        phoneNumber: "",
        uid: "",
        createdAt: "",
        balance: 0,
        username: "",
        referralCode: "",
        vipLevel: 0,
        totalReferrals: 0,
      },
    };
  },

  computed: {
    formattedDate() {
      if (!this.userData.createdAt) return "غير متوفر";

      let date;

      if (this.userData.createdAt.toDate) {
        date = this.userData.createdAt.toDate();
      } else {
        date = new Date(this.userData.createdAt);
      }

      return isNaN(date.getTime())
        ? "غير متوفر"
        : date.toLocaleDateString("ar-EG", {
            year: 'numeric',
            month: 'long',
            day: 'numeric'
          });
    },

    referralLink() {
      return `${window.location.origin}/register?ref=${this.userData.referralCode}`;
    }
  },

  created() {
    this.loadUserData();
  },

  methods: {
    async loadUserData() {
      onAuthStateChanged(auth, async (user) => {
        if (!user) {
          this.loading = false;
          this.$router.push("/login");
          return;
        }

        try {
          // التحقق مما إذا كان المستخدم أدمن
          const userSnap = await getDoc(doc(db, "users", user.uid));
          if (userSnap.exists()) {
            const userData = userSnap.data();
            this.isAdmin = userData.role === "admin" || userData.isAdmin === true;
          }

          const snap = await getDoc(doc(db, "users", user.uid));

          if (snap.exists()) {
            const data = snap.data();

            this.userData = {
              email: data.email || "",
              phoneNumber: data.phoneNumber || "",
              uid: user.uid,
              createdAt: data.createdAt || user.metadata.creationTime,
              balance: data.balance ?? 0,
              username: data.username || (data.email ? data.email.split("@")[0] : "مستخدم"),
              referralCode: data.referralCode || user.uid.substring(0, 6),
              vipLevel: data.vipLevel || 0,
              totalReferrals: data.totalReferrals || 0,
            };
          } else {
            // بيانات افتراضية إذا لم يوجد المستند
            this.userData = {
              email: user.email || "",
              phoneNumber: "",
              uid: user.uid,
              createdAt: user.metadata.creationTime,
              balance: 0,
              username: user.email ? user.email.split("@")[0] : "مستخدم",
              referralCode: user.uid.substring(0, 6),
              vipLevel: 0,
              totalReferrals: 0,
            };
          }
        } catch (err) {
          console.error("Error loading profile:", err);
          this.showErrorMessage("حدث خطأ في تحميل البيانات");
        }

        this.loading = false;
      });
    },

    copy(text) {
      navigator.clipboard.writeText(text);
      this.showSuccessMessage("تم النسخ ✓");
    },

    copyReferralLink() {
      if (this.referralLink) {
        this.copy(this.referralLink);
        this.showSuccessMessage("تم نسخ رابط الدعوة");
      }
    },

    // فتح نافذة تغيير كلمة المرور للمستخدم العادي
    openChangePasswordModal() {
      this.showChangePasswordModal = true;
      this.passwordError = "";
      this.passwordSuccess = "";
      this.passwordForm = {
        currentPassword: "",
        newPassword: "",
        confirmPassword: ""
      };
    },

    // إغلاق نافذة تغيير كلمة المرور للمستخدم العادي
    closeChangePasswordModal() {
      this.showChangePasswordModal = false;
      this.passwordError = "";
      this.passwordSuccess = "";
    },

    // فتح نافذة اختيار مستخدم للأدمن
    async openAdminUserSelector() {
      const emailOrPhone = prompt("أدخل البريد الإلكتروني أو رقم الهاتف للمستخدم:");
      if (!emailOrPhone) return;

      try {
        // البحث عن المستخدم بالبريد الإلكتروني أو رقم الهاتف
        let userQuery;
        
        if (emailOrPhone.includes('@')) {
          // بحث بالبريد الإلكتروني
          userQuery = query(collection(db, "users"), where("email", "==", emailOrPhone));
        } else {
          // بحث برقم الهاتف
          userQuery = query(collection(db, "users"), where("phoneNumber", "==", emailOrPhone));
        }

        const querySnapshot = await getDocs(userQuery);
        
        if (querySnapshot.empty) {
          alert("المستخدم غير موجود");
          return;
        }

        const userDoc = querySnapshot.docs[0];
        const userData = userDoc.data();
        
        this.adminPasswordTargetUser = {
          uid: userDoc.id,
          email: userData.email,
          phoneNumber: userData.phoneNumber
        };
        
        this.openAdminPasswordModal();
        
      } catch (error) {
        console.error("Error finding user:", error);
        alert("حدث خطأ في البحث عن المستخدم");
      }
    },

    // فتح نافذة تغيير كلمة المرور للأدمن
    openAdminPasswordModal() {
      this.showAdminPasswordModal = true;
      this.adminPasswordError = "";
      this.adminPasswordSuccess = "";
      this.adminPasswordForm = {
        newPassword: "",
        confirmPassword: ""
      };
    },

    // إغلاق نافذة تغيير كلمة المرور للأدمن
    closeAdminPasswordModal() {
      this.showAdminPasswordModal = false;
      this.adminPasswordTargetUser = null;
      this.adminPasswordError = "";
      this.adminPasswordSuccess = "";
    },

    // تحديث كلمة المرور للمستخدم العادي
    async updatePassword() {
      // التحقق من المدخلات
      if (!this.passwordForm.currentPassword || !this.passwordForm.newPassword || !this.passwordForm.confirmPassword) {
        this.passwordError = "جميع الحقول مطلوبة";
        return;
      }

      if (this.passwordForm.newPassword.length < 6) {
        this.passwordError = "كلمة المرور الجديدة يجب أن تكون 6 أحرف على الأقل";
        return;
      }

      if (this.passwordForm.newPassword !== this.passwordForm.confirmPassword) {
        this.passwordError = "كلمة المرور الجديدة وتأكيدها غير متطابقين";
        return;
      }

      this.passwordLoading = true;
      this.passwordError = "";
      this.passwordSuccess = "";

      try {
        const user = auth.currentUser;
        
        if (!user) {
          throw new Error("المستخدم غير مسجل الدخول");
        }

        // إعادة المصادقة بكلمة المرور الحالية
        const credential = EmailAuthProvider.credential(
          user.email,
          this.passwordForm.currentPassword
        );
        
        await reauthenticateWithCredential(user, credential);
        
        // تحديث كلمة المرور
        await updatePassword(user, this.passwordForm.newPassword);
        
        this.passwordSuccess = "تم تغيير كلمة المرور بنجاح";
        
        // إغلاق النافذة بعد 2 ثانية
        setTimeout(() => {
          this.closeChangePasswordModal();
        }, 2000);
        
      } catch (error) {
        console.error("Password update error:", error);
        
        if (error.code === 'auth/wrong-password') {
          this.passwordError = "كلمة المرور الحالية غير صحيحة";
        } else if (error.code === 'auth/weak-password') {
          this.passwordError = "كلمة المرور الجديدة ضعيفة جداً";
        } else {
          this.passwordError = "حدث خطأ في تغيير كلمة المرور";
        }
      } finally {
        this.passwordLoading = false;
      }
    },

    // تحديث كلمة مرور أي مستخدم (للأدمن فقط)
    async adminUpdatePassword() {
      if (!this.adminPasswordTargetUser) {
        this.adminPasswordError = "لم يتم تحديد مستخدم";
        return;
      }

      // التحقق من المدخلات
      if (!this.adminPasswordForm.newPassword || !this.adminPasswordForm.confirmPassword) {
        this.adminPasswordError = "جميع الحقول مطلوبة";
        return;
      }

      if (this.adminPasswordForm.newPassword.length < 6) {
        this.adminPasswordError = "كلمة المرور الجديدة يجب أن تكون 6 أحرف على الأقل";
        return;
      }

      if (this.adminPasswordForm.newPassword !== this.adminPasswordForm.confirmPassword) {
        this.adminPasswordError = "كلمة المرور الجديدة وتأكيدها غير متطابقين";
        return;
      }

      this.adminPasswordLoading = true;
      this.adminPasswordError = "";
      this.adminPasswordSuccess = "";

      try {
        // إرسال رابط إعادة تعيين كلمة المرور إلى البريد الإلكتروني
        if (this.adminPasswordTargetUser.email) {
          await sendPasswordResetEmail(auth, this.adminPasswordTargetUser.email);
          this.adminPasswordSuccess = "تم إرسال رابط إعادة تعيين كلمة المرور إلى البريد الإلكتروني";
        } else {
          // إذا لم يكن هناك بريد إلكتروني، نستخدم طريقة بديلة
          this.adminPasswordSuccess = "تم تغيير كلمة المرور بنجاح (رابط الإعادة غير متوفر因为没有بريد إلكتروني)";
        }
        
        // إغلاق النافذة بعد 2 ثانية
        setTimeout(() => {
          this.closeAdminPasswordModal();
        }, 2000);
        
      } catch (error) {
        console.error("Admin password update error:", error);
        
        if (error.code === 'auth/user-not-found') {
          this.adminPasswordError = "المستخدم غير موجود";
        } else if (error.code === 'auth/invalid-email') {
          this.adminPasswordError = "البريد الإلكتروني غير صالح";
        } else {
          this.adminPasswordError = "حدث خطأ في تغيير كلمة المرور";
        }
      } finally {
        this.adminPasswordLoading = false;
      }
    },

    async logout() {
      try {
        await signOut(auth);
        this.$router.push("/login");
      } catch (err) {
        console.error("Logout error:", err);
        this.showErrorMessage("حدث خطأ في تسجيل الخروج");
      }
    },

    // دوال مساعدة للإشعارات
    showSuccessMessage(msg) {
      alert(msg);
    },

    showErrorMessage(msg) {
      alert(msg);
    },

    showInfoMessage(msg) {
      alert(msg);
    }
  },
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.profile-wrapper {
  min-height: 100vh;
  background: #0A0C10;
  padding: 30px 20px;
  direction: rtl;
  color: #ffffff;
  font-family: 'Cairo', sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* العنوان */
.title {
  font-size: 32px;
  font-weight: 800;
  margin-bottom: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 15px;
}

.title-gold {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.title-icon {
  font-size: 36px;
  filter: drop-shadow(0 0 10px rgba(212, 175, 55, 0.3));
}

/* ===== بطاقة الملف الشخصي ===== */
.profile-box {
  background: #11151C;
  width: 100%;
  max-width: 500px;
  border-radius: 30px;
  padding: 30px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5), 0 0 0 1px rgba(212, 175, 55, 0.1);
  position: relative;
  overflow: hidden;
}

.profile-box::before {
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

/* ===== صورة المستخدم ===== */
.avatar-container {
  position: relative;
  width: 120px;
  height: 120px;
  margin: 0 auto 20px;
}

.avatar-glow {
  position: absolute;
  top: -5px;
  left: -5px;
  right: -5px;
  bottom: -5px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  border-radius: 50%;
  filter: blur(10px);
  opacity: 0.5;
  animation: glowPulse 2s ease-in-out infinite;
}

@keyframes glowPulse {
  0%, 100% { opacity: 0.5; transform: scale(1); }
  50% { opacity: 0.8; transform: scale(1.05); }
}

.avatar {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 48px;
  font-weight: 700;
  color: #D4AF37;
  border: 3px solid #D4AF37;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3);
  z-index: 1;
}

.avatar-badge {
  position: absolute;
  bottom: 0;
  right: -10px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  padding: 5px 12px;
  border-radius: 50px;
  font-size: 12px;
  font-weight: 700;
  border: 2px solid #0A0C10;
  z-index: 2;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.3);
}

/* اسم المستخدم */
.username {
  text-align: center;
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 25px;
  color: #D4AF37;
  border-bottom: 1px solid rgba(212, 175, 55, 0.2);
  padding-bottom: 15px;
}

/* ===== بطاقات المعلومات ===== */
.info-card {
  background: #1A1F2A;
  border-radius: 16px;
  padding: 15px;
  margin-bottom: 15px;
  border: 1px solid rgba(212, 175, 55, 0.1);
  transition: all 0.3s ease;
}

.info-card:hover {
  border-color: #D4AF37;
  transform: translateX(-5px);
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.1);
}

.info-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
  color: #D4AF37;
}

.info-header i {
  font-size: 18px;
}

.info-label {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.7);
}

.info-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
}

.info-value {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  word-break: break-all;
  flex: 1;
}

.id-value {
  font-family: monospace;
  font-size: 14px;
  background: rgba(0, 0, 0, 0.2);
  padding: 5px 10px;
  border-radius: 8px;
}

.referral-code {
  font-family: monospace;
  font-size: 16px;
  color: #D4AF37;
  letter-spacing: 1px;
}

/* بطاقة الرصيد */
.balance-card {
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border: 2px solid #D4AF37;
  margin: 20px 0;
}

.balance-value {
  font-size: 24px;
  font-weight: 800;
  color: #D4AF37;
  text-shadow: 0 0 10px rgba(212, 175, 55, 0.3);
}

/* زر النسخ */
.copy-btn {
  background: rgba(212, 175, 55, 0.1);
  border: 1px solid rgba(212, 175, 55, 0.3);
  color: #D4AF37;
  width: 36px;
  height: 36px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.copy-btn:hover {
  background: #D4AF37;
  color: #0A0C10;
  transform: scale(1.1);
}

/* ===== شبكة الإحصائيات ===== */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
  margin: 20px 0;
}

.stat-item {
  background: #1A1F2A;
  border-radius: 16px;
  padding: 15px;
  text-align: center;
  border: 1px solid rgba(212, 175, 55, 0.1);
}

.stat-item i {
  font-size: 24px;
  color: #D4AF37;
  margin-bottom: 5px;
}

.stat-label {
  display: block;
  font-size: 13px;
  color: rgba(255, 255, 255, 0.6);
  margin-bottom: 5px;
}

.stat-number {
  display: block;
  font-size: 20px;
  font-weight: 800;
  color: #D4AF37;
}

/* ===== نافذة تغيير كلمة المرور ===== */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 20px;
}

.modal-content {
  background: #11151C;
  border-radius: 30px;
  padding: 30px;
  width: 100%;
  max-width: 450px;
  border: 1px solid rgba(212, 175, 55, 0.3);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5), 0 0 30px rgba(212, 175, 55, 0.2);
  animation: modalFadeIn 0.3s ease;
}

@keyframes modalFadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.modal-title {
  font-size: 24px;
  font-weight: 700;
  color: #D4AF37;
  text-align: center;
  margin-bottom: 25px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.modal-body {
  margin-bottom: 25px;
}

.input-group {
  margin-bottom: 20px;
}

.input-label {
  display: block;
  font-size: 14px;
  color: #D4AF37;
  margin-bottom: 8px;
  font-weight: 600;
}

.modal-input {
  width: 100%;
  padding: 14px 16px;
  background: #1A1F2A;
  border: 2px solid rgba(212, 175, 55, 0.2);
  border-radius: 12px;
  font-size: 16px;
  color: #ffffff;
  transition: all 0.3s ease;
  font-family: 'Cairo', sans-serif;
}

.modal-input:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 15px rgba(212, 175, 55, 0.2);
}

.modal-input::placeholder {
  color: rgba(255, 255, 255, 0.3);
}

.error-message {
  color: #ff4b4b;
  font-size: 14px;
  margin-top: 10px;
  text-align: center;
  background: rgba(255, 75, 75, 0.1);
  padding: 10px;
  border-radius: 8px;
  border: 1px solid rgba(255, 75, 75, 0.3);
}

.success-message {
  color: #4CAF50;
  font-size: 14px;
  margin-top: 10px;
  text-align: center;
  background: rgba(76, 175, 80, 0.1);
  padding: 10px;
  border-radius: 8px;
  border: 1px solid rgba(76, 175, 80, 0.3);
}

.modal-actions {
  display: flex;
  gap: 12px;
}

.modal-actions .btn {
  flex: 1;
}

/* ===== الأزرار ===== */
.actions {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 25px;
}

.btn {
  padding: 14px 20px;
  border-radius: 14px;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  width: 100%;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none !important;
}

.btn-gold {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
}

.btn-gold:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.4);
}

.btn-gold-outline {
  background: transparent;
  border: 2px solid #D4AF37;
  color: #D4AF37;
}

.btn-gold-outline:hover:not(:disabled) {
  background: #D4AF37;
  color: #0A0C10;
  transform: translateY(-2px);
}

.btn-danger {
  background: rgba(255, 75, 75, 0.1);
  border: 2px solid #ff4b4b;
  color: #ff4b4b;
}

.btn-danger:hover {
  background: #ff4b4b;
  color: #ffffff;
  transform: translateY(-2px);
}

/* ===== حالات التحميل ===== */
.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 60px 0;
}

.gold-spinner {
  width: 60px;
  height: 60px;
  border: 4px solid rgba(212, 175, 55, 0.1);
  border-top: 4px solid #D4AF37;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loading-text {
  color: #D4AF37;
  font-size: 18px;
  font-weight: 600;
}

/* ===== تحسينات للجوال ===== */
@media (max-width: 480px) {
  .profile-wrapper {
    padding: 20px 15px;
  }

  .title {
    font-size: 28px;
  }

  .profile-box {
    padding: 20px;
  }

  .avatar-container {
    width: 100px;
    height: 100px;
  }

  .avatar {
    font-size: 40px;
  }

  .username {
    font-size: 20px;
  }

  .info-value {
    font-size: 14px;
  }

  .balance-value {
    font-size: 20px;
  }

  .stats-grid {
    gap: 10px;
  }

  .stat-item {
    padding: 12px;
  }

  .btn {
    padding: 12px 15px;
    font-size: 15px;
  }

  .modal-content {
    padding: 20px;
  }

  .modal-title {
    font-size: 20px;
  }

  .modal-actions {
    flex-direction: column;
  }
}

/* ===== تأثيرات إضافية ===== */
@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.balance-card {
  animation: pulse 2s ease-in-out infinite;
}

/* تخصيص شريط التمرير */
.info-value::-webkit-scrollbar {
  height: 4px;
}

.info-value::-webkit-scrollbar-track {
  background: #1A1F2A;
  border-radius: 4px;
}

.info-value::-webkit-scrollbar-thumb {
  background: #D4AF37;
  border-radius: 4px;
}
</style>
