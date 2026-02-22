<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Picko Pro</title>

<style>
:root{
--accent:#00ff88;
--bg:#000;
--nav-h:85px;
--vh:100vh;
}

*{
margin:0;
padding:0;
box-sizing:border-box;
-webkit-tap-highlight-color:transparent;
}

html,body{
height:var(--vh);
background:#000;
color:#fff;
font-family:-apple-system,BlinkMacSystemFont,"SF Pro Display",sans-serif;
overflow:hidden;
position:fixed;
width:100%;
}

.main{
height:var(--vh);
display:flex;
flex-direction:column;
}

.content{
flex:1;
display:flex;
justify-content:center;
align-items:center;
position:relative;
}

.tab{
display:none;
position:absolute;
inset:0;
justify-content:center;
align-items:center;
}

.tab.active{
display:flex;
}

/* NAV */
.nav{
position:fixed;
bottom:25px;
left:50%;
transform:translateX(-50%);
width:92%;
max-width:430px;
height:var(--nav-h);
background:rgba(20,20,20,.95);
border-radius:40px;
display:flex;
justify-content:space-around;
align-items:center;
backdrop-filter:blur(20px);
border:1px solid rgba(255,255,255,.08);
}

.nav-item{
flex:1;
text-align:center;
color:#444;
transition:.4s;
font-size:13px;
}

.nav-item.active{
color:var(--accent);
transform:translateY(-8px);
}

/* FINGER */
#surface{
position:absolute;
inset:0;
}

.ring{
position:fixed;
width:120px;
height:120px;
border-radius:50%;
border:4px solid var(--accent);
transform:translate(-50%,-50%);
pointer-events:none;
animation:spin 2s linear infinite;
box-shadow:0 0 20px var(--accent);
}

@keyframes spin{
from{transform:translate(-50%,-50%) rotate(0)}
to{transform:translate(-50%,-50%) rotate(360deg)}
}

/* DICE */
.dice-box{
width:120px;
height:120px;
perspective:600px;
}

.dice{
width:100%;
height:100%;
position:relative;
transform-style:preserve-3d;
transition:transform .7s cubic-bezier(.17,.84,.44,1);
}

.face{
position:absolute;
width:120px;
height:120px;
display:flex;
justify-content:center;
align-items:center;
font-size:55px;
font-weight:900;
border:4px solid var(--accent);
border-radius:22%;
background:#111;
backface-visibility:hidden;
}

.f1{transform:rotateY(0) translateZ(60px);}
.f2{transform:rotateX(90deg) translateZ(60px);}
.f3{transform:rotateY(90deg) translateZ(60px);}
.f4{transform:rotateY(-90deg) translateZ(60px);}
.f5{transform:rotateX(-90deg) translateZ(60px);}
.f6{transform:rotateY(180deg) translateZ(60px);}

.rolling{
animation:roll .6s linear infinite;
}

@keyframes roll{
to{transform:rotateX(360deg) rotateY(360deg);}
}

/* POINTER */
.pointer{
width:55px;
height:230px;
background:var(--accent);
clip-path:polygon(50% 0%,100% 100%,50% 85%,0% 100%);
transition:transform 3.5s cubic-bezier(.1,0,.1,1);
filter:drop-shadow(0 0 25px var(--accent));
}
</style>
</head>

<body>
<div class="main">
<div class="content">

<!-- FINGER -->
<div id="finger" class="tab active">
<div id="surface"></div>
</div>

<!-- DICE -->
<div id="dice" class="tab">
<div class="dice-box" onclick="rollDice()">
<div class="dice" id="diceEl">
<div class="face f1">1</div>
<div class="face f2">2</div>
<div class="face f3">3</div>
<div class="face f4">4</div>
<div class="face f5">5</div>
<div class="face f6">6</div>
</div>
</div>
</div>

<!-- POINTER -->
<div id="pointer" class="tab">
<div class="pointer" id="arrow" onclick="spin()"></div>
</div>

</div>

<div class="nav">
<div class="nav-item active" onclick="tab('finger',this)">☝️</div>
<div class="nav-item" onclick="tab('dice',this)">🎲</div>
<div class="nav-item" onclick="tab('pointer',this)">📍</div>
</div>

</div>

<script>
/* FIX MOBILE HEIGHT */
function fixVH(){
document.documentElement.style.setProperty('--vh',window.innerHeight+'px')
}
window.addEventListener('resize',fixVH)
fixVH()

/* TAB SWITCH */
function tab(id,el){
document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'))
document.querySelectorAll('.nav-item').forEach(n=>n.classList.remove('active'))
document.getElementById(id).classList.add('active')
el.classList.add('active')
}

/* FINGER */
const surface=document.getElementById('surface')
let fingers=new Map()
let deciding=false

surface.addEventListener('touchstart',e=>{
if(deciding)return
for(let t of e.touches){
if(!fingers.has(t.identifier)){
let r=document.createElement('div')
r.className='ring'
r.style.left=t.clientX+'px'
r.style.top=t.clientY+'px'
document.body.appendChild(r)
fingers.set(t.identifier,r)
}
}
if(fingers.size>=2){
setTimeout(selectWinner,3000)
}
})

surface.addEventListener('touchend',e=>{
let activeIds=[...e.touches].map(t=>t.identifier)
fingers.forEach((el,id)=>{
if(!activeIds.includes(id)){
el.remove()
fingers.delete(id)
}
})
})

function selectWinner(){
if(fingers.size<2||deciding)return
deciding=true
let ids=[...fingers.keys()]
let winner=ids[Math.floor(Math.random()*ids.length)]
fingers.forEach((el,id)=>{
if(id!==winner)el.style.opacity=0
})
navigator.vibrate?.([100,50,200])
setTimeout(()=>{
fingers.forEach(el=>el.remove())
fingers.clear()
deciding=false
},2000)
}

/* DICE */
function rollDice(){
let d=document.getElementById('diceEl')
d.classList.add('rolling')
setTimeout(()=>{
d.classList.remove('rolling')
const rotations=[
'rotateY(0deg)',
'rotateX(-90deg)',
'rotateY(90deg)',
'rotateY(-90deg)',
'rotateX(90deg)',
'rotateY(180deg)'
]
d.style.transform=rotations[Math.floor(Math.random()*6)]
navigator.vibrate?.(60)
},600)
}

/* POINTER */
let rot=0
function spin(){
if(deciding)return
deciding=true
rot+=Math.floor(Math.random()*360)+1440
document.getElementById('arrow').style.transform=`rotate(${rot}deg)`
setTimeout(()=>deciding=false,3500)
}
</script>
</body>
</html>
