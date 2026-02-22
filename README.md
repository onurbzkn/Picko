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
            --finger-bg: #000000; --dice-bg: #000000; --pointer-bg: #000000;
        }

        html, body {
            margin: 0; padding: 0; width: 100%; height: var(--vh);
            background-color: #000; color: white; overflow: hidden;
            font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", sans-serif;
            user-select: none; -webkit-user-select: none;
            touch-action: none; position: fixed;
        }

        .main-wrapper { position: relative; width: 100%; height: var(--vh); display: flex; flex-direction: column; }
        .content-area { flex: 1; position: relative; width: 100%; height: calc(var(--vh) - var(--nav-height)); }

        .tab-content { display: none; width: 100%; height: 100%; position: absolute; top: 0; left: 0; flex-direction: column; justify-content: center; align-items: center; }
        .tab-content.active { display: flex; animation: fadeIn 0.3s ease; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

        /* ARKA PLANLAR */
        #finger { background-color: var(--finger-bg); }
        #dice { background-color: var(--dice-bg); }
        #pointer { background-color: var(--pointer-bg); }

        /* FINGER */
        #finger-surface { width: 100%; height: 100%; position: absolute; z-index: 10; }
        .finger-circle { position: fixed; width: 120px; height: 120px; border-radius: 50%; border: 4px solid; transform: translate(-50%, -50%); pointer-events: none; animation: ringRotate 2s linear infinite; }
        @keyframes ringRotate { from { transform: translate(-50%, -50%) rotate(0deg); } to { transform: translate(-50%, -50%) rotate(360deg); } }
        .winner { transform: translate(-50%, -50%) scale(1.6) !important; box-shadow: 0 0 60px currentColor; z-index: 100; }

        /* DICE & POINTER TAM ORTALAMA */
        #dice-container { 
            display: flex; gap: 40px; flex-wrap: wrap; 
            justify-content: center; align-items: center; 
            width: 100%; height: 100%; 
            padding-bottom: 20px; /* Görsel denge */
        }
        .pointer-wrapper { 
            display: flex; justify-content: center; align-items: center; 
            width: 100%; height: 100%; 
            padding-bottom: 20px;
        }
        .scene { width: var(--dice-size); height: var(--dice-size); perspective: 600px; flex-shrink: 0; }
        .dice { width: 100%; height: 100%; position: relative; transform-style: preserve-3d; transition: transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .dice-face { position: absolute; width: 100%; height: 100%; background: #151515; border: 3px solid var(--dice-accent); display: flex; justify-content: center; align-items: center; font-size: calc(var(--dice-size) * 0.45); font-weight: 900; color: var(--dice-accent); border-radius: 20%; backface-visibility: hidden; }
        .f1 { transform: rotateY(0deg) translateZ(calc(var(--dice-size)/2)); }
        .f6 { transform: rotateY(180deg) translateZ(calc(var(--dice-size)/2)); }
        .f2 { transform: rotateX(90deg) translateZ(calc(var(--dice-size)/2)); }
        .f5 { transform: rotateX(-90deg) translateZ(calc(var(--dice-size)/2)); }
        .f3 { transform: rotateY(90deg) translateZ(calc(var(--dice-size)/2)); }
        .f4 { transform: rotateY(-90deg) translateZ(calc(var(--dice-size)/2)); }
        
        .rolling { animation: crazyRoll 0.5s linear infinite; }
        @keyframes crazyRoll { 0% { transform: rotateX(0deg) rotateY(0deg) rotateZ(0deg); } 100% { transform: rotateX(360deg) rotateY(360deg) rotateZ(360deg); } }

        /* POINTER */
        #pointer-arrow { width: 45px; height: 200px; background: var(--pointer-accent); clip-path: polygon(50% 0%, 100% 100%, 50% 85%, 0% 100%); filter: drop-shadow(0 0 25px var(--pointer-accent)); transition: transform 3.5s cubic-bezier(0.1, 0, 0.1, 1); }

        /* EFEKTLİ NAV BAR */
        .nav-bar { 
            position: fixed; bottom: 30px; left: 50%; transform: translateX(-50%); 
            width: 92%; max-width: 440px; height: var(--nav-height); 
            background: rgba(18, 18, 18, 0.96); backdrop-filter: blur(30px); 
            border-radius: 40px; display: flex; justify-content: space-around; align-items: center; 
            z-index: 2000; border: 1px solid rgba(255, 255, 255, 0.08);
        }
        .nav-item { 
            display: flex; flex-direction: column; align-items: center; 
            color: #444; flex: 1; transition: all 0.5s cubic-bezier(0.23, 1, 0.32, 1); 
            position: relative; cursor: pointer;
        }
        .nav-item i { font-style: normal; font-size: 28px; transition: all 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .nav-item span { font-size: 10px; font-weight: 800; text-transform: uppercase; margin-top: 4px; opacity: 0.6; transition: 0.3s; }
        
        .nav-item.active { color: var(--accent-color); }
        .nav-item.active i { transform: translateY(-14px) scale(1.35); filter: drop-shadow(0 0 12px var(--accent-color)); }
        .nav-item.active span { opacity: 1; transform: translateY(-4px); }

        /* SETTINGS */
        #settings { justify-content: flex-start; overflow-y: auto; padding-top: 60px; padding-bottom: 140px; background: #000; touch-action: pan-y; }
        .settings-view { width: 100%; display: none; flex-direction: column; align-items: center; }
        .settings-view.active { display: flex; }
        .settings-container { width: 90%; max-width: 450px; }
        .settings-group { background: var(--list-bg); border-radius: 16px; margin-bottom: 25px; overflow: hidden; width: 100%; }
        .settings-item { display: flex; justify-content: space-between; align-items: center; padding: 18px 16px; border-bottom: 1px solid var(--border-color); }
        .back-btn { align-self: flex-start; margin-left: 5%; margin-bottom: 20px; color: var(--accent-color); font-weight: 700; font-size: 19px; cursor: pointer; }
        .color-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 14px; width: 100%; justify-items: center; padding: 10px 0; }
        .color-dot { width: 42px; height: 42px; border-radius: 50%; border: 2.5px solid transparent; }
        .color-dot.active { border-color: white; transform: scale(1.1); box-shadow: 0 0 10px currentColor; }
        .grid-title { font-weight: 700; color: #8e8e93; font-size: 11px; text-transform: uppercase; margin-bottom: 5px; }

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
                            <div class="settings-item" onclick="showSubSettings('finger-settings')"><span>☝️ Finger Settings</span><span>›</span></div>
                            <div class="settings-item" onclick="showSubSettings('dice-settings')"><span>🎲 Dice Settings</span><span>›</span></div>
                            <div class="settings-item" onclick="showSubSettings('pointer-settings')"><span>📍 Pointer Settings</span><span>›</span></div>
                        </div>
                    </div>
                </div>
                <div id="finger-settings" class="settings-view"><div class="back-btn" onclick="hideSubSettings()">‹ Back</div><div class="settings-container" id="f-container"></div></div>
                <div id="dice-settings" class="settings-view"><div class="back-btn" onclick="hideSubSettings()">‹ Back</div><div class="settings-container" id="d-container"></div></div>
                <div id="pointer-settings" class="settings-view"><div class="back-btn" onclick="hideSubSettings()">‹ Back</div><div class="settings-container" id="p-container"></div></div>
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
        let isDeciding = false; let fingerGlowColor = '#00ff88'; let activeFingers = new Map(); let winnerCount = 1; let diceCount = 1;

        function showSubSettings(id) { 
            document.querySelectorAll('.settings-view').forEach(v => v.classList.remove('active'));
            document.getElementById(id).classList.add('active');
            renderSubSettings(id); 
        }
        function hideSubSettings() { document.querySelectorAll('.settings-view').forEach(v => v.classList.remove('active')); document.getElementById('settings-main').classList.add('active'); }

        function renderSubSettings(id) {
            const fCont = document.getElementById('f-container'); const dCont = document.getElementById('d-container'); const pCont = document.getElementById('p-container');
            if(id === 'finger-settings') {
                fCont.innerHTML = `<div class="settings-group"><div class="settings-item"><div style="width:100%"><div class="grid-title">Background</div><div class="color-grid">${createGrid(darkNeons, '--finger-bg')}</div></div></div><div class="settings-item"><div style="width:100%"><div class="grid-title">Glow Color</div><div class="color-grid">${createGrid(neons, 'glow')}</div></div></div></div>`;
            } else if(id === 'dice-settings') {
                dCont.innerHTML = `<div class="settings-group"><div class="settings-item"><div style="width:100%"><div class="grid-title">Background</div><div class="color-grid">${createGrid(darkNeons, '--dice-bg')}</div></div></div><div class="settings-item"><div style="width:100%"><div class="grid-title">Dice Color</div><div class="color-grid">${createGrid(neons, '--dice-accent', true)}</div></div></div></div>`;
            } else if(id === 'pointer-settings') {
                pCont.innerHTML = `<div class="settings-group"><div class="settings-item"><div style="width:100%"><div class="grid-title">Background</div><div class="color-grid">${createGrid(darkNeons, '--pointer-bg')}</div></div></div><div class="settings-item"><div style="width:100%"><div class="grid-title">Arrow Color</div><div class="color-grid">${createGrid(neons, '--pointer-accent')}</div></div></div></div>`;
            }
        }

        function createGrid(colors, varName, isDice = false) {
            return colors.map(c => `<div class="color-dot" style="background-color:${c}" onclick="updateColor('${varName}', '${c}', ${isDice}, this)"></div>`).join('');
        }

        function updateColor(varName, color, isDice, el) {
            if(varName === 'glow') fingerGlowColor = color;
            else document.documentElement.style.setProperty(varName, color);
            el.parentElement.querySelectorAll('.color-dot').forEach(d => d.classList.remove('active'));
            el.classList.add('active');
            if(isDice) renderDice();
        }

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
                    el.style.borderColor = fingerGlowColor; el.style.boxShadow = `0 0 15px ${fingerGlowColor}`;
                    document.body.appendChild(el); activeFingers.set(t.identifier, el);
                }
            }
            if (activeFingers.size >= 2) setTimeout(pickWinners, 3000);
        });
        surface.addEventListener('touchend', e => {
            const ids = new Set(Array.from(e.touches).map(t => t.identifier));
            activeFingers.forEach((el, id) => { if (!ids.has(id)) { el.remove(); activeFingers.delete(id); } });
        });

        function pickWinners() {
            if (activeFingers.size < 2 || isDeciding) return; isDeciding = true;
            const ids = Array.from(activeFingers.keys());
            const winners = ids.sort(() => 0.5 - Math.random()).slice(0, Math.min(winnerCount, ids.length - 1));
            activeFingers.forEach((el, id) => { winners.includes(id) ? el.classList.add('winner') : el.style.opacity = '0'; });
            if (navigator.vibrate) navigator.vibrate([100, 50, 200]);
            setTimeout(() => { activeFingers.forEach(el => el.remove()); activeFingers.clear(); isDeciding = false; }, 2000);
        }

        function renderDice() {
            const container = document.getElementById('dice-container'); container.innerHTML = '';
            for (let i = 0; i < diceCount; i++) container.innerHTML += `<div class="scene"><div class="dice"><div class="dice-face f1">1</div><div class="dice-face f6">6</div><div class="dice-face f2">2</div><div class="dice-face f5">5</div><div class="dice-face f3">3</div><div class="dice-face f4">4</div></div></div>`;
        }
        function setDiceCount(n, b) { diceCount = n; b.parentElement.querySelectorAll('.control-btn').forEach(x => x.classList.remove('active')); b.classList.add('active'); renderDice(); }
        function rollAllDice() {
            const dice = document.querySelectorAll('.dice');
            dice.forEach(d => { d.style.transition = 'none'; d.classList.add('rolling'); });
            setTimeout(() => {
                const r = ['rotateY(0deg)','rotateY(180deg)','rotateX(-90deg)','rotateX(90deg)','rotateY(-90deg)','rotateY(90deg)'];
                dice.forEach(d => { d.classList.remove('rolling'); d.style.transition = 'transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275)'; d.style.transform = r[Math.floor(Math.random()*6)]; });
                if(navigator.vibrate) navigator.vibrate(60);
            }, 600);
        }
        let curRot = 0;
        function spinPointer() { if (isDeciding) return; isDeciding = true; curRot += Math.floor(Math.random() * 360) + 1440; document.getElementById('pointer-arrow').style.transform = `rotate(${curRot}deg)`; setTimeout(() => isDeciding = false, 3500); }
        function setWinnerCount(n, btn) { winnerCount = n; btn.parentElement.querySelectorAll('.control-btn').forEach(b => b.classList.remove('active')); btn.classList.add('active'); }
        renderDice();
    </script>
</body>
</html>
