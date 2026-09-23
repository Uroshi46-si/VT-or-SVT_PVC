# VT-or-SVT_PVC
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-title" content="ECG判読">
<link rel="manifest" href="manifest.json">
<title>ECG判読支援（WCT / PVC）</title>
<style>
:root{--pri:#0a58ca;--vt:#c62828;--svt:#2e7d32;--muted:#555}
*{box-sizing:border-box}
body{margin:0;font-family:-apple-system,BlinkMacSystemFont,"Hiragino Sans",sans-serif;background:#f4f6f8;color:#222;padding-bottom:calc(env(safe-area-inset-bottom) + 20px)}
header{position:sticky;top:0;z-index:10;background:var(--pri);color:#fff;padding:calc(env(safe-area-inset-top) + 10px) 14px 10px;font-weight:700}
.badge{font-size:.72rem;font-weight:400;margin-left:8px;opacity:.9}
.tabs{display:flex;gap:6px;margin-top:8px}
.tab{background:rgba(255,255,255,.2)!important;color:#fff!important;margin:0!important;padding:8px 12px!important;font-size:.95rem!important}
.tab.active{background:#fff!important;color:var(--pri)!important}
#err{display:none;white-space:pre-wrap;background:#ffebee;color:#b71c1c;margin:10px;padding:10px;border-radius:8px;font-size:.85rem}
.card{background:#fff;margin:10px;padding:12px;border-radius:12px;box-shadow:0 1px 3px rgba(0,0,0,.08)}
h2{font-size:1.05rem;margin:0 0 8px}
button,.btn{font-size:1rem;padding:10px 14px;border-radius:10px;border:none;background:var(--pri);color:#fff;margin:4px 4px 4px 0;display:inline-block}
button.sec{background:#e3e8ef;color:#222}
button.yes{background:var(--vt)} button.no{background:#455a64}
button.sm{padding:8px 12px;font-size:.95rem}
#cv{width:100%;height:55vh;background:#222;border-radius:8px;touch-action:none;display:block;margin:8px 0}
.row{display:flex;flex-wrap:wrap;align-items:center;gap:6px;margin:4px 0}
input[type=number],select{font-size:1rem;padding:8px;border:1px solid #ccc;border-radius:8px;width:100px;background:#fff}
input[type=range]{flex:1}
#meas{font-size:1.3rem;font-weight:700}
.note{font-size:.85rem;color:var(--muted);line-height:1.55}
.warn{background:#fff3e0;border-left:4px solid #ef6c00;padding:8px 10px;border-radius:6px;font-size:.88rem;line-height:1.5;margin:8px 0}
.result{padding:14px;border-radius:12px;color:#fff;font-size:1.15rem;font-weight:700;margin-bottom:8px}
.result.VT{background:var(--vt)} .result.SVT{background:var(--svt)}
label.chk{display:block;padding:10px;border:1px solid #ddd;border-radius:8px;margin:4px 0;font-size:.95rem}
.log{font-size:.85rem;white-space:pre-wrap;background:#f0f0f0;padding:8px;border-radius:8px}
details.ex{background:#eef4ff;border:1px solid #c9dafc;border-radius:10px;padding:8px 10px;margin:10px 0}
details.ex summary{font-weight:700;color:var(--pri);cursor:pointer;font-size:.95rem}
details.ex div{font-size:.88rem;line-height:1.65;margin-top:6px}
details.ex p{margin:6px 0}
.lead{display:grid;grid-template-columns:64px 1fr auto;gap:6px;align-items:center;margin:4px 0}
.lead b{font-size:1rem}
.lead input{width:100%}
.fld{margin:10px 0}
.fld>label{display:block;font-size:.9rem;font-weight:600;margin-bottom:4px}
.fld select{width:100%}
.cand{border:1px solid #ddd;border-radius:10px;padding:10px;margin:8px 0}
.cand .cn{font-weight:700}
.cand ul{margin:6px 0 0 18px;padding:0;font-size:.88rem;line-height:1.55}
.score{float:right;font-size:.8rem;background:#e3e8ef;border-radius:8px;padding:2px 8px;font-weight:400}
.neg{color:#b71c1c}
.hide{display:none}
footer{font-size:.75rem;color:#777;margin:10px 14px;line-height:1.5}
</style>
</head>
<body>
<header>ECG判読支援<span id="net" class="badge"></span>
  <div class="tabs">
    <button id="tabW" class="tab active" onclick="tab('W')">VT/SVT鑑別</button>
    <button id="tabP" class="tab" onclick="tab('P')">PVC起源推定</button>
  </div>
</header>
<div id="err"></div>

<div class="card">
  <h2>① 心電図画像</h2>
  <label class="btn" for="file">📷 写真を撮影 / 選択</label>
  <input id="file" type="file" accept="image/*" style="display:none">
  <canvas id="cv"></canvas>
  <div class="row"><span>計測値：</span><span id="meas">—</span></div>
  <div class="row">キャリパー：
    <button id="mT" class="sm" onclick="setMode('t')">時間（縦線）</button>
    <button id="mA" class="sec sm" onclick="setMode('a')">振幅（横線）</button>
  </div>
  <div class="row">
    <button class="sec sm" onclick="zoomC(1.5)">＋拡大</button>
    <button class="sec sm" onclick="zoomC(1/1.5)">－縮小</button>
    <button class="sec sm" onclick="fit();req()">全体表示</button>
    <button class="sec sm" onclick="centerCal()">キャリパーを中央へ</button>
  </div>
  <div class="row"><span style="color:#0097a7">水色線</span>
    <button class="sec sm" onclick="nudge(1,-1)">−</button><button class="sec sm" onclick="nudge(1,1)">＋</button>
    <span style="color:#d81b60;margin-left:8px">ピンク線</span>
    <button class="sec sm" onclick="nudge(2,-1)">−</button><button class="sec sm" onclick="nudge(2,1)">＋</button>
  </div>
  <div class="row">
    <button class="sec sm" onclick="set40(1)">水色線から40 ms</button>
    <button class="sec sm" onclick="set40(2)">ピンク線の手前40 ms</button>
  </div>
  <div class="row"><span>傾き補正</span><input id="rot" type="range" min="-15" max="15" step="0.25" value="0"><span id="rotv">0</span>°</div>
  <p class="note">1本指ドラッグ：移動／2本指ピンチ：拡大縮小／線の近くを触ってドラッグ：キャリパー移動。−/＋ボタンは、時間モードでは左右、振幅モードでは上下に微調整します。</p>
</div>

<div class="card">
  <h2>② 較正（最初に1回）</h2>
  <div class="row">紙送り速度
    <select id="speed"><option value="25" selected>25 mm/s</option><option value="50">50 mm/s</option></select>
    感度
    <select id="gain"><option value="10" selected>10 mm/mV</option><option value="5">5 mm/mV</option><option value="20">20 mm/mV</option></select>
  </div>
  <div class="row">2本の縦線の間 ＝ 大マス <input id="boxes" type="number" inputmode="decimal" value="5"> 個
    <button onclick="calibrate()">較正する</button></div>
  <div id="calStat" class="note">未較正</div>
  <p class="note">「時間（縦線）」モードで、方眼の太線（大マス：5 mm）に2本の線を合わせてから「較正する」を押します。振幅は、方眼が正方形であるとみなしてこの較正から換算します。遠近による歪みがあるため、計測したい誘導の近くで較正すると精度が上がります。</p>
</div>

<div class="card" id="wiz"></div>
<div class="card hide" id="pvc"></div>

<footer>本アプリは判読を支援する参考ツールで、診断を確定するものではありません。画像は端末内だけで処理し、外部には送信しません。<br>
主な根拠：Brugada P, et al. Circulation 1991;83:1649-59／Vereckei A, et al. Heart Rhythm 2008;5:89-98／Kaiser E, et al. Europace 2015;17:1422-7／Wellens HJJ, et al. Am J Med 1978;64:27-33／Kindwall KE, et al. Am J Cardiol 1988;61:1279-83／Enriquez A, et al. JACC Clin EP 2024;10:1206-22／Daniels DV, et al. Circulation 2006;113:1659-66<br>
<span id="ver"></span></footer>

<script>
const APP_VER='1.3.0';
const $=id=>document.getElementById(id);
window.addEventListener('error',e=>{
  const d=$('err'); if(!d) return;
  d.style.display='block';
  d.textContent+='エラー：'+e.message+'（'+(e.lineno||'?')+'行目）\nこの画面のスクリーンショットを送ってください。\n';
});
$('ver').textContent='バージョン '+APP_VER;

/* ---------- オフライン対応 ---------- */
function setNet(){
  const ready='serviceWorker' in navigator && navigator.serviceWorker.controller;
  $('net').textContent=(navigator.onLine?'オンライン':'オフライン')+(ready?'・オフライン対応済':'');
}
if('serviceWorker' in navigator){
  navigator.serviceWorker.register('sw.js').then(()=>navigator.serviceWorker.ready).then(setNet).catch(()=>{});
  navigator.serviceWorker.addEventListener('controllerchange',setNet);
}
window.addEventListener('online',setNet); window.addEventListener('offline',setNet); setNet();

/* ---------- タブ ---------- */
function tab(t){
  $('wiz').classList.toggle('hide',t!=='W'); $('pvc').classList.toggle('hide',t!=='P');
  $('tabW').classList.toggle('active',t==='W'); $('tabP').classList.toggle('active',t==='P');
}

/* ---------- 画像・キャリパー ---------- */
const cv=$('cv'),ctx=cv.getContext('2d');
let img=null,view={s:1,x:0,y:0},rot=0,cal={x1:-50,x2:50,y1:-30,y2:30},mode='t',msPerPx=null,pending=false;

$('file').addEventListener('change',e=>{
  const f=e.target.files[0]; if(!f) return;
  const url=URL.createObjectURL(f), im=new Image();
  im.onload=()=>{
    const MAX=3000, k=Math.min(1,MAX/Math.max(im.naturalWidth,im.naturalHeight));
    const c=document.createElement('canvas');
    c.width=Math.round(im.naturalWidth*k); c.height=Math.round(im.naturalHeight*k);
    c.getContext('2d').drawImage(im,0,0,c.width,c.height);
    img=c; URL.revokeObjectURL(url);
    msPerPx=null; rot=0; $('rot').value=0; $('rotv').textContent='0';
    fit(); updateCal(); req();
  };
  im.src=url; e.target.value='';
});
function fit(){
  if(!img) return;
  const W=cv.clientWidth,H=cv.clientHeight;
  view.s=Math.min(W/img.width,H/img.height); view.x=W/2; view.y=H/2;
  cal.x1=-img.width*0.08; cal.x2=img.width*0.08;
  cal.y1=-img.height*0.05; cal.y2=img.height*0.05;
}
function centerCal(){
  const cx=(cv.clientWidth/2-view.x)/view.s, cy=(cv.clientHeight/2-view.y)/view.s;
  cal.x1=cx-40/view.s; cal.x2=cx+40/view.s; cal.y1=cy-30/view.s; cal.y2=cy+30/view.s; req();
}
function nudge(n,d){ cal[(mode==='t'?'x':'y')+n]+=d/view.s; req(); }
function setMode(m){
  mode=m; $('mT').className=(m==='t'?'':'sec ')+'sm'; $('mA').className=(m==='a'?'':'sec ')+'sm'; req();
}
function set40(n){
  if(msPerPx==null){ alert('先に②で較正してください'); return; }
  setMode('t'); const w=40/msPerPx;
  if(n===1) cal.x2=cal.x1+w; else cal.x1=cal.x2-w;
  req();
}
function zoomAt(x,y,k){
  const wx=(x-view.x)/view.s, wy=(y-view.y)/view.s;
  view.s=Math.max(0.03,Math.min(40,view.s*k));
  view.x=x-wx*view.s; view.y=y-wy*view.s; req();
}
function zoomC(k){ zoomAt(cv.clientWidth/2,cv.clientHeight/2,k); }
$('rot').addEventListener('input',e=>{ rot=parseFloat(e.target.value); $('rotv').textContent=rot; req(); });

function req(){ if(!pending){ pending=true; requestAnimationFrame(()=>{pending=false;draw();}); } }
function draw(){
  const dpr=window.devicePixelRatio||1, W=cv.clientWidth, H=cv.clientHeight;
  if(cv.width!==Math.round(W*dpr)||cv.height!==Math.round(H*dpr)){ cv.width=Math.round(W*dpr); cv.height=Math.round(H*dpr); }
  ctx.setTransform(1,0,0,1,0,0); ctx.fillStyle='#222'; ctx.fillRect(0,0,cv.width,cv.height);
  if(!img){
    ctx.fillStyle='#aaa'; ctx.font=(16*dpr)+'px sans-serif'; ctx.textAlign='center';
    ctx.fillText('写真を読み込んでください',cv.width/2,cv.height/2); updateMeas(); return;
  }
  ctx.setTransform(dpr*view.s,0,0,dpr*view.s,dpr*view.x,dpr*view.y);
  ctx.save(); ctx.rotate(rot*Math.PI/180); ctx.drawImage(img,-img.width/2,-img.height/2); ctx.restore();
  const yT=-view.y/view.s, yB=(H-view.y)/view.s, xL=-view.x/view.s, xR=(W-view.x)/view.s;
  ctx.fillStyle='rgba(255,235,59,0.15)'; ctx.lineWidth=1.5/view.s;
  const cols=[['1','#00e5ff'],['2','#ff4081']];
  if(mode==='t'){
    ctx.fillRect(Math.min(cal.x1,cal.x2),yT,Math.abs(cal.x2-cal.x1),yB-yT);
    cols.forEach(([n,c])=>{ ctx.strokeStyle=c; ctx.beginPath(); ctx.moveTo(cal['x'+n],yT); ctx.lineTo(cal['x'+n],yB); ctx.stroke(); });
  } else {
    ctx.fillRect(xL,Math.min(cal.y1,cal.y2),xR-xL,Math.abs(cal.y2-cal.y1));
    cols.forEach(([n,c])=>{ ctx.strokeStyle=c; ctx.beginPath(); ctx.moveTo(xL,cal['y'+n]); ctx.lineTo(xR,cal['y'+n]); ctx.stroke(); });
  }
  ctx.setTransform(dpr,0,0,dpr,0,0);
  cols.forEach(([n,c])=>{
    ctx.fillStyle=c;
    if(mode==='t'){ const sx=cal['x'+n]*view.s+view.x; ctx.beginPath(); ctx.arc(sx,H-16,10,0,7); ctx.fill(); ctx.beginPath(); ctx.arc(sx,16,10,0,7); ctx.fill(); }
    else { const sy=cal['y'+n]*view.s+view.y; ctx.beginPath(); ctx.arc(16,sy,10,0,7); ctx.fill(); ctx.beginPath(); ctx.arc(W-16,sy,10,0,7); ctx.fill(); }
  });
  const t=measText();
  ctx.font='bold 17px sans-serif'; const tw=ctx.measureText(t).width;
  ctx.fillStyle='rgba(0,0,0,.65)'; ctx.fillRect(30,8,tw+20,30);
  ctx.fillStyle='#fff'; ctx.textAlign='left'; ctx.fillText(t,40,29);
  updateMeas();
}
window.addEventListener('resize',req);

function pos(e){ const r=cv.getBoundingClientRect(); return {x:e.clientX-r.left,y:e.clientY-r.top}; }
const ptrs=new Map(); let drag=null,pinch=null;
function startSingle(p){
  let d1,d2;
  if(mode==='t'){ d1=Math.abs(cal.x1*view.s+view.x-p.x); d2=Math.abs(cal.x2*view.s+view.x-p.x); }
  else { d1=Math.abs(cal.y1*view.s+view.y-p.y); d2=Math.abs(cal.y2*view.s+view.y-p.y); }
  drag = Math.min(d1,d2)<24 ? {t:'cal',n:d1<=d2?'1':'2'} : {t:'pan',sx:p.x,sy:p.y,vx:view.x,vy:view.y};
}
cv.addEventListener('pointerdown',e=>{
  if(!img) return; e.preventDefault(); cv.setPointerCapture(e.pointerId);
  const p=pos(e); ptrs.set(e.pointerId,p);
  if(ptrs.size===1) startSingle(p);
  else if(ptrs.size===2){
    const [a,b]=[...ptrs.values()];
    pinch={d:Math.hypot(a.x-b.x,a.y-b.y)||1,s:view.s,wx:((a.x+b.x)/2-view.x)/view.s,wy:((a.y+b.y)/2-view.y)/view.s};
    drag=null;
  }
});
cv.addEventListener('pointermove',e=>{
  if(!ptrs.has(e.pointerId)) return;
  const p=pos(e); ptrs.set(e.pointerId,p);
  if(pinch&&ptrs.size>=2){
    const [a,b]=[...ptrs.values()], d=Math.hypot(a.x-b.x,a.y-b.y)||1;
    const ns=Math.max(0.03,Math.min(40,pinch.s*d/pinch.d));
    view.s=ns; view.x=(a.x+b.x)/2-pinch.wx*ns; view.y=(a.y+b.y)/2-pinch.wy*ns;
  } else if(drag){
    if(drag.t==='cal'){ if(mode==='t') cal['x'+drag.n]=(p.x-view.x)/view.s; else cal['y'+drag.n]=(p.y-view.y)/view.s; }
    else { view.x=drag.vx+p.x-drag.sx; view.y=drag.vy+p.y-drag.sy; }
  }
  req();
});
function up(e){
  ptrs.delete(e.pointerId);
  if(ptrs.size<2) pinch=null;
  if(ptrs.size===1){ const p=[...ptrs.values()][0]; drag={t:'pan',sx:p.x,sy:p.y,vx:view.x,vy:view.y}; }
  if(ptrs.size===0) drag=null;
}
cv.addEventListener('pointerup',up); cv.addEventListener('pointercancel',up);
cv.addEventListener('wheel',e=>{ e.preventDefault(); const p=pos(e); zoomAt(p.x,p.y,Math.exp(-e.deltaY*0.002)); },{passive:false});
document.addEventListener('gesturestart',e=>e.preventDefault());

/* ---------- 較正・計測 ---------- */
function speed(){ return parseFloat($('speed').value); }
function gain(){ return parseFloat($('gain').value); }
function msPerBox(){ return 5/speed()*1000; }
function curMs(){ return msPerPx==null?null:Math.abs(cal.x2-cal.x1)*msPerPx; }
function curMm(){ return msPerPx==null?null:Math.abs(cal.y2-cal.y1)*msPerPx*speed()/1000; }
function measText(){
  if(msPerPx==null) return '未較正';
  if(mode==='t') return Math.round(curMs())+' ms';
  const mm=curMm(); return mm.toFixed(1)+' mm（'+(mm/gain()).toFixed(2)+' mV）';
}
function updateMeas(){ $('meas').textContent=!img?'—':(msPerPx==null?'未較正（②で較正してください）':measText()); }
function calibrate(){
  const n=parseFloat($('boxes').value), dx=Math.abs(cal.x2-cal.x1);
  if(!img){ alert('先に写真を読み込んでください'); return; }
  if(mode!=='t'){ alert('較正は「時間（縦線）」モードで行ってください'); return; }
  if(!(n>0)||dx<3){ alert('2本の線を大マスの太線に合わせ、マス数を入力してください'); return; }
  msPerPx=n*msPerBox()/dx; updateCal(); req();
}
function updateCal(){
  $('calStat').textContent = msPerPx==null ? '未較正'
    : '較正済み：大マス1個 = '+msPerBox()+' ms（1 px ≈ '+msPerPx.toFixed(2)+' ms）';
}
$('speed').addEventListener('change',()=>{ msPerPx=null; updateCal(); req(); });
$('gain').addEventListener('change',req);
function useCalTo(id,type){
  if(msPerPx==null){ alert('先に②で較正してください'); return; }
  if(mode!==type){ alert(type==='t'?'キャリパーを「時間（縦線）」にして計測してください':'キャリパーを「振幅（横線）」にして計測してください'); return; }
  const v=type==='t'?Math.round(curMs()):Math.round(curMm()*10)/10;
  const el=$(id); el.value=v; el.dispatchEvent(new Event('input'));
}
function copyText(t){
  if(navigator.clipboard) navigator.clipboard.writeText(t).then(()=>alert('コピーしました')).catch(()=>prompt('コピーしてください',t));
  else prompt('コピーしてください',t);
}

/* ---------- 解説 ---------- */
const EX={
pre:{t:'なぜ前提の確認が必要か',b:`
<p>Brugada法は、電気生理学的検査で機序が確定した<b>規則的な</b>wide QRS頻拍を対象に作られ、検証されています（554例、抗不整脈薬は投与されていない症例）。Vereckei法も同じく規則的なwide QRS頻拍が対象です。不規則な頻拍（例：WPW症候群に合併した心房細動）は想定外です。</p>
<p><b>副伝導路を順行するSVT</b>（antidromic AVRTなど）：心室の興奮が副伝導路の付着部から心筋を直接伝わり、His-Purkinje系を使わないため、VTとほぼ区別できません。洞調律時にデルタ波があれば疑います。</p>
<p><b>高K血症、Ic群薬などのNaチャネル遮断薬</b>：心筋とHis-Purkinje系の伝導が全体に遅くなるため、SVTでも初期の興奮が遅く、VTに似た所見になります。</p>
<p><b>心室ペーシング</b>：心室を直接刺激して興奮させるため、VTに似た波形になります。</p>`},
algo:{t:'3つのアルゴリズムの違い',b:`
<p><b>推奨（aVRの初期R波 → Brugada法）</b>：Kaiserら（2015）の提案です。aVRの初期R波は短時間で判定でき、判定者の間で一致しやすく、陽性尤度比も高い指標でした。これが陰性なら、次にBrugada法で評価します。</p>
<p><b>Vereckei aVR法</b>：aVRだけで判定します。原著（2008年、483例）では正診率91.5%とBrugada法（85.5%）を上回りました。一方で、Step 3・4は細かい計測が必要で、Kaiserらの研究では「虫眼鏡が必要なことが多く、時間がかかるほど誤りが増えた」と報告されています。</p>
<p><b>Brugada法</b>：胸部誘導を使う古典的な方法です。原著では感度0.987、特異度0.965でしたが、他の施設での検証では特異度がもっと低い報告が多く、判定者の間でも意見が割れやすい方法です（Kaiser：κ=−0.088）。</p>`},
avr:{t:'aVRの初期R波がVTを示す理由',b:`
<p>aVRは、心臓を右上方（心基部の方向）から見る誘導です。</p>
<p><b>変行伝導（SVT）の場合</b>：初期の興奮はHis-Purkinje系を通り、中隔から左下方へ向かいます。aVRから<b>遠ざかる</b>向きなので、QRSの初期は陰性（Q、QS、rS）になります。</p>
<p><b>初期にR波がある場合</b>：初期の興奮がaVRに<b>向かって</b>いる、つまり心尖部や下壁から右上方へ興奮が広がっていることを意味します。正常な伝導系を使う変行伝導では通常見られないため、VTを示唆します。</p>
<p><b>判定の性能</b>：速く、判定者の間で一致しやすい指標です（Kaiser 2015：平均約9秒、κ=0.61、陽性尤度比18.16）。ただし初期R波がなくてもVTは否定できず、同じ研究ではVTの確率が72.6%から約60%に下がるだけでした。</p>`},
v2:{t:'aVRの初期r波・q波が40 msを超えるとVTといえる理由',b:`
<p>このステップが見ているのは、QRSの<b>初期の成分の幅</b>、つまり心室の興奮が始まる速さです。</p>
<p><b>変行伝導の場合</b>：初期の興奮はブロックされていない脚からHis-Purkinje系で速やかに広がるので、初期のr波やq波は細く鋭くなります。</p>
<p><b>VTの場合</b>：興奮は起源から心筋細胞を順に伝わるためゆっくりで、初期のr波やq波が40 msを超えて幅広くなります。初期のr波がある場合は、それ自体が「aVR方向（上方）に向かう興奮」でもあります。</p>
<p><b>このアプリでの扱い</b>：QS型（最初から最後まで陰性）の場合は初期の成分を区切れないため、このステップでは評価せず、Step 3（ノッチ）に進みます。</p>`},
v3:{t:'陰性QRSの下行脚のノッチがVTを示す理由',b:`
<p>陰性で始まり主に陰性のQRSで、最初に下向きに振れていく部分（下行脚）を見ます。</p>
<p><b>変行伝導の場合</b>：初期はHis-Purkinje系で速やかに伝わるので、下行脚はなめらかで急峻です。</p>
<p><b>VTの場合</b>：初期の興奮が心筋の中をゆっくり、不均一に伝わるため、下行脚にノッチ（切れ込み・段差）ができます。</p>
<p><b>注意</b>：ノッチは小さいことが多く、画像の解像度に左右されます。拡大して判断し、はっきりしなければ「いいえ」として次のステップで評価してください。</p>`},
v4:{t:'Vi/Vt ≤1がVTを示す理由',b:`
<p><b>Vi</b>＝QRSの最初の40 msに波形が上下に動いた幅、<b>Vt</b>＝最後の40 msに上下に動いた幅です。どちらも同じ心拍で測ります。</p>
<p><b>変行伝導の場合</b>：初期は伝導系で速く興奮するので最初の40 msで大きく振れ（Viが大きい）、遅れはブロック側の興奮が起こる終盤に集中します（Vtが小さい）。そのためVi/Vt >1になります。</p>
<p><b>VTの場合</b>：初期は心筋伝導で遅く、ゆっくり振れます（Viが小さい）。終盤には興奮が伝導系に入り込んで速くなることがあり（Vtが大きい）、Vi/Vt ≤1になります。</p>
<p><b>計測のしかた</b>：時間モードで「水色線から40 ms」を押し、水色線をQRSの開始点に合わせます。その区間の上下の振れ幅を振幅モードで測ります。終わりの40 msは、ピンク線をQRSの終了点に合わせて「ピンク線の手前40 ms」を押し、同じように測ります。上下両方に振れていれば、合計の振れ幅を使います。</p>
<p><b>注意</b>：最も細かい計測が必要なステップで、判定者の間で差が出やすいことが報告されています（Kaiser 2015）。</p>`},
b1:{t:'胸部誘導にRS complexがないとVTといえる理由',b:`
<p><b>変行伝導の場合</b>：ブロックされていない側の脚から興奮が速やかに始まり、遅れてブロック側へ広がります。興奮の向きが途中で変わるため、胸部誘導のどこかにRS complex（向かってきて、去っていく波形）が必ず現れます。原著でも、SVTの全例でRS complexが1つ以上の誘導にありました。</p>
<p><b>RS complexがない場合</b>：V1〜V6がすべて陽性（positive concordance）、すべて陰性（negative concordance）、あるいはQR型などになっている状態です。これは、興奮が心室の1点から一方向に広がっていることを示し、VTに特徴的です。</p>
<p><b>原著の成績</b>：特異度1.00、感度0.21。QR、QRS、QS、単相性R、rSR′はRS complexに含めません。</p>
<p><b>例外</b>：左後壁の副伝導路を順行するantidromic AVRTは、positive concordanceになることがあります。</p>`},
b2:{t:'RS間隔が100 msを超えるとVTといえる理由',b:`
<p>RS間隔（R波の立ち上がりからS波の最深点まで）は、心室興奮の<b>初期部分</b>にかかる時間を表します。</p>
<p><b>変行伝導の場合</b>：初期の興奮はHis-Purkinje系で速く、遅れはQRSの後半に集中するので、RS間隔は短くなります。</p>
<p><b>VTの場合</b>：興奮は心筋細胞から心筋細胞へとゆっくり伝わるため、QRSの始めから遅く、RS間隔が延長します。</p>
<p><b>この基準の由来</b>：LBBB型の頻拍でV1/V2の「R開始からS最深点まで >60 ms」がVTを示すというKindwallらの報告を、Brugadaらがすべての胸部誘導に広げたものです。原著では、RS間隔とQRS幅に有意な相関はありませんでした。前半の解析では、RS間隔が100 msを超えたSVTは1例もありませんでした。</p>
<p><b>注意</b>：Ic群薬、アミオダロン、高K血症、副伝導路の順行伝導があると、SVTでもRS間隔が延長することがあります。</p>`},
b3:{t:'房室解離がVTを示す理由',b:`
<p><b>SVTの場合</b>：頻拍の回路に心房または房室結節が含まれるため、通常は心房と心室が1対1で連動しています。</p>
<p><b>房室解離がある場合</b>：心房と無関係に心室が興奮し続けていることを示します。原著では、SVTでは1例も見られませんでした（特異度1.00）。capture beat（幅の狭いQRS）やfusion beat（中間的な形のQRS）も房室解離の証拠です。</p>
<p><b>注意</b>：多くのVTは室房伝導を保っているため、房室解離が認識できるのはVTの約21%だけでした（感度が低い）。</p>`},
b4:{t:'形態基準の考え方と、両方の誘導を要求する理由',b:`
<p>形態基準が見ているのは<b>「その波形が、正常なHis-Purkinje系を使った典型的な脚ブロックとして説明できるか」</b>です。変行伝導は典型的な脚ブロックの形になり、VTは典型から外れた形になります。</p>
<p>原著では1/3以上の症例でV1とV6の判定が食い違ったため、Brugada法は<b>両方</b>がVT基準を満たすときだけVTと判定します。このステップで感度は0.82から0.987に上がり、特異度は0.98から0.965に下がります。</p>`},
b4R:{t:'RBBB型でのV1・V6の見方',b:`
<p><b>V1</b>：右脚ブロックの変行伝導では、中隔が左→右に興奮して小さなr波が出て、続いて左室のS波、遅れた右室のR′波が出て3相性（rsR′）になります。<b>単相性R・qR型</b>は、この正常な中隔の初期興奮がないことを示し、左室起源のVTを示唆します。</p>
<p><b>V6</b>：右脚ブロックでは左室は正常に興奮するので、大きなR波と小さなs波（R/S >1）になります。<b>R/S <1、QS型、QR型</b>は、興奮が心尖部付近から始まるVTを示唆します。</p>`},
b4L:{t:'LBBB型でのV1/V2・V6の見方（Kindwall基準）',b:`
<p><b>V1/V2</b>：左脚ブロックの変行伝導では右室が右脚から速やかに興奮するため、r波は細く（30 ms以下）、S波は急峻に下がります（R開始からS最深点まで60 ms以下）。<b>R波の幅 >30 ms、R開始からS最深点まで >60 ms、S波の下行脚のノッチ</b>は、初期の興奮が遅いことを示し、VTを示唆します。</p>
<p><b>V6</b>：左脚ブロックでは中隔が右→左に興奮するため、V6の初期にQ波は出ません。<b>V6にQ波がある</b>場合は通常の左脚ブロックでは説明できず、VTを示唆します。</p>`},
p_bb:{t:'脚ブロックの型から分かること',b:`<p>V1は右室の前面に近い誘導です。左室起源では興奮がV1に向かうのでR波優位（RBBB型）、右室や中隔起源では遠ざかるのでS波優位（LBBB型）になります。ただし、LVOT（大動脈冠尖）や中隔の左側から起こる場合もLBBB型になることがあります。</p>`},
p_inf:{t:'下壁誘導から分かること',b:`<p>下壁誘導は心臓を下から見ています。流出路など心臓の上方から始まる興奮は下へ向かうので陽性（下方軸）、下壁や心尖部から始まる興奮は上へ向かうので陰性（上方軸）になります。</p>`},
p_I:{t:'I誘導から分かること',b:`<p>I誘導は左右の方向を見ています。右側（RVOT自由壁、三尖弁輪など）から始まると興奮は左へ向かうのでI誘導は陽性、左側（LCC、LV summit、僧帽弁輪の側壁など）から始まると右へ向かうので陰性になります。</p>`},
p_tz:{t:'移行帯から分かること',b:`<p>移行帯は、起源が前方か後方かを反映します。前方にあるRVOTでは興奮が後方へ向かうため移行帯が遅く（V4以降）、後方にあるLVOTでは前方へ向かうため早く（V2以前）なります。V3はどちらもありうるので、洞調律の移行帯との比較が参考になります（PVCの方が早ければLVOT寄り、遅ければRVOT寄り）。心臓の回転や電極の位置の影響を受けます。</p>`},
p_mdi:{t:'MDI（maximum deflection index）とは',b:`<p>MDI＝（胸部誘導で、QRS開始から最大の振れの頂点までの時間。正負は問わず、誘導の中で最も短いもの）÷ QRS幅。心外膜側から起こる興奮は、心筋をゆっくり通ってから刺激伝導系に入るため、初期が遅くなりMDIが大きくなります。MDI ≥0.55で心外膜起源を示唆するとされます（Daniels 2006）。</p>`}
};
function ex(k,open){
  const e=EX[k]; if(!e) return '';
  return `<details class="ex"${open?' open':''}><summary>なぜこの基準か：${e.t}</summary><div>${e.b}</div></details>`;
}

/* ---------- VT/SVT判定の流れ ---------- */
const H=[]; let cur='pre', RES=null, morphType=null, algo='combo';
const ALGO={combo:'推奨（aVRの初期R波 → Brugada法）',vk:'Vereckei aVR法',bru:'Brugada法'};
const LEADS=['V1','V2','V3','V4','V5','V6'];
const RS={}; const VK={w:'',vi:'',vt:''};
const ST_KS='Kaiserら（Europace 2015、51例・153解析）：陽性尤度比18.16。検査前のVT確率72.6% → 検査後98%。';
const ST_VK='Vereckei 2008（483例のWCT）：aVR法全体で正診率91.5%、VT診断の感度96.5%、特異度75.0%。ただしKaiser 2015（51例）では感度89.2%、特異度28.6%で、施設や判定者によって成績が大きく異なります。';
const OUT={
  v1:{dx:'VT',ex:'avr',reason:'aVRでQRSの初期にR波あり（Vereckei aVR Step 1）',stat:()=>algo==='vk'?ST_VK:ST_KS,log:'aVRの初期R波：あり'},
  v2:{dx:'VT',ex:'v2',reason:'aVRの初期r波・q波の幅が40 msを超える（Vereckei Step 2）',stat:ST_VK},
  v3:{dx:'VT',ex:'v3',reason:'陰性で始まる主に陰性のQRSの下行脚にノッチ（Vereckei Step 3）',stat:ST_VK,log:'下行脚のノッチ：あり'},
  v4vt:{dx:'VT',ex:'v4',reason:'Vi/Vt ≤1（Vereckei Step 4）',stat:ST_VK},
  v4svt:{dx:'SVT',ex:'v4',reason:'Vi/Vt >1（Vereckei Step 4）のため、SVT（変行伝導）と判定',stat:ST_VK},
  b1:{dx:'VT',ex:'b1',reason:'V1〜V6のすべてでRS complexなし（Brugada Step 1）',stat:'Brugada 1991（554例）：Step 1の感度0.21、特異度1.00。',log:'胸部誘導のRS complex：どの誘導にもない'},
  b2:{dx:'VT',ex:'b2',reason:'最長RS間隔が100 msを超える（Brugada Step 2）',stat:'Brugada 1991：Step 2までの累積で感度0.66、特異度0.98。'},
  b3:{dx:'VT',ex:'b3',reason:'房室解離あり（Brugada Step 3）',stat:'Brugada 1991：Step 3までの累積で感度0.82、特異度0.98。',log:'房室解離：あり'},
  b4vt:{dx:'VT',ex:'b4',reason:'V1-2とV6の両方でVTの形態基準を満たす（Brugada Step 4）',stat:'Brugada 1991：4ステップ全体でVT診断の感度0.987、特異度0.965。'},
  b4svt:{dx:'SVT',ex:'b4',reason:'4ステップすべて陰性のため、VTを除外してSVT（変行伝導）と判定',stat:'Brugada 1991：SVT with aberrancy診断の感度0.965、特異度0.987。'}
};
const MORPH={
  R:{name:'RBBB型',
     a:{title:'V1',items:['単相性R波','QR型','RS型'],note:'3相性（rsR′、rSR′）はSVT寄りなのでチェックしません。'},
     b:{title:'V6',items:['R/S比 <1（rS型）','QS型','QR型'],note:'3相性（qRs）やR/S比 >1はSVT寄りです。'}},
  L:{name:'LBBB型',
     a:{title:'V1またはV2',items:['R波の幅 >30 ms','R波の開始 → S波の最深点 >60 ms','S波の下行脚にノッチ（切れ込み）'],note:'1つでも当てはまればVT寄りです（Kindwall基準）。幅は上のキャリパーで計測できます。'},
     b:{title:'V6',items:['QR型（Q波あり）','QS型'],note:'原著では、V6に何らかのQ波があればVT寄りとして扱います。'}}
};

function go(id,log){ H.push({from:cur,text:log}); cur=id; render(); }
function fin(key,log,reasonOverride){
  const o=OUT[key]; H.push({from:cur,text:log||o.log});
  RES=Object.assign({},o);
  RES.stat=typeof o.stat==='function'?o.stat():o.stat;
  if(reasonOverride) RES.reason=reasonOverride;
  cur='res'; render();
}
function back(){ const h=H.pop(); if(h){ cur=h.from; render(); } }
function restart(){
  H.length=0; cur='pre'; RES=null; morphType=null;
  LEADS.forEach(l=>delete RS[l]); VK.w=''; VK.vi=''; VK.vt=''; render();
}
function startAlgo(a){ algo=a; go(a==='bru'?'b1':'v1','アルゴリズム：'+ALGO[a]); }

function leadRows(){
  return LEADS.map(l=>`<div class="lead"><b>${l}</b>
    <input id="rs_${l}" type="number" inputmode="decimal" placeholder="RSなし＝空欄" value="${RS[l]??''}" oninput="RS['${l}']=this.value">
    <button class="sec sm" onclick="useCalTo('rs_${l}','t')">キャリパー値</button></div>`).join('');
}

const STEPS={
pre:()=>`<h2>③ 判定　0. 前提の確認</h2>
 <div class="warn">血行動態が不安定な場合は、判読より治療（ACLSに準じた同期電気ショックなど）を優先してください。</div>
 <p class="note">対象：<b>規則的な</b>wide QRS頻拍（QRS幅 ≥120 ms）。次の場合は判定を誤りやすくなります：洞調律時にデルタ波がある、高K血症、抗不整脈薬の内服、心室ペーシング。</p>
 ${ex('pre')}
 <p><b>使うアルゴリズムを選んでください</b></p>
 <button onclick="startAlgo('combo')">推奨：aVRの初期R波 → Brugada法</button>
 <button class="sec" onclick="startAlgo('vk')">Vereckei aVR法（4ステップ）</button>
 <button class="sec" onclick="startAlgo('bru')">Brugada法（4ステップ）</button>
 ${ex('algo')}`,
v1:()=>`<h2>${algo==='vk'?'Step 1（Vereckei aVR法）':'Step A（Vereckei aVR Step 1）'}</h2>
 <p><b>aVR誘導で、QRSの始まりが上向きのR波（R型・Rs型）ですか？</b></p>
 <p class="note">rS、Qr、QSのように初期が下向き、または小さなrからすぐ下がる波形は「いいえ」です。</p>
 ${ex('avr')}
 <button class="yes" onclick="fin('v1')">はい → VT</button>
 <button class="no" onclick="go('${algo==='vk'?'v2':'b1'}','aVRの初期R波：なし')">いいえ → 次へ</button>`,
v2:()=>`<h2>Step 2（Vereckei aVR法）</h2>
 <p><b>aVRで、QRSの初期のr波（rS型）またはq波（Qr型）の幅は何msですか？</b></p>
 <p class="note">QRSの開始点から、初期のr波またはq波が終わって次の波に切り替わる点までを測ります。</p>
 <div class="lead"><b>幅</b><input id="vk_w" type="number" inputmode="decimal" placeholder="ms" value="${VK.w}" oninput="VK.w=this.value">
 <button class="sec sm" onclick="useCalTo('vk_w','t')">キャリパー値</button></div>
 ${ex('v2')}
 <button onclick="judgeV2()">判定</button>
 <button class="sec" onclick="go('v3','aVRの初期r/q波：評価できない（QS型など）')">QS型など、初期の成分を区切れない → 次へ</button>`,
v3:()=>`<h2>Step 3（Vereckei aVR法）</h2>
 <p><b>陰性で始まり主に陰性のQRS（QS、Qr、rSなど）で、下行脚（下向きに振れていく部分）にノッチがありますか？</b></p>
 <p class="note">QRSが主に陽性の場合は「いいえ」を選んでください。</p>
 ${ex('v3')}
 <button class="yes" onclick="fin('v3')">はい → VT</button>
 <button class="no" onclick="go('v4','下行脚のノッチ：なし／該当しない')">いいえ → 次へ</button>`,
v4:()=>`<h2>Step 4（Vereckei aVR法）</h2>
 <p><b>aVRで、Vi（最初の40 msの振れ幅）とVt（最後の40 msの振れ幅）を入力してください。</b></p>
 <p class="note">単位はmm（小マス）です。比をとるので、感度の設定は結果に影響しません。計測のしかたは下の解説を参照してください。</p>
 <div class="lead"><b>Vi</b><input id="vk_vi" type="number" inputmode="decimal" placeholder="mm" value="${VK.vi}" oninput="VK.vi=this.value">
 <button class="sec sm" onclick="useCalTo('vk_vi','a')">振幅キャリパー値</button></div>
 <div class="lead"><b>Vt</b><input id="vk_vt" type="number" inputmode="decimal" placeholder="mm" value="${VK.vt}" oninput="VK.vt=this.value">
 <button class="sec sm" onclick="useCalTo('vk_vt','a')">振幅キャリパー値</button></div>
 ${ex('v4',true)}
 <button onclick="judgeV4()">判定</button>`,
b1:()=>`<h2>Step 1（Brugada）</h2>
 <p><b>V1〜V6のどの誘導にもRS complexがありませんか？</b></p>
 <p class="note">RS complex＝R波にS波が続く組（RS、Rs、rS）です。QR、QRS、QS、単相性R、rSR′はRS complexに<b>含めません</b>（原著）。</p>
 ${ex('b1')}
 <button class="yes" onclick="fin('b1')">はい（どこにもない）→ VT</button>
 <button class="no" onclick="go('b2','胸部誘導のRS complex：1誘導以上にあり')">いいえ（1誘導以上にある）</button>`,
b2:()=>`<h2>Step 2（Brugada）</h2>
 <p><b>RS complexがある誘導について、RS間隔（ms）を入力してください。</b></p>
 <p class="note">R波の立ち上がり（QRSの開始点）から、S波の最も深い点までを測ります。RS complexがない誘導や、S波の底がはっきりしない誘導は空欄にしてください。最長の誘導はアプリが自動で選びます。</p>
 ${leadRows()}
 ${ex('b2')}
 <button onclick="judgeRS()">判定</button>`,
b3:()=>`<h2>Step 3（Brugada）</h2>
 <p><b>房室解離がありますか？</b></p>
 <p class="note">P波がQRSと無関係なリズムで出ている、または捕捉収縮（capture beat）や融合収縮（fusion beat）がある場合です。II、III、aVF、V1で、QRSやT波に重なった「ずれ」も探してください。</p>
 ${ex('b3')}
 <button class="yes" onclick="fin('b3')">はい → VT</button>
 <button class="no" onclick="go('b4','房室解離：なし／判断できない')">いいえ／判断できない</button>`,
b4:()=>`<h2>Step 4（Brugada）形態基準</h2>
 ${ex('b4')}
 <p><b>QRSの形はどちらに近いですか？（V1で判断）</b></p>
 <button class="sec" onclick="setMorph('R')">RBBB型（V1で主に上向き）</button>
 <button class="sec" onclick="setMorph('L')">LBBB型（V1で主に下向き）</button>
 <div id="morph"></div>`,
res:()=>`<div class="result ${RES.dx}">判定：${RES.dx==='VT'?'VT（心室頻拍）疑い':'SVT with aberrancy（変行伝導を伴う上室頻拍）疑い'}</div>
 <p><b>根拠：</b>${RES.reason}</p><p class="note">${RES.stat}</p>
 ${ex(RES.ex,true)}
 ${RES.ex==='b4'&&morphType?ex('b4'+morphType):''}
 <div class="warn">${RES.dx==='VT'
   ?'最終診断には、臨床情報、既往歴、過去の心電図との比較、必要に応じて電気生理学的検査が必要です。'
   :'除外による判定です。心筋梗塞の既往や構造的心疾患がある場合はVTの事前確率が高いため、判断に迷うときはVTとして対応するのが安全です。ベラパミル静注など、VTに有害となり得る治療は特に慎重に判断してください。'}</div>
 <h2>回答の履歴</h2><div class="log">${H.map(h=>'・'+h.text).join('\n')}</div>
 <button onclick="copyReport()">結果をコピー</button>`
};

function render(){
  let h=STEPS[cur]();
  if(H.length) h+=`<div style="margin-top:12px"><button class="sec sm" onclick="back()">← 1つ戻る</button><button class="sec sm" onclick="restart()">最初から</button></div>`;
  $('wiz').innerHTML=h;
}
function judgeV2(){
  const v=parseFloat(VK.w);
  if(isNaN(v)||v<=0){ alert('幅（ms）を入力するか、「初期の成分を区切れない → 次へ」を選んでください'); return; }
  const log='aVRの初期r/q波の幅：'+v+' ms'+(v>=35&&v<=45?'（境界値：計測誤差に注意）':'');
  if(v>40) fin('v2',log,'aVRの初期r波・q波の幅が40 msを超える（'+v+' ms、Vereckei Step 2）');
  else go('v3',log);
}
function judgeV4(){
  const vi=parseFloat(VK.vi), vt=parseFloat(VK.vt);
  if(!(vi>0)||!(vt>0)){ alert('ViとVtを両方入力してください（mm）'); return; }
  const r=vi/vt;
  let log='Vi='+vi+' mm、Vt='+vt+' mm → Vi/Vt='+r.toFixed(2);
  if(r>=0.9&&r<=1.1) log+='（境界値：計測誤差に注意）';
  if(r<=1) fin('v4vt',log,'Vi/Vt ≤1（'+r.toFixed(2)+'、Vereckei Step 4）');
  else fin('v4svt',log,'Vi/Vt >1（'+r.toFixed(2)+'、Vereckei Step 4）のため、SVT（変行伝導）と判定');
}
function judgeRS(){
  const ent=LEADS.map(l=>[l,parseFloat(RS[l])]).filter(e=>!isNaN(e[1])&&e[1]>0);
  if(!ent.length){ alert('RS complexがある誘導に、RS間隔（ms）を1つ以上入力してください'); return; }
  const list=ent.map(e=>e[0]+'='+e[1]+' ms').join('、');
  const [ml,mv]=ent.slice().sort((a,b)=>b[1]-a[1])[0];
  let log='RS間隔の計測：'+list+' → 最長 '+ml+' '+mv+' ms';
  if(mv>=95&&mv<=105) log+='（境界値：計測誤差に注意）';
  if(mv>100) fin('b2',log,'最長RS間隔が100 msを超える（'+ml+'で'+mv+' ms、Brugada Step 2）');
  else go('b3',log);
}
function setMorph(t){
  morphType=t; const m=MORPH[t];
  const grp=(g,cls)=>`<p style="margin:10px 0 4px"><b>${g.title}</b>（当てはまるものにすべてチェック）</p>`+
    g.items.map(it=>`<label class="chk"><input type="checkbox" class="${cls}" value="${it}"> ${it}</label>`).join('')+
    `<p class="note">${g.note}</p>`;
  $('morph').innerHTML=`<p><b>${m.name}</b>として評価します</p>`+ex('b4'+t)+grp(m.a,'ga')+grp(m.b,'gb')+
    `<button onclick="judgeMorph()">判定</button>`;
}
function judgeMorph(){
  const m=MORPH[morphType];
  const a=[...document.querySelectorAll('.ga:checked')].map(x=>x.value);
  const b=[...document.querySelectorAll('.gb:checked')].map(x=>x.value);
  const log='形態基準（'+m.name+'）：'+m.a.title+'＝'+(a.join('・')||'該当なし')+'／'+m.b.title+'＝'+(b.join('・')||'該当なし');
  if(a.length&&b.length) fin('b4vt',log); else fin('b4svt',log);
}
function copyReport(){
  copyText('WCT鑑別（'+new Date().toLocaleString('ja-JP')+'）\n'+H.map(h=>'・'+h.text).join('\n')+
    '\n判定：'+RES.dx+'\n根拠：'+RES.reason+'\n'+RES.stat+'\n※判読支援ツールによる参考結果（v'+APP_VER+'）');
}

/* ---------- PVC起源推定 ---------- */
const PF=[
 {id:'bb',label:'V1のQRSの主成分（脚ブロックの型）',ex:'p_bb',opts:[['unk','不明'],['L','陰性優位（LBBB型）'],['R','陽性優位（RBBB型）']]},
 {id:'inf',label:'下壁誘導（II・III・aVF）',ex:'p_inf',opts:[['unk','不明'],['pos','すべて陽性（下方軸）'],['neg','すべて陰性（上方軸）'],['mix','混在（例：II陰性・III陽性）']]},
 {id:'r3',label:'下壁誘導のR波の高さ（下方軸のとき）',opts:[['unk','不明・該当なし'],['III','III > II'],['II','II ≥ III']]},
 {id:'I',label:'I誘導',ex:'p_I',opts:[['unk','不明'],['pos','陽性'],['iso','等電位・二相性'],['neg','陰性（rS・QS）']]},
 {id:'avl',label:'aVL誘導',opts:[['unk','不明'],['pos','陽性'],['neg','陰性']]},
 {id:'qavl',label:'aVLの陰性がaVRより深い（Q波の深さの比 aVL/aVR >1.4）',opts:[['unk','不明'],['y','はい'],['n','いいえ']]},
 {id:'tz',label:'胸部誘導の移行帯（R≥Sになる最初の誘導）',ex:'p_tz',opts:[['unk','不明'],['conc','陽性一致（V1〜V6すべてR優位）'],['V1','V1'],['V2','V2'],['V3','V3'],['V4','V4'],['V5','V5'],['V6','V6'],['negc','移行なし（陰性一致）']]},
 {id:'tzs',label:'洞調律の移行帯との比較（特に移行帯V3のとき）',opts:[['unk','不明・洞調律の記録なし'],['earlier','PVCの方が早い'],['same','同じ'],['later','PVCの方が遅い']]},
 {id:'v1m',label:'V1の波形',opts:[['unk','不明'],['QS','QS'],['rS','rS'],['qrS','qrS'],['WM','W型・M型（多相性）'],['qR','qR'],['R','単相性R・Rs'],['rsR','rsR′（3相性）']]},
 {id:'notch',label:'下壁誘導のノッチ',opts:[['unk','不明'],['y','あり'],['n','なし']]},
 {id:'pd',label:'疑似デルタ波（QRS初期のゆっくりした立ち上がり）',opts:[['unk','不明'],['y','あり'],['n','なし']]}
];
const PV={qrs:'',tmd:''}; let PVCRES=null;
const TZ={conc:0,V1:1,V2:2,V3:3,V4:4,V5:5,V6:6,negc:7};

function renderPVC(){
  let h=`<h2>PVC起源推定</h2>
  <div class="warn">心電図による起源の推定は目安です。電極の位置、心臓の回転、体格、構造的心疾患によって波形は変わり、最終的な部位はマッピングで決まります。このモジュールは、一般に報告されている心電図所見を点数化したもので、検証済みの予測モデルではありません。</div>
  <p class="note">PVCの1拍について、分かる項目だけ選んでください（不明のままでも構いません）。</p>`;
  h+=PF.map(fd=>`<div class="fld"><label>${fd.label}</label>
    <select id="p_${fd.id}" onchange="PV['${fd.id}']=this.value">${fd.opts.map(o=>`<option value="${o[0]}"${(PV[fd.id]||'unk')===o[0]?' selected':''}>${o[1]}</option>`).join('')}</select>
    ${fd.ex?ex(fd.ex):''}</div>`).join('');
  h+=`<div class="fld"><label>QRS幅とMDI（任意）</label>
    <div class="lead"><b>QRS幅</b><input id="p_qrs" type="number" inputmode="decimal" placeholder="ms" value="${PV.qrs}" oninput="PV.qrs=this.value;updMDI()">
    <button class="sec sm" onclick="useCalTo('p_qrs','t')">キャリパー値</button></div>
    <div class="lead"><b>最大振れ<br>までの時間</b><input id="p_tmd" type="number" inputmode="decimal" placeholder="ms" value="${PV.tmd}" oninput="PV.tmd=this.value;updMDI()">
    <button class="sec sm" onclick="useCalTo('p_tmd','t')">キャリパー値</button></div>
    <div id="mdiv" class="note"></div>${ex('p_mdi')}</div>
  <button onclick="runPVC()">起源を推定</button>
  <button class="sec" onclick="resetPVC()">リセット</button>
  <div id="pvcOut"></div>`;
  $('pvc').innerHTML=h; updMDI();
}
function getMDI(){ const q=parseFloat(PV.qrs), t=parseFloat(PV.tmd); return (q>0&&t>0)?t/q:null; }
function updMDI(){ const m=getMDI(), d=$('mdiv'); if(d) d.textContent=m==null?'MDI：—（両方入力すると計算します）':'MDI = '+m.toFixed(2)+(m>=0.55?'（≥0.55：心外膜起源を示唆）':''); }
function resetPVC(){ Object.keys(PV).forEach(k=>delete PV[k]); PV.qrs=''; PV.tmd=''; PVCRES=null; renderPVC(); }

function candidates(f){
  const t=TZ[f.tz], q=f.qrs, m=f.mdi;
  const L=f.bb==='L', Rb=f.bb==='R';
  const infP=f.inf==='pos', infN=f.inf==='neg', infM=f.inf==='mix';
  const Ipos=f.I==='pos', Iiso=f.I==='iso', Ineg=f.I==='neg';
  const early=t!==undefined&&t<=2, late=t>=4&&t<=6, mid35=t>=3&&t<=5;
  const wide=q>=140, narrow=q>0&&q<130;
  return [
  {name:'右室流出路（RVOT）',gate:!Rb&&!infN&&!infM,r:[
    [L,2,'LBBB型'],[infP,2,'下方軸'],[late,3,'移行帯がV4以降（遅い）'],
    [f.tz==='V3',1,'移行帯V3（RVOTとLVOTの境界）'],
    [f.tz==='V3'&&f.tzs==='later',2,'移行帯V3で、洞調律より遅い'],
    [f.tz==='V3'&&f.tzs==='earlier',-2,'移行帯V3で、洞調律より早い（LVOT寄り）'],
    [early,-3,'移行帯がV2以前（LVOT寄り）'],[Ineg||Iiso,0.5,'I誘導が陰性・等電位']],
   sub:()=>{const a=[];
     if(wide||f.notch==='y') a.push('自由壁寄り（QRS幅 ≥140 ms、下壁誘導のノッチ）'); else if(q>0) a.push('中隔寄り（QRSが比較的狭い）');
     if(Ineg) a.push('前方・左寄り（I誘導が陰性）'); if(Ipos) a.push('後方・右寄り（I誘導が陽性）');
     a.push('肺動脈起源も鑑別（下壁誘導のR波がより高く、aVLがより深い陰性になりやすい）'); return a.join('／');},
   d:'LBBB型・下方軸で、移行帯がV4以降になるのが典型的です。自由壁側はQRSが広く下壁誘導にノッチが出やすく、中隔側はQRSが比較的狭くなります。'},
  {name:'右冠尖（RCC）',gate:!Rb&&!infN,r:[
    [L,1,'LBBB型'],[infP,2,'下方軸'],[f.tz==='V3',2,'移行帯V3'],[f.tz==='V2',1,'移行帯V2'],
    [f.tzs==='earlier',1,'洞調律より移行帯が早い'],[Ipos,1,'I誘導が陽性'],[f.r3==='II',1,'II ≥ III'],[late,-2,'移行帯がV4以降（RVOT寄り）']],
   d:'LVOTの中で最も右前方にあり、RVOTの後方の中隔側に接しています。LBBB型・下方軸で移行帯はV2〜V3、I誘導は陽性になりやすいとされます。'},
  {name:'左冠尖（LCC）',gate:!infN,r:[
    [infP,2,'下方軸'],[early,2,'移行帯がV2以前'],[f.tz==='V3',1,'移行帯V3'],[f.tzs==='earlier',1,'洞調律より移行帯が早い'],
    [f.v1m==='WM',2,'V1がW型・M型'],[Ineg||Iiso,1,'I誘導が陰性・等電位'],[f.r3==='III',1,'III > II'],[late,-2,'移行帯がV4以降（RVOT寄り）']],
   d:'左後方にあるため移行帯が早く、興奮が右へ向かうのでI誘導は陰性〜等電位になりやすいです。V1が多相性（W型・M型）になることがあります。'},
  {name:'左右冠尖接合部（L-RCC）',gate:!Rb&&!infN,r:[
    [f.v1m==='qrS',3,'V1がqrS型'],[infP,1,'下方軸'],[f.tz==='V2'||f.tz==='V3',1,'移行帯V2〜V3']],
   d:'V1のqrS型（小さなq、r、深いS）が特徴的とされます。'},
  {name:'大動脈弁-僧帽弁接合部（AMC）',gate:!L&&!infN,r:[
    [Rb,1,'RBBB型'],[infP,2,'下方軸'],[f.v1m==='qR',3,'V1がqR型'],[f.tz==='conc'||f.tz==='V1',1,'移行帯がV1以前・陽性一致'],[Ineg||Iiso,1,'I誘導が陰性・等電位']],
   d:'左室基部の前方にあり、V1がqR型になるのが特徴的とされます。'},
  {name:'LV summit（心外膜側：GCV/AIVの周辺）',gate:!infN,r:[
    [infP,1,'下方軸'],[m>=0.55,3,'MDI ≥0.55'],[f.qavl==='y',2,'aVL/aVRのQ波比 >1.4'],[Ineg,2,'I誘導が陰性'],
    [f.pd==='y',1,'疑似デルタ波'],[early,1,'移行帯が早い'],[Rb,0.5,'RBBB型'],[f.r3==='III',0.5,'III > II']],
   d:'左室の最上部の心外膜側です。心外膜から心筋をゆっくり伝わるため立ち上がりが遅く（疑似デルタ波、MDIの延長）、左上方から始まる興奮でaVLやI誘導が深い陰性になりやすいです。'},
  {name:'僧帽弁輪（前側壁）',gate:!L&&!infN,r:[
    [Rb,1,'RBBB型'],[f.tz==='conc',2,'胸部誘導の陽性一致'],[infP||infM,1,'下方軸・混在'],[Ineg,1,'I誘導が陰性'],
    [f.notch==='y',1,'下壁誘導の後半のノッチ'],[f.v1m==='R',1,'V1が単相性R・Rs']],
   d:'左室の基部（後方）から前方へ向かう興奮のため、胸部誘導が陽性一致になりやすいです。側壁寄りほどI誘導が陰性になり、下壁誘導の後半にノッチが出やすいとされます。'},
  {name:'僧帽弁輪（後壁・後中隔）',gate:!L&&!infP,r:[
    [Rb,1,'RBBB型'],[infN,2,'上方軸'],[f.tz==='conc',2,'胸部誘導の陽性一致'],[f.v1m==='R',1,'V1が単相性R・Rs']],
   d:'後方の基部から前上方へ向かう興奮のため、上方軸と胸部誘導の陽性一致になりやすいです。'},
  {name:'前側壁乳頭筋',gate:!L&&!infN,r:[
    [Rb,1,'RBBB型'],[infP,1,'下方軸'],[infM,2,'下壁誘導の不一致（II陰性・III陽性）'],[mid35,1,'移行帯V3〜V5'],
    [wide,1,'QRS幅 ≥140 ms'],[Ineg,0.5,'I誘導が陰性'],[f.v1m==='qR',-1,'V1のqR型（AMC寄り）'],[narrow,-1,'QRSが狭い（刺激伝導系寄り）']],
   d:'RBBB型、下方軸、移行帯V3〜V5で、ときに下壁誘導の不一致（II陰性・III陽性）を示すとされます（Enriquez 2024）。刺激伝導系から起こる場合よりQRSが広い傾向です。'},
  {name:'後内側乳頭筋',gate:!L&&!infP,r:[
    [Rb,1,'RBBB型'],[infN,2,'上方軸'],[mid35,1,'移行帯V3〜V5'],[wide,1,'QRS幅 ≥140 ms'],
    [f.v1m==='rsR',-1,'V1のrsR′（刺激伝導系寄り）'],[narrow,-1,'QRSが狭い（刺激伝導系寄り）']],
   d:'RBBB型・上方軸で、左脚後枝から起こる場合と似ています。QRS幅（乳頭筋の方が広い）やV1の形（刺激伝導系ではrsR′）で区別します。'},
  {name:'左脚の分枝（刺激伝導系）',gate:!L,r:[
    [Rb,1,'RBBB型'],[narrow,3,'QRS幅 <130 ms'],[f.v1m==='rsR',2,'V1がrsR′'],[wide,-2,'QRS幅 ≥140 ms']],
   sub:()=>infN?'左脚後枝の領域（上方軸）':infP?'左脚前枝の領域（下方軸）':'',
   d:'刺激伝導系から出た興奮はすぐにPurkinje網に入るため、QRSが比較的狭く、右脚ブロック＋分枝ブロックに近い形（V1のrsR′）になります。'},
  {name:'三尖弁輪',gate:!Rb,r:[
    [L,1,'LBBB型'],[Ipos,2,'I誘導が陽性'],[f.avl==='pos',1,'aVLが陽性'],[f.v1m==='QS',1,'V1がQS型'],
    [infN||infM,1,'下壁誘導が陰性・混在'],[late,0.5,'移行帯がV4以降']],
   sub:()=>((f.tz==='V2'||f.tz==='V3')&&f.v1m==='QS')?'中隔側（移行帯が早め、V1がQS）':(late&&(wide||f.notch==='y'))?'自由壁側（移行帯が遅く、QRSが広い・ノッチあり）':'',
   d:'右室の基部から左へ向かう興奮のため、I誘導とaVLが陽性になりやすいです。中隔側はQRSが狭く移行帯が早め、自由壁側は移行帯が遅く、QRSが広くノッチを伴いやすいとされます。'},
  {name:'右室の調節帯・右室乳頭筋',gate:!Rb&&!infP,r:[
    [L,1,'LBBB型'],[infN,2,'上方軸'],[late,2,'移行帯がV4以降'],[Ipos,0.5,'I誘導が陽性']],
   d:'LBBB型・上方軸で移行帯が遅く（V4以降）なります。右脚に近いため、胸部誘導の立ち下がりが比較的急峻とされます。'},
  {name:'心尖部（左室・右室）',gate:!infP,r:[
    [infN,2,'上方軸'],[f.tz==='negc',3,'胸部誘導の陰性一致']],
   d:'心尖部から心基部へ向かう興奮のため、下壁誘導も胸部誘導も陰性（陰性一致）になります。構造的心疾患の評価が特に重要です。'},
  {name:'下壁の心外膜側（crux周辺）',gate:!infP,r:[
    [infN,2,'上方軸'],[m>=0.55,2,'MDI ≥0.55'],[f.pd==='y',1,'疑似デルタ波'],[early,1,'移行帯が早い']],
   d:'心臓の下面で、房室間溝と心室間溝が交わる部分です。心外膜から起こるため立ち上がりが遅く、下壁誘導がQS型になりやすいとされます。'}
  ];
}
function runPVC(){
  const f={}; PF.forEach(fd=>f[fd.id]=PV[fd.id]||'unk');
  f.qrs=parseFloat(PV.qrs)||0; f.mdi=getMDI();
  const known=PF.filter(fd=>f[fd.id]!=='unk').length+(f.qrs?1:0);
  if(known<2){ alert('少なくとも2項目以上（特に脚ブロックの型と下壁誘導）を選んでください'); return; }
  const res=candidates(f).filter(c=>c.gate).map(c=>{
    const hit=c.r.filter(x=>x[0]);
    return {c,score:hit.reduce((s,x)=>s+x[1],0),hit};
  }).filter(x=>x.score>=3).sort((a,b)=>b.score-a.score).slice(0,5);
  const inputs=PF.filter(fd=>f[fd.id]!=='unk').map(fd=>fd.label+'：'+fd.opts.find(o=>o[0]===f[fd.id])[1]);
  if(f.qrs) inputs.push('QRS幅：'+f.qrs+' ms');
  if(f.mdi!=null) inputs.push('MDI：'+f.mdi.toFixed(2));
  let h='<h2 style="margin-top:14px">推定結果</h2>';
  if(!res.length){
    h+='<p class="note">入力された所見では、候補を絞り込めませんでした。脚ブロックの型、下壁誘導、I誘導、移行帯を追加してください。</p>';
    PVCRES=null;
  } else {
    h+=res.map((x,i)=>{
      const sub=x.c.sub?x.c.sub():'';
      return `<div class="cand"><div class="cn">${i+1}. ${x.c.name}<span class="score">スコア ${x.score}</span></div>
      ${sub?`<div class="note">${sub}</div>`:''}
      <ul>${x.hit.map(y=>`<li class="${y[1]<0?'neg':''}">${y[1]<0?'（反する所見）':''}${y[2]}（${y[1]>0?'+':''}${y[1]}）</li>`).join('')}</ul>
      <details class="ex"><summary>この部位の特徴</summary><div><p>${x.c.d}</p></div></details></div>`;
    }).join('');
    h+='<p class="note">スコアは、所見が一致した度合いを示す相対的な目安で、確率ではありません。上位の候補どうしのスコア差が小さいときは、隣接する部位の間で区別がついていないと考えてください。</p>';
    PVCRES='PVC起源推定（'+new Date().toLocaleString('ja-JP')+'）\n【入力】\n・'+inputs.join('\n・')+'\n【候補】\n'+
      res.map((x,i)=>(i+1)+'. '+x.c.name+'（スコア '+x.score+'）'+(x.c.sub&&x.c.sub()?'：'+x.c.sub():'')+'\n   根拠：'+x.hit.map(y=>y[2]+'('+(y[1]>0?'+':'')+y[1]+')').join('、')).join('\n')+
      '\n※判読支援ツールによる参考結果（v'+APP_VER+'）';
    h+='<button onclick="copyText(PVCRES)">結果をコピー</button>';
  }
  $('pvcOut').innerHTML=h;
  $('pvcOut').scrollIntoView({behavior:'smooth'});
}

render(); renderPVC(); req();
</script>
</body>
</html>
