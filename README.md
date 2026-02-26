<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Picko Pro</title>
    <style>
        :root {
            --bg-color: #000000; 
            --accent-green: #00ff88; 
            --accent-orange: #ff8c00;
            --nav-height: 85px; 
            --list-bg: #1c1c1e;
            --dice-size: 95px;
            --vh: 100vh;
        }

        * { -webkit-tap-highlight-color: transparent !important; outline: none !important; box-sizing: border-box; }

        html, body {
            margin: 0; padding: 0; width: 100%; height: var(--vh);
            background-color: #000; 
            background: radial-gradient(circle at center, #111 0%, #000 100%);
            color: white; overflow: hidden;
            font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", sans-serif;
            user-select: none; -webkit-user-select: none;
            touch-action: none; position: fixed;
        }

        /* SABİT NEON BAŞLIK - TİTREME YOK */
        .app-header {
            position: absolute; top: 50px; left: 0; width: 100%;
            text-align: center; z-index: 3000; pointer-events: none;
        }

        .app-header h1 {
            font-size: 42px; font-weight: 900; margin: 0;
            color: var(--accent-green);
            text-transform: uppercase;
            text-shadow: 0 0 10px rgba(0, 255, 136, 0.8), 0 0 30px rgba(0, 255, 136, 0.4);
        }

        /* KONTROLLER - İÇERİĞİ İTMEYEN YAPI */
        .top-controls { 
            position: absolute; top: 130px; left: 50%; transform: translateX(-50%); 
            display: flex; gap: 5px; background: rgba(255,255,255,0.08); 
            padding: 5px; border-radius: 40px; z-index: 1100; backdrop-filter: blur(10px);
            border: 1px solid rgba(255,255,255,0.1);
        }
        .control-btn { 
            background: transparent; border: none; color: #888; 
            padding: 10px 20px; border-radius: 30px; font-weight: 800; font-size: 13px; 
            transition: 0.3s;
        }
        .control-btn.active { background: var(--accent-orange); color: black; }

        .main-wrapper { position: relative; width: 100%; height: var(--vh); display: flex; flex-direction: column; }
        
        /* TAM MERKEZLEME ALANI */
        .content-area { 
            flex: 1; position: relative; width: 100%; 
            display: flex; justify-content: center; align-items: center; 
        }

        .tab-content { 
            display: none; width: 100%; height: 100%; 
            flex-direction: column; justify-content: center; align-items: center; 
        }
        .tab-content.active { display: flex; }

        /* GÖRSELDEKİ TURUNCU NEON ZARLAR */
        #dice-container { display: flex; gap: 35px; justify-content: center; align-items: center; }
        .scene { width: var(--dice-size); height: var(--dice-size); perspective: 600px; }
        .dice { width: 100%; height: 100%; position: relative; transform-style: preserve-3d; transition: transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .dice-face { 
            position: absolute; width: 100%; height: 100%; background: #111; 
            border: 3.5px solid var(--accent-orange); 
            display: flex; justify-content: center; align-items: center; 
            font-size: 45px; font-weight: 900; color: var(--accent-green); border-radius: 18px; 
            backface-visibility: hidden; 
            box-shadow: 0 0 15px rgba(255, 140, 0, 0.3), inset 0 0 10px rgba(255, 140, 0, 0.2);
        }
        
        /* ZAR YÖNLERİ */
        .f1 { transform: rotateY(0deg) translateZ(calc(var(--dice-size)/2)); }
        .f6 { transform: rotateY(180deg) translateZ(calc(var(--dice-size)/2)); }
        .f2 { transform: rotateX(90deg) translateZ(calc(var(--dice-size)/2)); }
        .f5 { transform: rotateX(-90deg) translateZ(calc(var(--dice-size)/2)); }
        .f3 { transform: rotateY(90deg) translateZ(calc(var(--dice-size)/2)); }
        .f4 { transform: rotateY(-90deg) translateZ(calc(var(--dice-size)/2)); }
        
        .rolling { animation: crazyRoll 0.5s linear infinite; }
        @keyframes crazyRoll { 0% { transform: rotateX(0deg) rotateY(0deg); } 100% { transform: rotateX(360deg) rotateY(360deg); } }

        /* OK TASARIMI */
        .pointer-wrapper { display: flex; justify-content: center; align-items: center; }
        #pointer-arrow { 
            width: 55px; height: 230px; background: var(--accent-green); 
            clip-path: polygon(50% 0%, 100% 100%, 50% 85%, 0% 100%); 
            filter: drop-shadow(0 0 20px var(--accent-green)); 
            transition: transform 3.5s cubic-bezier(0.1, 0, 0.1, 1); 
        }

        /* PARMAK SEÇİCİ HALKALAR */
        #finger-surface { width: 100%; height: 100%; position: absolute; top: 0; left: 0; z-index: 10; }
        .finger-circle {
            position: fixed; width: 110px; height: 110px; border-radius: 50%; border: 4px solid var(--accent-green);
            transform: translate(-50%, -50%); pointer-events: none;
            box-shadow: 0 0 20px var(--accent-green);
        }
        .finger-circle.winner { animation: winnerPulse 0.5s ease infinite alternate; border-color: var(--accent-orange); box-shadow: 0 0 30px var(--accent-orange); }
        @keyframes winnerPulse { from { transform: translate(-50%, -50%) scale(1); } to { transform: translate(-50%, -50%) scale(1.3); } }

        /* NAV BAR - PREMIUM BLUR */
        .nav-bar { 
            position: fixed; bottom: 30px; left: 50%; transform: translateX(-50%); 
            width: 92%; max-width: 420px; height: var(--nav-height); 
            background: rgba(20, 20, 20, 0.85); backdrop-filter: blur(25px); 
            border-radius: 40px; display: flex; justify-content: space-around; align-items: center; 
            z-index: 2000; border: 1px solid rgba(255, 255, 255, 0.08);
        }
        .nav-item { display: flex; flex-direction: column; align-items: center; color: #555; flex: 1; transition: 0.3s; }
        .nav-item i { font-style: normal; font-size: 24px; margin-bottom: 4px; transition: 0.3s; }
        .nav-item span { font-size: 10px; font-weight: 700; text-transform: uppercase; }
        .nav-item.active { color: var(--accent-green); }
        .nav-item.active i { transform: translateY(-8px) scale(1.2); filter: drop-shadow(0 0 10px var(--accent-green)); }

        /* AYARLAR */
        #settings { padding-top: 150px; justify-content: flex-start; overflow-y: auto; }
        .settings-view { width: 90%; display: flex; flex-direction: column; }
        .settings-group { background: #111; border-radius: 20px; overflow: hidden; border: 1px solid #222; }
        .settings-item { padding: 20px; border-bottom: 1px solid #222; display: flex; justify-content: space-between; }
    </style>
</head>
<body>

    <div class="app-header">
        <h1>Picko Pro</h1>
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
                <div class="settings-view">
                    <h2 style="margin-bottom: 20px;">Ayarlar</h2>
                    <div class="settings-group">
                        <div class="settings-item"><span>Versiyon</span><span>Pro 2.0</span></div>
                        <div class="settings-item"><span>Hız</span><span>Ultra</span></div>
                    </div>
                </div>
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
        // Ekran Boyutu Sabitleme
        function fixVH() { document.documentElement.style.setProperty('--vh', `${window.innerHeight}px`); }
        window.addEventListener('resize', fixVH); fixVH();

        let isDeciding = false; 
        let activeFingers = new Map(); 
        let winnerCount = 1; 
        let diceCount = 1;

        function switchTab(tabId, el) {
            if (isDeciding) return;
            document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(i => i.classList.remove('active'));
            document.getElementById(tabId).classList.add('active'); el.classList.add('active');
            if(tabId === 'dice') renderDice();
        }

        // PARMAK SEÇİCİ MANTIĞI
        const surface = document.getElementById('finger-surface');
        surface.addEventListener('touchstart', e => {
            e.preventDefault(); if (isDeciding) return;
            for (let t of e.touches) {
                if (!activeFingers.has(t.identifier)) {
                    const el = document.createElement('div'); el.className = 'finger-circle';
                    el.style.left = t.clientX + 'px'; el.style.top = t.clientY + 'px';
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
                else el.style.opacity = '0.2';
            });
            if (navigator.vibrate) navigator.vibrate([100, 50, 150]);
            setTimeout(() => { 
                activeFingers.forEach(el => el.remove()); 
                activeFingers.clear(); isDeciding = false; 
            }, 3000);
        }

        // ZAR MANTIĞI
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
            if (navigator.vibrate) navigator.vibrate(50);
            setTimeout(() => {
                const r = ['rotateY(0deg)','rotateY(180deg)','rotateX(-90deg)','rotateX(90deg)','rotateY(-90deg)','rotateY(90deg)'];
                dice.forEach(d => { 
                    d.classList.remove('rolling'); 
                    d.style.transition = 'transform 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275)'; 
                    d.style.transform = r[Math.floor(Math.random()*6)]; 
                });
            }, 600);
        }

        // OK MANTIĞI
        let curRot = 0;
        function spinPointer() { 
            if (isDeciding) return; isDeciding = true; 
            curRot += Math.floor(Math.random() * 360) + 1800; 
            document.getElementById('pointer-arrow').style.transform = `rotate(${curRot}deg)`; 
            setTimeout(() => isDeciding = false, 3500); 
        }

        function setDiceCount(n, b) { diceCount = n; b.parentElement.querySelectorAll('.control-btn').forEach(x => x.classList.remove('active')); b.classList.add('active'); renderDice(); }
        function setWinnerCount(n, b) { winnerCount = n; b.parentElement.querySelectorAll('.control-btn').forEach(x => x.classList.remove('active')); b.classList.add('active'); }
        
        renderDice();
    </script>
</body>
</html>
