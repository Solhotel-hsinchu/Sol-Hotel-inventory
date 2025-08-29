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
    h1, h2 { text-align: center; }
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
    .day:hover { background: #d0e6ff; }
    .selected {
      background: #4caf50 !important;
      color: white;
      font-weight: bold;
    }
    .inventory, .summary {
      margin-top: 20px;
      padding: 15px;
      background: white;
      border-radius: 8px;
      box-shadow: 0 0 6px rgba(0,0,0,0.1);
    }
    input, button {
      padding: 6px;
      margin: 3px;
    }
    li { margin: 4px 0; }
    .out { color: red; font-weight: bold; }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 8px;
      text-align: center;
    }
    th { background: #f0f0f0; }
    .download-btn {
      display: block;
      margin: 15px auto 0;
      padding: 8px 16px;
      background: #4caf50;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
    }
    .download-btn:hover {
      background: #45a049;
    }
  </style>
</head>
<body>
  <h1>📅 跨日期庫存系統</h1>
  <div id="calendar" class="calendar"></div>

  <div class="inventory">
    <h2 id="selectedDateTitle"></h2>
    <div>
      <input type="text" id="item1" placeholder="品項1 (如：加床)">
      <input type="number" id="qty1" placeholder="數量" min="1"><br>
      <input type="text" id="item2" placeholder="品項2 (如：嬰兒床)">
      <input type="number" id="qty2" placeholder="數量" min="1"><br>
      <input type="text" id="item3" placeholder="品項3">
      <input type="number" id="qty3" placeholder="數量" min="1"><br>
      <input type="text" id="item4" placeholder="品項4">
      <input type="number" id="qty4" placeholder="數量" min="1"><br>
      <input type="text" id="item5" placeholder="品項5">
      <input type="number" id="qty5" placeholder="數量" min="1"><br>
      <button onclick="addItems()">新增品項</button>
    </div>
    <ul id="inventoryList"></ul>
  </div>

  <div class="summary">
    <h2>📊 總庫存表 (2025/09/01 ~ 12/31)</h2>
    <table id="summaryTable">
      <thead>
        <tr>
          <th>品項</th>
          <th>總剩餘數量</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
    <button class="download-btn" onclick="downloadExcel()">📥 下載 Excel 報表</button>
  </div>

  <!-- 引入 SheetJS -->
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

  <script>
    const calendarEl = document.getElementById("calendar");
    const inventoryList = document.getElementById("inventoryList");
    const selectedDateTitle = document.getElementById("selectedDateTitle");
    const summaryTable = document.querySelector("#summaryTable tbody");
    let selectedDate = "2025-09-01"; // 預設開始日

    // 渲染 2025/09/01 ~ 2025/12/31
    function renderCalendar() {
      const start = new Date("2025-09-01");
      const end = new Date("2025-12-31");

      calendarEl.innerHTML = "";
      let date = new Date(start);

      while (date <= end) {
        const isoDate = date.toISOString().split("T")[0];
        const div = document.createElement("div");
        div.className = "day";
        div.innerText = `${date.getMonth() + 1}/${date.getDate()}`;

        if (isoDate === selectedDate) {
          div.classList.add("selected");
        }

        div.onclick = () => {
          selectedDate = isoDate;
          renderCalendar();
          loadInventory();
        };

        calendarEl.appendChild(div);
        date.setDate(date.getDate() + 1);
      }
    }

    function loadInventory() {
      selectedDateTitle.innerText = "📦 " + selectedDate + " 的庫存";
      inventoryList.innerHTML = "";
      let data = JSON.parse(localStorage.getItem("inventory")) || {};
      let items = data[selectedDate] || [];
      items.forEach((item, index) => {
        let li = document.createElement("li");
        if (item.qty > 0) {
          li.innerHTML = `${item.name} - 剩餘 ${item.qty} 
            <button onclick="useItem(${index})">使用1</button>`;
        } else {
          li.innerHTML = `${item.name} - <span class="out">⚠️ 庫存不足</span>`;
        }
        inventoryList.appendChild(li);
      });
      updateSummary();
    }

    function addItems() {
      let data = JSON.parse(localStorage.getItem("inventory")) || {};
      if (!data[selectedDate]) data[selectedDate] = [];

      for (let i = 1; i <= 5; i++) {
        let name = document.getElementById("item" + i).value.trim();
        let qty = parseInt(document.getElementById("qty" + i).value);
        if (name && qty > 0) {
          data[selectedDate].push({ name, qty });
          document.getElementById("item" + i).value = "";
          document.getElementById("qty" + i).value = "";
        }
      }
      localStorage.setItem("inventory", JSON.stringify(data));
      loadInventory();
    }

    function useItem(index) {
      let data = JSON.parse(localStorage.getItem("inventory")) || {};
      if (!data[selectedDate]) return;

      if (data[selectedDate][index].qty > 0) {
        data[selectedDate][index].qty--;
      }
      localStorage.setItem("inventory", JSON.stringify(data));
      loadInventory();
    }

    // 更新總庫存表
    function updateSummary() {
      let data = JSON.parse(localStorage.getItem("inventory")) || {};
      let totals = {};

      Object.values(data).forEach(dayItems => {
        dayItems.forEach(item => {
          if (!totals[item.name]) totals[item.name] = 0;
          totals[item.name] += item.qty;
        });
      });

      summaryTable.innerHTML = "";
      for (let [name, qty] of Object.entries(totals)) {
        let row = document.createElement("tr");
        row.innerHTML = `
          <td>${name}</td>
          <td>${qty > 0 ? qty : '<span class="out">⚠️ 庫存不足</span>'}</td>
        `;
        summaryTable.appendChild(row);
      }
    }

    // 匯出 Excel (總表 + 每日明細)
    function downloadExcel() {
      let data = JSON.parse(localStorage.getItem("inventory")) || {};

      // ====== 總表 ======
      let totals = {};
      Object.values(data).forEach(dayItems => {
        dayItems.forEach(item => {
          if (!totals[item.name]) totals[item.name] = 0;
          totals[item.name] += item.qty;
        });
      });

      let summarySheetData = [["品項", "總剩餘數量"]];
      for (let [name, qty] of Object.entries(totals)) {
        summarySheetData.push([name, qty]);
      }
      let wsSummary = XLSX.utils.aoa_to_sheet(summarySheetData);

      // ====== 每日明細 ======
      let detailSheetData = [["日期", "品項", "剩餘數量"]];
      Object.keys(data).sort().forEach(date => {
        data[date].forEach(item => {
          detailSheetData.push([date, item.name, item.qty]);
        });
      });
      let wsDetail = XLSX.utils.aoa_to_sheet(detailSheetData);

      // ====== 建立 Excel 檔 ======
      let wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, wsSummary, "總庫存表");
      XLSX.utils.book_append_sheet(wb, wsDetail, "每日明細");
      XLSX.writeFile(wb, "庫存報表.xlsx");
    }

    renderCalendar();
    loadInventory();
  </script>
</body>
</html>
