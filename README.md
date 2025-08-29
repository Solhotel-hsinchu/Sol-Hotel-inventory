<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<title>每日代號批量使用明細庫存系統</title>
<style>
body{font-family:Arial,sans-serif;background:#f7f7f7;margin:20px;}
h1,h2{text-align:center;}
.calendar{display:grid;grid-template-columns:repeat(7,1fr);gap:5px;margin-bottom:20px;}
.day{background:#fff;padding:10px;text-align:center;cursor:pointer;border:1px solid #ddd;border-radius:6px;}
.day:hover{background:#d0e6ff;}
.selected{background:#4caf50 !important;color:white;font-weight:bold;}
.inventory,.summary{margin-top:20px;padding:15px;background:white;border-radius:8px;box-shadow:0 0 6px rgba(0,0,0,0.1);}
ul{list-style:none;padding:0;}
li{margin:5px 0;}
input,button,textarea{padding:4px;margin:2px;width:300px;}
.out{color:red;font-weight:bold;}
.download-btn{display:block;margin:15px auto;padding:8px 16px;background:#4caf50;color:white;border:none;border-radius:6px;cursor:pointer;}
.download-btn:hover{background:#45a049;}
</style>
</head>
<body>
<h1>📅 每日代號批量使用明細庫存系統</h1>
<div id="calendar" class="calendar"></div>

<div class="inventory">
<h2 id="selectedDateTitle"></h2>
<div>
<label>訂房代號:</label>
<input type="text" id="bookingCode" placeholder="四碼代號"><br>
<label>批量輸入品項數量（格式: 品項:數量, 品項:數量）:</label><br>
<textarea id="batchInput" rows="2" placeholder="加床:1, 嬰兒床:1, 嬰兒澡盆:2"></textarea><br>
<button onclick="useBatch()">使用</button>
</div>

<h3>📦 當日剩餘庫存</h3>
<ul id="inventoryList"></ul>

<h3>📋 當日使用明細</h3>
<ul id="usageList"></ul>
</div>

<div class="summary">
<h2>📊 匯出 Excel 報表</h2>
<button class="download-btn" onclick="downloadExcel()">📥 下載 Excel 報表</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<script>
const calendarEl=document.getElementById("calendar");
const selectedDateTitle=document.getElementById("selectedDateTitle");
const inventoryList=document.getElementById("inventoryList");
const usageList=document.getElementById("usageList");

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
  let usage=JSON.parse(localStorage.getItem("usage"))||{};
  if(!data[date]){
    data[date]=fixedItems.map(item=>({name:item.name, qty:item.qty}));
    localStorage.setItem("inventory",JSON.stringify(data));
  }
  if(!usage[date]){ usage[date]={}; localStorage.setItem("usage",JSON.stringify(usage)); }
  return data;
}

// 日曆
function renderCalendar(){
  const start=new Date("2025-09-01");
  const end=new Date("2025-12-31");
  calendarEl.innerHTML="";
  let date=new Date(start);
  while(date<=end){
    const iso=date.toISOString().split("T")[0];
    const div=document.createElement("div");
    div.className="day";
    div.innerText=`${date.getMonth()+1}/${date.getDate()}`;
    if(iso===selectedDate) div.classList.add("selected");
    div.onclick=()=>{selectedDate=iso;renderCalendar();loadData();};
    calendarEl.appendChild(div);
    date.setDate(date.getDate()+1);
  }
}

// 載入資料
function loadData(){
  selectedDateTitle.innerText=`📦 ${selectedDate} 庫存`;
  let data=initDailyInventory(selectedDate);
  let items=data[selectedDate];

  // 顯示庫存
  inventoryList.innerHTML="";
  items.forEach(it=>{
    let li=document.createElement("li");
    li.textContent=`${it.name} - 剩餘 ${it.qty}`;
    if(it.qty<=0){ li.classList.add("out"); li.textContent+=" ⚠️ 庫存不足"; }
    inventoryList.appendChild(li);
  });

  // 顯示使用明細
  let usage=JSON.parse(localStorage.getItem("usage"))||{};
  let usageToday=usage[selectedDate]||{};
  usageList.innerHTML="";
  Object.keys(usageToday).forEach(code=>{
    let line=`訂房代號${code} 使用了: `;
    let parts=[];
    Object.keys(usageToday[code]).forEach(name=>{
      parts.push(`${name}*${usageToday[code][name]}`);
    });
    line+=parts.join(", ");
    let li=document.createElement("li"); li.textContent=line;
    usageList.appendChild(li);
  });
}

// 批量使用
function useBatch(){
  let code=document.getElementById("bookingCode").value.trim();
  let batch=document.getElementById("batchInput").value.trim();
  if(!code.match(/^\d{4}$/)){ alert("請輸入正確四碼代號"); return; }
  if(!batch){ alert("請輸入品項數量"); return; }

  let data=initDailyInventory(selectedDate);
  let items=data[selectedDate];
  let usage=JSON.parse(localStorage.getItem("usage"))||{};
  if(!usage[selectedDate]) usage[selectedDate]={};
  if(!usage[selectedDate][code]) usage[selectedDate][code]={};

  let entries=batch.split(",").map(s=>s.trim());
  for(let e of entries){
    let [name,val]=e.split(":").map(s=>s.trim());
    let qty=parseInt(val);
    if(isNaN(qty)||qty<1){ alert("格式錯誤或數量不正確"); return; }
    let target=items.find(it=>it.name===name);
    if(!target){ alert(`品項 ${name} 不存在`); return; }
    if(target.qty<qty){ alert(`庫存不足: ${name}`); return; }

    target.qty-=qty;
    if(!usage[selectedDate][code][name]) usage[selectedDate][code][name]=0;
    usage[selectedDate][code][name]+=qty;
  }

  localStorage.setItem("inventory",JSON.stringify(data));
  localStorage.setItem("usage",JSON.stringify(usage));
  document.getElementById("batchInput").value="";
  loadData();
}

// 匯出 Excel
function downloadExcel(){
  let data=JSON.parse(localStorage.getItem("usage"))||{};
  let rows=[["日期","訂房代號","品項","數量"]];
  Object.keys(data).sort().forEach(date=>{
    let day=data[date];
    Object.keys(day).forEach(code=>{
      let items=day[code];
      Object.keys(items).forEach(name=>{
        rows.push([date,code,name,items[name]]);
      });
    });
  });
  let ws=XLSX.utils.aoa_to_sheet(rows);
  let wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,"每日使用明細");
  XLSX.writeFile(wb,"每日使用明細.xlsx");
}

renderCalendar();
loadData();
</script>
</body>
</html>
