<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Picko Pro</title>
    
    <link rel="manifest" href="manifest.json">

    <style>
        :root {
            --bg-color: #000000; --accent-color: #00ff88; 
            --nav-height: 85px; --list-bg: #1c1c1e; --border-color: #38383a;
            --dice-size: 90px; --dice-accent: #00ff88; --pointer-accent: #00ff88;
            --vh: 100vh;
        }

        html, body {
            margin: 0; padding: 0; width: 100%; height: var(--vh);
            background-color: #000; color: white; overflow: hidden;
            font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", sans-serif;
            user-select: none; -webkit-user-select: none;
            touch-action: none; -webkit-tap-highlight-color: transparent;
            position: fixed; top: 0; left: 0;
        }

        .main-wrapper { 
            position: relative; width: 100%; height: var(--vh); 
            display: flex; flex-direction: column; 
        }

        .content-area { 
            flex: 1; position: relative; width: 100%; 
            height: calc(var(--vh) - var(--nav-height)); 
        }

        .tab-content {
            display: none; width: 100%; height: 100%;
            position: absolute; top: 0; left: 0;
            flex-direction: column; justify-content: center; align-items: center;
        }
        .tab-content.active { display: flex; }

        /* --- FINGER --- */
        #finger-surface { width: 100%; height: 100%; position: absolute; z-index: 10; }
        .finger-circle { 
            position: fixed; width: 120px; height: 120px; 
            border-radius: 50%; border: 4px solid; 
            transform: translate(-50%, -50%); pointer-events: none;
            animation: ringRotate 2s linear infinite, ringPulse 1.5s ease-in-out infinite;
        }
        @keyframes ringRotate { from { transform: translate(-50%, -50%) rotate(0deg); } to { transform: translate(-50%, -50%) rotate(360deg); } }
        @keyframes ringPulse { 0%, 100% { opacity: 0.8; transform: translate(-50%, -50%) scale(1); } 50% { opacity: 1; transform: translate(-50%, -50%) scale(1.1); } }
        .winner { animation: winner-pulse 0.6s infinite alternate !important; transform: translate(-50%, -50%) scale(1.6) !important; box-shadow: 0 0 60px currentColor; z-index: 100; }
        @keyframes winner-pulse { from { opacity: 1; } to { opacity: 0.6; } }

        /* --- DICE & POINTER (YUKARI KAYDIRILDI) --- */
        #dice-container {
            display: flex; gap: 30px; flex-wrap: wrap; 
            justify-content: center; align-items: center;
            width: 100%; height: 100%;
            margin-bottom: 40px; /* İçeriği biraz yukarı iter */
        }
        .pointer-wrapper {
            margin-bottom: 60px; /* Pointer'ı yukarı iter */
            display: flex; justify-content: center; align-items: center;
        }
        .scene { width: var(--dice-size); height: var(--dice-size); perspective: 600px; }
        .dice { width: 100%; height: 100%; position: relative; transform-style: preserve-3d; transition: transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .dice-face { position: absolute; width: 100%; height: 100%; background: #151515; border: 2.5px solid var(--dice-accent); display: flex; justify-content: center; align-items: center; font-size: calc(var(--dice-size) * 0.45); font-weight: 900; color: var(--dice-accent); border-radius: 20%; backface-visibility: hidden; }
        .f1 { transform: rotateY(0deg) translateZ(calc(var(--dice-size)/2)); }
        .f6 { transform: rotateY(180deg) translateZ(calc(var(--dice-size)/2)); }
        .f2 { transform: rotateX(90deg) translateZ(calc(var(--dice-size)/2)); }
        .f5 { transform: rotateX(-90deg) translateZ(calc(var(--dice-size)/2)); }
        .f3 { transform: rotateY(90deg) translateZ(calc(var(--dice-size)/2)); }
        .f4 { transform: rotateY(-90deg) translateZ(calc(var(--dice-size)/2)); }
        .rolling { animation: superRoll 0.5s linear infinite; }
        @keyframes superRoll { 0% { transform: rotateX(0deg) rotateY(0deg); } 100% { transform: rotateX(360deg) rotateY(360deg); } }

        /* --- SETTINGS: İSTEDİĞİN GİBİ DÜZELTİLDİ --- */
        #settings { justify-content: flex-start; overflow-y: auto; padding-top: 60px; padding-bottom: 140px; background: #000; touch-action: pan-y; -webkit-overflow-scrolling: touch; }
        .settings-container { width: 90%; max-width: 450px; margin: 0 auto; }
        .settings-group { background: var(--list-bg); border-radius: 16px; margin-bottom: 25px; overflow: hidden; width: 100%; }
        .settings-item { display: flex; justify-content: space-between; align-items: center; padding: 16px; border-bottom: 1px solid var(--border-color); cursor: pointer; }
        .settings-item:last-child { border-bottom: none; }
        .item-label { display: flex; align-items: center; gap: 14px; font-size: 17px; }
        .item-icon { width: 34px; height: 34px; border-radius: 10px; display: flex; justify-content: center; align-items: center; font-size: 18px; }
        .chevron { color: #8e8e93; font-size: 18px; font-weight: bold; }
        .back-btn { align-self: flex-start; margin-left: 5%; margin-bottom: 20px; color: var(--accent-color); font-weight: 700; font-size: 19px; cursor: pointer; padding: 10px; }

        .color-grid-container { width: 100%; display: flex; flex-direction: column; align-items: center; }
        .grid-title { font-weight: 700; color: #8e8e93; font-size: 12px; text-transform: uppercase; margin-bottom: 15px; align-self: flex-start; }
        .color-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 14px; width: 100%; justify-items: center; }
        .color-dot { width: 44px; height: 44px; border-radius: 50%; border: 2.5px solid transparent; }
        .color-dot.active { border-color: white; transform: scale(1.1); box-shadow: 0 0 12px currentColor; }

        .size-grid { display: flex; gap: 10px; width: 100%; }
        .size-btn { flex: 1; padding: 14px; background: #2c2c2e; border: none; border-radius: 12px; color: white; font-weight: 700; }
        .size-btn.active { background: var(--accent-color); color: black; }

        /* --- NAV BAR --- */
        .nav-bar { position: fixed; bottom: 30px; left: 50%; transform: translateX(-50%); width: 92%; max-width: 440px; height: var(--nav-height); background: rgba(18, 18, 18, 0.95); backdrop-filter: blur(30px); border-radius: 40px; display: flex; justify-content: space-around; align-items: center; z-index: 2000; border: 1px solid rgba(255, 255, 255, 0.1); }
        .nav-item { display: flex; flex-direction: column; align-items: center; color: #444; transition: 0.4s cubic-bezier(0.23, 1, 0.32, 1); flex: 1; }
        .nav-item.active { color: var(--accent-color); }
        .nav-item i { font-style: normal; font-size: 28px; }
        .nav-item.active i { transform: translateY(-12px) scale(1.3); filter: drop-shadow(0 0 8px var(--accent-color)); }
        .nav-item span { font-size: 10px; font-weight: 700; text-transform: uppercase; margin-top: 2px; }

        #pointer-arrow { width: 45px; height: 200px; background: var(--pointer-accent); clip-path: polygon(50% 0%, 100% 100%, 50% 85%, 0% 100%); filter: drop-shadow(0 0 20px var(--pointer-accent)); transition: transform 3.5s cubic-bezier(0.1, 0, 0.1, 1); }
        .top-controls { position: absolute; top: 70px; left: 50%; transform: translateX(-50%); display: flex; gap: 10px; background: rgba(255,255,255,0.1); padding: 8px 16px; border-radius: 30px; z-index: 1100; }
        .control-btn { background: transparent; border: none; color: #666; padding: 10px 20px; border-radius: 20px; font-weight: 800; }
        .control-btn.active { background: rgba(255, 255, 255, 0.2); color: white; }
    </style>
</head>
<body>

    <div class="main-wrapper">
        <div class="content-area">
            <div id="finger" class="tab-content active"><div class="top-controls"><button class="control-btn active" onclick="setWinnerCount(1, this)">1</button><button class="control-btn" onclick="setWinnerCount(2, this)">2</button></div><div id="finger-surface"></div></div>
            <div id="dice" class="tab-content"><div class="top-controls"><button class="control-btn active" onclick="setDiceCount(1, this)">1</button><button class="control-btn" onclick="setDiceCount(2, this)">2</button></div><div id="dice-container" onclick="rollAllDice()"></div></div>
            <div id="pointer" class="tab-content"><div class="pointer-wrapper" onclick="spinPointer()"><div id="pointer-arrow"></div></div></div>
            <div id="settings" class="tab-content">
                <div id="settings-main" class="settings-view active">
                    <h1 style="align-self: flex-start; margin-left: 5%; font-size: 36px; font-weight: 800; margin-bottom: 25px;">Settings</h1>
                    <div class="settings-container">
                        <div class="settings-group">
                            <div class="settings-item" onclick="showSubSettings('finger-settings')"><div class="item-label"><div class="item-icon" style="background:#34c759;">☝️</div><span>Finger Settings</span></div><span class="chevron">›</span></div>
                            <div class="settings-item" onclick="showSubSettings('dice-settings')"><div class="item-label"><div class="item-icon" style="background:#5856d6;">🎲</div><span>Dice Settings</span></div><span class="chevron">›</span></div>
                            <div class="settings-item" onclick="showSubSettings('pointer-settings')"><div class="item-label"><div class="item-icon" style="background:#ff9500;">📍</div><span>Pointer Settings</span></div><span class="chevron">›</span></div>
                        </div>
                    </div>
                </div>
                <div id="finger-settings" class="settings-view"><div class="back-btn" onclick="hideSubSettings()">‹ Back</div><div class="settings-container"><div class="settings-group"><div class="settings-item"><div class="color-grid-container"><div class="grid-title">Background</div><div class="color-grid" id="f-bg-grid"></div></div></div><div class="settings-item"><div class="color-grid-container"><div class="grid-title">Glow Color</div><div class="color-grid" id="f-gl-grid"></div></div></div></div></div></div>
                <div id="dice-settings" class="settings-view"><div class="back-btn" onclick="hideSubSettings()">‹ Back</div><div class="settings-container"><div class="settings-group"><div class="settings-item"><div class="color-grid-container"><div class="grid-title">Size</div><div class="size-grid"><button class="size-btn" onclick="setDiceSize(65, this)">Small</button><button class="size-btn active" onclick="setDiceSize(90, this)">Medium</button><button class="size-btn" onclick="setDiceSize(120, this)">Large</button></div></div></div><div class="settings-item"><div class="color-grid-container"><div class="grid-title">Background</div><div class="color-grid" id="d-bg-grid"></div></div></div><div class="settings-item"><div class="color-grid-container"><div class="grid-title">Dice Color</div><div class="color-grid" id="d-ac-grid"></div></div></div></div></div></div>
                <div id="pointer-settings" class="settings-view"><div class="back-btn" onclick="hideSubSettings()">‹ Back</div><div class="settings-container"><div class="settings-group"><div class="settings-item"><div class="color-grid-container"><div class="grid-title">Background</div><div class="color-grid" id="p-bg-grid"></div></div></div><div class="settings-item"><div class="color-grid-container"><div class="grid-title">Arrow Color</div><div class="color-grid" id="p-ac-grid"></div></div></div></div></div></div>
            </div>
        </div>
        <nav class="nav-bar">
            <div class="nav-item active" onclick="switchTab('finger', this)"><i>☝️</i><span>Finger</span></div>
            <div class="nav-item" onclick="switchTab('dice', this)"><i>🎲</i><span>Dice</span></div>
            <div class="nav-item" onclick="switchTab('pointer', this)"><i>📍</i><span>Pointer</span></div>
            <div class="nav-item" onclick="switchTab('settings', this)"><i>⚙️</i><span>Settings</span></div>
        </nav>
    </div>

    <script>
        function fixVH() { document.documentElement.style.setProperty('--vh', `${window.innerHeight}px`); }
        window.addEventListener('resize', fixVH); fixVH();

        const neons = ['#00FF88', '#FF00FF', '#00D0FF', '#FFCC00', '#FF4444', '#7FFF00', '#FF007F', '#00FFFF', '#FFFF00', '#FF8C00'];
        const darkNeons = ['#001a0d', '#1a001a', '#000d1a', '#1a1a00', '#1a0000', '#0d1a00', '#1a000d', '#001a1a', '#1a1a0d', '#1a0d00'];
        let isDeciding = false; let fingerGlowColor = '#00ff88'; let activeFingers = new Map(); let winnerCount = 1; let decideTimeout; let vibrateInterval;

        function showSubSettings(id) { document.querySelectorAll('.settings-view').forEach(v => v.classList.remove('active')); document.getElementById(id).classList.add('active'); }
        function hideSubSettings() { document.querySelectorAll('.settings-view').forEach(v => v.classList.remove('active')); document.getElementById('settings-main').classList.add('active'); }

        function initGrids() {
            darkNeons.forEach(c => createDot('f-bg-grid', c, '--finger-bg')); neons.forEach(c => createDot('f-gl-grid', c, 'glow'));
            darkNeons.forEach(c => createDot('d-bg-grid', c, '--dice-bg')); neons.forEach(c => createDot('d-ac-grid', c, '--dice-accent', true));
            darkNeons.forEach(c => createDot('p-bg-grid', c, '--pointer-bg')); neons.forEach(c => createDot('p-ac-grid', c, '--pointer-accent'));
        }

        function createDot(gridId, color, varName, isDice = false) {
            const d = document.createElement('div'); d.className = 'color-dot'; d.style.backgroundColor = color;
            d.onclick = () => {
                if(varName === 'glow') fingerGlowColor = color; else document.documentElement.style.setProperty(varName, color);
                if(isDice) renderDice(); d.parentElement.querySelectorAll('.color-dot').forEach(x => x.classList.remove('active')); d.classList.add('active');
            };
            document.getElementById(gridId).appendChild(d);
        }

        function setDiceSize(px, btn) { document.documentElement.style.setProperty('--dice-size', px + 'px'); btn.parentElement.querySelectorAll('.size-btn').forEach(b => b.classList.remove('active')); btn.classList.add('active'); renderDice(); }

        function switchTab(tabId, el) {
            if (isDeciding) return;
            document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(i => i.classList.remove('active'));
            document.getElementById(tabId).classList.add('active'); el.classList.add('active');
            if(tabId === 'dice') renderDice(); hideSubSettings();
        }

        const surface = document.getElementById('finger-surface');
        surface.addEventListener('touchstart', e => {
            e.preventDefault(); if (isDeciding) return;
            for (let t of e.touches) {
                if (!activeFingers.has(t.identifier) && activeFingers.size < 5) {
                    const el = document.createElement('div'); el.className = 'finger-circle';
                    el.style.left = t.clientX + 'px'; el.style.top = t.clientY + 'px';
                    el.style.borderColor = fingerGlowColor; el.style.boxShadow = `0 0 15px ${fingerGlowColor}`;
                    document.body.appendChild(el); activeFingers.set(t.identifier, el);
                }
            }
            if (activeFingers.size >= 2) {
                if (!vibrateInterval) vibrateInterval = setInterval(() => { if(navigator.vibrate) navigator.vibrate(40); }, 150);
                if (!decideTimeout) decideTimeout = setTimeout(pickWinners, 3000);
            }
        });

        surface.addEventListener('touchend', e => {
            const ids = new Set(Array.from(e.touches).map(t => t.identifier));
            activeFingers.forEach((el, id) => { if (!ids.has(id)) { el.remove(); activeFingers.delete(id); } });
            if (activeFingers.size < 2) { clearTimeout(decideTimeout); decideTimeout = null; clearInterval(vibrateInterval); vibrateInterval = null; }
        });

        function pickWinners() {
            if (activeFingers.size < 2) return; isDeciding = true; clearInterval(vibrateInterval); vibrateInterval = null;
            const ids = Array.from(activeFingers.keys());
            const winners = ids.sort(() => 0.5 - Math.random()).slice(0, Math.min(winnerCount, ids.length - 1));
            activeFingers.forEach((el, id) => { el.style.animation = 'none'; winners.includes(id) ? el.classList.add('winner') : el.style.opacity = '0'; });
            if (navigator.vibrate) navigator.vibrate([150, 50, 200]);
            setTimeout(() => { activeFingers.forEach(el => el.remove()); activeFingers.clear(); decideTimeout = null; isDeciding = false; }, 2000);
        }

        function renderDice() {
            const container = document.getElementById('dice-container'); container.innerHTML = '';
            const dCount = document.querySelector('#dice .control-btn.active').innerText;
            for (let i = 0; i < parseInt(dCount); i++) container.innerHTML += `<div class="scene"><div class="dice"><div class="dice-face f1">1</div><div class="dice-face f6">6</div><div class="dice-face f2">2</div><div class="dice-face f5">5</div><div class="dice-face f3">3</div><div class="dice-face f4">4</div></div></div>`;
        }
        function setDiceCount(n, b) { b.parentElement.querySelectorAll('.control-btn').forEach(x => x.classList.remove('active')); b.classList.add('active'); renderDice(); }
        function rollAllDice() {
            const dice = document.querySelectorAll('.dice'); dice.forEach(d => { d.style.transition = 'none'; d.classList.add('rolling'); });
            setTimeout(() => {
                const r = ['rotateY(0deg)','rotateY(180deg)','rotateX(-90deg)','rotateX(90deg)','rotateY(-90deg)','rotateY(90deg)'];
                dice.forEach(d => { d.classList.remove('rolling'); d.style.transition = 'transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275)'; d.style.transform = r[Math.floor(Math.random()*6)]; });
                if(navigator.vibrate) navigator.vibrate(60);
            }, 500);
        }

        let curRot = 0;
        function spinPointer() { if (isDeciding) return; isDeciding = true; curRot += Math.floor(Math.random() * 360) + 1440; document.getElementById('pointer-arrow').style.transform = `rotate(${curRot}deg)`; if(navigator.vibrate) navigator.vibrate(50); setTimeout(() => isDeciding = false, 3500); }
        function setWinnerCount(n, btn) { event.stopPropagation(); winnerCount = n; btn.parentElement.querySelectorAll('.control-btn').forEach(b => b.classList.remove('active')); btn.classList.add('active'); }
        
        initGrids(); renderDice();
    </script>
</body>
</html>
