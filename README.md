<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>庫存系統與月曆互動版</title>
<style>
    body {
        font-family: Arial, sans-serif;
        background-color: #f4f4f4;
        text-align: center;
        margin: 0;
        padding: 20px;
    }
    h1 { margin-bottom: 10px; }
    .inventory, .calendar { margin: 20px auto; }
    table { margin: 0 auto; border-collapse: collapse; }
    td, th { border: 1px solid #555; padding: 8px 12px; width: 50px; height: 50px; }
    th { background-color: #ddd; }
    .product-btn, #restockBtn {
        margin: 5px; padding: 10px 20px; cursor: pointer;
        border: none; border-radius: 5px; background-color: #4CAF50; color: white;
        transition: 0.3s;
    }
    .product-btn:disabled {
        background-color: #ccc;
        cursor: not-allowed;
    }
    .low-stock { background-color: #ffeb3b; }
    .out-stock { background-color: #f44336; color: white; }
    .today { background-color: #2196F3; color: white; font-weight: bold; }
</style>
</head>
<body>

<h1>庫存系統</h1>
<div class="inventory">
    <table id="inventoryTable">
        <tr><th>產品</th><th>庫存</th></tr>
    </table>
</div>
<div class="buttons" id="buttons"></div>
<button id="restockBtn" onclick="restock()">補貨全部產品</button>

<h1>本月月曆</h1>
<div class="calendar" id="calendar"></div>

<script>
// 初始化庫存
let products = {
    "產品A": 8,
    "產品B": 8,
    "產品C": 8,
    "產品D": 8,
    "產品E": 8
};

let usageLog = {}; // 記錄每日期庫存使用

// 更新庫存表格
function updateInventory() {
    const table = document.getElementById("inventoryTable");
    table.innerHTML = "<tr><th>產品</th><th>庫存</th></tr>";
    for (let p in products) {
        const row = table.insertRow();
        const nameCell = row.insertCell(0);
        const stockCell = row.insertCell(1);
        nameCell.innerText = p;
        stockCell.innerText = products[p];

        if (products[p] === 0) {
            stockCell.className = "out-stock";
        } else if (products[p] <= 3) {
            stockCell.className = "low-stock";
        } else {
            stockCell.className = "";
        }
    }

    // 更新按鈕狀態
    const btns = document.querySelectorAll(".product-btn");
    btns.forEach(btn => {
        const prod = btn.dataset.product;
        btn.disabled = products[prod] === 0;
    });
}

// 建立使用按鈕
function createButtons() {
    const btnDiv = document.getElementById("buttons");
    for (let p in products) {
        const btn = document.createElement("button");
        btn.innerText = `使用 ${p}`;
        btn.className = "product-btn";
        btn.dataset.product = p;
        btn.onclick = () => useProduct(p);
        btnDiv.appendChild(btn);
    }
}

// 使用產品
function useProduct(product) {
    if (products[product] > 0) {
        products[product]--;
        updateInventory();

        const todayStr = new Date().toISOString().split('T')[0];
        if (!usageLog[todayStr]) usageLog[todayStr] = [];
        usageLog[todayStr].push(product);

        showCalendar(); // 更新月曆顯示使用紀錄
    }
}

// 補貨
function restock() {
    for (let p in products) products[p] = 8;
    updateInventory();
    showCalendar();
}

// 顯示月曆
function showCalendar() {
    const calDiv = document.getElementById("calendar");
    const now = new Date();
    const year = now.getFullYear();
    const month = now.getMonth();
    const firstDay = new Date(year, month, 1).getDay();
    const lastDate = new Date(year, month + 1, 0).getDate();

    let table = "<table><tr>";
    const weekDays = ["日","一","二","三","四","五","六"];
    weekDays.forEach(day => table += `<th>${day}</th>`);
    table += "</tr><tr>";

    for (let i = 0; i < firstDay; i++) table += "<td></td>";

    for (let date = 1; date <= lastDate; date++) {
        const dateStr = `${year}-${String(month+1).padStart(2,'0')}-${String(date).padStart(2,'0')}`;
        let classes = "";
        if (date === now.getDate()) classes += "today";
        if (usageLog[dateStr]) classes += " low-stock"; // 標示有使用紀錄

        table += `<td class="${classes}" title="${usageLog[dateStr]?usageLog[dateStr].join(', '):''}">${date}</td>`;
        if ((date + firstDay) % 7 === 0) table += "</tr><tr>";
    }

    table += "</tr></table>";
    calDiv.innerHTML = table;
}

// 初始化
updateInventory();
createButtons();
showCalendar();
</script>

</body>
</html>
