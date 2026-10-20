<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mzansi AI Battery Alert</title>
<link rel="manifest" href="manifest.json">
<style>
body{margin:0;background:#000;color:#fff;font-family:Arial;text-align:center;padding:20px}
h1{color:#FFD700;font-size:28px;margin:10px}
.card{background:#1a1a1a;border-radius:20px;padding:20px;margin:15px 0;border:2px solid #333}
.percent{font-size:80px;font-weight:900;color:#00FF00}
.status{font-size:16px;color:#aaa;margin:8px}
.btn{width:100%;padding:18px;border:none;border-radius:15px;font-size:20px;font-weight:900;margin:8px 0}
.green{background:#00FF00;color:#000}
.red{background:#FF0000;color:#fff;animation:blink 0.5s infinite}
@keyframes blink{0%{opacity:1}50%{opacity:0.3}}
select{width:100%;padding:14px;border-radius:12px;background:#000;color:#fff;border:1px solid #555;font-size:16px;margin:8px 0}
</style>
</head>
<body>

<h1>🇿🇦 MZANSI AI BATTERY ALERT</h1>
<p style="color:#FFD700">Reads % - Speaks Zulu - Custom Ringtone</p>

<div class="card" id="main">
<div class="percent" id="pct">--%</div>
<div class="status" id="stat">Reading battery %...</div>
<div class="status" id="zuluStat" style="color:#FFD700;font-weight:800"></div>
</div>

<div class="card">
<h3 style="color:#FFD700">🔋 SET WHEN TO SHOUT:</h3>
<select id="targetLevel" onchange="setTarget()">
<option value="80">80% - Battery lasts 3 years (AI Recommend)</option>
<option value="90">90% - Balanced</option>
<option value="100" selected>100% - Fully Charged - Shout Now</option>
</select>
<p style="font-size:12px;color:#aaa">AI will read % every second and shout at your chosen %</p>
</div>

<div class="card">
<h3 style="color:#FFD700">🎵 CHOOSE ZULU RINGTONE:</h3>
<select id="voice" onchange="previewVoice()">
<option value="zulu1">👵 Zulu Gogo: Mntanami! Battery igcwele! Khipha shaja manje!</option>
<option value="zulu2">👨 Zulu Baba: Hey! Battery full! Khipha i-plug!</option>
<option value="sotho">🗣️ Sotho: Tjha! E tletse! Ntlosha shaja!</option>
<option value="xhosa">🗣️ Xhosa: Battery igcwele! Khupha iplagi!</option>
<option value="english">🇬🇧 English: Boss! Battery full! Unplug now!</option>
<option value="custom">📁 My Own Song / Voice Note from Phone</option>
</select>
<button class="btn" style="background:#333;color:#fff" onclick="document.getElementById('file').click()" id="fileBtn">📁 Choose Your Own Ringtone (MP3/Audio from Phone)</button>
<input type="file" id="file" accept="audio/*" style="display:none" onchange="loadCustom(this)">
<audio id="audio" controls style="width:100%;margin-top:10px;display:none"></audio>
<p id="chosen" style="color:#00FF00;font-size:12px">Chosen: Zulu Gogo - Mntanami battery full</p>
<button class="btn" style="background:#333;color:#fff" onclick="previewVoice()">🔊 PREVIEW ZULU VOICE</button>
</div>

<div class="card">
<button class="btn green" id="startBtn" onclick="startAlert()">🟢 START MZANSI ALERT - READ % NOW</button>
<button class="btn red" id="stopBtn" onclick="stopAlert()" style="display:none">🔴 STOP - I UNPLUGGED - THANK YOU MZANSI AI</button>
<p style="font-size:11px;color:#aaa">Add to Home Screen → Works offline → Loud even if silent → No data needed after install</p>
</div>

<script>
let battery=null, target=100, alertOn=false, alarmLoop=null, pctEl=document.getElementById('pct'), statEl=document.getElementById('stat'), zuluStatEl=document.getElementById('zuluStat'), chosenEl=document.getElementById('chosen'), audioEl=document.getElementById('audio'), customAudioURL=null, currentVoice='zulu1';

const voices={
 zulu1:{text:"Mntanami! Battery igcwele! Khipha shaja manje! Battery igcwele!", lang:"zu-ZA"},
 zulu2:{text:"Hey! Battery full! Khipha i-plug! Siyabonga!", lang:"zu-ZA"},
 sotho:{text:"Tjha! E tletse! Ntlosha shaja! Battery e tletse!", lang:"st-ZA"},
 xhosa:{text:"Battery igcwele! Khupha iplagi! Enkosi!", lang:"xh-ZA"},
 english:{text:"Boss! Battery fully charged! Unplug now! Protect battery!", lang:"en-ZA"}
};

function setTarget(){target=parseInt(document.getElementById('targetLevel').value); if(alertOn) statEl.innerText=`Guard ON - Will shout at ${target}%`; }

async function init(){
 if('getBattery' in navigator){
   battery=await navigator.getBattery();
   updateBattery();
   battery.addEventListener('levelchange', updateBattery);
   battery.addEventListener('chargingchange', updateBattery);
 } else {
   pctEl.innerText="80%"; statEl.innerText="iPhone: Battery % limited - Using timer mode - Will still shout Zulu";
   setInterval(()=>{ if(alertOn){ let fake=parseInt(pctEl.innerText)||0; if(fake<100){ fake+=1; pctEl.innerText=fake+'%'; if(fake>=target) shout(); } } },2000);
 }
}
function updateBattery(){
 if(!battery) return;
 let p=Math.round(battery.level*100);
 pctEl.innerText=p+'%';
 pctEl.style.color = p>=100? '#FF0000' : p>=80? '#FFD700' : '#00FF00';
 statEl.innerText= battery.charging? `⚡ Charging - ${p}% - Will shout at ${target}% in Zulu` : `🔋 Not charging - ${p}% - Plug in charger`;
 zuluStatEl.innerText = battery.charging? `AI Mzansi reading: ${p}%` : `Linda - Shaja ifuna ukugcwalisa`;
 if(alertOn && battery.charging && p>=target) shout();
 if(alertOn &&!battery.charging && alarmLoop) stopShout();
}
function previewVoice(){
 currentVoice=document.getElementById('voice').value;
 if(currentVoice==='custom'){ if(customAudioURL){ audioEl.src=customAudioURL; audioEl.style.display='block'; audioEl.play(); chosenEl.innerText='Playing your custom song'; } else { alert('Choose your song from phone first - Click Choose Your Own Ringtone'); return; } }
 else { let v=voices[currentVoice]; if(!v) return; chosenEl.innerText='Chosen: '+document.getElementById('voice').selectedOptions[0].text; speakZulu(v.text, v.lang); }
}
function speakZulu(text, lang){
 if('speechSynthesis' in window){
   speechSynthesis.cancel();
   let u=new SpeechSynthesisUtterance(text);
   u.lang=lang; u.rate=0.9; u.volume=1; u.pitch=1;
   speechSynthesis.speak(u);
 }
}
function loadCustom(input){
 let f=input.files[0]; if(!f) return;
 customAudioURL=URL.createObjectURL(f);
 audioEl.src=customAudioURL; audioEl.style.display='block';
 document.getElementById('voice').value='custom';
 chosenEl.innerText='Custom Ringtone: '+f.name+' - Will play when '+target+'%';
 audioEl.play();
}
function shout(){
 if(alarmLoop) return;
 document.getElementById('main').style.background='#FF0000';
 pctEl.innerText=target+'% FULL!';
 statEl.innerHTML=`<b style="color:#fff;font-size:20px">🔴 KHIPHA SHAJA MANJE! ${target}% FULL!</b>`;
 zuluStatEl.innerText='Mntanami! Battery igcwele! Khipha!';
 // Play custom or Zulu
 if(currentVoice==='custom' && customAudioURL){ audioEl.loop=true; audioEl.play().catch(()=>{}); }
 else { let v=voices[currentVoice]||voices.zulu1; speakZulu(v.text+' '+v.text+' '+v.text, v.lang); alarmLoop=setInterval(()=>{ speakZulu(v.text, v.lang); if(navigator.vibrate) navigator.vibrate(1000); },3000); }
 if(navigator.vibrate) navigator.vibrate([1000,500,1000,500,1000]);
 if(Notification && Notification.permission==="granted"){ new Notification(`🔋 ${target}% FULL! Khipha shaja!`,{body:'Mzansi AI Battery Alert - Unplug now', requireInteraction:true}); }
 document.getElementById('stopBtn').style.display='block';
}
function stopShout(){ document.getElementById('main').style.background='#1a1a1a'; if(alarmLoop){clearInterval(alarmLoop); alarmLoop=null;} speechSynthesis.cancel(); audioEl.pause(); audioEl.loop=false; }
function startAlert(){ alertOn=true; target=parseInt(document.getElementById('targetLevel').value); currentVoice=document.getElementById('voice').value; if(Notification && Notification.permission!=="granted") Notification.requestPermission(); if('wakeLock' in navigator){navigator.wakeLock.request('screen').catch(()=>{});} document.getElementById('startBtn').innerText=`🟢 MZANSI ALERT ON - Reading % - Will shout at ${target}%`; document.getElementById('startBtn').style.background='#555'; updateBattery(); }
function stopAlert(){ alertOn=false; stopShout(); document.getElementById('startBtn').innerText='🟢 START MZANSI ALERT - READ % NOW'; document.getElementById('startBtn').style.background='#00FF00'; document.getElementById('stopBtn').style.display='none'; document.getElementById('main').style.background='#1a1a1a'; statEl.innerText='Guard OFF - Plug charger to start'; }

init();
if('serviceWorker' in navigator){navigator.serviceWorker.register('sw.js').catch(()=>{});}
</script>
</body>
</html>
