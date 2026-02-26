<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Picko Pro</title>
    <style>
        :root {
            --bg-color: #000000; --accent-color: #00ff88; 
            --nav-height: 85px; --list-bg: #1c1c1e; --border-color: #38383a;
            --dice-size: 95px; --dice-accent: #00ff88; --pointer-accent: #00ff88;
            --vh: 100vh;
            --finger-bg: #000000; --dice-bg: #000000; --pointer-bg: #000000;
        }

        * { -webkit-tap-highlight-color: transparent !important; outline: none !important; box-sizing: border-box; }

        html, body {
            margin: 0; padding: 0; width: 100%; height: var(--vh);
            background-color: #000; color: white; overflow: hidden;
            font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", sans-serif;
            user-select: none; -webkit-user-select: none;
            touch-action: none; position: fixed;
        }

        /* BAŞLIK ARTIK YER KAPLAMIYOR - TAM NEON */
        .app-header {
            position: absolute; top: 0; left: 0; width: 100%;
            padding: 40px 25px; z-index: 3000; pointer-events: none;
        }

        .app-header h1 {
            font-size: 38px; font-weight: 900; margin: 0;
            color: #fff; text-transform: uppercase; letter-spacing: -1px;
            text-shadow: 0 0 10px #00ffff, 0 0 20px #00ffff, 0 0 40px #00ffff;
        }

        .main-wrapper { position: relative; width: 100%; height: var(--vh); display: flex; flex-direction: column; }
        
        /* CONTENT AREA: EKRANIN TAMAMINI KAPLAR VE MERKEZLER */
        .content-area { 
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            display: flex; justify-content: center; align-items: center; z-index: 1;
        }

        .tab-content { 
            display: none; width: 100%; height: 100%; 
            flex-direction: column; justify-content: center; align-items: center; 
        }
        .tab-content.active { display: flex; animation: fadeIn 0.3s ease; }

        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

        /* ARKA PLANLAR */
        #finger { background-color: var(--finger-bg); }
        #dice { background-color: var(--dice-bg); }
        #pointer { background-color: var(--pointer-bg); }

        /* FINGER SURFACE */
        #finger-surface { width: 100%; height: 100%; position: absolute; z-index: 10; top: 0; left: 0; }
      
        .finger-circle {
            position: fixed; width: 120px; height: 120px; border-radius: 50%; border: 4px solid;
            transform: translate(-50%, -50%); pointer-events: none;
            animation: ringRotate 2s linear infinite; backdrop-filter: blur(4px);
        }
        .finger-circle.winner { animation: winnerPulse 0.8s ease infinite alternate; }

        @keyframes winnerPulse { from { transform: translate(-50%, -50%) scale(1); } to { transform: translate(-50%, -50%) scale(1.3); } }
        @keyframes ringRotate { from { transform: translate(-50%, -50%) rotate(0deg); } to { transform: translate(-50%, -50%) rotate(360deg); } }

        /* MERKEZİ DÜZEN */
        #dice-container { 
            display: flex; gap: 40px; flex-wrap: wrap; 
            justify-content: center; align-items: center; 
        }
        .pointer-wrapper { 
            display: flex; justify-content: center; align-items: center; 
        }

        /* ZAR TASARIMI - ÖZDEŞ VE NET */
        .scene { width: var(--dice-size); height: var(--dice-size); perspective: 600px; }
        .dice { width: 100%; height: 100%; position: relative; transform-style: preserve-3d; transition: transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .dice-face { 
            position: absolute; width: 100%; height: 100%; background: #111; border: 3px solid var(--dice-accent); 
            display: flex; justify-content: center; align-items: center; 
            font-size: 42px; font-weight: 900; color: var(--dice-accent); border-radius: 20%; 
            backface-visibility: hidden; box-shadow: inset 0 0 15px rgba(0,255,136,0.1);
        }
        .f1 { transform: rotateY(0deg) translateZ(calc(var(--dice-size)/2)); }
        .f6 { transform: rotateY(180deg) translateZ(calc(var(--dice-size)/2)); }
        .f2 { transform: rotateX(90deg) translateZ(calc(var(--dice-size)/2)); }
        .f5 { transform: rotateX(-90deg) translateZ(calc(var(--dice-size)/2)); }
        .f3 { transform: rotateY(90deg) translateZ(calc(var(--dice-size)/2)); }
        .f4 { transform: rotateY(-90deg) translateZ(calc(var(--dice-size)/2)); }
        
        .rolling { animation: crazyRoll 0.5s linear infinite; }
        @keyframes crazyRoll { 0% { transform: rotateX(0deg) rotateY(0deg) rotateZ(0deg); } 100% { transform: rotateX(360deg) rotateY(360deg) rotateZ(360deg); } }

        /* OK TASARIMI */
        #pointer-arrow { 
            width: 50px; height: 220px; background: var(--pointer-accent); 
            clip-path: polygon(50% 0%, 100% 100%, 50% 85%, 0% 100%); 
            filter: drop-shadow(0 0 25px var(--pointer-accent)); 
            transition: transform 3.5s cubic-bezier(0.1, 0, 0.1, 1); 
        }

        /* KONTROLLER */
        .top-controls { 
            position: absolute; top: 120px; left: 50%; transform: translateX(-50%); 
            display: flex; gap: 10px; background: rgba(255,255,255,0.08); 
            padding: 6px 12px; border-radius: 30px; z-index: 1100; backdrop-filter: blur(10px);
        }
        .control-btn { background: transparent; border: none; color: #666; padding: 8px 16px; border-radius: 20px; font-weight: 800; font-size: 13px; }
        .control-btn.active { background: rgba(255, 255, 255, 0.15); color: white; }

        /* NAV BAR */
        .nav-bar { 
            position: fixed; bottom: 35px; left: 50%; transform: translateX(-50%); 
            width: 90%; max-width: 400px; height: var(--nav-height); 
            background: rgba(18, 18, 18, 0.9); backdrop-filter: blur(25px); 
            border-radius: 40px; display: flex; justify-content: space-around; align-items: center; 
            z-index: 2000; border: 1px solid rgba(255, 255, 255, 0.08);
        }
        .nav-item { display: flex; flex-direction: column; align-items: center; color: #444; flex: 1; transition: 0.4s ease; }
        .nav-item i { font-style: normal; font-size: 26px; transition: 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .nav-item span { font-size: 10px; font-weight: 700; text-transform: uppercase; margin-top: 4px; opacity: 0.5; }
        .nav-item.active { color: var(--accent-color); }
        .nav-item.active i { transform: translateY(-12px) scale(1.3); filter: drop-shadow(0 0 10px var(--accent-color)); }
        .nav-item.active span { opacity: 1; }

        /* SETTINGS DÜZENİ */
        #settings { justify-content: flex-start; overflow-y: auto; padding-top: 130px; padding-bottom: 140px; }
        .settings-view { width: 90%; display: none; flex-direction: column; }
        .settings-view.active { display: flex; }
        .settings-group { background: var(--list-bg); border-radius: 20px; margin-bottom: 20px; overflow: hidden; }
        .settings-item { display: flex; justify-content: space-between; align-items: center; padding: 20px; border-bottom: 1px solid var(--border-color); }
        .back-btn { color: var(--accent-color); font-weight: 700; margin-bottom: 20px; }
        .color-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 12px; padding: 10px 0; }
        .color-dot { width: 35px; height: 35px; border-radius: 50%; border: 2px solid transparent; }
        .color-dot.active { border-color: white; transform: scale(1.1); }
    </style>
</head>
<body>
    <div class="app-header">
        <h1>Picko</h1>
    </div>

    <div class="main-wrapper">
        <div class="content-area">
            <div id="finger" class="tab-content active">
                <div class="top-controls">
                    <button class="control-btn active" onclick="setWinnerCount(1, this)">1 KİŞİ</button>
                    <button class="control-btn" onclick="setWinnerCount(2, this)">2 KİŞİ</button>
                </div>
                <div id="finger-surface"></div>
            </div>
            
            <div id="dice" class="tab-content">
                <div class="top-controls">
                    <button class="control-btn active" onclick="setDiceCount(1, this)">1 ZAR</button>
                    <button class="control-btn" onclick="setDiceCount(2, this)">2 ZAR</button>
                </div>
                <div id="dice-container" onclick="rollAllDice()"></div>
            </div>
            
            <div id="pointer" class="tab-content">
                <div class="pointer-wrapper" onclick="spinPointer()">
                    <div id="pointer-arrow"></div>
                </div>
            </div>

            <div id="settings" class="tab-content">
                <div id="settings-main" class="settings-view active">
                    <h2 style="font-size: 32px; margin-bottom: 20px;">Ayarlar</h2>
                    <div class="settings-group">
                        <div class="settings-item" onclick="showSubSettings('finger-settings')"><span>☝️ Parmak Ayarları</span><span>›</span></div>
                        <div class="settings-item" onclick="showSubSettings('dice-settings')"><span>🎲 Zar Ayarları</span><span>›</span></div>
                        <div class="settings-item" onclick="showSubSettings('pointer-settings')"><span>📍 Ok Ayarları</span><span>›</span></div>
                    </div>
                </div>
                <div id="finger-settings" class="settings-view"><div class="back-btn" onclick="hideSubSettings()">‹ Geri</div><div id="f-container"></div></div>
                <div id="dice-settings" class="settings-view"><div class="back-btn" onclick="hideSubSettings()">‹ Geri</div><div id="d-container"></div></div>
                <div id="pointer-settings" class="settings-view"><div class="back-btn" onclick="hideSubSettings()">‹ Geri</div><div id="p-container"></div></div>
            </div>
        </div>

        <nav class="nav-bar">
            <div class="nav-item active" onclick="switchTab('finger', this)"><i>☝️</i><span>Seçici</span></div>
            <div class="nav-item" onclick="switchTab('dice', this)"><i>🎲</i><span>Zar</span></div>
            <div class="nav-item" onclick="switchTab('pointer', this)"><i>📍</i><span>Ok</span></div>
            <div class="nav-item" onclick="switchTab('settings', this)"><i>⚙️</i><span>Ayar</span></div>
        </nav>
    </div>

    <script>
        function fixVH() { document.documentElement.style.setProperty('--vh', `${window.innerHeight}px`); }
        window.addEventListener('resize', fixVH); fixVH();

        const neons = ['#00FF88', '#FF00FF', '#00D0FF', '#FFCC00', '#FF4444', '#7FFF00', '#FF007F', '#00FFFF', '#FFFF00', '#FF8C00'];
        const darkNeons = ['#000000', '#1a001a', '#000d1a', '#1a1a00', '#1a0000', '#0d1a00', '#1a000d', '#001a1a', '#1a1a0d', '#1a0d00'];
        let isDeciding = false; let fingerGlowColor = '#00ff88'; let activeFingers = new Map(); let winnerCount = 1; let diceCount = 1;

        function switchTab(tabId, el) {
            if (isDeciding) return;
            document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(i => i.classList.remove('active'));
            document.getElementById(tabId).classList.add('active'); el.classList.add('active');
            if(tabId === 'dice') renderDice();
        }

        const surface = document.getElementById('finger-surface');
        surface.addEventListener('touchstart', e => {
            e.preventDefault(); if (isDeciding) return;
            for (let t of e.touches) {
                if (!activeFingers.has(t.identifier) && activeFingers.size < 5) {
                    const el = document.createElement('div'); el.className = 'finger-circle';
                    el.style.left = t.clientX + 'px'; el.style.top = t.clientY + 'px';
                    el.style.borderColor = fingerGlowColor; el.style.boxShadow = `0 0 20px ${fingerGlowColor}`;
                    document.body.appendChild(el); activeFingers.set(t.identifier, el);
                }
            }
            if (activeFingers.size >= 2) {
                clearTimeout(window.pickTimer);
                window.pickTimer = setTimeout(pickWinners, 2500);
            }
        });

        surface.addEventListener('touchmove', e => {
            for (let t of e.touches) {
                if (activeFingers.has(t.identifier)) {
                    const el = activeFingers.get(t.identifier);
                    el.style.left = t.clientX + 'px'; el.style.top = t.clientY + 'px';
                }
            }
        });

        surface.addEventListener('touchend', e => {
            const ids = new Set(Array.from(e.touches).map(t => t.identifier));
            activeFingers.forEach((el, id) => { if (!ids.has(id)) { el.remove(); activeFingers.delete(id); } });
            if (activeFingers.size < 2) clearTimeout(window.pickTimer);
        });

        function pickWinners() {
            if (activeFingers.size < 2 || isDeciding) return; isDeciding = true;
            const ids = Array.from(activeFingers.keys());
            const winners = ids.sort(() => 0.5 - Math.random()).slice(0, Math.min(winnerCount, ids.length - 1));
            activeFingers.forEach((el, id) => { 
                if (winners.includes(id)) el.classList.add('winner');
                else el.style.opacity = '0';
            });
            if (navigator.vibrate) navigator.vibrate([100, 50, 100]);
            setTimeout(() => { 
                activeFingers.forEach(el => el.remove()); 
                activeFingers.clear(); isDeciding = false; 
            }, 2500);
        }

        function renderDice() {
            const container = document.getElementById('dice-container'); container.innerHTML = '';
            for (let i = 0; i < diceCount; i++) {
                container.innerHTML += `<div class="scene"><div class="dice">
                    <div class="dice-face f1">1</div><div class="dice-face f6">6</div>
                    <div class="dice-face f2">2</div><div class="dice-face f5">5</div>
                    <div class="dice-face f3">3</div><div class="dice-face f4">4</div>
                </div></div>`;
            }
        }

        function rollAllDice() {
            if (isDeciding) return;
            const dice = document.querySelectorAll('.dice');
            dice.forEach(d => { d.style.transition = 'none'; d.classList.add('rolling'); });
            if (navigator.vibrate) navigator.vibrate(40);
            setTimeout(() => {
                const r = ['rotateY(0deg)','rotateY(180deg)','rotateX(-90deg)','rotateX(90deg)','rotateY(-90deg)','rotateY(90deg)'];
                dice.forEach(d => { 
                    d.classList.remove('rolling'); 
                    d.style.transition = 'transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275)'; 
                    d.style.transform = r[Math.floor(Math.random()*6)]; 
                });
            }, 600);
        }

        let curRot = 0;
        function spinPointer() { 
            if (isDeciding) return; isDeciding = true; 
            curRot += Math.floor(Math.random() * 360) + 1800; 
            document.getElementById('pointer-arrow').style.transform = `rotate(${curRot}deg)`; 
            setTimeout(() => isDeciding = false, 3500); 
        }

        function setDiceCount(n, b) { diceCount = n; b.parentElement.querySelectorAll('.control-btn').forEach(x => x.classList.remove('active')); b.classList.add('active'); renderDice(); }
        function setWinnerCount(n, b) { winnerCount = n; b.parentElement.querySelectorAll('.control-btn').forEach(x => x.classList.remove('active')); b.classList.add('active'); }
        
        // Ayarlar Fonksiyonları
        function showSubSettings(id) { 
            document.querySelectorAll('.settings-view').forEach(v => v.classList.remove('active'));
            document.getElementById(id).classList.add('active');
            renderSubSettings(id); 
        }
        function hideSubSettings() { document.querySelectorAll('.settings-view').forEach(v => v.classList.remove('active')); document.getElementById('settings-main').classList.add('active'); }
        function renderSubSettings(id) {
            const conts = { 'finger-settings': 'f-container', 'dice-settings': 'd-container', 'pointer-settings': 'p-container' };
            const target = document.getElementById(conts[id]);
            if (!target) return;
            if (id === 'finger-settings') target.innerHTML = `<div class="settings-group"><div class="settings-item"><div style="width:100%">Glow Rengi<div class="color-grid">${neons.map(c => `<div class="color-dot" style="background:${c}" onclick="fingerGlowColor='${c}'"></div>`).join('')}</div></div></div></div>`;
            if (id === 'dice-settings') target.innerHTML = `<div class="settings-group"><div class="settings-item"><div style="width:100%">Zar Rengi<div class="color-grid">${neons.map(c => `<div class="color-dot" style="background:${c}" onclick="document.documentElement.style.setProperty('--dice-accent', '${c}'); renderDice()"></div>`).join('')}</div></div></div></div>`;
            if (id === 'pointer-settings') target.innerHTML = `<div class="settings-group"><div class="settings-item"><div style="width:100%">Ok Rengi<div class="color-grid">${neons.map(c => `<div class="color-dot" style="background:${c}" onclick="document.documentElement.style.setProperty('--pointer-accent', '${c}')"></div>`).join('')}</div></div></div></div>`;
        }

        renderDice();
    </script>
</body>
</html>
