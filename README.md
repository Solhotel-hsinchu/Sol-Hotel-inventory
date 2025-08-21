<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <title>跨日期庫存系統</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f7f7f7;
      margin: 20px;
    }
    h1 { text-align: center; }
    .calendar {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 5px;
      margin-bottom: 20px;
    }
    .day {
      background: #fff;
      padding: 10px;
      text-align: center;
      cursor: pointer;
      border: 1px solid #ddd;
      border-radius: 6px;
    }
    .day:hover {
      background: #d0e6ff;
    }
    .selected {
      background: #4caf50 !important;
      color: white;
      font-weight: bold;
    }
    .inventory {
      margin-top: 20px;
      padding: 15px;
      background: white;
      border-radius: 8px;
      box-shadow: 0 0 6px rgba(0,0,0,0.1);
    }
    input, button {
      padding: 6px;
      margin: 5px 0;
    }
  </style>
</head>
<body>
  <h1>📅 跨日期庫存系統</h1>
  <div id="calendar" class="calendar"></div>

  <div class="inventory">
    <h2 id="selectedDateTitle"></h2>
    <div>
      <input type="text" id="itemName" placeholder="品項名稱">
      <input type="number" id="itemQty" placeholder="數量" min="1">
      <button onclick="addItem()">新增品項</button>
    </div>
    <ul id="inventoryList"></ul>
  </div>

  <script>
    const calendarEl = document.getElementById("calendar");
    const inventoryList = document.getElementById("inventoryList");
    const selectedDateTitle = document.getElementById("selectedDateTitle");
    let selectedDate = new Date().toISOString().split("T")[0]; // 預設今天

    function renderCalendar() {
      const today = new Date();
      const year = today.getFullYear();
      const month = today.getMonth();
      const firstDay = new Date(year, month, 1);
      const lastDay = new Date(year, month + 1, 0);

      calendarEl.innerHTML = "";
      for (let i = 1; i <= lastDay.getDate(); i++) {
        const date = new Date(year, month, i).toISOString().split("T")[0];
        const div = document.createElement("div");
        div.className = "day";
        div.innerText = i;

        if (date === selectedDate) {
          div.classList.add("selected");
        }

        div.onclick = () => {
          selectedDate = date;
          renderCalendar();
          loadInventory();
        };
        calendarEl.appendChild(div);
      }
    }

    function loadInventory() {
      selectedDateTitle.innerText = "📦 " + selectedDate + " 的庫存";
      inventoryList.innerHTML = "";
      let data = JSON.parse(localStorage.getItem("inventory")) || {};
      let items = data[selectedDate] || [];
      items.forEach((item, index) => {
        let li = document.createElement("li");
        li.innerHTML = `${item.name} - 剩餘 ${item.qty} 
          <button onclick="useItem(${index})">使用1</button>`;
        inventoryList.appendChild(li);
      });
    }

    function addItem() {
      let name = document.getElementById("itemName").value.trim();
      let qty = parseInt(document.getElementById("itemQty").value);
      if (!name || qty <= 0) return alert("請輸入正確品項與數量！");

      let data = JSON.parse(localStorage.getItem("inventory")) || {};
      if (!data[selectedDate]) data[selectedDate] = [];
      data[selectedDate].push({ name, qty });
      localStorage.setItem("inventory", JSON.stringify(data));

      document.getElementById("itemName").value = "";
      document.getElementById("itemQty").value = "";
      loadInventory();
    }

    function useItem(index) {
      let data = JSON.parse(localStorage.getItem("inventory")) || {};
      if (!data[selectedDate]) return;

      data[selectedDate][index].qty--;
      if (data[selectedDate][index].qty <= 0) {
        data[selectedDate].splice(index, 1); // 移除沒庫存的
      }
      localStorage.setItem("inventory", JSON.stringify(data));
      loadInventory();
    }

    renderCalendar();
    loadInventory();
  </script>
</body>
</html>
