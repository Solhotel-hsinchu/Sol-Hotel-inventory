<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>庫存系統與彩色月曆</title>
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
    td, th { border: 1px solid #555; padding: 5px; width: 60px; height: 60px; vertical-align: top; position: relative; }
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
    #log { margin-top: 10px; font-weight: bold; min-height: 24px; }
    .tag {
        display: inline-block;
        width: 10px;
        height: 10px;
        border-radius: 50%;
        margin: 1px;
    }
    .A { background-color: #f44336; }  /* 產品A紅 */
    .B { background-color: #ff9800; }  /* 產品B橙 */
    .C { background-color: #ffeb3b; }  /* 產品C黃 */
    .D { background-color: #4CAF50; }  /* 產品D綠 */
    .E { background-color: #2196F3; }  /* 產品E藍 */
    td span { font-size: 10px; display: block; margin-top: 2px; }
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
<div id="log"></div>

<script>
// 初始化庫存
let products = { "產品A":8, "產品B":8, "產品C":8, "產品D":8, "產品E":8 };
let usageLog = {}; // 每日期庫存使用

const productColors = { "產品A":"A","產品B":"B","產品C":"C","產品D":"D","產品E":"E" };

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

        if (products[p] === 0) stockCell.className = "out-stock";
        else if (products[p] <= 3) stockCell.className = "low-stock";
        else stockCell.className = "";
    }

    document.querySelectorAll(".product-btn").forEach(btn => {
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

        showCalendar();
        showLog(todayStr);
    }
}

// 補貨
function restock() {
    for (let p in products) products[p] = 8;
    updateInventory();
    showCalendar();
    document.getElementById("log").innerText = "";
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
    ["日","一","二","三","四","五","六"].forEach(d=>table+=`<th>${d}</th>`);
    table+="</tr><tr>";

    for(let i=0;i<firstDay;i++) table+="<td></td>";

    for(let date=1; date<=lastDate; date++){
        const dateStr = `${year}-${String(month+1).padStart(2,'0')}-${String(date).padStart(2,'0')}`;
        let classes = date===now.getDate() ? "today" : "";
        let tags = "";
        if(usageLog[dateStr]){
            usageLog[dateStr].forEach(p=>{
                tags += `<span class="tag ${productColors[p]}" title="${p}"></span>`;
            });
        }
        table += `<td class="${classes}" onclick="showLog('${dateStr}')" title="${usageLog[dateStr]?usageLog[dateStr].join(', '):''}">${date}${tags}</td>`;
        if((date + firstDay) % 7 === 0) table+="</tr><tr>";
    }

    table+="</tr></table>";
    calDiv.innerHTML = table;
}

// 顯示使用紀錄
function showLog(dateStr){
    const logDiv = document.getElementById("log");
    if(usageLog[dateStr] && usageLog[dateStr].length>0)
        logDiv.innerText = `${dateStr} 使用產品: ${usageLog[dateStr].join(', ')}`;
    else
        logDiv.innerText = `${dateStr} 沒有使用產品`;
}

// 初始化
updateInventory();
createButtons();
showCalendar();
</script>

</body>
</html>
