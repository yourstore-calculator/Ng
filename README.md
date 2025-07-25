# Ng
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>حاسبة الأسعار</title>
  <style>
    body {
      font-family: 'Tahoma', sans-serif;
      background: #f4f4f4;
      margin: 0;
      padding: 0;
      direction: rtl;
      color: #333;
    }
    header {
      text-align: center;
      padding: 20px;
    }
    header img {
      max-width: 120px;
    }
    .container {
      max-width: 900px;
      margin: auto;
      padding: 20px;
      background: #fff;
      border-radius: 10px;
    }
    h2 {
      margin-top: 30px;
      color: #333;
    }
    label {
      display: block;
      margin: 15px 0 5px;
    }
    select, input {
      width: 100%;
      padding: 10px;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    button {
      background: #333;
      color: white;
      padding: 10px 25px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      margin-top: 15px;
    }
    .result {
      background: #eee;
      padding: 15px;
      margin-top: 20px;
      border-radius: 10px;
    }
    .summary {
      font-weight: bold;
      font-size: 18px;
      color: #222;
      padding-top: 20px;
    }
  </style>
</head>
<body>

  <header>
    <img src="https://i.imgur.com/BO6aZDu.png" alt="شعار الشركة">
  </header>

  <div class="container">
    <h2>💰 حاسبة الأسعار</h2>

    <!-- 🧾 النوع الرئيسي -->
    <label>اختر القسم الرئيسي:</label>
    <select id="mainType" onchange="populateSubTypes()">
      <option disabled selected>-- اختر --</option>
      <option value="نوافذ">نوافذ</option>
      <option value="أبواب">أبواب</option>
      <option value="أبواب السحب">أبواب السحب</option>
    </select>

    <!-- ✅ الأنواع الفرعية -->
    <label>اختر النوع الفرعي:</label>
    <select id="subType" onchange="handleAdditions()"></select>

    <!-- 📐 المقاسات -->
    <label>الطول (متر):</label>
    <input type="number" id="height" step="0.01">

    <label>العرض (متر):</label>
    <input type="number" id="width" step="0.01">

    <!-- 🔢 الكمية -->
    <label>الكمية:</label>
    <input type="number" id="quantity" value="1">

    <!-- ➕ إضافات -->
    <div id="extrasContainer" style="margin-top: 15px;"></div>

    <!-- 🧮 أزرار -->
    <button onclick="calculate()">احسب</button>
    <button onclick="clearResults()">🧹 محو النتائج</button>
    <button onclick="downloadWord()">💾 حفظ كـ Word</button>

    <!-- 📦 النتائج -->
    <div id="results"></div>

    <!-- 📊 ملخص -->
    <div id="summaryContainer" class="summary"></div>
  </div>

<script>
const prices = {
  "دبل جلاس دبل فريم - ثابت": 34,
  "دبل جلاس دبل فريم - حركة": 73,
  "دبل جلاس دبل فريم - حركتين": 92,
  "دبل جلاس سنجل فريم - ثابت": 26,
  "دبل جلاس سنجل فريم - حركة": 46,
  "دبل جلاس سنجل فريم - حركتين": 58,
  "سنجل جلاس سنجل فريم - ثابت": 20,
  "سنجل جلاس سنجل فريم - حركة": 43,
  "سنجل جلاس سنجل فريم - حركتين": 47,
  "نوافذ السلايدنج": 10,
  "النوافذ الكهربائية": 102,
  "سكاي لايت - بدون مكينة": 56,
  "سكاي لايت - مع مكينة": 145,
  "كارتن وول - خفيف": 45,
  "كارتن وول - ثقيل": 56,
  "باب مدخل - زينك": 66,
  "باب مدخل - ستيل": 120,
  "باب مدخل - كاست المنيوم": 168,
  "WPC - فارغ": 45,
  "WPC - مع خشب": 50,
  "WPC - ضد الصوت": 60,
  "WPC - فريم ألمنيوم": 67,
  "WPC - سلايد": 65,
  "ألمنيوم - فارغ": 65,
  "ألمنيوم - مع خشب": 75,
  "ألمنيوم - فل ألمنيوم": 85,
  "ألمنيوم - مخفي": 110,
  "ألمنيوم - خارجي": 61,
  "دورات مياه - جديد": 55,
  "دورات مياه - قديم": 45,
  "دورات مياه - مخفي زجاجي": 65,
  "سحب داخلي زجاج": 38,
  "سحب داخلي متين": 41,
  "سحب خارجي جز مفتوح": 55,
  "سحب خارجي جزئين": 58,
  "سحب WPC": 61
};

const factors = {
  "default": 0.13,
  "WPC": 0.11,
  "ألمنيوم": 0.11,
  "دورات مياه": 0.11,
  "أبواب المداخل": 0.2,
  "أبواب الحدائق": 0.2,
  "الحواجز": 0.05
};

let allResults = [];

function populateSubTypes() {
  const type = document.getElementById("mainType").value;
  const sub = document.getElementById("subType");
  sub.innerHTML = "";
  const options = {
    "نوافذ": [
      "دبل جلاس دبل فريم - ثابت",
      "دبل جلاس دبل فريم - حركة",
      "دبل جلاس دبل فريم - حركتين",
      "دبل جلاس سنجل فريم - ثابت",
      "دبل جلاس سنجل فريم - حركة",
      "دبل جلاس سنجل فريم - حركتين",
      "سنجل جلاس سنجل فريم - ثابت",
      "سنجل جلاس سنجل فريم - حركة",
      "سنجل جلاس سنجل فريم - حركتين",
      "نوافذ السلايدنج",
      "النوافذ الكهربائية",
      "سكاي لايت - بدون مكينة",
      "سكاي لايت - مع مكينة",
      "كارتن وول - خفيف",
      "كارتن وول - ثقيل"
    ],
    "أبواب": [
      "باب مدخل - زينك",
      "باب مدخل - ستيل",
      "باب مدخل - كاست المنيوم",
      "WPC - فارغ",
      "WPC - مع خشب",
      "WPC - ضد الصوت",
      "WPC - فريم ألمنيوم",
      "ألمنيوم - فارغ",
      "ألمنيوم - مع خشب",
      "ألمنيوم - فل ألمنيوم",
      "ألمنيوم - مخفي",
      "ألمنيوم - خارجي",
      "دورات مياه - جديد",
      "دورات مياه - قديم",
      "دورات مياه - مخفي زجاجي"
    ],
    "أبواب السحب": [
      "سحب داخلي زجاج",
      "سحب داخلي متين",
      "سحب خارجي جز مفتوح",
      "سحب خارجي جزئين",
      "سحب WPC"
    ]
  };

  options[type].forEach(opt => {
    const el = document.createElement("option");
    el.textContent = opt;
    sub.appendChild(el);
  });

  handleAdditions();
}

function handleAdditions() {
  const subType = document.getElementById("subType").value;
  const container = document.getElementById("extrasContainer");
  container.innerHTML = "";

  const extras = [];

  // ستارة داخلية
  if (subType.includes("دبل جلاس") || subType.includes("السلايدنج") || subType.includes("سحب")) {
    extras.push(`
      <label>📏 مقاس الستارة (الطول × العرض بالمتر):</label>
      <input type="number" id="curtainHeight" step="0.01" placeholder="الطول">
      <input type="number" id="curtainWidth" step="0.01" placeholder="العرض">
    `);
  }

  // الشبك
  if (subType.includes("دبل جلاس") || subType.includes("سنجل جلاس")) {
    extras.push(`
      <label>🕸 نوع الشبك:</label>
      <select id="meshType">
        <option value="">بدون</option>
        <option value="ثابت">ثابت</option>
        <option value="سلايد">سلايد</option>
        <option value="فولدينج">فولدينج</option>
        <option value="باب">باب</option>
      </select>
    `);
  }

  container.innerHTML = extras.join("");
}
  function calculate() {
  const subType = document.getElementById("subType").value;
  const width = parseFloat(document.getElementById("width").value);
  const height = parseFloat(document.getElementById("height").value);
  const quantity = parseInt(document.getElementById("quantity").value);
  const curtainH = parseFloat(document.getElementById("curtainHeight")?.value || 0);
  const curtainW = parseFloat(document.getElementById("curtainWidth")?.value || 0);
  const meshType = document.getElementById("meshType")?.value || "";

  if (!subType || !width || !height || !quantity) {
    alert("يرجى إدخال جميع البيانات المطلوبة");
    return;
  }

  let basePrice = prices[subType] || 0;
  let area = width * height;
  let factor = 0.13;

  // تعديل حسابات أبواب WPC والألمنيوم ودورات المياه (مقاس 2.2 × 1)
  const fixedHeight = 2.2;
  const fixedWidth = 1;
  let usedHeight = height;
  let usedWidth = width;

  if (
    subType.includes("WPC") ||
    subType.includes("ألمنيوم") ||
    subType.includes("دورات مياه")
  ) {
    usedHeight = Math.max(height, fixedHeight);
    usedWidth = Math.max(width, fixedWidth);
    area = usedHeight * usedWidth;
  }

  // الشحن
  if (subType.includes("WPC")) factor = 0.11;
  if (subType.includes("ألمنيوم")) factor = 0.11;
  if (subType.includes("دورات مياه")) factor = 0.11;
  if (subType.includes("مدخل") || subType.includes("حديقة")) factor = 0.2;

  const shipping = area * factor * 48;

  // السعر الأساسي
  let total = 0;
  if (subType.includes("باب مدخل")) {
    total = (area * basePrice) + 10;
  } else if (subType.includes("السلايدنج")) {
    total = (area * basePrice) + 10;
  } else if (subType.includes("باب")) {
    total = basePrice;
  } else {
    total = area * basePrice;
  }

  // إضافات: ستارة داخلية
  let curtainTotal = 0;
  if (curtainH && curtainW) {
    curtainTotal = curtainH * curtainW * 26;
  }

  // إضافات: الشبك
  let meshTotal = 0;
  if (meshType) {
    meshTotal = 25;
  }

  const finalPerItem = total + shipping + curtainTotal + meshTotal;
  const finalAll = finalPerItem * quantity;

  const resultText = `
    <div class="result">
      <p><strong>النوع:</strong> ${subType}</p>
      <p><strong>المقاس:</strong> ${usedHeight.toFixed(2)} × ${usedWidth.toFixed(2)} متر</p>
      <p><strong>الكمية:</strong> ${quantity}</p>
      <p><strong>السعر الأساسي:</strong> ${total.toFixed(2)} ريال</p>
      <p><strong>سعر الشحن:</strong> ${shipping.toFixed(2)} ريال</p>
      ${curtainTotal ? `<p><strong>سعر الستارة:</strong> ${curtainTotal.toFixed(2)} ريال</p>` : ""}
      ${meshTotal ? `<p><strong>سعر الشبك:</strong> ${meshTotal.toFixed(2)} ريال (${meshType})</p>` : ""}
      <p><strong>الإجمالي:</strong> ${finalAll.toFixed(2)} ريال</p>
    </div>
  `;

  document.getElementById("results").innerHTML += resultText;
  allResults.push(finalAll);
  updateSummary();
}

function updateSummary() {
  const sum = allResults.reduce((a, b) => a + b, 0);
  const officeFee = sum * 0.04;
  const final = sum + officeFee;

  document.getElementById("summaryContainer").innerHTML = `
    <p>الإجمالي: ${sum.toFixed(2)} ريال</p>
    <p>عمولة المكتب (4%): ${officeFee.toFixed(2)} ريال</p>
    <p><strong>المبلغ النهائي: ${final.toFixed(2)} ريال</strong></p>
    <p style="color:#888; font-size:14px;">* الأسعار شاملة الشحن ولا تشمل التركيب</p>
  `;
}

function clearResults() {
  document.getElementById("results").innerHTML = "";
  document.getElementById("summaryContainer").innerHTML = "";
  allResults = [];
}

// تحميل النتائج كـ Word
function downloadWord() {
  let content = "<h2>تفاصيل الطلب</h2>";
  content += document.getElementById("results").innerHTML;
  content += document.getElementById("summaryContainer").innerHTML;

  const blob = new Blob(['\ufeff', content], {
    type: 'application/msword'
  });

  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = 'النتائج.doc';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}
</script>

</body>
</html>
