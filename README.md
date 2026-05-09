<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>武財神趙公明 - 互動祈福求籤</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            overflow: hidden;
            background: #1a0a00;
            font-family: 'Microsoft YaHei', '微軟正黑體', serif;
            cursor: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="gold"><path d="M12 2l3 7h7l-5 4 2 7-6-5-6 5 2-7-5-4h7z"/></svg>') 12 12, auto;
        }
        #gameCanvas { position: fixed; top: 0; left: 0; width: 100%; height: 100%; }
        
        .ui-layer {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none; z-index: 10;
        }
        .top-bar {
            position: absolute; top: 15px; left: 50%; transform: translateX(-50%);
            display: flex; gap: 20px; align-items: center;
            background: rgba(0,0,0,0.7); border: 2px solid #d4af37;
            border-radius: 50px; padding: 8px 20px;
        }
        .stat-item { display: flex; align-items: center; gap: 5px; color: #ffd700; font-size: 15px; font-weight: bold; }
        .stat-icon { font-size: 18px; }
        
        .side-panel {
            position: absolute; right: 10px; top: 50%; transform: translateY(-50%);
            display: flex; flex-direction: column; gap: 10px; pointer-events: all;
        }
        .power-btn {
            width: 50px; height: 50px; border-radius: 50%; border: 2px solid #d4af37;
            background: rgba(139,69,19,0.85); color: #ffd700; font-size: 20px;
            cursor: pointer; transition: all 0.3s; display: flex; align-items: center; justify-content: center;
            box-shadow: 0 0 15px rgba(212,175,55,0.3);
        }
        .power-btn:hover { transform: scale(1.1); box-shadow: 0 0 25px rgba(255,215,0,0.6); }
        .power-btn.cooldown { opacity: 0.4; cursor: not-allowed; }
        .power-btn.highlight { animation: pulse 2s infinite; border-color: #ff6347; }
        @keyframes pulse { 0%,100%{transform:scale(1)} 50%{transform:scale(1.05)} }
        
        .combo-display {
            position: absolute; top: 80px; left: 50%; transform: translateX(-50%);
            font-size: 28px; color: #ff6347; font-weight: bold; opacity: 0; pointer-events: none;
            text-shadow: 0 0 15px rgba(255,99,71,0.8);
        }
        .combo-display.show { opacity: 1; animation: pop 0.5s ease-out; }
        @keyframes pop { 0%{transform:translateX(-50%) scale(0.5)} 50%{transform:translateX(-50%) scale(1.2)} 100%{transform:translateX(-50%) scale(1)} }
        
        .floating-text {
            position: absolute; color: #ffd700; font-size: 20px; font-weight: bold;
            pointer-events: none; text-shadow: 0 0 10px rgba(255,215,0,0.8);
            animation: floatUp 1s ease-out forwards; z-index: 100;
        }
        @keyframes floatUp { 0%{opacity:1;transform:translateY(0) scale(1)} 100%{opacity:0;transform:translateY(-100px) scale(1.3)} }
        
        .mute-btn {
            position: absolute; bottom: 15px; right: 15px; pointer-events: all;
            background: rgba(0,0,0,0.6); border: 2px solid #d4af37; color: #ffd700;
            padding: 8px 12px; border-radius: 8px; cursor: pointer; font-size: 13px;
        }
        
        /* 求籤視窗 */
        .fortune-modal {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.9); display: none; align-items: center; justify-content: center;
            z-index: 1000; pointer-events: all;
        }
        .fortune-modal.active { display: flex; }
        .fortune-box {
            width: 90%; max-width: 450px; max-height: 85vh; overflow-y: auto;
            background: linear-gradient(135deg, #2c1810, #1a0a00);
            border: 3px solid #d4af37; border-radius: 15px; padding: 25px;
            position: relative; box-shadow: 0 0 50px rgba(212,175,55,0.3);
        }
        .fortune-title { text-align: center; color: #ffd700; font-size: 24px; margin-bottom: 15px; letter-spacing: 3px; }
        .fortune-desc { color: #daa520; font-size: 14px; line-height: 1.8; text-align: center; margin-bottom: 15px; padding: 10px; border: 1px solid rgba(212,175,55,0.3); border-radius: 8px; background: rgba(0,0,0,0.3); }
        .fortune-btn {
            display: block; width: 100%; padding: 12px; margin: 8px 0;
            background: linear-gradient(135deg, #8b0000, #dc143c); color: #ffd700;
            border: 2px solid #ffd700; border-radius: 50px; font-size: 16px;
            cursor: pointer; font-weight: bold; letter-spacing: 2px;
        }
        .fortune-btn:hover { transform: scale(1.02); box-shadow: 0 0 20px rgba(220,20,60,0.5); }
        .fortune-input { width: 100%; padding: 10px; margin: 8px 0; background: rgba(0,0,0,0.5); border: 2px solid #d4af37; border-radius: 8px; color: #ffd700; font-size: 15px; text-align: center; }
        .fortune-input::placeholder { color: rgba(218,165,32,0.5); }
        
        .step { display: none; animation: fadeIn 0.4s; }
        .step.active { display: block; }
        @keyframes fadeIn { from{opacity:0;transform:translateY(15px)} to{opacity:1;transform:translateY(0)} }
        
        /* 籤筒 */
        .tube-wrap { width: 100px; height: 160px; margin: 15px auto; position: relative; cursor: pointer; }
        .tube-top { width: 80px; height: 25px; background: linear-gradient(90deg, #d4af37, #ffd700); border-radius: 50%; margin: 0 auto; position: relative; top: 10px; z-index: 2; border: 2px solid #8b4513; }
        .tube-body { width: 70px; height: 130px; background: linear-gradient(90deg, #8b4513, #a0522d, #8b4513); border-radius: 0 0 35px 35px; margin: 0 auto; border: 2px solid #d4af37; position: relative; overflow: hidden; }
        .tube-shake { animation: shake 0.4s ease-in-out infinite; }
        @keyframes shake { 0%,100%{transform:rotate(0)} 25%{transform:rotate(-12deg) translateX(-8px)} 75%{transform:rotate(12deg) translateX(8px)} }
        .stick { width: 6px; height: 100px; background: linear-gradient(90deg, #d4af37, #ffd700); border-radius: 3px; position: absolute; top: -80px; left: 50%; transform: translateX(-50%); transition: all 0.6s ease-out; }
        .stick.drop { top: 15px; box-shadow: 0 0 20px rgba(255,215,0,0.8); }
        .progress-bar { width: 100%; height: 5px; background: rgba(0,0,0,0.5); border-radius: 3px; margin: 10px 0; overflow: hidden; }
        .progress-fill { height: 100%; background: linear-gradient(90deg, #ffd700, #ff8c00); width: 0%; transition: width 0.3s; }
        
        /* 擲筊 */
        .cup-box { display: flex; justify-content: center; gap: 30px; margin: 20px 0; height: 120px; align-items: center; }
        .cup { width: 50px; height: 50px; border-radius: 50%; background: linear-gradient(135deg, #4a4a4a, #2c2c2c); border: 2px solid #d4af37; transition: all 0.3s; }
        .cup.flat { background: linear-gradient(135deg, #666, #444); }
        .cup.convex { background: radial-gradient(circle at 30% 30%, #666, #222); box-shadow: inset -4px -4px 8px rgba(0,0,0,0.5); }
        .cup.tossing { animation: toss 0.5s ease-out; }
        @keyframes toss { 0%{transform:translateY(0) rotate(0)} 50%{transform:translateY(-60px) rotate(360deg)} 100%{transform:translateY(0) rotate(720deg)} }
        .cup-label { text-align: center; margin-top: 8px; color: #ffd700; font-size: 13px; }
        .sacred-result { font-size: 28px; text-align: center; margin: 15px 0; min-height: 40px; }
        .result-holy { color: #ffd700; text-shadow: 0 0 15px rgba(255,215,0,0.8); }
        .result-laugh { color: #87ceeb; }
        .result-dark { color: #999; }
        
        /* 籤詩 */
        .poem-box { background: rgba(139,69,19,0.2); border: 2px solid #d4af37; border-radius: 12px; padding: 15px; margin: 15px 0; text-align: center; }
        .poem-num { font-size: 36px; color: #dc143c; font-weight: bold; text-shadow: 0 0 15px rgba(220,20,60,0.5); margin-bottom: 8px; }
        .poem-luck { display: inline-block; padding: 4px 15px; border-radius: 15px; font-size: 18px; font-weight: bold; margin-bottom: 12px; }
        .luck-great { background: linear-gradient(135deg, #ffd700, #ff8c00); color: #8b0000; }
        .luck-good { background: linear-gradient(135deg, #98fb98, #32cd32); color: #006400; }
        .luck-mid { background: linear-gradient(135deg, #f0e68c, #daa520); color: #8b4513; }
        .luck-bad { background: linear-gradient(135deg, #d3d3d3, #a9a9a9); color: #4a4a4a; }
        .luck-terrible { background: linear-gradient(135deg, #8b0000, #dc143c); color: #ffd700; }
        .poem-text { color: #ffd700; font-size: 20px; line-height: 2; letter-spacing: 2px; margin: 10px 0; }
        .poem-explain { color: #daa520; font-size: 14px; line-height: 1.8; margin-top: 12px; padding-top: 12px; border-top: 1px solid rgba(212,175,55,0.3); }
        .aspect-box { display: flex; flex-wrap: wrap; gap: 8px; justify-content: center; margin-top: 12px; }
        .aspect-tag { padding: 4px 12px; border-radius: 12px; font-size: 13px; border: 1px solid; }
        .aspect-wealth { border-color: #ffd700; color: #ffd700; background: rgba(255,215,0,0.1); }
        .aspect-career { border-color: #87ceeb; color: #87ceeb; background: rgba(135,206,235,0.1); }
        .aspect-love { border-color: #ff69b4; color: #ff69b4; background: rgba(255,105,180,0.1); }
        .aspect-health { border-color: #98fb98; color: #98fb98; background: rgba(152,251,152,0.1); }
        
        .close-x { position: absolute; top: 10px; right: 10px; width: 30px; height: 30px; border-radius: 50%; background: rgba(139,69,19,0.8); border: 2px solid #d4af37; color: #ffd700; font-size: 18px; cursor: pointer; display: flex; align-items: center; justify-content: center; }
        .close-x:hover { background: rgba(220,20,60,0.8); transform: rotate(90deg); }
        
        .start-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.9); display: flex; flex-direction: column;
            align-items: center; justify-content: center; z-index: 100; pointer-events: all;
        }
        .start-title { font-size: 36px; color: #ffd700; text-shadow: 0 0 20px rgba(255,215,0,0.8); margin-bottom: 8px; letter-spacing: 5px; }
        .start-sub { font-size: 16px; color: #ff6b6b; margin-bottom: 25px; }
        .start-desc { color: #daa520; font-size: 14px; max-width: 400px; text-align: center; line-height: 1.8; margin-bottom: 20px; padding: 12px; border: 1px solid rgba(212,175,55,0.3); border-radius: 8px; background: rgba(0,0,0,0.4); }
        .start-btn { padding: 12px 35px; font-size: 20px; background: linear-gradient(135deg, #8b0000, #dc143c); color: #ffd700; border: 2px solid #ffd700; border-radius: 50px; cursor: pointer; font-weight: bold; letter-spacing: 3px; box-shadow: 0 0 25px rgba(220,20,60,0.5); }
        .start-btn:hover { transform: scale(1.05); box-shadow: 0 0 40px rgba(255,215,0,0.6); }
        .hidden { display: none !important; }
        
        @media (max-width: 600px) {
            .start-title { font-size: 26px; }
            .fortune-box { padding: 15px; width: 95%; }
            .top-bar { gap: 12px; padding: 6px 12px; }
            .stat-item { font-size: 13px; }
            .power-btn { width: 42px; height: 42px; font-size: 16px; }
        }
    </style>
</head>
<body>

<canvas id="gameCanvas"></canvas>

<div class="ui-layer">
    <div class="top-bar">
        <div class="stat-item"><span class="stat-icon">💰</span><span id="wealth">0</span></div>
        <div class="stat-item"><span class="stat-icon">⭐</span><span id="blessing">0</span></div>
        <div class="stat-item"><span class="stat-icon">🔥</span><span id="combo">連擊 x1</span></div>
        <div class="stat-item"><span class="stat-icon">🎋</span><span id="fcount">求籤 0</span></div>
    </div>
    <div class="combo-display" id="comboDisplay">連擊 x5!</div>
    <div class="side-panel">
        <button class="power-btn highlight" id="fortuneBtn" title="求籤 (Q)">🎋</button>
        <button class="power-btn" id="thunderBtn" title="雷電 (Space)">⚡</button>
        <button class="power-btn" id="tigerBtn" title="虎嘯 (Z)">🐯</button>
        <button class="power-btn" id="rainBtn" title="元寶雨 (X)">🌧️</button>
    </div>
    <button class="mute-btn" id="muteBtn">🔊 開啟</button>
</div>

<!-- 求籤 -->
<div class="fortune-modal" id="fmodal">
    <div class="fortune-box">
        <button class="close-x" id="fclose">×</button>
        
        <div class="step active" id="s1">
            <h2 class="fortune-title">🙏 淨心稟告</h2>
            <div class="fortune-desc">
                <p>向趙公明財神爺稟告：</p>
                <p style="margin-top:8px;color:#ffd700;font-style:italic;">
                    財神在上，弟子（姓名）<br>今日求問運勢，請賜籤指點
                </p>
            </div>
            <input type="text" class="fortune-input" id="uname" placeholder="您的姓名（選填）" maxlength="8">
            <select class="fortune-input" id="qtype">
                <option value="wealth">💰 求財運</option>
                <option value="career">💼 求事業</option>
                <option value="love">💕 求姻緣</option>
                <option value="health">🍀 求健康</option>
                <option value="general">🎯 綜合運勢</option>
            </select>
            <button class="fortune-btn" id="b1">我已淨心，開始求籤</button>
        </div>
        
        <div class="step" id="s2">
            <h2 class="fortune-title">🎋 搖動籤筒</h2>
            <div class="fortune-desc">雙手握筒，默念所求，輕搖至籤掉落</div>
            <div class="tube-wrap" id="twrap">
                <div class="tube-top"></div>
                <div class="tube-body" id="tbody">
                    <div class="stick" id="fstick"></div>
                </div>
            </div>
            <div class="progress-bar"><div class="progress-fill" id="pfill"></div></div>
            <p style="text-align:center;color:#daa520;font-size:13px;">點擊籤筒搖動</p>
        </div>
        
        <div class="step" id="s3">
            <h2 class="fortune-title">🎲 擲筊確認</h2>
            <div class="fortune-desc">連續三個<strong style="color:#ffd700">聖杯</strong>（一平一凸）方為靈籤</div>
            <div class="cup-box">
                <div><div class="cup flat" id="c1"></div><div class="cup-label" id="l1">待擲</div></div>
                <div><div class="cup flat" id="c2"></div><div class="cup-label" id="l2">待擲</div></div>
            </div>
            <div class="sacred-result" id="sresult"></div>
            <button class="fortune-btn" id="btoss">擲筊</button>
            <button class="fortune-btn hidden" id="bnext" style="background:linear-gradient(135deg,#006400,#228b22);">查看籤詩</button>
        </div>
        
        <div class="step" id="s4">
            <h2 class="fortune-title">📜 籤詩解說</h2>
            <div class="poem-box" id="pbox">
                <div class="poem-num" id="pnum">第壹籤</div>
                <div class="poem-luck luck-great" id="pluck">上上籤</div>
                <div class="poem-text" id="ptext">龍虎相隨在門前<br>財源滾滾入家園<br>此時正是亨通日<br>何必憂愁問上蒼</div>
                <div class="poem-explain" id="pexplain">此籤象徵財運亨通，龍虎護佑，諸事大吉。</div>
                <div class="aspect-box" id="paspect"></div>
            </div>
            <button class="fortune-btn" id="bagain">再求一籤</button>
            <button class="fortune-btn" id="bclose2" style="background:linear-gradient(135deg,#4a4a4a,#2c2c2c);">返回遊戲</button>
        </div>
    </div>
</div>

<div class="start-screen" id="startScreen">
    <h1 class="start-title">武財神趙公明</h1>
    <p class="start-sub">黑虎玄壇 · 迎祥納福 · 求籤問運</p>
    <div class="start-desc">
        <p>趙公明，道教尊為「正一玄壇元帥」</p>
        <p>黑面濃鬚，頭戴鐵冠，手執鐵鞭，身跨黑虎</p>
        <p>能驅雷役電，除瘟禳災，買賣求財</p>
        <p>點擊祈福收集元寶，或求籤問運勢！</p>
    </div>
    <button class="start-btn" id="startBtn">開始祈福</button>
</div>

<script>
// ===== 音效系統 =====
class SoundSystem {
    constructor() { this.ctx=null; this.muted=false; this.initd=false; }
    init() { if(this.initd)return; this.ctx=new(window.AudioContext||window.webkitAudioContext)(); this.initd=true; }
    tone(f,d,t='sine',v=0.3){ if(this.muted||!this.ctx)return; const o=this.ctx.createOscillator(),g=this.ctx.createGain(); o.type=t; o.frequency.setValueAtTime(f,this.ctx.currentTime); g.gain.setValueAtTime(v,this.ctx.currentTime); g.gain.exponentialRampToValueAtTime(0.01,this.ctx.currentTime+d); o.connect(g); g.connect(this.ctx.destination); o.start(); o.stop(this.ctx.currentTime+d); }
    coin(){ if(this.muted||!this.ctx)return; const n=this.ctx.currentTime; this.tone(1200,0.3,'sine',0.3); setTimeout(()=>this.tone(1800,0.2,'sine',0.2),50); setTimeout(()=>this.tone(2400,0.15,'square',0.1),100); }
    whip(){ if(this.muted||!this.ctx)return; const n=this.ctx.currentTime,bs=this.ctx.sampleRate*0.5,b=this.ctx.createBuffer(1,bs,this.ctx.sampleRate),d=b.getChannelData(0); for(let i=0;i<bs;i++)d[i]=(Math.random()*2-1)*Math.pow(1-i/bs,2); const ns=this.ctx.createBufferSource(); ns.buffer=b; const f=this.ctx.createBiquadFilter(); f.type='bandpass'; f.frequency.setValueAtTime(800,n); f.frequency.exponentialRampToValueAtTime(100,n+0.3); f.Q.value=1; const g=this.ctx.createGain(); g.gain.setValueAtTime(0.5,n); g.gain.exponentialRampToValueAtTime(0.01,n+0.5); ns.connect(f); f.connect(g); g.connect(this.ctx.destination); ns.start(n); }
    tiger(){ if(this.muted||!this.ctx)return; const n=this.ctx.currentTime,o=this.ctx.createOscillator(),g=this.ctx.createGain(); o.type='sawtooth'; o.frequency.setValueAtTime(80,n); o.frequency.linearRampToValueAtTime(120,n+0.5); o.frequency.linearRampToValueAtTime(60,n+1); g.gain.setValueAtTime(0.4,n); g.gain.linearRampToValueAtTime(0.6,n+0.3); g.gain.exponentialRampToValueAtTime(0.01,n+1.2); const f=this.ctx.createBiquadFilter(); f.type='lowpass'; f.frequency.setValueAtTime(300,n); f.frequency.linearRampToValueAtTime(600,n+0.5); f.frequency.linearRampToValueAtTime(200,n+1); o.connect(f); f.connect(g); g.connect(this.ctx.destination); o.start(n); o.stop(n+1.2); }
    thunder(){ if(this.muted||!this.ctx)return; const n=this.ctx.currentTime,bs=this.ctx.sampleRate*2,b=this.ctx.createBuffer(1,bs,this.ctx.sampleRate),d=b.getChannelData(0); for(let i=0;i<bs;i++)d[i]=(Math.random()*2-1)*Math.pow(1-i/bs,0.5); const ns=this.ctx.createBufferSource(); ns.buffer=b; const f=this.ctx.createBiquadFilter(); f.type='lowpass'; f.frequency.setValueAtTime(1000,n); f.frequency.exponentialRampToValueAtTime(100,n+2); const g=this.ctx.createGain(); g.gain.setValueAtTime(0.8,n); g.gain.exponentialRampToValueAtTime(0.01,n+2); ns.connect(f); f.connect(g); g.connect(this.ctx.destination); ns.start(n); this.tone(50,1.5,'sawtooth',0.4); this.tone(40,1,'square',0.2); }
    shake(){ if(this.muted||!this.ctx)return; const n=this.ctx.currentTime,bs=this.ctx.sampleRate*0.3,b=this.ctx.createBuffer(1,bs,this.ctx.sampleRate),d=b.getChannelData(0); for(let i=0;i<bs;i++)d[i]=(Math.random()*2-1)*Math.sin(i/bs*Math.PI); const ns=this.ctx.createBufferSource(); ns.buffer=b; const f=this.ctx.createBiquadFilter(); f.type='bandpass'; f.frequency.setValueAtTime(600,n); f.frequency.linearRampToValueAtTime(400,n+0.3); const g=this.ctx.createGain(); g.gain.setValueAtTime(0.4,n); g.gain.exponentialRampToValueAtTime(0.01,n+0.3); ns.connect(f); f.connect(g); g.connect(this.ctx.destination); ns.start(n); }
    stick(){ if(this.muted||!this.ctx)return; this.tone(800,0.1,'sine',0.3); setTimeout(()=>this.tone(1200,0.15,'sine',0.2),50); setTimeout(()=>this.tone(600,0.2,'triangle',0.25),100); }
    toss(){ if(this.muted||!this.ctx)return; const n=this.ctx.currentTime,bs=this.ctx.sampleRate*0.2,b=this.ctx.createBuffer(1,bs,this.ctx.sampleRate),d=b.getChannelData(0); for(let i=0;i<bs;i++)d[i]=(Math.random()*2-1)*(1-i/bs); const ns=this.ctx.createBufferSource(); ns.buffer=b; const f=this.ctx.createBiquadFilter(); f.type='highpass'; f.frequency.setValueAtTime(2000,n); const g=this.ctx.createGain(); g.gain.setValueAtTime(0.5,n); g.gain.exponentialRampToValueAtTime(0.01,n+0.2); ns.connect(f); f.connect(g); g.connect(this.ctx.destination); ns.start(n); }
    holy(){ [523,659,784,1046].forEach((f,i)=>setTimeout(()=>this.tone(f,0.4,'sine',0.2),i*150)); }
    laugh(){ this.tone(400,0.3,'sine',0.2); setTimeout(()=>this.tone(350,0.3,'sine',0.2),200); }
    dark(){ this.tone(200,0.5,'sawtooth',0.2); setTimeout(()=>this.tone(150,0.5,'sawtooth',0.15),300); }
    melody(){ if(this.muted||!this.ctx)return; const n=[261,293,329,392,329,293]; let d=0; n.forEach((f,i)=>{setTimeout(()=>this.tone(f,0.8,'sine',0.12),d); d+=800; }); }
    toggle(){ this.muted=!this.muted; return this.muted; }
}

// ===== 籤詩資料 =====
const POEMS=[
    {n:"第一籤",l:"上上籤",c:"luck-great",t:"龍虎相隨在門前\n財源滾滾入家園\n此時正是亨通日\n何必憂愁問上蒼",e:"此籤象徵財運亨通，龍虎護佑。趙公明騎黑虎、執鐵鞭，正是龍虎精神之象。無論事業投資或偏財運勢皆大吉，宜把握時機，大膽進取。",a:{wealth:"大吉",career:"順遂",love:"美滿",health:"安康"}},
    {n:"第二籤",l:"上吉籤",c:"luck-good",t:"金銀滿庫福自來\n貴人相助運開懷\n莫道前程多險阻\n玄壇元帥護君財",e:"財庫豐盈，貴人提攜。趙公明率五路財神，象徵多方財源匯聚。雖有小阻，終能化險為夷。宜多行善積德。",a:{wealth:"上吉",career:"有貴人",love:"平穩",health:"無礙"}},
    {n:"第三籤",l:"中吉籤",c:"luck-mid",t:"黑虎咆哮震四方\n威風凜凜護玄壇\n財來須仗辛勤力\n坐享其成恐未安",e:"黑虎顯威，護法鎮煞。此籤示財運需靠努力經營，不可投機取巧。勤奮不懈，終有收穫。",a:{wealth:"中吉",career:"需努力",love:"緩進",health:"注意勞累"}},
    {n:"第四籤",l:"中平籤",c:"luck-mid",t:"鐵鞭高舉鎮乾坤\n善惡分明報應真\n財運平平須守舊\n靜待時機莫強求",e:"鐵鞭象徵公正與規矩。此籤示財運平平，宜守不宜攻。靜心等待時機，保守經營為上。",a:{wealth:"平平",career:"守舊",love:"平淡",health:"調養"}},
    {n:"第五籤",l:"下下籤",c:"luck-terrible",t:"迷霧重重遮財路\n心急如火反招誤\n且將慾念暫收起\n虔誠悔改求玄壇",e:"此籤示財運不順，可能因貪念或急躁所致。宜反省自身，誠心懺悔，暫緩重大財務決策。",a:{wealth:"不佳",career:"受阻",love:"波折",health:"欠安"}},
    {n:"第六籤",l:"上吉籤",c:"luck-good",t:"五路財神齊降臨\n東西南北聚黃金\n此月正值亨通時\n投資置產皆順心",e:"五路財神到，八方財源聚。此籤大吉，象徵財運全面開展。宜把握時機，但勿過度擴張。",a:{wealth:"大吉",career:"高升",love:"甜蜜",health:"健朗"}},
    {n:"第七籤",l:"中平籤",c:"luck-mid",t:"日精月華養元神\n趙帥當年修此身\n財運須隨緣分至\n強求不得反勞神",e:"趙公明原為日精之一，後修煉得道。此籤示財運隨緣，不可強求。宜修身養性，充實自我。",a:{wealth:"隨緣",career:"進修",love:"順其自然",health:"養生"}},
    {n:"第八籤",l:"上上籤",c:"luck-great",t:"正一玄壇號令行\n雷電風雨皆聽令\n財如春雨連綿至\n福似東海萬年盈",e:"趙公明受封「正一玄壇元帥」，能驅雷役電。此籤象徵財運如春雨連綿不絕，福氣如東海浩瀚無邊。",a:{wealth:"極佳",career:"飛黃騰達",love:"天作之合",health:"百病不侵"}},
    {n:"第九籤",l:"中下籤",c:"luck-bad",t:"財神門前燒香忙\n心誠不夠意彷徨\n欲得元寶須先捨\n布施福田種吉祥",e:"此籤示求財之心雖切，但誠意不足。趙公明掌管賞罰，提醒財富來自於付出與分享。宜多行善布施。",a:{wealth:"需努力",career:"波折",love:"誠意為上",health:"心寬則安"}},
    {n:"第十籤",l:"上吉籤",c:"luck-good",t:"招寶納珍兩天尊\n左右護持財運新\n舊債清償無牽絆\n輕裝上陣步青雲",e:"招寶天尊、納珍天尊為趙公明麾下。此籤示財運新生，舊有財務問題將得解決。適合開創新事業。",a:{wealth:"新生",career:"轉機",love:"新緣",health:"煥然一新"}},
    {n:"第十一籤",l:"中吉籤",c:"luck-mid",t:"峨眉山頭雲霧深\n九老洞中煉丹心\n財運如登山險路\n一步一腳印自真",e:"趙公明於峨眉山九老洞修煉得道。此籤示財運如登山，需一步一腳印，踏實經營。適合長期規劃。",a:{wealth:"漸進",career:"穩步",love:"細水長流",health:"適度運動"}},
    {n:"第十二籤",l:"大吉籤",c:"luck-great",t:"封神榜上顯威靈\n金龍如意賜康寧\n財福壽喜齊聚會\n闔家歡樂滿門庭",e:"趙公明受姜子牙封神，為「金龍如意正一龍虎玄壇真君」。此籤為全運大吉，財福壽喜皆至，闔家平安。",a:{wealth:"全盛",career:"巔峰",love:"圓滿",health:"長壽"}}
];
const NUMS=["壹","貳","參","肆","伍","陸","柒","捌","玖","拾","拾壹","拾貳"];

// ===== 求籤系統 =====
class FortuneSystem{
    constructor(snd){ this.snd=snd; this.step=1; this.fortune=null; this.tossC=0; this.tossR=[]; this.shaking=false; this.fcount=0; this.setup(); }
    setup(){
        document.getElementById('fortuneBtn').addEventListener('click',()=>this.open());
        document.getElementById('fclose').addEventListener('click',()=>this.close());
        document.getElementById('b1').addEventListener('click',()=>this.go(2));
        document.getElementById('twrap').addEventListener('click',()=>this.shake());
        document.getElementById('btoss').addEventListener('click',()=>this.toss());
        document.getElementById('bnext').addEventListener('click',()=>{this.go(4);this.show();});
        document.getElementById('bagain').addEventListener('click',()=>this.reset());
        document.getElementById('bclose2').addEventListener('click',()=>this.close());
        window.addEventListener('keydown',(e)=>{ if(e.key==='q'||e.key==='Q')this.open(); if(document.getElementById('fmodal').classList.contains('active')){ if(e.key===' '&&this.step===2)this.shake(); if(e.key==='Escape')this.close(); } });
    }
    open(){ if(!this.snd.initd)this.snd.init(); document.getElementById('fmodal').classList.add('active'); this.reset(); this.snd.tone(600,0.3,'sine',0.2); }
    close(){ document.getElementById('fmodal').classList.remove('active'); this.snd.tone(400,0.2,'sine',0.15); }
    reset(){
        this.step=1; this.tossC=0; this.tossR=[]; this.shaking=false;
        document.getElementById('fstick').classList.remove('drop');
        document.getElementById('tbody').classList.remove('tube-shake');
        document.getElementById('pfill').style.width='0%';
        ['c1','c2'].forEach(id=>{const el=document.getElementById(id); el.className='cup flat';});
        document.getElementById('l1').textContent='待擲'; document.getElementById('l1').className='cup-label';
        document.getElementById('l2').textContent='待擲'; document.getElementById('l2').className='cup-label';
        document.getElementById('sresult').innerHTML='';
        document.getElementById('btoss').classList.remove('hidden');
        document.getElementById('bnext').classList.add('hidden');
        document.querySelectorAll('.step').forEach(s=>s.classList.remove('active'));
        document.getElementById('s1').classList.add('active');
    }
    go(n){ document.getElementById('s'+this.step).classList.remove('active'); document.getElementById('s'+n).classList.add('active'); this.step=n; this.snd.tone(500+n*100,0.3,'sine',0.2); }
    shake(){ if(this.shaking||this.step!==2)return; this.shaking=true; document.getElementById('tbody').classList.add('tube-shake'); this.snd.shake(); let p=0; const iv=setInterval(()=>{p+=3; document.getElementById('pfill').style.width=p+'%'; if(p>=100){clearInterval(iv); this.finShake();} },30); }
    finShake(){
        document.getElementById('tbody').classList.remove('tube-shake');
        document.getElementById('fstick').classList.add('drop'); this.snd.stick();
        const qt=document.getElementById('qtype').value;
        const w=qt==='wealth'?[0.12,0.11,0.10,0.09,0.05,0.12,0.08,0.12,0.06,0.11,0.10,0.12]:qt==='general'?Array(12).fill(1/12):[0.11,0.10,0.10,0.10,0.08,0.11,0.09,0.11,0.08,0.10,0.10,0.11];
        let r=Math.random(),c=0,idx=0; for(let i=0;i<12;i++){c+=w[i]; if(r<=c){idx=i; break;} }
        this.fortune=POEMS[idx];
        setTimeout(()=>{this.go(3); this.shaking=false;},1200);
    }
    toss(){
        if(this.tossC>=3)return;
        const c1=document.getElementById('c1'), c2=document.getElementById('c2');
        c1.classList.add('tossing'); c2.classList.add('tossing'); this.snd.toss();
        setTimeout(()=>{
            c1.classList.remove('tossing'); c2.classList.remove('tossing');
            const rand=Math.random(), res=rand<0.6?'holy':rand<0.85?'laugh':'dark';
            if(res==='holy'){ c1.className='cup flat'; c2.className='cup convex'; }
            else if(res==='laugh'){ c1.className='cup flat'; c2.className='cup flat'; }
            else { c1.className='cup convex'; c2.className='cup convex'; }
            this.tossR.push(res); this.tossC++;
            const txt={holy:'聖杯',laugh:'笑杯',dark:'陰杯'};
            const cls={holy:'result-holy',laugh:'result-laugh',dark:'result-dark'};
            document.getElementById('l1').textContent=txt[res];
            document.getElementById('l1').className='cup-label '+cls[res];
            if(res==='holy')this.snd.holy(); else if(res==='laugh')this.snd.laugh(); else this.snd.dark();
            if(this.tossC===3){
                const holy=this.tossR.filter(x=>x==='holy').length;
                const sr=document.getElementById('sresult');
                if(holy===3){ sr.innerHTML='<span class="result-holy">✨ 三聖杯確認！靈籤 ✨</span>'; this.snd.holy(); document.getElementById('btoss').classList.add('hidden'); document.getElementById('bnext').classList.remove('hidden'); }
                else if(holy>=2){ sr.innerHTML='<span class="result-holy">🙏 神明應允，可參考 🙏</span>'; document.getElementById('btoss').classList.add('hidden'); document.getElementById('bnext').classList.remove('hidden'); }
                else { sr.innerHTML='<span class="result-dark">⚠️ 示意不明，建議重求 ⚠️</span>'; document.getElementById('btoss').textContent='重新擲筊'; this.tossC=0; this.tossR=[]; setTimeout(()=>{document.getElementById('l1').textContent='待擲'; document.getElementById('l1').className='cup-label';},2000); }
            }
        },500);
    }
    show(){
        if(!this.fortune)return; const f=this.fortune;
        document.getElementById('pnum').textContent='第'+NUMS[POEMS.indexOf(f)]+'籤';
        document.getElementById('pluck').textContent=f.l; document.getElementById('pluck').className='poem-luck '+f.c;
        document.getElementById('ptext').innerHTML=f.t.replace(/\n/g,'<br>');
        document.getElementById('pexplain').textContent=f.e;
        const ab=document.getElementById('paspect'); ab.innerHTML='';
        const am={wealth:['aspect-wealth','💰 財運'],career:['aspect-career','💼 事業'],love:['aspect-love','💕 姻緣'],health:['aspect-health','🍀 健康']};
        Object.entries(f.a).forEach(([k,v])=>{ const s=document.createElement('span'); s.className='aspect-tag '+am[k][0]; s.textContent=am[k][1]+'：'+v; ab.appendChild(s); });
        this.fcount++; document.getElementById('fcount').textContent='求籤 '+this.fcount;
        const rew=f.c==='luck-great'?100:f.c==='luck-good'?50:f.c==='luck-mid'?25:10;
        window.game?.addWealth(rew);
        this.snd.holy();
    }
}

// ===== 遊戲主體 =====
class Game{
    constructor(){
        this.canvas=document.getElementById('gameCanvas'); this.ctx=this.canvas.getContext('2d');
        this.snd=new SoundSystem(); this.fortune=new FortuneSystem(this.snd);
        this.w=window.innerWidth; this.h=window.innerHeight; this.canvas.width=this.w; this.canvas.height=this.h;
        this.wealth=0; this.bless=0; this.combo=0; this.lastClick=0;
        this.yuanbaos=[]; this.particles=[]; this.thunders=[]; this.bgOff=0;
        this.god={x:this.w/2,y:this.h/2+50,s:1,ts:1,whip:0,whipOn:false,roar:false};
        this.playing=false; this.flash=0; this.cd={thunder:0,tiger:0,rain:0};
        this.setup(); this.resize();
        setInterval(()=>{if(this.playing)this.snd.melody();},6000);
        window.game=this;
    }
    setup(){
        window.addEventListener('resize',()=>this.resize());
        document.getElementById('startBtn').addEventListener('click',()=>this.start());
        this.canvas.addEventListener('click',e=>this.click(e));
        this.canvas.addEventListener('touchstart',e=>{e.preventDefault(); const t=e.touches[0]; this.click({clientX:t.clientX,clientY:t.clientY});});
        window.addEventListener('keydown',e=>{ if(!this.playing)return; if(e.key===' ')this.thunder(); if(e.key==='z'||e.key==='Z')this.tiger(); if(e.key==='x'||e.key==='X')this.rain(); });
        document.getElementById('thunderBtn').addEventListener('click',()=>this.thunder());
        document.getElementById('tigerBtn').addEventListener('click',()=>this.tiger());
        document.getElementById('rainBtn').addEventListener('click',()=>this.rain());
        document.getElementById('muteBtn').addEventListener('click',e=>{ const m=this.snd.toggle(); e.target.textContent=m?'🔇 關閉':'🔊 開啟'; });
    }
    resize(){ this.w=window.innerWidth; this.h=window.innerHeight; this.canvas.width=this.w; this.canvas.height=this.h; this.god.x=this.w/2; this.god.y=this.h/2+50; }
    start(){ document.getElementById('startScreen').classList.add('hidden'); this.snd.init(); this.playing=true; this.snd.melody(); this.loop(); }
    addWealth(a){ this.wealth+=a; this.bless+=Math.floor(a/5); this.updateUI(); const x=this.w/2+(Math.random()-0.5)*200,y=this.h/2; this.floatText(x,y,'+'+a+' 元寶'); this.spawnPart(x,y,'#ffd700',8); this.snd.coin(); }
    click(e){
        if(!this.playing)return;
        const r=this.canvas.getBoundingClientRect(),x=e.clientX-r.left,y=e.clientY-r.top;
        const now=Date.now(); if(now-this.lastClick<1000)this.combo++; else this.combo=1; this.lastClick=now;
        document.getElementById('combo').textContent='連擊 x'+this.combo;
        if(this.combo>=5){ const d=document.getElementById('comboDisplay'); d.textContent='連擊 x'+this.combo+'!'; d.classList.add('show'); setTimeout(()=>d.classList.remove('show'),1000); }
        const v=Math.floor(Math.random()*10*this.combo)+5;
        this.yuanbaos.push({x:x,y:y,vx:(Math.random()-0.5)*4,vy:-4-Math.random()*4,r:Math.random()*Math.PI*2,rs:(Math.random()-0.5)*0.15,s:0,ts:1,life:1,val:v});
        this.spawnPart(x,y,'#ffd700',10); this.floatText(x,y,'+'+v+' 元寶');
        this.snd.coin(); this.wealth+=v; this.bless+=Math.floor(v/5); this.updateUI();
        this.god.ts=1.12; setTimeout(()=>this.god.ts=1,150);
        this.god.whipOn=true; setTimeout(()=>this.god.whipOn=false,300); this.snd.whip();
    }
    thunder(){ if(this.cd.thunder>0||!this.playing)return; this.cd.thunder=5; this.snd.thunder(); for(let i=0;i<5;i++){ setTimeout(()=>{ const x=Math.random()*this.w; this.thunders.push({x:x,y:0,ty:this.h,w:2+Math.random()*3,life:0.3}); this.yuanbaos.push({x:x,y:this.h*0.3,vx:0,vy:-3,r:0,rs:0,s:1,ts:1,life:1,val:50}); this.spawnPart(x,this.h*0.3,'#00ffff',12); },i*200); } this.flash=1; this.cdBtn('thunderBtn',5); }
    tiger(){ if(this.cd.tiger>0||!this.playing)return; this.cd.tiger=8; this.snd.tiger(); this.god.roar=true; setTimeout(()=>this.god.roar=false,2000); for(let i=0;i<20;i++){ setTimeout(()=>{ const a=(Math.PI*2/20)*i,d=180+Math.random()*80; const x=this.god.x+Math.cos(a)*d,y=this.god.y+Math.sin(a)*d; this.yuanbaos.push({x:x,y:y,vx:Math.cos(a)*3,vy:Math.sin(a)*3,r:Math.random()*Math.PI*2,rs:0.1,s:0,ts:1,life:1,val:30}); this.spawnPart(x,y,'#ff6347',5); },i*50); } this.cdBtn('tigerBtn',8); }
    rain(){ if(this.cd.rain>0||!this.playing)return; this.cd.rain=10; let c=0; const iv=setInterval(()=>{ if(c>=30){clearInterval(iv); return;} const x=Math.random()*this.w; this.yuanbaos.push({x:x,y:-40,vx:0,vy:2+Math.random()*2,r:0,rs:0.05,s:0,ts:1,life:1,val:20}); this.snd.coin(); c++; },100); this.cdBtn('rainBtn',10); }
    cdBtn(id,s){ const b=document.getElementById(id); b.classList.add('cooldown'); let r=s; const t=setInterval(()=>{ r--; if(r<=0){clearInterval(t); b.classList.remove('cooldown'); this.cd[id.replace('Btn','')]=0;} },1000); }
    spawnPart(x,y,c,n){ for(let i=0;i<n;i++){ const a=(Math.PI*2/n)*i+Math.random()*0.5,sp=2+Math.random()*4; this.particles.push({x:x,y:y,vx:Math.cos(a)*sp,vy:Math.sin(a)*sp,life:1,color:c,size:3+Math.random()*4}); } }
    floatText(x,y,t){ const el=document.createElement('div'); el.className='floating-text'; el.textContent=t; el.style.left=x+'px'; el.style.top=y+'px'; document.body.appendChild(el); setTimeout(()=>el.remove(),1000); }
    updateUI(){ document.getElementById('wealth').textContent=this.wealth; document.getElementById('blessing').textContent=this.bless; }
    
    update(){
        if(!this.playing)return; this.bgOff+=0.5;
        this.god.s+=(this.god.ts-this.god.s)*0.1; this.god.whip+=0.1;
        this.yuanbaos=this.yuanbaos.filter(yb=>{
            yb.x+=yb.vx; yb.y+=yb.vy; yb.vy+=0.18; yb.r+=yb.rs; yb.s+=(yb.ts-yb.s)*0.1;
            if(yb.x<0||yb.x>this.w)yb.vx*=-0.8; if(yb.y>this.h-40){yb.vy*=-0.6; yb.y=this.h-40;}
            yb.life-=0.004; return yb.life>0;
        });
        this.particles=this.particles.filter(p=>{ p.x+=p.vx; p.y+=p.vy; p.vy+=0.1; p.life-=0.02; return p.life>0; });
        this.thunders=this.thunders.filter(t=>{ t.life-=0.016; return t.life>0; });
        if(this.flash>0){this.flash-=0.05; if(this.flash<0)this.flash=0;}
    }
    
    draw(){
        this.ctx.fillStyle='#1a0a00'; this.ctx.fillRect(0,0,this.w,this.h);
        // 背景
        const g=this.ctx.createRadialGradient(this.w/2,this.h/2,0,this.w/2,this.h/2,this.w*0.8);
        g.addColorStop(0,'#2d1b00'); g.addColorStop(0.5,'#1a0a00'); g.addColorStop(1,'#0d0500');
        this.ctx.fillStyle=g; this.ctx.fillRect(0,0,this.w,this.h);
        // 祥雲
        for(let i=0;i<5;i++){ const x=((i*300+this.bgOff*0.3)%(this.w+400))-200,y=80+i*70+Math.sin(this.bgOff*0.01+i)*15; this.drawCloud(x,y,0.7+i*0.15); }
        // 地面
        const gg=this.ctx.createRadialGradient(this.w/2,this.h,0,this.w/2,this.h,this.w*0.6);
        gg.addColorStop(0,'rgba(139,69,19,0.4)'); gg.addColorStop(1,'rgba(0,0,0,0)');
        this.ctx.fillStyle=gg; this.ctx.fillRect(0,this.h-180,this.w,180);
        // 雷電
        this.thunders.forEach(t=>{
            this.ctx.save(); this.ctx.strokeStyle=`rgba(200,220,255,${t.life*2})`; this.ctx.lineWidth=t.w; this.ctx.shadowColor='#00ffff'; this.ctx.shadowBlur=15;
            this.ctx.beginPath(); this.ctx.moveTo(t.x,t.y); let cx=t.x,cy=t.y; for(let i=0;i<10;i++){ cx+=(Math.random()-0.5)*35; cy+=(t.ty-t.y)/10; this.ctx.lineTo(cx,cy); } this.ctx.stroke(); this.ctx.restore();
        });
        // 元寶
        this.yuanbaos.forEach(yb=>{ this.ctx.save(); this.ctx.translate(yb.x,yb.y); this.ctx.rotate(yb.r); this.ctx.scale(yb.s,yb.s); this.drawYuanbao(); this.ctx.restore(); });
        // 粒子
        this.particles.forEach(p=>{ this.ctx.save(); this.ctx.globalAlpha=p.life; this.ctx.fillStyle=p.color; this.ctx.shadowColor=p.color; this.ctx.shadowBlur=8; this.ctx.beginPath(); this.ctx.arc(p.x,p.y,p.size*p.life,0,Math.PI*2); this.ctx.fill(); this.ctx.restore(); });
        // 趙公明
        this.drawGod();
        // 閃屏
        if(this.flash>0){ this.ctx.fillStyle=`rgba(255,255,255,${this.flash*0.25})`; this.ctx.fillRect(0,0,this.w,this.h); }
    }
    drawCloud(x,y,s){ this.ctx.save(); this.ctx.translate(x,y); this.ctx.scale(s,s); this.ctx.fillStyle='rgba(255,215,0,0.12)'; this.ctx.beginPath(); this.ctx.arc(0,0,50,0,Math.PI*2); this.ctx.arc(40,-8,42,0,Math.PI*2); this.ctx.arc(80,4,45,0,Math.PI*2); this.ctx.arc(35,15,38,0,Math.PI*2); this.ctx.fill(); this.ctx.restore(); }
    drawYuanbao(){
        const g=this.ctx.createRadialGradient(-8,-8,0,0,0,35); g.addColorStop(0,'#fff8dc'); g.addColorStop(0.3,'#ffd700'); g.addColorStop(1,'#daa520'); this.ctx.fillStyle=g;
        this.ctx.beginPath(); this.ctx.moveTo(-25,0); this.ctx.quadraticCurveTo(-25,-20,0,-25); this.ctx.quadraticCurveTo(25,-20,25,0); this.ctx.quadraticCurveTo(25,12,0,16); this.ctx.quadraticCurveTo(-25,12,-25,0); this.ctx.fill();
        this.ctx.beginPath(); this.ctx.ellipse(0,-22,12,8,0,0,Math.PI*2); this.ctx.fill();
        this.ctx.fillStyle='rgba(255,255,255,0.35)'; this.ctx.beginPath(); this.ctx.ellipse(-8,-12,6,4,-0.5,0,Math.PI*2); this.ctx.fill();
    }
    drawGod(){
        const g=this.god,s=g.s,x=g.x,y=g.y; this.ctx.save(); this.ctx.translate(x,y); this.ctx.scale(s,s);
        // 光環
        const hg=this.ctx.createRadialGradient(0,-70,15,0,-70,130); hg.addColorStop(0,'rgba(255,215,0,0.25)'); hg.addColorStop(0.5,'rgba(255,140,0,0.12)'); hg.addColorStop(1,'rgba(255,215,0,0)');
        this.ctx.fillStyle=hg; this.ctx.beginPath(); this.ctx.arc(0,-70,130,0,Math.PI*2); this.ctx.fill();
        // 黑虎
        this.drawTiger(-70,15);
        // 身體
        this.ctx.fillStyle='#8b0000'; this.ctx.beginPath(); this.ctx.moveTo(-45,-45); this.ctx.lineTo(45,-45); this.ctx.lineTo(60,90); this.ctx.lineTo(-60,90); this.ctx.closePath(); this.ctx.fill();
        this.ctx.strokeStyle='#ffd700'; this.ctx.lineWidth=2; this.ctx.beginPath(); this.ctx.moveTo(-45,-45); this.ctx.lineTo(45,-45); this.ctx.lineTo(60,90); this.ctx.stroke();
        // 頭
        this.ctx.fillStyle='#2c1810'; this.ctx.beginPath(); this.ctx.arc(0,-90,40,0,Math.PI*2); this.ctx.fill();
        // 鬍鬚
        this.ctx.fillStyle='#1a1a1a'; this.ctx.beginPath(); this.ctx.moveTo(-25,-80); this.ctx.quadraticCurveTo(0,-35,25,-80); this.ctx.lineTo(22,-65); this.ctx.quadraticCurveTo(0,-18,-22,-65); this.ctx.closePath(); this.ctx.fill();
        // 眼睛
        this.ctx.fillStyle='#ff0000'; this.ctx.beginPath(); this.ctx.arc(-13,-95,7,0,Math.PI*2); this.ctx.arc(13,-95,7,0,Math.PI*2); this.ctx.fill();
        this.ctx.fillStyle='#ffd700'; this.ctx.beginPath(); this.ctx.arc(-13,-97,3,0,Math.PI*2); this.ctx.arc(13,-97,3,0,Math.PI*2); this.ctx.fill();
        // 鐵冠
        this.ctx.fillStyle='#4a4a4a'; this.ctx.fillRect(-30,-140,60,22); this.ctx.fillStyle='#ffd700'; this.ctx.fillRect(-25,-135,50,4); this.ctx.beginPath(); this.ctx.moveTo(0,-140); this.ctx.lineTo(-8,-158); this.ctx.lineTo(8,-158); this.ctx.closePath(); this.ctx.fill();
        // 右手鐵鞭
        this.ctx.save(); this.ctx.translate(55,-55); this.ctx.rotate(g.whipOn?Math.sin(g.whip*3)*0.5:-0.25);
        this.ctx.fillStyle='#4a4a4a'; this.ctx.fillRect(-4,-70,8,90);
        for(let i=0;i<7;i++){ this.ctx.fillStyle=i%2===0?'#ffd700':'#8b4513'; this.ctx.beginPath(); this.ctx.arc(0,-78-i*13,5,0,Math.PI*2); this.ctx.fill(); }
        if(g.whipOn){ this.ctx.strokeStyle='rgba(255,215,0,0.5)'; this.ctx.lineWidth=2; this.ctx.beginPath(); this.ctx.moveTo(0,-70); this.ctx.lineTo(0,-195); this.ctx.stroke(); }
        this.ctx.restore();
        // 左手元寶
        this.ctx.save(); this.ctx.translate(-55,-35); this.ctx.rotate(-0.2); this.drawYuanbao(); this.ctx.restore();
        if(g.roar){ this.ctx.translate(Math.sin(Date.now()*0.05)*4,Math.sin(Date.now()*0.05)*4); }
        this.ctx.restore();
    }
    drawTiger(x,y){
        this.ctx.save(); this.ctx.translate(x,y);
        this.ctx.fillStyle='#1a1a1a'; this.ctx.beginPath(); this.ctx.ellipse(0,0,80,45,0,0,Math.PI*2); this.ctx.fill();
        this.ctx.strokeStyle='#2c2c2c'; this.ctx.lineWidth=3; for(let i=-55;i<55;i+=25){ this.ctx.beginPath(); this.ctx.moveTo(i,-35); this.ctx.lineTo(i+8,35); this.ctx.stroke(); }
        this.ctx.fillStyle='#1a1a1a'; this.ctx.beginPath(); this.ctx.arc(-60,-18,30,0,Math.PI*2); this.ctx.fill();
        this.ctx.fillStyle=this.god.roar?'#ff4500':'#ffd700'; this.ctx.shadowColor=this.god.roar?'#ff4500':'#ffd700'; this.ctx.shadowBlur=this.god.roar?15:8;
        this.ctx.beginPath(); this.ctx.arc(-68,-22,5,0,Math.PI*2); this.ctx.arc(-52,-22,5,0,Math.PI*2); this.ctx.fill(); this.ctx.shadowBlur=0;
        this.ctx.fillStyle='#ff0000'; this.ctx.beginPath(); this.ctx.moveTo(-78,-8); this.ctx.quadraticCurveTo(-60,8,-42,-8); this.ctx.fill();
        this.ctx.fillStyle='#fff'; this.ctx.beginPath(); this.ctx.moveTo(-73,-6); this.ctx.lineTo(-68,4); this.ctx.lineTo(-63,-6); this.ctx.fill();
        this.ctx.strokeStyle='#1a1a1a'; this.ctx.lineWidth=10; this.ctx.beginPath(); this.ctx.moveTo(70,0); this.ctx.quadraticCurveTo(115,-25,105,-50); this.ctx.stroke();
        if(this.god.roar){ this.ctx.fillStyle='rgba(255,100,0,0.5)'; this.ctx.beginPath(); this.ctx.arc(105,-50,12,0,Math.PI*2); this.ctx.fill(); }
        this.ctx.restore();
    }
    loop(){ if(!this.playing)return; this.update(); this.draw(); requestAnimationFrame(()=>this.loop()); }
}

window.onload=()=>new Game();
</script>

</body>
</html>
