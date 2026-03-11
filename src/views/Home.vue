<!-- الشريط العلوي -->
<div class="top-bar">
  <div class="top-left">
    <i class="fas fa-headset icon"></i>
    <i class="fas fa-envelope icon"></i>
  </div>

  <div class="user-box">
    <div class="welcome">مرحباً، {{ username }}</div>
    <div class="balance">الرصيد: <strong>{{ balance }} USDT</strong></div>
  </div>
</div>

<!-- شريط الأزرار -->
<div class="quick-buttons">
  <button class="quick-btn">
    <span class="btn-icon">⏱</span>
    وفر الوقت
  </button>
  <button class="quick-btn">
    <span class="btn-icon">💰</span>
    وفر المال
  </button>
</div>

<!-- شريط الإعلان -->
<div class="notice-bar">
  <span class="notice-icon">🔊</span>
  شراكة عالمية مع Amazon, eBay, Etsy, Walmart وأكثر!
</div>

<!-- القائمة الرئيسية -->
<div class="grid-menu">
  <div
    class="item"
    v-for="item in menu"
    :key="item.title"
    @click="go(item.route)"
  >
    <div class="icon-box">
      <i :class="item.icon"></i>
    </div>
    <p>{{ item.title }}</p>
  </div>

  <!-- زر الشركة لعرض الإعلان -->
  <div class="item company-item" @click="showCompanyAd">
    <div class="icon-box gold-glow">
      <i class="fas fa-building"></i>
    </div>
    <p>الشركة</p>
  </div>
</div>

<!-- تم إزالة البانر بالكامل -->

<!-- إعلان Popup فاخر -->
<div id="companyAd" class="ad-overlay" v-if="showAd" @click.self="closeAd">
  <div class="ad-box">
    <div class="ad-header">
      <h2>✨ إعلان الشركة ✨</h2>
      <button class="close-btn" @click="closeAd">×</button>
    </div>

    <div class="ad-content">
      <p>
        مرحباً بالجميع 🌟
        <br><br>
        يسرّنا أن نعرّفكم بشركة Palm Treasure التي تأسست في سنغافورة بتاريخ 5 فبراير 2026، وهي شركة استثمارية متخصصة في مجال التجارة الإلكترونية. تمتلك الشركة فريقاً تقنياً محترفاً وخبرة مالية قوية، ويقع مقرها الرئيسي حالياً في منطقة الأعمال المركزية في سنغافورة.
        <br><br>
        وانطلاقاً من رؤيتنا للتوسع وبناء شبكة تعاون واسعة، قمنا بإنشاء فروع ووكالات في عدد من الدول العربية مثل لبنان، الجزائر، ليبيا، والعراق، ونسعى خلال المرحلة القادمة إلى توسيع نشاطنا وانتشارنا في مختلف دول الشرق الأوسط.
        <br><br>
        لقد جاء تأسيس هذه الشركة بعد دراسة عميقة لما حدث خلال عامي 2024 و2025، حيث ظهرت العديد من المنصات الوهمية التي خدعت الكثير من الناس ولم تستمر طويلاً. ومن هنا كان هدفنا واضحاً: تحويل هذا المجال إلى منصة حقيقية وموثوقة يستفيد منها الأعضاء كما تستفيد منها الشركة، ضمن نظام عادل وشفاف.
        <br><br>
        وقد تحقق هذا المشروع بجهود كبيرة من الفريق التقني تحت إشراف المهندس أليكس ديروب، الذي لعب دوراً أساسياً في تطوير النظام التقني للشركة. وتقديراً لجهوده المميزة ومساهمته في نجاح هذا المشروع، تمت ترقيته إلى منصب نائب المدير.
        <br><br>
        نحن في Palm Treasure نؤمن بأن النجاح الحقيقي يجب أن يكون متاحاً للجميع، لذلك تم تصميم هذا المشروع ليكون فرصة متاحة لكل الناس، وخاصة للطبقة المتوسطة وذوي الدخل المحدود، حتى يتمكنوا من تحسين أوضاعهم والمشاركة في فرص الاقتصاد الرقمي.
        <br><br>
        ولهذا السبب تم وضع نظام واضح وقوانين عادلة تضمن حماية حقوق الموظفين والأعضاء قبل حقوق المستثمرين، مع مراعاة الظروف الاقتصادية والاجتماعية في المجتمعات العربية.
        <br><br>
        وسيتم تطبيق هذا النظام المبرمج في معظم الدول العربية ابتداءً من 1 مارس 2026 وحتى نهاية عام 2028، وخلال هذه الفترة سيتم تقييم الأداء والنتائج. وبعد ذلك قد يتم تطوير النظام وإجراء بعض التعديلات بناءً على كفاءة الأعضاء وجهودهم وإخلاصهم في العمل.
        <br><br>
        كما نعمل منذ الآن على التخطيط لمجموعة من الفرص المستقبلية والمشاريع الجديدة التي ستوفر مزيداً من الإمكانيات لأعضاء الشركة في السنوات القادمة.
        <br><br>
        نتمنى للجميع التوفيق، ونسعد بانضمامكم إلى مجتمع Palm Treasure 🌴
        <br>
        ━━━━━━━━━━━━━━━
        <br><br>
        📊 خطط العضوية والأرباح في Palm Treasure
        <br><br>
        حرصاً على توفير فرص تناسب الجميع، قمنا بتصميم مستويات VIP متعددة تبدأ من المستوى المجاني وحتى مستويات النخبة ذات العوائد المرتفعة.
      </p>
      
      <div class="vip-table">
        <h3>🎯 خطط الاستثمار VIP</h3>
        <div class="vip-row">
          <span class="vip-level">VIP 1</span>
          <span class="vip-recharge">💰 0 USDT</span>
          <span class="vip-daily">📈 0.3 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 2</span>
          <span class="vip-recharge">💰 50 USDT</span>
          <span class="vip-daily">📈 1.6 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 3</span>
          <span class="vip-recharge">💰 100 USDT</span>
          <span class="vip-daily">📈 3.25 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 4</span>
          <span class="vip-recharge">💰 300 USDT</span>
          <span class="vip-daily">📈 10 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 5</span>
          <span class="vip-recharge">💰 900 USDT</span>
          <span class="vip-daily">📈 33 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 6</span>
          <span class="vip-recharge">💰 1350 USDT</span>
          <span class="vip-daily">📈 51 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 7</span>
          <span class="vip-recharge">💰 1800 USDT</span>
          <span class="vip-daily">📈 70 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 8</span>
          <span class="vip-recharge">💰 3600 USDT</span>
          <span class="vip-daily">📈 150 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 9</span>
          <span class="vip-recharge">💰 7200 USDT</span>
          <span class="vip-daily">📈 330 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 10</span>
          <span class="vip-recharge">💰 14400 USDT</span>
          <span class="vip-daily">📈 700 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 11</span>
          <span class="vip-recharge">💰 18800 USDT</span>
          <span class="vip-daily">📈 1600 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 12</span>
          <span class="vip-recharge">💰 37600 USDT</span>
          <span class="vip-daily">📈 3500 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 13</span>
          <span class="vip-recharge">💰 75200 USDT</span>
          <span class="vip-daily">📈 7500 USDT/يوم</span>
        </div>
        <div class="vip-row">
          <span class="vip-level">VIP 14</span>
          <span class="vip-recharge">💰 150400 USDT</span>
          <span class="vip-daily">📈 16000 USDT/يوم</span>
        </div>
      </div>

      <div class="commission-info">
        <h3>🤝 نظام العمولات</h3>
        <p>المستوى 1: 6% | المستوى 2: 2% | المستوى 3: 1%</p>
      </div>
    </div>

    <button class="ad-btn" @click="closeAd">أنا أعرف</button>
  </div>
</div>
