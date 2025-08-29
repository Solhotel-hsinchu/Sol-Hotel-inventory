<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<title>跨日期庫存系統</title>
<style>
body { font-family: Arial, sans-serif; background: #f7f7f7; margin: 20px; }
h1,h2{text-align:center;}
.calendar{display:grid;grid-template-columns:repeat(7,1fr);gap:5px;margin-bottom:20px;}
.day{background:#fff;padding:10px;text-align:center;cursor:pointer;border:1px solid #ddd;border-radius:6px;}
.day:hover{background:#d0e6ff;}
.selected{background:#4caf50 !important;color:white;font-weight:bold;}
.inventory,.summary{margin-top:20px;padding:15px;background:white;border-radius:8px;box-shadow:0 0 6px rgba(0,0,0,0.1);}
input,button,select{padding:6px;margin:3px;}
li{margin:4px 0;}
.out{color:red;font-weight:bold;}
.download-btn{display:block;margin:15px auto 0;padding:8px 16px;background:#4caf50;color:white;border:none;border-radius:6px;cursor:pointer;}
.download-btn:hover{background:#45a049;}
</style>
</head>
<body>
<h1>📅 跨日期庫存系統</h1>
<div id="calendar" class="calendar"></div>

<div class="inventory">
<h2 id="selectedDateTitle"></h2>
<div id="inputContainer">
<script>
for(let i=1;i<=10;i++){
document.write(`
<input type="text" id="item${i}" placeholder="品項${i}">
<input type="number" id="qty${i}" placeholder="數量" min="1">
<input type="text" id="code${i}" placeholder="四碼代號"><br>
`);
}
</script>
<button onclick="addItems()">新增品項</button>
</div>
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
  inventoryList.innerHTML="";
  let data=JSON.parse(localStorage.getItem("inventory"))||{};
  let items=data[selectedDate]||[];
  items.forEach((item,index)=>{
    let li=document.createElement("li");
    li.innerHTML=`
      ${item.name} - 剩餘 ${item.qty} - 代號 ${item.code} 
      ${item.qty>0?`<button onclick="useItem(${index})">使用1</button>`:'<span class="out">⚠️ 庫存不足</span>'}
    `;
    inventoryList.appendChild(li);
  });
}

function addItems(){
  let data=JSON.parse(localStorage.getItem("inventory"))||{};
  if(!data[selectedDate]) data[selectedDate]=[];
  for(let i=1;i<=10;i++){
    let name=document.getElementById(`item${i}`).value.trim();
    let qty=parseInt(document.getElementById(`qty${i}`).value);
    let code=document.getElementById(`code${i}`).value.trim();
    if(name&&qty>0&&code&&/^\d{4}$/.test(code)){
      data[selectedDate].push({name,qty,code});
      document.getElementById(`item${i}`).value="";
      document.getElementById(`qty${i}`).value="";
      document.getElementById(`code${i}`).value="";
    }
  }
  localStorage.setItem("inventory",JSON.stringify(data));
  loadInventory();
}

// 使用1功能，選擇代號扣庫
function useItem(index){
  let data=JSON.parse(localStorage.getItem("inventory"))||{};
  if(!data[selectedDate]) return;
  // 先找所有同品項的代號列表
  const item=data[selectedDate][index];
  let sameItems=data[selectedDate].filter(x=>x.name===item.name && x.qty>0);
  if(sameItems.length>1){
    let codeList=sameItems.map(x=>x.code);
    let code=prompt(`請輸入代號使用1，選項: ${codeList.join(",")}`);
    let target=sameItems.find(x=>x.code===code);
    if(target) target.qty--;
  } else if(sameItems.length===1){
    sameItems[0].qty--;
  }
  localStorage.setItem("inventory",JSON.stringify(data));
  loadInventory();
}

function downloadExcel(){
  let data=JSON.parse(localStorage.getItem("inventory"))||{};
  let detailSheetData=[["日期","品項","數量","訂房代號"]];
  Object.keys(data).sort().forEach(date=>{
    data[date].forEach(item=>{
      detailSheetData.push([date,item.name,item.qty,item.code]);
    });
  });
  let wsDetail=XLSX.utils.aoa_to_sheet(detailSheetData);
  let wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,wsDetail,"每日明細");
  XLSX.writeFile(wb,"庫存報表.xlsx");
}

renderCalendar();
loadInventory();
</script>
</body>
</html>
