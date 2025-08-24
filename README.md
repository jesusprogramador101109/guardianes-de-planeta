<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Guardianes del Planeta</title>
<style>
  :root{
    --green:#2e7d32;
    --green-light:#66bb6a;
    --blue:#0277bd;
    --gray:#f5f5f5;
    --dark:#333;
  }
  body{
    margin:0;
    font-family:system-ui,sans-serif;
    background:var(--gray);
    color:var(--dark);
    display:flex;
    flex-direction:column;
    min-height:100vh;
  }
  header{
    background:var(--green);
    color:#fff;
    text-align:center;
    padding:1rem;
  }
  main{
    flex:1;
    display:flex;
    justify-content:center;
    align-items:center;
    padding:0.5rem;
  }
  .container{
    width:100%;
    max-width:960px;
    background:#fff;
    border-radius:12px;
    box-shadow:0 4px 8px rgba(0,0,0,0.1);
    padding:12px;
    text-align:center;
  }
  button{
    background:var(--blue);
    color:#fff;
    border:none;
    padding:1rem 1.2rem;
    border-radius:10px;
    margin:0.5rem;
    cursor:pointer;
    font-size:1.1rem;
    width:100%;
    max-width:280px;
  }
  button:disabled{
    background:#aaa;
    cursor:not-allowed;
  }
  .hud{
    display:flex;
    justify-content:space-around;
    background:var(--green-light);
    padding:0.5rem;
    margin-bottom:1rem;
    border-radius:8px;
    font-size:1rem;
    font-weight:bold;
    color:#fff;
    flex-wrap:wrap;
    gap:6px;
  }
  /* Minijuego 1 */
  .bins{display:flex;justify-content:space-around;margin-top:1rem;flex-wrap:wrap;gap:0.5rem;}
  .bin{flex:1;min-width:90px;padding:1rem;border:2px dashed var(--green);border-radius:8px;}
  .item{display:inline-block;background:var(--blue);color:#fff;padding:0.6rem 1.1rem;border-radius:6px;margin:4px;cursor:grab;font-size:1.6rem;}
  /* Minijuego 2 */
  .tap-area{position:relative;height:40vh;max-height:320px;background:#e0f7fa;border-radius:10px;overflow:hidden;}
  .faucet{position:absolute;width:60px;height:60px;background:var(--blue);border-radius:50%;display:flex;justify-content:center;align-items:center;color:#fff;font-size:1.8rem;cursor:pointer;user-select:none;}
  .waste-bar{height:22px;background:#ccc;margin-top:10px;border-radius:10px;overflow:hidden;}
  .waste-fill{height:100%;width:0;background:red;transition:width 0.1s;}
  /* Minijuego 3 */
  .grid{display:grid;grid-template-columns:repeat(5,1fr);grid-gap:6px;justify-content:center;}
  .cell{width:100%;aspect-ratio:1;background:#c8e6c9;border-radius:6px;display:flex;justify-content:center;align-items:center;font-size:1.5rem;cursor:pointer;}
  .bar{height:22px;background:#ccc;margin:10px;border-radius:10px;overflow:hidden;}
  .fill{height:100%;width:50%;background:var(--blue);transition:width 0.3s;}
</style>
</head>
<body>
<header>
  <h1>Guardianes del Planeta</h1>
</header>
<main>
  <div class="container">
    <div id="hud" class="hud" style="display:none;">
      <div>Nivel: <span id="nivel">1</span></div>
      <div>Puntos: <span id="puntos">0</span></div>
      <div>Objetivo: <span id="objetivo">0/30</span></div>
    </div>
    <div id="scene"></div>
  </div>
</main>

<script>
"use strict";
/* ==== Estado global ==== */
const TARGET_SORT=10, TARGET_TAPS=15, TARGET_TREES=5;
let gameState={nivel:1,puntos:0,objetivo:0,scene:null,timers:[],raf:null};

/* ==== Utilidades ==== */
const qs=s=>document.querySelector(s);
const qsa=s=>Array.from(document.querySelectorAll(s));
function clearTimers(){gameState.timers.forEach(clearInterval);gameState.timers=[];if(gameState.raf){cancelAnimationFrame(gameState.raf);gameState.raf=null;}}
function goTo(scene){clearTimers();qs("#scene").innerHTML="";mountScene(scene);updateHud();}
function updateHud(){qs("#hud").style.display=(gameState.scene==="inicio"||gameState.scene==="final")?"none":"flex";qs("#nivel").textContent=gameState.nivel;qs("#puntos").textContent=gameState.puntos;qs("#objetivo").textContent=gameState.objetivo+"/30";}

/* ==== Escenas ==== */
function mountScene(name){gameState.scene=name;
 if(name==="inicio"){
   qs("#scene").innerHTML=`<h2>Guardianes del Planeta</h2><p>Aprende a cuidar el medio ambiente jugando tres retos.</p><button id="play">Jugar</button>`;
   qs("#play").onclick=()=>{gameState.nivel=1;gameState.puntos=0;gameState.objetivo=0;goTo("juego1");};
 }
 if(name==="juego1"){game1();}
 if(name==="juego2"){game2();}
 if(name==="juego3"){game3();}
 if(name==="final"){
   qs("#scene").innerHTML=`<h2>¡Felicidades, has ayudado al planeta!</h2><p>Puntos totales: ${gameState.puntos}</p><button id="restart">Reiniciar</button>`;
   qs("#restart").onclick=()=>goTo("inicio");
 }
}

/* ==== Juego 1 ==== */
function game1(){
 qs("#scene").innerHTML=`<h2>Minijuego 1 - Separar residuos</h2><p>Toca o arrastra los objetos al contenedor correcto.</p><div id="items"></div><div class="bins"><div class="bin" data-type="org">Orgánico</div><div class="bin" data-type="pla">Plástico</div><div class="bin" data-type="pap">Papel</div></div>`;
 const itemsEl=qs("#items");const types=["org","pla","pap"];let score=0;
 for(let i=0;i<TARGET_SORT;i++){let t=types[Math.floor(Math.random()*3)];let el=document.createElement("div");el.className="item";el.textContent=t==="org"?"🍎":t==="pla"?"🧴":"📄";el.draggable=true;el.dataset.type=t;itemsEl.appendChild(el);}
 qsa(".item").forEach(it=>{
   it.addEventListener("dragstart",e=>{e.dataTransfer.setData("type",it.dataset.type);});
   it.addEventListener("click",()=>binDrop(it));
   it.addEventListener("touchstart",()=>binDrop(it));
 });
 qsa(".bin").forEach(bin=>{
   bin.addEventListener("dragover",e=>e.preventDefault());
   bin.addEventListener("drop",e=>{
     let type=e.dataTransfer.getData("type");let el=itemsEl.querySelector(`.item[data-type=${type}]`);
     if(type===bin.dataset.type && el){el.remove();score++;if(score>=TARGET_SORT){gameState.puntos+=10;gameState.objetivo+=10;gameState.nivel=2;goTo("juego2");}}
   });
 });
 function binDrop(it){let target=qsa(".bin").find(b=>b.dataset.type===it.dataset.type);if(target){it.remove();score++;if(score>=TARGET_SORT){gameState.puntos+=10;gameState.objetivo+=10;gameState.nivel=2;goTo("juego2");}}}
}

/* ==== Juego 2 ==== */
function game2(){
 qs("#scene").innerHTML=`<h2>Minijuego 2 - Ahorra agua</h2><p>Toca los grifos antes que se llene la barra.</p><div class="tap-area" id="tapArea"></div><div class="waste-bar"><div class="waste-fill" id="wasteFill"></div></div>`;
 const tapArea=qs("#tapArea"), wasteFill=qs("#wasteFill");let taps=0, waste=0;
 function spawn(){
   if(tapArea.children.length<3){
     let f=document.createElement("div");f.className="faucet";f.textContent="🚰";
     f.style.left=Math.random()*(tapArea.clientWidth-60)+"px";f.style.top=Math.random()*(tapArea.clientHeight-60)+"px";tapArea.appendChild(f);
     const remove=()=>{taps++;f.remove();waste=Math.max(0,waste-0.1);if(taps>=TARGET_TAPS){gameState.puntos+=10;gameState.objetivo+=10;gameState.nivel=3;goTo("juego3");}};
     f.addEventListener("click",remove);f.addEventListener("touchstart",remove);
   }
 }
 let spawnTimer=setInterval(spawn,1000);gameState.timers.push(spawnTimer);
 function animate(){waste+=0.002;if(waste>1)waste=1;wasteFill.style.width=(waste*100)+"%";if(waste>=1){taps=0;waste=0;tapArea.innerHTML="";}gameState.raf=requestAnimationFrame(animate);}animate();
}

/* ==== Juego 3 ==== */
function game3(){
 qs("#scene").innerHTML=`<h2>Minijuego 3 - Plantar árboles</h2><p>Toca para plantar y cuida el agua.</p><div class="grid" id="grid"></div><div class="bar"><div class="fill" id="waterFill"></div></div>`;
 const grid=qs("#grid"), waterFill=qs("#waterFill");let water=0.5,adults=0;
 for(let i=0;i<25;i++){let c=document.createElement("div");c.className="cell";c.dataset.stage=0;
   const plant=()=>{if(c.dataset.stage==0){c.dataset.stage=1;c.textContent="🌱";water=Math.min(1,water+0.2);}};
   c.addEventListener("click",plant);c.addEventListener("touchstart",plant);grid.appendChild(c);}
 function tick(){water=Math.max(0,water-0.001);waterFill.style.width=(water*100)+"%";adults=0;
 qsa(".cell").forEach(c=>{let s=+c.dataset.stage;if(s>0&&Math.random()<0.01&&water>0.1){s++;c.dataset.stage=s;if(s==2)c.textContent="🌳";if(s>=3)c.textContent="🌲";}if(s>=3)adults++;});
 if(adults>=TARGET_TREES){gameState.puntos+=10;gameState.objetivo+=10;gameState.nivel=4;goTo("final");}
 gameState.raf=requestAnimationFrame(tick);}tick();
}

/* ==== Inicio ==== */
mountScene("inicio");
</script>
</body>
</html>
