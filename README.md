<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>庫存系統與月曆</title>
<style>
    body {
        font-family: Arial, sans-serif;
        text-align: center;
        background-color: #f4f4f4;
        margin: 0;
        padding: 20px;
    }
    h1 { margin-bottom: 10px; }
    .inventory, .calendar { margin: 20px auto; }
    .product-btn { margin: 5px; padding: 10px 20px; cursor: pointer; }
    table { margin: 0 auto; border-collapse: collapse; }
    td, th { border: 1px solid #555; padding: 5px 10px; width: 40px; height: 40px; }
    th { background-color: #ddd; }
</style>
</head>
<body>

<h1>庫存系統</h1>
<div class="inventory" id="inventory"></div>
<div class="buttons" id="buttons"></div>

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

// 更新庫存顯示
function updateInventory() {
    const invDiv = document.getElementById("inventory");
    invDiv.innerHTML = "";
    for (let p in products) {
        invDiv.innerHTML += `${p}: ${products[p]}<br>`;
    }
}

// 建立使用按鈕
function createButtons() {
    const btnDiv = document.getElementById("buttons");
    for (let p in products) {
        const btn = document.createElement("button");
        btn.innerText = `使用 ${p}`;
        btn.className = "product-btn";
        btn.onclick = () => useProduct(p);
        btnDiv.appendChild(btn);
    }
}

// 使用產品，扣除庫存
function useProduct(product) {
    if (products[product] > 0) {
        products[product]--;
        updateInventory();
    } else {
        alert(`${product} 庫存已經用完!`);
    }
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
        table += `<td>${date}</td>`;
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
