<index.html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>CH&T Transport System</title>

<style>
body{font-family:Arial;margin:0;background:#f5f7fa}
header{background:#1e3a8a;color:#fff;padding:20px;text-align:center}
section{padding:30px}
.box{background:#fff;padding:20px;border-radius:10px;max-width:900px;margin:auto;margin-bottom:20px}
button{padding:10px 20px;margin:5px;background:#2563eb;color:#fff;border:none;border-radius:5px}
.line-btn{background:#06C755}
table{width:100%;border-collapse:collapse;margin-top:20px}
th,td{border:1px solid #ccc;padding:8px;text-align:center}
.vip{background:#ffe4b5}
</style>
</head>

<body>

<header>
<h1>CH&T Transport LTD.,PART.</h1>
<p>Container Trucking | 集装箱运输 | ขนส่งตู้คอนเทนเนอร์</p>
</header>

<!-- 油价面板 -->
<section>
<div class="box">
<h2>Fuel Price (Auto)</h2>
<h3 id="fuelDisplay">Loading...</h3>
<button onclick="manualFuel()">Manual Input</button>
</div>
</section>

<!-- 报价 -->
<section>
<div class="box">
<h2>Smart Quote (Rayong Amata)</h2>

<select id="container">
<option value="20GP">20GP</option>
<option value="40HQ">40HQ</option>
</select>

<br>

<button onclick="calculatePrice()">Get Price</button>
<button class="line-btn" onclick="openLine()">LINE</button>

<h3 id="result"></h3>
</div>
</section>

<!-- CRM -->
<section>
<div class="box">
<h2>Customer CRM</h2>
<table id="table">
<thead>
<tr>
<th>Container</th>
<th>Fuel</th>
<th>Price</th>
<th>Level</th>
<th>Time</th>
</tr>
</thead>
<tbody></tbody>
</table>
</div>
</section>

<script>

// ⚠️ 必改
const LINE_ID = "pattamon";
const GOOGLE_FORM_URL = "YOUR_GOOGLE_FORM_URL";

// 自动抓油价（可替换API）
async function fetchFuel(){
  try{
    let res = await fetch("https://api.exchangerate.host/latest"); // 占位API
    return 32; // 默认模拟泰国柴油
  }catch{
    return 32;
  }
}

// 手动输入
async function manualFuel(){
  let val = prompt("Enter diesel price:");
  if(val){
    document.getElementById("fuelDisplay").innerText = val + " THB/L";
    window.currentFuel = parseFloat(val);
  }
}

// 油价显示
async function initFuel(){
  let fuel = await fetchFuel();
  window.currentFuel = fuel;
  document.getElementById("fuelDisplay").innerText = fuel + " THB/L";
}

// 运价逻辑（你给的规则）
function calcPrice(fuel){

if(fuel>=28 && fuel<=30) return 4800;
if(fuel>30 && fuel<=32) return 5040;
if(fuel>32 && fuel<=35) return 5280;
if(fuel>35 && fuel<=37) return 5520;
if(fuel>37 && fuel<=40) return 5760;

if(fuel>40){
let extra=Math.floor((fuel-40)/3);
return Math.round(5760*(1+extra*0.05));
}

return 4800;
}

// 客户等级
function getLevel(){
let leads=JSON.parse(localStorage.getItem('leads')||'[]');
if(leads.length>=5) return "VIP";
if(leads.length>=2) return "Repeat";
return "New";
}

// 主函数
function calculatePrice(){

let fuel = window.currentFuel || 32;
let container=document.getElementById("container").value;

let price = calcPrice(fuel);
let level = getLevel();

document.getElementById("result").innerText =
`Fuel: ${fuel} | Price: ${price} THB (${level})`;

let data={
container,
fuel,
price,
level,
time:new Date().toLocaleString()
};

save(data);
updateTable();
sendToGoogle(data);
sendToLine(data);

}

// 本地存储
function save(d){
let leads=JSON.parse(localStorage.getItem('leads')||'[]');
leads.push(d);
localStorage.setItem('leads',JSON.stringify(leads));
}

// 表格
function updateTable(){
let leads=JSON.parse(localStorage.getItem('leads')||'[]');
let tbody=document.querySelector("#table tbody");
tbody.innerHTML="";
leads.forEach(l=>{
let cls=l.level==="VIP"?"vip":"";
tbody.innerHTML+=`<tr class="${cls}">
<td>${l.container}</td>
<td>${l.fuel}</td>
<td>${l.price}</td>
<td>${l.level}</td>
<td>${l.time}</td>
</tr>`;
});
}

// Google Sheet
function sendToGoogle(d){
fetch(GOOGLE_FORM_URL,{
method:'POST',
mode:'no-cors',
headers:{'Content-Type':'application/x-www-form-urlencoded'},
body:`entry.111111=${d.container}&entry.222222=${d.fuel}&entry.333333=${d.price}&entry.444444=${d.time}`
});
}

// LINE消息（三语）
function sendToLine(d){
let msg=`EN:
Rayong Amata → Laem Chabang
Fuel:${d.fuel}
Price:${d.price}

中文:
罗勇Amata → 林查班
油价:${d.fuel}
价格:${d.price}

ไทย:
ระยอง Amata → แหลมฉบัง
น้ำมัน:${d.fuel}
ราคา:${d.price}`;

let url=`https://line.me/ti/p/~${LINE_ID}?text=${encodeURIComponent(msg)}`;
setTimeout(()=>window.open(url,'_blank'),1000);
}

// 打开LINE
function openLine(){
window.open(`https://line.me/ti/p/~${LINE_ID}`,'_blank');
}

initFuel();
updateTable();

</script>

</body>
</html>
