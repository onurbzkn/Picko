<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Picko Pro - Final</title>
    <style>
        :root {
            --bg-color: #0b101b;
            --accent: #00f2ff;
            --secondary: #7000ff;
            --text: #ffffff;
            --die-size: 80px;
            --nav-height: 75px;
            --nav-safe: env(safe-area-inset-bottom, 0px);
            --nav-total: calc(var(--nav-height) + var(--nav-safe));
        }

        body {
            margin: 0; padding: 0;
            background: var(--bg-color);
            color: var(--text);
            font-family: 'Segoe UI', sans-serif;
            overflow: hidden;
            min-height: 100vh;
            min-height: 100dvh;
            user-select: none;
            -webkit-user-select: none;
        }

        /* Navigasyon */
        .bottom-nav {
            position: fixed;
            left: 50%;
            transform: translateX(-50%);
            bottom: 0;
            width: min(100%, 720px);
            height: var(--nav-height);
            padding-bottom: var(--nav-safe);
            background: rgba(15, 20, 35, 0.95);
            backdrop-filter: blur(10px);
            display: flex; justify-content: space-around; align-items: center;
            border-top: 1px solid rgba(255,255,255,0.05);
            z-index: 1000;
            box-sizing: content-box;
        }
        .nav-item { color: #444; cursor: pointer; font-weight: bold; font-size: 11px; transition: 0.3s; text-transform: uppercase; text-align: center; }
        .nav-item.active { color: var(--accent); text-shadow: 0 0 15px var(--accent); }

        .tab-content { display: none; height: calc(100dvh - var(--nav-total)); width: 100%; position: relative; }
        .tab-content.active { display: flex; flex-direction: column; align-items: center; justify-content: center; }

        /* Finger Geri Sayım - Tam Orta */
        #countdown { 
            position: absolute; 
            top: 15%; 
            left: 0;
            right: 0;
            margin: 0 auto;
            font-size: 5rem; 
            font-weight: 900; 
            color: var(--accent); 
            text-shadow: 0 0 25px var(--accent); 
            z-index: 20; 
            text-align: center;
            pointer-events: none;
        }

        #finger-area { width: 100%; height: 100%; touch-action: none; position: relative; }
        .touch-circle {
            position: absolute; width: 90px; height: 90px;
            border: 5px solid var(--accent); border-radius: 50%;
            transform: translate(-50%, -50%); pointer-events: none;
            box-shadow: 0 0 25px var(--accent); z-index: 10;
        }
        .touch-circle.winner { border-color: #00ff88; box-shadow: 0 0 50px #00ff88; scale: 1.3; transition: all 0.3s ease; }
        .touch-circle.loser { border-color: #ff3366; opacity: 0.2; scale: 0.8; transition: all 0.3s ease; }

        /* 3D Dice */
        #dice { cursor: pointer; background: radial-gradient(circle, #1a2a4a 0%, #0b101b 100%); }
        .scene { width: var(--die-size); height: var(--die-size); perspective: 600px; }
        .cube {
            width: 100%; height: 100%; position: relative; transform-style: preserve-3d;
            transform: rotateX(-30deg) rotateY(45deg);
            transition: transform 1.2s cubic-bezier(0.15, 0.85, 0.35, 1.2);
        }
        .cube-face {
            position: absolute; width: var(--die-size); height: var(--die-size);
            background: white; border: 1px solid #ddd; display: flex;
            align-items: center; justify-content: center; font-size: 2rem; font-weight: 900; color: #111; border-radius: 12px;
        }
        .front { transform: rotateY(0deg) translateZ(40px); }
        .back { transform: rotateY(180deg) translateZ(40px); }
        .right { transform: rotateY(90deg) translateZ(40px); }
        .left { transform: rotateY(-90deg) translateZ(40px); }
        .top { transform: rotateX(90deg) translateZ(40px); }
        .bottom { transform: rotateX(-90deg) translateZ(40px); }

        /* Flow */
        #arrow {
            width: 0; height: 0; border-left: 35px solid transparent; border-right: 35px solid transparent;
            border-bottom: 170px solid var(--accent); filter: drop-shadow(0 0 20px var(--accent));
            transition: transform 4s cubic-bezier(0.15, 0, 0.15, 1); transform-origin: 50% 100%; margin-bottom: 170px;
        }

        /* Bilgi Metinleri - En Alt */
        .hint { 
            position: absolute; 
            bottom: 30px; 
            left: 0;
            right: 0;
            text-align: center;
            font-size: 10px; 
            color: rgba(255,255,255,0.4); 
            text-transform: uppercase; 
            letter-spacing: 2px; 
            pointer-events: none;
        }

        /* Ayarlar */
        .settings-list { width: 85%; max-width: 400px; }
        .setting-item {
            background: rgba(255,255,255,0.05); padding: 15px; border-radius: 15px;
            margin-bottom: 15px; display: flex; justify-content: space-between; align-items: center;
        }
        .color-dots { display: flex; gap: 10px; }
        .dot { width: 30px; height: 30px; border-radius: 50%; cursor: pointer; border: 2px solid transparent; }
        .dot.active { border-color: white; transform: scale(1.1); }
        .switch { position: relative; display: inline-block; width: 40px; height: 20px; }
        .switch input { opacity: 0; width: 0; height: 0; }
        .slider { position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0; background-color: #333; transition: .4s; border-radius: 20px; }
        .slider:before { position: absolute; content: ""; height: 14px; width: 14px; left: 3px; bottom: 3px; background-color: white; transition: .4s; border-radius: 50%; }
        input:checked + .slider { background-color: var(--accent); }
        input:checked + .slider:before { transform: translateX(20px); }
    </style>
</head>
<body>

    <div id="countdown"></div>

    <div class="content">
        <div id="finger" class="tab-content active">
            <div id="finger-area"></div>
            <div class="hint" data-tr="EN AZ 2 PARMAKLA DOKUN" data-en="TOUCH WITH AT LEAST 2 FINGERS">EN AZ 2 PARMAKLA DOKUN</div>
        </div>

        <div id="dice" class="tab-content" onclick="rollDice()">
            <div style="position: absolute; top: 20px; display: flex; gap: 10px;">
                <button onclick="event.stopPropagation(); setDice(1)" style="background:rgba(255,255,255,0.1); color:white; border:none; padding:10px 18px; border-radius:12px; font-weight:bold;">1 ZAR</button>
                <button onclick="event.stopPropagation(); setDice(2)" style="background:rgba(255,255,255,0.1); color:white; border:none; padding:10px 18px; border-radius:12px; font-weight:bold;">2 ZAR</button>
                <button onclick="event.stopPropagation(); setDice(3)" style="background:rgba(255,255,255,0.1); color:white; border:none; padding:10px 18px; border-radius:12px; font-weight:bold;">3 ZAR</button>
            </div>
            <div id="dice-display" style="display:flex; gap:40px;"></div>
            <div class="hint" data-tr="ATMAK İÇİN DOKUN" data-en="TOUCH TO ROLL">ATMAK İÇİN DOKUN</div>
        </div>

        <div id="flow" class="tab-content" onclick="spinArrow()">
            <div style="position:relative; display:flex; align-items:center; justify-content:center;">
                <div id="arrow"></div>
                <div style="position:absolute; width:30px; height:30px; background:#fff; border-radius:50%; box-shadow:0 0 20px #fff;"></div>
            </div>
            <div class="hint" data-tr="DÖNDÜRMEK İÇİN DOKUN" data-en="TOUCH TO SPIN">DÖNDÜRMEK İÇİN DOKUN</div>
        </div>

        <div id="settings" class="tab-content">
            <h2 style="margin-bottom:30px;" data-tr="AYARLAR" data-en="SETTINGS">AYARLAR</h2>
            <div class="settings-list">
                <div class="setting-item">
                    <span data-tr="Tema Rengi" data-en="Theme Color">Tema Rengi</span>
                    <div class="color-dots">
                        <div class="dot" style="background:#00f2ff;" onclick="setTheme('#00f2ff')"></div>
                        <div class="dot" style="background:#00ff88;" onclick="setTheme('#00ff88')"></div>
                        <div class="dot" style="background:#ff00ea;" onclick="setTheme('#ff00ea')"></div>
                        <div class="dot" style="background:#ff9500;" onclick="setTheme('#ff9500')"></div>
                    </div>
                </div>
                <div class="setting-item">
                    <span data-tr="Titreşim" data-en="Vibration">Titreşim</span>
                    <label class="switch"><input type="checkbox" id="vibrateToggle" checked><span class="slider"></span></label>
                </div>
                <div class="setting-item">
                    <span data-tr="Ses Efektleri" data-en="Sound Effects">Ses Efektleri</span>
                    <label class="switch"><input type="checkbox" id="soundToggle" checked><span class="slider"></span></label>
                </div>
                <div class="setting-item">
                    <span data-tr="Dil / Language" data-en="Language / Dil">Dil</span>
                    <button onclick="toggleLang()" id="langBtn" style="background:none; border:1px solid #555; color:white; padding:5px 10px; border-radius:8px;">TR</button>
                </div>
            </div>
            <div style="margin-top:20px; font-size:12px; opacity:0.3;">Picko v4.1 - Hocama Özel</div>
        </div>
    </div>

    <nav class="bottom-nav">
        <div class="nav-item active" onclick="showTab('finger', this)">Finger</div>
        <div class="nav-item" onclick="showTab('dice', this)">Dice</div>
        <div class="nav-item" onclick="showTab('flow', this)">Flow</div>
        <div class="nav-item" onclick="showTab('settings', this)">Settings</div>
    </nav>

    <script>
        let currentLang = 'tr';
        const config = { accent: '#00f2ff', vibrate: true, sound: true };

        function showTab(id, el) {
            document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            document.getElementById(id).classList.add('active');
            el.classList.add('active');
            if(id === 'dice') setDice(1);
        }

        function setTheme(color) {
            document.documentElement.style.setProperty('--accent', color);
            config.accent = color;
        }

        function toggleLang() {
            currentLang = currentLang === 'tr' ? 'en' : 'tr';
            document.getElementById('langBtn').innerText = currentLang.toUpperCase();
            document.querySelectorAll('[data-tr]').forEach(el => {
                el.innerText = el.getAttribute('data-' + currentLang);
            });
        }

        function playSound(type) {
            if(!document.getElementById('soundToggle').checked) return;
            const ctx = new (window.AudioContext || window.webkitAudioContext)();
            const osc = ctx.createOscillator();
            const gain = ctx.createGain();
            osc.connect(gain); gain.connect(ctx.destination);
            if(type === 'roll') {
                osc.type = 'square'; osc.frequency.setValueAtTime(150, ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.1);
                osc.start(); osc.stop(ctx.currentTime + 0.1);
            } else if(type === 'win') {
                osc.type = 'sine'; osc.frequency.setValueAtTime(440, ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(880, ctx.currentTime + 0.2);
                gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.3);
                osc.start(); osc.stop(ctx.currentTime + 0.3);
            }
        }

        /* Finger Logic */
        const fingerArea = document.getElementById('finger-area');
        const countdownEl = document.getElementById('countdown');
        let touches = {}, timer = null, isFinished = false;

        fingerArea.addEventListener('touchstart', e => {
            if(isFinished) resetFinger();
            Array.from(e.changedTouches).forEach(t => {
                const dot = document.createElement('div');
                dot.className = 'touch-circle'; dot.id = 't-' + t.identifier;
                dot.style.left = t.pageX + 'px'; dot.style.top = t.pageY + 'px';
                fingerArea.appendChild(dot);
                touches[t.identifier] = dot;
            });
            checkTimer();
        });

        fingerArea.addEventListener('touchmove', e => {
            e.preventDefault();
            Array.from(e.changedTouches).forEach(t => {
                const dot = document.getElementById('t-' + t.identifier);
                if(dot) { dot.style.left = t.pageX + 'px'; dot.style.top = t.pageY + 'px'; }
            });
        }, { passive: false });

        fingerArea.addEventListener('touchend', e => {
            Array.from(e.changedTouches).forEach(t => {
                const dot = document.getElementById('t-' + t.identifier);
                if(dot) dot.remove();
                delete touches[t.identifier];
            });
            checkTimer();
        });

        function checkTimer() {
            const count = Object.keys(touches).length;
            if (count > 1) { if (!timer) startCountdown(); } else { stopCountdown(); }
        }

        function startCountdown() {
            let timeLeft = 3; countdownEl.innerText = timeLeft;
            timer = setInterval(() => {
                timeLeft--; countdownEl.innerText = timeLeft || "";
                if (timeLeft <= 0) { clearInterval(timer); selectWinner(); }
            }, 1000);
        }

        function stopCountdown() { clearInterval(timer); timer = null; countdownEl.innerText = ""; }

        function selectWinner() {
            const keys = Object.keys(touches);
            if(keys.length < 2) return;
            const winnerKey = keys[Math.floor(Math.random() * keys.length)];
            keys.forEach(k => touches[k].classList.add(k === winnerKey ? 'winner' : 'loser'));
            isFinished = true; countdownEl.innerText = "!";
            playSound('win');
            if(document.getElementById('vibrateToggle').checked && window.navigator.vibrate) window.navigator.vibrate([100, 50, 100]);
        }

        function resetFinger() { fingerArea.innerHTML = ""; touches = {}; isFinished = false; stopCountdown(); }

        /* 3D Dice Logic */
        function setDice(n) {
            const container = document.getElementById('dice-display');
            container.innerHTML = "";
            for(let i=0; i<n; i++) {
                const scene = document.createElement('div');
                scene.className = 'scene';
                scene.innerHTML = `<div class="cube" id="cube-${i}">
                    <div class="cube-face front">1</div><div class="cube-face back">6</div>
                    <div class="cube-face right">3</div><div class="cube-face left">4</div>
                    <div class="cube-face top">2</div><div class="cube-face bottom">5</div>
                </div>`;
                container.appendChild(scene);
            }
        }

        function rollDice() {
            const cubes = document.querySelectorAll('.cube');
            playSound('roll');
            if(document.getElementById('vibrateToggle').checked && window.navigator.vibrate) window.navigator.vibrate(50);
            cubes.forEach(cube => {
                const result = Math.floor(Math.random() * 6) + 1;
                const xExtra = (Math.floor(Math.random() * 4) + 4) * 360;
                const yExtra = (Math.floor(Math.random() * 4) + 4) * 360;
                let x = 0, y = 0;
                switch(result) {
                    case 1: x = 0; y = 0; break;
                    case 2: x = -90; y = 0; break;
                    case 3: x = 0; y = -90; break;
                    case 4: x = 0; y = 90; break;
                    case 5: x = 90; y = 0; break;
                    case 6: x = 0; y = 180; break;
                }
                cube.style.transform = `rotateX(${x + xExtra}deg) rotateY(${y + yExtra}deg)`;
            });
        }

        /* Flow Logic */
        let currentRotation = 0;
        function spinArrow() {
            const arrow = document.getElementById('arrow');
            currentRotation += 2000 + Math.floor(Math.random() * 2000);
            arrow.style.transform = `rotate(${currentRotation}deg)`;
            playSound('roll');
            if(document.getElementById('vibrateToggle').checked && window.navigator.vibrate) window.navigator.vibrate(20);
        }

        setDice(1);
    </script>
</body>
</html>
