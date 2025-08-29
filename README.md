<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<title>跨日期固定庫存系統（可修改每日庫存）</title>
<style>
body{font-family:Arial,sans-serif;background:#f7f7f7;margin:20px;}
h1,h2{text-align:center;}
.calendar{display:grid;grid-template-columns:repeat(7,1fr);gap:5px;margin-bottom:20px;}
.day{background:#fff;padding:10px;text-align:center;cursor:pointer;border:1px solid #ddd;border-radius:6px;}
.day:hover{background:#d0e6ff;}
.selected{background:#4caf50 !important;color:white;font-weight:bold;}
.inventory,.summary{margin-top:20px;padding:15px;background:white;border-radius:8px;box-shadow:0 0 6px rgba(0,0,0,0.1);}
li{margin:4px 0;}
.out{color:red;font-weight:bold;}
.download-btn{display:block;margin:15px auto 0;padding:8px 16px;background:#4caf50;color:white;border:none;border-radius:6px;cursor:pointer;}
.download-btn:hover{background:#45a049;}
select,button,input{padding:4px;margin-left:5px;width:80px;}
input.qtyInput{width:50px;}
</style>
</head>
<body>
<h1>📅 跨日期固定庫存系統</h1>
<div id="calendar" class="calendar"></div>

<div class="inventory">
<h2 id="selectedDateTitle"></h2>
<ul id="inventoryList"></ul>
</div>

<div class="summary">
<h2>📊 匯出 Excel 報表</h2>
<button class="download-btn" onclick="downloadExcel()">📥 下載 Excel 報表</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<script>
const calendarEl=document.getElementById("calendar");
const inventoryList=document.getElementById("inventoryList");
const selectedDateTitle=document.getElementById("selectedDateTitle");
let selectedDate="2025-09-01";

const fixedItems=[
  {name:"加床", qty:13},
  {name:"嬰兒床", qty:8},
  {name:"嬰兒澡盆", qty:11},
  {name:"床圍", qty:3},
  {name:"消毒鍋", qty:4}
];

// 初始化每日庫存
function initDailyInventory(date){
  let data=JSON.parse(localStorage.getItem("inventory"))||{};
  if(!data[date]){
    data[date]=fixedItems.map(item=>{
      let code=Math.floor(1000+Math.random()*9000).toString();
      return {name:item.name, qty:item.qty, code:code};
    });
    localStorage.setItem("inventory",JSON.stringify(data));
  }
  return data;
}

function renderCalendar(){
  const start=new Date("2025-09-01");
  const end=new Date("2025-12-31");
  calendarEl.innerHTML="";
  let date=new Date(start);
  while(date<=end){
    const isoDate=date.toISOString().split("T")[0];
    const div=document.createElement("div");
    div.className="day";
    div.innerText=`${date.getMonth()+1}/${date.getDate()}`;
    if(isoDate===selectedDate) div.classList.add("selected");
    div.onclick=()=>{selectedDate=isoDate;renderCalendar();loadInventory();};
    calendarEl.appendChild(div);
    date.setDate(date.getDate()+1);
  }
}

function loadInventory(){
  selectedDateTitle.innerText=`📦 ${selectedDate} 的庫存`;
  let data=initDailyInventory(selectedDate);
  let items=data[selectedDate];
  inventoryList.innerHTML="";
  items.forEach((item,index)=>{
    let li=document.createElement("li");
    li.innerHTML=`
      ${item.name} - 剩餘 
      <input type="number" class="qtyInput" id="qty${index}" value="${item.qty}" min="0">
      - 代號 ${item.code} 
      ${item.qty>0?
        `<select id="sel${index}">
          ${getAvailableCodes(item.name).map(c=>`<option value="${c}">${c}</option>`).join("")}
        </select>
        <button onclick="useSelected(${index})">使用1</button>`:
        '<span class="out">⚠️ 庫存不足</span>'}
      <button onclick="updateQty(${index})">更新數量</button>
    `;
    inventoryList.appendChild(li);
  });
}

// 取得當天該品項可用代號
function getAvailableCodes(name){
  let data=JSON.parse(localStorage.getItem("inventory"))||{};
  return (data[selectedDate]||[]).filter(x=>x.name===name && x.qty>0).map(x=>x.code);
}

// 使用下拉選擇代號扣庫
function useSelected(index){
  let data=JSON.parse(localStorage.getItem("inventory"))||{};
  let select=document.getElementById(`sel${index}`);
  if(!select) return;
  let code=select.value;
  let item=data[selectedDate].find(x=>x.code===code);
  if(item && item.qty>0) item.qty--;
  localStorage.setItem("inventory",JSON.stringify(data));
  loadInventory();
}

// 更新每日庫存數量
function updateQty(index){
  let data=JSON.parse(localStorage.getItem("inventory"))||{};
  let input=document.getElementById(`qty${index}`);
  let val=parseInt(input.value);
  if(isNaN(val)||val<0) return alert("請輸入正確數量");
  data[selectedDate][index].qty=val;
  localStorage.setItem("inventory",JSON.stringify(data));
  loadInventory();
}

// 匯出 Excel
function downloadExcel(){
  let data=JSON.parse(localStorage.getItem("inventory"))||{};
  let detailSheetData=[["日期","品項","數量","訂房代號"]];
  Object.keys(data).sort().forEach(date=>{
    data[date].forEach(item=>{
      detailSheetData.push([date,item.name,item.qty,item.code]);
    });
  });
  let ws=XLSX.utils.aoa_to_sheet(detailSheetData);
  let wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,"每日明細");
  XLSX.writeFile(wb,"庫存報表.xlsx");
}

renderCalendar();
loadInventory();
</script>
</body>
</html>
