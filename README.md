<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>بطاقات البضاعة / الأشخاص</title>
<style>
  body {
    display: flex;
    flex-direction: column;
    align-items: center;
    min-height: 100vh;
    margin: 20px;
    font-family: "Cairo", sans-serif;
    background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  }

  #top-bar {
    display: none;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    margin-bottom: 15px;
  }

  #searchBox {
    padding: 10px;
    width: 300px;
    border-radius: 8px;
    border: 1px solid #ccc;
    font-size: 1rem;
    text-align: center;
  }

  #totalSum {
    font-weight: bold;
    color: #007bff;
    font-size: 1.2rem;
  }

  #storage-controls {
    display: none;
    gap: 10px;
    margin: 15px 0;
  }

  button {
    padding: 10px 18px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    color: white;
    font-weight: bold;
    transition: 0.2s;
  }

  button:hover { opacity: 0.85; }
  .save-btn { background: #28a745; }

  .container {
    display: none;
    flex-wrap: wrap;
    gap: 15px;
    justify-content: center;
    max-width: 1300px;
    padding: 20px;
  }

  .item-card {
    width: 130px;
    height: 150px;
    background: #ffffff;
    border-radius: 12px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    transition: all 0.3s ease;
    border: 1px solid #dee2e6;
    cursor: pointer;
  }

  .item-card:hover {
    transform: scale(1.05);
    box-shadow: 0 8px 14px rgba(0,0,0,0.15);
  }

  .item-number {
    font-size: 0.8rem;
    font-weight: bold;
    color: #007bff;
    margin-bottom: 6px;
  }

  .item-input {
    width: 90%;
    padding: 6px;
    border: 1px solid #ccc;
    border-radius: 6px;
    font-size: 0.8rem;
    text-align: center;
  }

  .item-total {
    margin-top: 6px;
    font-size: 0.9rem;
    color: #28a745;
    font-weight: bold;
  }

  .modal {
    display: none;
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
    background: rgba(0,0,0,0.6);
    z-index: 1000;
    justify-content: center;
    align-items: center;
  }

  .modal-content {
    background: #fff;
    padding: 25px;
    border-radius: 12px;
    width: 90%;
    max-width: 500px;
    position: relative;
  }

  .close-btn {
    position: absolute;
    top: 15px; left: 15px;
    cursor: pointer;
    font-size: 22px;
    color: #666;
  }

  table {
    width: 100%;
    margin-top: 10px;
    border-collapse: collapse;
  }

  th, td {
    padding: 10px;
    text-align: right;
    border-bottom: 1px solid #eee;
  }

  th {
    background: #f8f9fa;
    width: 40%;
  }

  .edit-input {
    width: 100%;
    padding: 6px;
    border: 1px solid #ccc;
    border-radius: 6px;
  }

  #lock-screen {
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
    background: #ffffff;
    z-index: 2000;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  #lock-screen input {
    padding: 10px;
    width: 200px;
    border: 1px solid #ccc;
    border-radius: 8px;
    text-align: center;
    font-size: 1rem;
    margin-bottom: 10px;
  }

  #lock-screen button {
    background: #007bff;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 8px;
    cursor: pointer;
  }

  #lock-screen button:hover {
    background: #0056b3;
  }

</style>
</head>
<body>

<!-- شاشة القفل -->
<div id="lock-screen">
  <h2>🔒 أدخل الكود السري لفتح الصفحة</h2>
  <input type="password" id="unlockCode" placeholder="أدخل الكود">
  <button onclick="unlockPage()">دخول</button>
  <p id="errorMsg" style="color:red;margin-top:10px;"></p>
</div>

<div id="top-bar">
  <input type="text" id="searchBox" placeholder="🔍 ابحث عن الاسم...">
  <div id="totalSum">إجمالي جميع الدفعات: 0</div>
</div>

<div id="storage-controls">
  <button class="save-btn" onclick="saveData()">💾 حفظ البيانات</button>
</div>

<div class="container"></div>

<!-- نافذة التفاصيل -->
<div id="itemModal" class="modal">
  <div class="modal-content">
    <span class="close-btn" onclick="closeModal()">&times;</span>
    <table>
      <tr><th>رقم البطاقة</th><td id="modalNumber"></td></tr>
      <tr><th>الاسم</th><td><input type="text" id="modalName" class="edit-input"></td></tr>
      <tr><th>الدفعة 1</th><td><input type="number" id="modalQty1" class="edit-input" min="0" oninput="calculateTotal()"></td></tr>
      <tr><th>الدفعة 2</th><td><input type="number" id="modalQty2" class="edit-input" min="0" oninput="calculateTotal()"></td></tr>
      <tr><th>الدفعة 3</th><td><input type="number" id="modalQty3" class="edit-input" min="0" oninput="calculateTotal()"></td></tr>
      <tr><th>المجموع</th><td id="modalTotal">0</td></tr>
      <tr><th>التاريخ</th><td><input type="date" id="modalDate" class="edit-input"></td></tr>
    </table>

    <div style="text-align:left; margin-top:15px;">
      <button class="save-btn" onclick="saveModal()">حفظ</button>
      <button class="close-btn" style="color:white;background:#6c757d;border:none;padding:10px 18px;border-radius:6px;" onclick="closeModal()">إلغاء</button>
    </div>
  </div>
</div>

<script>
const itemsData = new Map();
const container = document.querySelector('.container');
let currentNumber = 1;
const SECRET_CODE = "1294"; // الكود المطلوب لفتح الصفحة
const EDIT_CODE = "1293"; // كود تعديل البيانات

// 🔐 فتح الصفحة بعد إدخال الكود
function unlockPage() {
  const inputCode = document.getElementById("unlockCode").value.trim();
  if (inputCode === SECRET_CODE) {
    document.getElementById("lock-screen").style.display = "none";
    document.getElementById("top-bar").style.display = "flex";
    document.getElementById("storage-controls").style.display = "flex";
    document.querySelector(".container").style.display = "flex";
    initApp();
  } else {
    document.getElementById("errorMsg").textContent = "❌ الكود غير صحيح!";
  }
}

// 🧮 حساب مجموع الدفعات
function calculateTotal() {
  const q1 = +document.getElementById('modalQty1').value || 0;
  const q2 = +document.getElementById('modalQty2').value || 0;
  const q3 = +document.getElementById('modalQty3').value || 0;
  document.getElementById('modalTotal').textContent = q1 + q2 + q3;
}

// 🧾 جمع جميع المبالغ
function updateTotalSum() {
  let sum = 0;
  itemsData.forEach(d => sum += (+d.qty1 || 0) + (+d.qty2 || 0) + (+d.qty3 || 0));
  document.getElementById('totalSum').textContent = "إجمالي جميع الدفعات: " + sum.toFixed(2);
}

// 🧱 إنشاء البطاقات
function createCards() {
  container.innerHTML = '';
  for (let i = 1; i <= 200; i++) {
    const card = document.createElement('div');
    card.className = 'item-card';
    card.innerHTML = `
      <div class="item-number">#${i}</div>
      <input class="item-input" data-id="${i}" placeholder="الاسم" readonly>
      <div class="item-total" id="total_${i}">المجموع: 0</div>
    `;
    card.addEventListener('click', () => openModal(i));

    const saved = localStorage.getItem(`item_${i}`);
    if (saved) {
      const data = JSON.parse(saved);
      itemsData.set(i, data);
      card.querySelector('.item-input').value = data.name;
      const total = (+data.qty1 || 0) + (+data.qty2 || 0) + (+data.qty3 || 0);
      card.querySelector('.item-total').textContent = "المجموع: " + total;
    }
    container.appendChild(card);
  }
  updateTotalSum();
}

// 🔍 فتح النافذة
function openModal(id) {
  const data = itemsData.get(id) || {name:"",qty1:0,qty2:0,qty3:0,date:new Date().toISOString().split('T')[0]};
  currentNumber = id;
  document.getElementById('modalNumber').textContent = id;
  document.getElementById('modalName').value = data.name;
  document.getElementById('modalQty1').value = data.qty1;
  document.getElementById('modalQty2').value = data.qty2;
  document.getElementById('modalQty3').value = data.qty3;
  document.getElementById('modalDate').value = data.date;
  calculateTotal();
  document.getElementById('itemModal').style.display = 'flex';
}

// 💾 حفظ البيانات مع كود سري للتعديل فقط
function saveModal() {
  const code = prompt("🔒 أدخل الكود السري للتعديل:");
  if (code !== EDIT_CODE) {
    alert("❌ الكود غير صحيح. لن يتم الحفظ.");
    closeModal();
    return;
  }

  const data = {
    name: document.getElementById('modalName').value,
    qty1: +document.getElementById('modalQty1').value || 0,
    qty2: +document.getElementById('modalQty2').value || 0,
    qty3: +document.getElementById('modalQty3').value || 0,
    date: document.getElementById('modalDate').value
  };

  itemsData.set(currentNumber, data);
  localStorage.setItem(`item_${currentNumber}`, JSON.stringify(data));

  const input = document.querySelector(`[data-id="${currentNumber}"]`);
  if (input) input.value = data.name;

  const total = data.qty1 + data.qty2 + data.qty3;
  document.getElementById(`total_${currentNumber}`).textContent = "المجموع: " + total;

  updateTotalSum();
  closeModal();
}

function closeModal() {
  document.getElementById('itemModal').style.display = 'none';
}

function saveData() {
  localStorage.setItem('all_items', JSON.stringify([...itemsData]));
  alert('✅ تم حفظ جميع البيانات بنجاح!');
}

function searchCards() {
  const value = document.getElementById('searchBox').value.trim();
  document.querySelectorAll('.item-card').forEach(card => {
    const input = card.querySelector('.item-input');
    card.style.display = (!value || (input.value && input.value.includes(value))) ? 'flex' : 'none';
  });
}

function initApp() {
  const saved = localStorage.getItem('all_items');
  if (saved) {
    const data = new Map(JSON.parse(saved));
    data.forEach((v,k)=>itemsData.set(k,v));
  }
  createCards();
  document.getElementById('searchBox').addEventListener('input', searchCards);
}
</script>

</body>
</html>
