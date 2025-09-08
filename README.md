<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<title>Werewolf Party</title>
<style>
body { 
  font-family: 'Kanit', Arial, sans-serif; 
  background: linear-gradient(to bottom, #f5f7fa, #c3cfe2); 
  color: #222; 
  text-align: center; 
  margin: 0;
  padding: 0;
}
h1 { font-size: 3em; margin-top: 20px; }
h2 { font-size: 2em; margin: 15px 0; }
.card { 
  border: 2px solid #555; 
  padding: 30px; 
  margin: 20px auto; 
  width: 260px; 
  background: #fff; 
  border-radius: 15px; 
  font-size: 1.5em; 
  color: #222;
  box-shadow: 0 0 15px rgba(0,0,0,0.2);
  position: relative;
}
.card::before { content: "🃏"; position: absolute; top: 5px; right: 10px; font-size: 1.2em; }
button { padding: 15px 30px; margin: 10px; border: none; border-radius: 8px; cursor: pointer; font-size: 1.2em; transition: 0.2s; }
button:hover { opacity: 0.85; transform: scale(1.05); }
.btn { background: #4CAF50; color: white; }
.btn-close { background: #f44336; color: white; }
input[type=number] { width: 70px; font-size: 1.2em; text-align: center; border-radius: 5px; border: 1px solid #ccc; padding: 5px; }
.role-input { margin: 8px; font-size: 1.2em; }
#nightResult { margin-top: 15px; color: #D35400; font-size: 1.4em; }
#voteTimer { margin-top: 15px; font-size:1.5em; color:#2E86C1; font-weight:bold; }
.player-btn { margin: 5px; padding: 10px 20px; font-size: 1.2em; border-radius:5px; color:white; background:#4CAF50; }
#nightChoice { margin-top:20px; display:none; }
</style>
</head>
<body>
<h1>🐺 Werewolf Party 🃏</h1>

<div id="setup">
  <h2>เลือกจำนวนบทบาท</h2>
  <div class="role-input">หมาป่า: <input type="number" id="wolfCount" value="1" min="0"></div>
  <div class="role-input">ผู้ทำนาย: <input type="number" id="seerCount" value="1" min="0"></div>
  <div class="role-input">บอดี้การ์ด: <input type="number" id="guardCount" value="1" min="0"></div>
  <div class="role-input">ชาวบ้าน: <input type="number" id="villCount" value="3" min="0"></div>
  <button class="btn" onclick="setupGame()">เริ่มเกม</button>
</div>

<div id="game" style="display:none;">
  <h2 id="status">แจกการ์ด...</h2>
  <div class="card" id="cardArea">กดปุ่มเพื่อหยิบการ์ด</div>
  <button class="btn" id="drawBtn" onclick="drawCard()">หยิบการ์ด</button>
  <button class="btn-close" id="closeBtn" style="display:none;" onclick="closeCard()">ปิดการ์ด</button>

  <div id="actionButtons" style="display:none;">
    <button class="btn" onclick="showNightChoices()">กลางคืน</button>
    <button class="btn" onclick="dayVote()">กลางวัน/โหวต</button>
  </div>

  <div id="nightResult"></div>
  <div id="voteTimer"></div>

  <div id="nightChoice">
    <h3>หมาป่าเลือกใคร?</h3>
    <div id="wolfOptions"></div>
    <h3>บอดี้การ์ดเลือกใคร?</h3>
    <div id="guardOptions"></div>
    <h3>ผู้ทำนายตรวจใคร?</h3>
    <div id="seerOptions"></div>
    <button class="btn" onclick="resolveNight()">ดำเนินการกลางคืน</button>
  </div>
</div>

<script>
let roles=[], players=[], current=0, revealed=false, round=1;
let wolfTarget=null, guardTarget=null, seerTarget=null;

function setupGame() {
  const wolf = parseInt(document.getElementById("wolfCount").value)||0;
  const seer = parseInt(document.getElementById("seerCount").value)||0;
  const guard = parseInt(document.getElementById("guardCount").value)||0;
  const vill = parseInt(document.getElementById("villCount").value)||0;
  const total = wolf+seer+guard+vill;
  if(total<5){ alert("ควรมีผู้เล่นอย่างน้อย 5 คน!"); return; }

  roles=[];
  for(let i=0;i<wolf;i++) roles.push("หมาป่า");
  for(let i=0;i<seer;i++) roles.push("ผู้ทำนาย");
  for(let i=0;i<guard;i++) roles.push("บอดี้การ์ด");
  for(let i=0;i<vill;i++) roles.push("ชาวบ้าน");
  roles=shuffle(roles);

  players=Array.from({length:total},(_,i)=>({id:i+1,role:null,alive:true}));
  current=0;

  document.getElementById("setup").style.display="none";
  document.getElementById("game").style.display="block";
  document.getElementById("status").innerText="ผู้เล่นคนที่ 1 หยิบการ์ด";
  document.getElementById("cardArea").innerText="การ์ดยังไม่ถูกเปิด";
}

function drawCard(){
  if(current>=players.length){
    document.getElementById("status").innerText="แจกการ์ดครบแล้ว!";
    document.getElementById("cardArea").innerText="ทุกคนมีการ์ดแล้ว 🎉";
    document.getElementById("drawBtn").style.display="none";
    document.getElementById("actionButtons").style.display="block";
    return;
  }
  const role = roles.pop();
  players[current].role=role;
  document.getElementById("cardArea").innerText="คุณคือ: "+role;
  document.getElementById("drawBtn").style.display="none";
  document.getElementById("closeBtn").style.display="inline-block";
  revealed=true;
}

function closeCard(){
  if(!revealed) return;
  revealed=false;
  current++;
  document.getElementById("cardArea").innerText="การ์ดยังไม่ถูกเปิด";
  document.getElementById("closeBtn").style.display="none";
  if(current<players.length){
    document.getElementById("status").innerText="ส่งมือถือให้ผู้เล่นคนที่ "+(current+1);
    document.getElementById("drawBtn").style.display="inline-block";
  } else {
    document.getElementById("status").innerText="แจกการ์ดครบแล้ว!";
    document.getElementById("actionButtons").style.display="block";
  }
}

function showNightChoices(){
  wolfTarget=null; guardTarget=null; seerTarget=null;
  const alive=players.filter(p=>p.alive);

  const wolfDiv=document.getElementById("wolfOptions"); wolfDiv.innerHTML="";
  alive.forEach(p=>{
    let b=document.createElement("button"); 
    b.className="player-btn"; 
    b.innerText=`ผู้เล่น ${p.id} (${p.role})`; 
    b.onclick=()=>{ wolfTarget=p.id; updateNightButtons(); }; 
    wolfDiv.appendChild(b);
  });

  const guardDiv=document.getElementById("guardOptions"); guardDiv.innerHTML="";
  alive.forEach(p=>{
    let b=document.createElement("button"); 
    b.className="player-btn"; 
    b.innerText=`ผู้เล่น ${p.id} (${p.role})`; 
    b.onclick=()=>{ guardTarget=p.id; updateNightButtons(); }; 
    guardDiv.appendChild(b);
  });

  const seerDiv=document.getElementById("seerOptions"); seerDiv.innerHTML="";
  alive.forEach(p=>{
    let b=document.createElement("button"); 
    b.className="player-btn"; 
    b.innerText=`ผู้เล่น ${p.id} (${p.role})`; 
    b.onclick=()=>{ seerTarget=p.id; updateNightButtons(); }; 
    seerDiv.appendChild(b);
  });

  document.getElementById("nightChoice").style.display="block";
}

function updateNightButtons(){
  document.querySelectorAll("#wolfOptions .player-btn").forEach(b=>{
    if(parseInt(b.innerText.split(" ")[1])===wolfTarget) b.style.background="#f44336";
    else b.style.background="#4CAF50";
  });
  document.querySelectorAll("#guardOptions .player-btn").forEach(b=>{
    if(parseInt(b.innerText.split(" ")[1])===guardTarget) b.style.background="#f44336";
    else b.style.background="#4CAF50";
  });
  document.querySelectorAll("#seerOptions .player-btn").forEach(b=>{
    if(parseInt(b.innerText.split(" ")[1])===seerTarget) b.style.background="#f44336";
    else b.style.background="#4CAF50";
  });
}

function resolveNight(){
  const resultDiv=document.getElementById("nightResult");
  resultDiv.innerHTML="";
  if(wolfTarget && wolfTarget!==guardTarget){
    players.find(p=>p.id===wolfTarget).alive=false;
    resultDiv.innerHTML+=`💀 ผู้เล่น ${wolfTarget} ถูกฆ่าเมื่อคืนนี้!<br>`;
  } else if(wolfTarget){
    resultDiv.innerHTML+=`✨ ผู้เล่น ${wolfTarget} รอดเพราะบอดี้การ์ดช่วยไว้!<br>`;
  }

  if(seerTarget){
    const text=players.find(p=>p.id===seerTarget).role==="หมาป่า" ? "ใช่ หมาป่า" : "ไม่ใช่หมาป่า";
    resultDiv.innerHTML+=`🔮 ผู้ทำนายตรวจผู้เล่น ${seerTarget}: ${text}<br>`;
  }

  document.getElementById("nightChoice").style.display="none";
  checkWin();
}

function dayVote(){
  let alive=players.filter(p=>p.alive);
  let timeLeft=6*60;
  const timerDiv=document.getElementById("voteTimer");
  timerDiv.innerText=formatTime(timeLeft);

  let countdown=setInterval(()=>{
    timeLeft--;
    timerDiv.innerText=formatTime(timeLeft);
    if(timeLeft<=0){
      clearInterval(countdown);
      timerDiv.innerText="หมดเวลาโหวต!";
      alert("⏰ หมดเวลาโหวตแล้ว!");
      checkWin();
    }
  },1000);

  let vote=parseInt(prompt("โหวตออก (พิมพ์หมายเลข)"));
  clearInterval(countdown);
  timerDiv.innerText="";
  players.forEach(p=>{ if(p.id===vote) p.alive=false; });
  alert("☠️ ผู้เล่น "+vote+" ถูกโหวตออก!");
  round++;
  checkWin();
}

function checkWin(){
  let wolves=players.filter(p=>p.alive&&p.role==="หมาป่า");
  let villagers=players.filter(p=>p.alive&&p.role!=="หมาป่า");
  if(wolves.length===0){ alert("🎉 ชาวบ้านชนะ! 🎉"); }
  else if(wolves.length>=villagers.length){ alert("🐺 หมาป่าชนะ! 🐺"); }
}

function shuffle(arr){for(let i=arr.length-1;i>0;i--){let j=Math.floor(Math.random()*(i+1));[arr[i],arr[j]]=[arr[j],arr[i]];} return arr;}
function formatTime(sec){let m=Math.floor(sec/60),s=sec%60;return `${m.toString().padStart(2,'0')}:${s.toString().padStart(2,'0')}`;}
</script>
</body>
</html>
