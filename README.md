<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>庫存系統與月份切換月曆</title>
<style>
body { font-family: Arial, sans-serif; background-color: #f4f4f4; text-align: center; margin:0; padding:20px;}
h1 { margin-bottom:10px; }
.inventory, .calendar { margin:20px auto; }
table { margin:0 auto; border-collapse:collapse; }
td, th { border:1px solid #555; padding:5px; width:60px; height:60px; vertical-align:top; position:relative; }
th { background-color:#ddd; }
.product-btn, #restockBtn, #prevMonth, #nextMonth { margin:5px; padding:10px 20px; cursor:pointer; border:none; border-radius:5px; background-color:#4CAF50; color:white; transition:0.3s;}
.product-btn:disabled { background-color:#ccc; cursor:not-allowed; }
.low-stock { background-color:#ffeb3b; }
.out-stock { background-color:#f44336; color:white; }
.today { background-color:#2196F3; color:white; font-weight:bold; }
#log { margin-top:10px; font-weight:bold; min-height:24px; }
.tag { display:inline-block; width:10px; height:10px; border-radius:50%; margin:1px;}
.A { background-color:#f44336; }  /* 產品A紅 */
.B { background-color:#ff9800; }  /* 產品B橙 */
.C { background-color:#ffeb3b; }  /* 產品C黃 */
.D { background-color:#4CAF50; }  /* 產品D綠 */
.E { background-color:#2196F3; }  /* 產品E藍 */
td span { font-size:10px; display:block; margin-top:2px; }
.month-nav { margin:10px 0; font-weight:bold; }
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

<h1>月曆</h1>
<div class="month-nav">
    <button id="prevMonth" onclick="changeMonth(-1)">上一個月</button>
    <span id="currentMonth"></span>
    <button id="nextMonth" onclick="changeMonth(1)">下一個月</button>
</div>
<div class="calendar" id="calendar"></div>
<div id="log"></div>

<script>
// 初始資料
let products = { "產品A":8, "產品B":8, "產品C":8, "產品D":8, "產品E":8 };
let usageLog = {}; // YYYY-MM-DD => [產品名稱]
const productColors = { "產品A":"A","產品B":"B","產品C":"C","產品D":"D","產品E":"E" };

// 目前月曆顯示年月
let currentYear = new Date().getFullYear();
let currentMonth = new Date().getMonth(); // 0~11

// 更新庫存
function updateInventory() {
    const table = document.getElementById("inventoryTable");
    table.innerHTML = "<tr><th>產品</th><th>庫存</th></tr>";
    for(let p in products){
        const row=table.insertRow();
        const nameCell=row.insertCell(0);
        const stockCell=row.insertCell(1);
        nameCell.innerText=p;
        stockCell.innerText=products[p];
        if(products[p]===0) stockCell.className="out-stock";
        else if(products[p]<=3) stockCell.className="low-stock";
        else stockCell.className="";
    }
    document.querySelectorAll(".product-btn").forEach(btn=>{
        btn.disabled = products[btn.dataset.product]===0;
    });
}

// 建立按鈕
function createButtons(){
    const btnDiv=document.getElementById("buttons");
    for(let p in products){
        const btn=document.createElement("button");
        btn.innerText=`使用 ${p}`;
        btn.className="product-btn";
        btn.dataset.product=p;
        btn.onclick=()=>useProduct(p);
        btnDiv.appendChild(btn);
    }
}

// 使用產品
function useProduct(product){
    if(products[product]>0){
        products[product]--;
        updateInventory();
        const todayStr = new Date().toISOString().split('T')[0];
        if(!usageLog[todayStr]) usageLog[todayStr]=[];
        usageLog[todayStr].push(product);
        showCalendar();
        showLog(todayStr);
    }
}

// 補貨
function restock(){
    for(let p in products) products[p]=8;
    updateInventory();
    showCalendar();
    document.getElementById("log").innerText="";
}

// 顯示月曆
function showCalendar(){
    const calDiv=document.getElementById("calendar");
    const now=new Date();
    const year=currentYear;
    const month=currentMonth;
    const firstDay=new Date(year,month,1).getDay();
    const lastDate=new Date(year,month+1,0).getDate();
    document.getElementById("currentMonth").innerText=`${year} 年 ${month+1} 月`;

    let table="<table><tr>";
    ["日","一","二","三","四","五","六"].forEach(d=>table+=`<th>${d}</th>`);
    table+="</tr><tr>";

    for(let i=0;i<firstDay;i++) table+="<td></td>";

    for(let date=1;date<=lastDate;date++){
        const dateStr=`${year}-${String(month+1).padStart(2,'0')}-${String(date).padStart(2,'0')}`;
        let classes="";
        if(date===new Date().getDate() && year===now.getFullYear() && month===now.getMonth()) classes="today";

        let tags="";
        if(usageLog[dateStr]){
            usageLog[dateStr].forEach(p=>{
                tags+=`<span class="tag ${productColors[p]}" title="${p}"></span>`;
            });
        }

        table+=`<td class="${classes}" onclick="showLog('${dateStr}')" title="${usageLog[dateStr]?usageLog[dateStr].join(', '):''}">${date}${tags}</td>`;
        if((date+firstDay)%7===0) table+="</tr><tr>";
    }
    table+="</tr></table>";
    calDiv.innerHTML=table;
}

// 顯示當天使用紀錄
function showLog(dateStr){
    const logDiv=document.getElementById("log");
    if(usageLog[dateStr] && usageLog[dateStr].length>0)
        logDiv.innerText=`${dateStr} 使用產品: ${usageLog[dateStr].join(', ')}`;
    else
        logDiv.innerText=`${dateStr} 沒有使用產品`;
}

// 切換月份
function changeMonth(offset){
    currentMonth += offset;
    if(currentMonth<0){ currentMonth=11; currentYear--; }
    else if(currentMonth>11){ currentMonth=0; currentYear++; }
    showCalendar();
    document.getElementById("log").innerText="";
}

// 初始化
updateInventory();
createButtons();
showCalendar();
</script>

</body>
</html>
