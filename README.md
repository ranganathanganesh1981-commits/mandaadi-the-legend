<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>MANDAADI: THE LEGEND</title>
<style>
*{box-sizing:border-box}
html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#052033;color:white;font-family:Arial,sans-serif;touch-action:none}
canvas{display:block}
button,input,select{padding:10px;border:1px solid #56edff;border-radius:10px;background:#092c42;color:white;font-weight:bold;max-width:100%}
button:active{background:#00a9cb;color:#00151d}
input{width:100%;margin:5px 0}
#menu{position:absolute;inset:0;z-index:5;display:flex;align-items:center;justify-content:center;background:radial-gradient(#125c75,#03131e);padding:12px;text-align:center}
#panel{width:min(500px,100%);max-height:96%;overflow:auto;padding:18px;border:1px solid #55eaff;border-radius:18px;background:#061d2bed}
h1{font-size:clamp(32px,7vw,54px);margin:8px 0;color:#66f4ff;text-shadow:0 0 20px #00bce9}
#developer{font-size:13px;letter-spacing:1.5px;color:#ffe49b;margin:12px 0}
.sub{letter-spacing:4px;font-size:12px}
.small{font-size:12px;color:#c8eaf2;line-height:1.6}
.primary{width:100%;padding:15px;background:#00c6e8;color:#002334;font-size:16px;margin-top:8px}
.row{display:flex;gap:8px;justify-content:center;align-items:center;flex-wrap:wrap;margin:10px 0}
.row>*{flex:1}
#hud{display:none;position:absolute;inset:0;z-index:2;pointer-events:none}
#top{position:absolute;top:6px;left:6px;right:6px;display:flex;justify-content:space-between;gap:4px}
.info{background:#041d2bed;border:1px solid #53e9ff;border-radius:8px;padding:7px;font-size:10px;line-height:1.5}
#controls{position:absolute;bottom:8px;left:8px;right:8px;display:flex;justify-content:space-between;align-items:end;gap:8px}
.group{display:flex;gap:7px;pointer-events:auto}
.group button{min-width:52px;height:55px;font-size:16px}
.group.right{flex-direction:column;align-items:stretch}
.group.right button{width:108px;min-height:35px;height:auto;padding:7px 4px;font-size:11px}
#boost{background:#ff6d16;color:white;border-color:#ffe08a}
#boost.active{background:#ffe08a;color:#19232b;box-shadow:0 0 18px #ff9b24}
#message{position:absolute;top:24%;left:50%;transform:translateX(-50%);background:#032b40ed;padding:12px;border-radius:10px;border:1px solid #55edff;display:none;text-align:center;max-width:90%;font-size:13px}
#rotate{display:none;position:absolute;inset:0;z-index:10;background:#031521;align-items:center;justify-content:center;text-align:center}
@media(orientation:portrait){#rotate{display:flex}}
</style>
</head>
<body>

<div id="rotate">
  <div>
    <div style="font-size:48px">📱 ↻</div>
    <h2>ROTATE YOUR PHONE</h2>
    <p>Play MANDAADI in landscape mode.</p>
  </div>
</div>

<div id="menu">
<div id="panel">
<h1>MANDAADI</h1>
<div class="sub">THE LEGEND</div>
<div id="developer">GAME DEVELOPER<br>NITHISH KUMAR</div>
<div style="font-size:46px">⛵🌊</div>
<h3>OCEAN SAILING CHAMPIONSHIP</h3>
<p class="small">ISLANDS • BOAT RACING • LIVE 2-PLAYER ROOM</p>
<p class="small">PLAYER ID: <b id="pid">Loading...</b></p>

<div class="row">
<label class="small">DIFFICULTY<br>
<select id="level">
<option value="0">LEVEL 1</option>
<option value="1">LEVEL 2</option>
<option value="2">LEVEL 3</option>
</select>
</label>
<label class="small">LAPS<br>
<select id="laps">
<option>1</option>
<option selected>2</option>
<option>3</option>
</select>
</label>
</div>

<div class="row">
<button id="createRoom">CREATE ONLINE ROOM</button>
<button id="joinRoom">JOIN ROOM</button>
</div>

<input id="roomInput" placeholder="Enter friend's room code">
<p class="small" id="roomStatus">Online multiplayer: not connected</p>
<button class="primary" id="start">START CHAMPIONSHIP</button>

<div class="row">
<button id="invite">INVITE FRIENDS</button>
<button id="copyid">COPY PLAYER ID</button>
</div>

<p class="small" id="status">Loading 3D ocean...</p>
<p class="small">Free hosting prototype • Internet required for online multiplayer</p>
</div>
</div>

<div id="hud">
<div id="top">
<div class="info" id="stats">MANDAADI</div>
<div class="info" id="checkpoint">CHECKPOINT 1</div>
<div class="info" id="wind">SAIL: DOWN</div>
</div>

<div id="message"></div>

<div id="controls">
<div class="group">
<button id="left">◀ LEFT</button>
<button id="right">RIGHT ▶</button>
</div>

<div class="group right">
<button id="sail">SAIL ▲</button>
<button id="boost">🚀 பாயைத் தூக்கு டா!</button>
<button id="brake">BRAKE</button>
<button id="pause">PAUSE</button>
<button id="map">MAP VIEW</button>
</div>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>

<script>
(()=>{
"use strict";

const $=id=>document.getElementById(id);

const SUPABASE_URL="https://hziethqnwmdxgpglvnze.supabase.co";
const SUPABASE_KEY="sb_publishable_KD5Vd4OE3rKtSpIbXqpYzw_WD-SJGQN";

let supabase=null,channel=null,roomCode="",isHost=false;
let remoteSeen=false,lastBroadcast=0;

try{
 if(window.supabase&&SUPABASE_URL.startsWith("https://")&&!SUPABASE_KEY.includes("YOUR_")){
  supabase=window.supabase.createClient(SUPABASE_URL,SUPABASE_KEY);
 }
}catch(e){console.error(e)}

let playerID;
try{
 playerID=localStorage.getItem("mandaadi_player_id");
 if(!playerID){
  playerID="MD-"+Math.random().toString(36).slice(2,8).toUpperCase();
  localStorage.setItem("mandaadi_player_id",playerID);
 }
}catch(e){
 playerID="MD-"+Math.random().toString(36).slice(2,8).toUpperCase();
}

$("pid").textContent=playerID;

const newCode=()=>Math.random().toString(36).slice(2,7).toUpperCase();

function setRoomStatus(s){
 $("roomStatus").textContent=s;
}

function setRoomURL(code){
 try{return location.href.split("?")[0]+"?room="+encodeURIComponent(code)}
 catch(e){return code}
}

async function leaveChannel(){
 if(channel&&supabase){
  try{await supabase.removeChannel(channel)}catch(e){}
 }
 channel=null;
 remoteSeen=false;
}

async function connectRoom(code,host){
 if(!supabase){
  setRoomStatus("Supabase library/key unavailable. Check internet and reload.");
  return false;
 }

 await leaveChannel();

 roomCode=code.toUpperCase();
 isHost=host;

 channel=supabase.channel("mandaadi-room-"+roomCode,{
  config:{broadcast:{self:false}}
 });

 channel.on("broadcast",{event:"race-state"},({payload})=>{
  if(!payload||payload.playerID===playerID||!boats.length)return;

  const b=boats[1];
  b.t=Number(payload.t)||0;
  b.lateral=Number(payload.lateral)||0;
  b.speed=Number(payload.speed)||0;
  b.sail.visible=!!payload.sail;
  remoteSeen=true;

  $("wind").textContent="ROOM "+roomCode+" • FRIEND ONLINE";
 });

 return await new Promise(resolve=>{
  const timeout=setTimeout(()=>{
   setRoomStatus("Connection timed out. Check internet and try again.");
   resolve(false);
  },12000);

  channel.subscribe(status=>{
   if(status==="SUBSCRIBED"){
    clearTimeout(timeout);

    setRoomStatus("Room "+roomCode+" connected • share code with friend");
    $("roomInput").value=roomCode;

    const url=setRoomURL(roomCode);
    history.replaceState(null,"",url);

    resolve(true);

    channel.send({
     type:"broadcast",
     event:"race-state",
     payload:{
      playerID,
      hello:true,
      t:0,
      lateral:0,
      speed:0,
      sail:false
     }
    });
   }else if(status==="CHANNEL_ERROR"||status==="TIMED_OUT"){
    clearTimeout(timeout);
    setRoomStatus("Realtime connection failed. Check Supabase settings.");
    resolve(false);
   }
  });
 });
}

$("createRoom").onclick=async()=>{
 const code=newCode();
 setRoomStatus("Creating room...");
 const ok=await connectRoom(code,true);

 if(ok){
  try{
   await navigator.clipboard.writeText(setRoomURL(code));
   setRoomStatus("Room "+code+" ready. Invite link copied!");
  }catch(e){
   setRoomStatus("Room "+code+" ready. Share this code with your friend.");
  }

  alert("Room code: "+code+"\nShare this code/link with your friend.");
 }
};

$("joinRoom").onclick=async()=>{
 const code=$("roomInput").value.trim().toUpperCase()
  ||new URLSearchParams(location.search).get("room")
  ||"";

 if(!code){
  setRoomStatus("Enter your friend's room code first.");
  return;
 }

 setRoomStatus("Joining room "+code+"...");
 await connectRoom(code,false);
};

$("copyid").onclick=async()=>{
 try{
  await navigator.clipboard.writeText(playerID);
  alert("Player ID copied: "+playerID);
 }catch(e){
  prompt("Copy Player ID",playerID);
 }
};

$("invite").onclick=async()=>{
 const code=roomCode||new URLSearchParams(location.search).get("room");

 const text=code
  ?"Join my MANDAADI live race! Room code: "+code+"\n"+setRoomURL(code)
  :"Create/join a room first, then invite your friend.";

 try{
  if(navigator.share){
   await navigator.share({
    title:"MANDAADI: THE LEGEND",
    text,
    url:code?setRoomURL(code):undefined
   });
  }else{
   await navigator.clipboard.writeText(text);
   alert("Invite copied.");
  }
 }catch(e){
  prompt("Copy invite",text);
 }
};

if(new URLSearchParams(location.search).get("room")){
 $("roomInput").value=new URLSearchParams(location.search).get("room");
 setRoomStatus("Invite link detected. Tap JOIN ROOM to connect.");
}

if(!window.THREE){
 $("status").textContent="3D library not loaded. Connect internet and reload.";
 return;
}

const T=THREE;

let scene,camera,renderer,curve,boats=[];
let ready=false,playing=false,paused=false;
let level=0,laps=2,steer=0,sailHeld=false;
let boostHeld=false,braking=false,overview=false;
let last=performance.now(),messageTimer=0;

const notice=$("message");

function say(t){
 notice.textContent=t;
 notice.style.display="block";
 messageTimer=2.5;
}

const mat=(c,r=1)=>new T.MeshStandardMaterial({color:c,roughness:r});

try{
 scene=new T.Scene();
 scene.background=new T.Color(0x5fc7e5);
 scene.fog=new T.Fog(0x5fc7e5,180,700);

 camera=new T.PerspectiveCamera(62,innerWidth/innerHeight,.1,1500);

 renderer=new T.WebGLRenderer({
  antialias:false,
  powerPreference:"high-performance"
 });

 renderer.setPixelRatio(Math.min(devicePixelRatio,1.4));
 renderer.setSize(innerWidth,innerHeight);
 renderer.outputColorSpace=T.SRGBColorSpace;

 document.body.insertBefore(renderer.domElement,$("menu"));

 scene.add(new T.HemisphereLight(0xcaf7ff,0x456655,2.2));

 const sun=new T.DirectionalLight(0xffe9b9,2);
 sun.position.set(-100,140,70);
 scene.add(sun);

 const ocean=mat(0x087da4,.5);
 const sand=mat(0xe7d09b);
 const green=mat(0x27834b);
 const wood=mat(0x754329);
 const plank=mat(0xb37b43);
 const rock=mat(0x677173);
 const cyan=mat(0x56edff);
 const white=mat(0xffffff);
 const dark=mat(0x26383c);
 const orange=mat(0xff6b27);

 function add(geo,m,parent=scene,pos=[0,0,0]){
  const o=new T.Mesh(geo,m);
  o.position.set(...pos);
  parent.add(o);
  return o;
 }

 function block(p,pos,size,m){
  return add(new T.BoxGeometry(...size),m,p,pos);
 }

 function group(p=scene,pos=[0,0,0]){
  const g=new T.Group();
  g.position.set(...pos);
  p.add(g);
  return g;
 }

 function cylinder(p,pos,r1,r2,h,m,n=8){
  return add(new T.CylinderGeometry(r1,r2,h,n),m,p,pos);
 }

 const sea=add(
  new T.PlaneGeometry(1400,1400),
  ocean,scene,[0,-1,0]
 );
 sea.rotation.x=-Math.PI/2;

 function island(x,z,s,type=0){
  const g=group(scene,[x,0,z]);

  add(new T.CylinderGeometry(s*.75,s,2,12),sand,g,[0,-.2,0]);
  add(new T.CylinderGeometry(s*.62,s*.78,3,12),green,g,[0,1,0]);
  add(new T.DodecahedronGeometry(s*.6),type?rock:green,g,[0,2.4,0]);

  for(let i=0;i<5;i++){
   const a=i*2.4;
   const px=Math.cos(a)*s*.36;
   const pz=Math.sin(a)*s*.36;

   cylinder(g,[px,3,pz],.18,.27,2.4,wood,6);

   for(let j=0;j<4;j++){
    const ang=j*Math.PI/2;

    const leaf=block(
     g,
     [px+Math.cos(ang)*1.1,4.1,pz+Math.sin(ang)*1.1],
     [2.2,.13,.38],
     green
    );

    leaf.rotation.y=-ang;
   }
  }
 }

 island(-100,20,24,1);
 island(-60,-55,20);
 island(-5,-85,28,1);
 island(50,-45,22);
 island(80,10,20);
 island(45,55,25);
 island(-10,60,22);
 island(-105,85,21);

 const volcano=group(scene,[-5,0,-85]);
 add(new T.ConeGeometry(17,23,12),rock,volcano,[0,11,0]);
 add(new T.ConeGeometry(3,4,8),orange,volcano,[0,22,0]);

 const ship=group(scene,[105,0,-25]);
 block(ship,[0,1,0],[14,2.6,5],wood);
 block(ship,[0,4,0],[.7,8,.7],dark);
 block(ship,[-2,4,0],[.25,5,.25],wood);
 block(ship,[0,5,0],[4,.3,.15],plank);

 const pts=[
  new T.Vector3(-115,0,65),
  new T.Vector3(-135,0,22),
  new T.Vector3(-105,0,-20),
  new T.Vector3(-60,0,-42),
  new T.Vector3(-8,0,-45),
  new T.Vector3(40,0,-15),
  new T.Vector3(66,0,20),
  new T.Vector3(35,0,50),
  new T.Vector3(-15,0,52),
  new T.Vector3(-65,0,45)
 ];

 curve=new T.CatmullRomCurve3(pts,true,"catmullrom",.3);
 const route=curve.getPoints(500);

 for(let i=0;i<route.length;i+=5){
  const p=route[i];
  const n=route[(i+2)%route.length];

  const m=block(scene,[p.x,-.25,p.z],[2.1,.12,.55],cyan);
  m.rotation.y=Math.atan2(n.x-p.x,n.z-p.z);
 }

 for(let i=0;i<8;i++){
  const p=curve.getPointAt(i/8);
  const b=group(scene,[p.x,0,p.z]);

  cylinder(b,[0,.3,0],.65,.8,1.4,i===0?green:cyan,8);
  cylinder(b,[0,1.4,0],.12,.12,1,white);
  block(b,[.7,1.8,0],[1.3,.6,.12],i===0?green:cyan);
 }

 function makeBoat(color){
  const b=group();

  block(b,[0,0,0],[2.5,.55,6.2],wood);
  block(b,[0,.32,-.1],[2.1,.16,5.3],plank);
  block(b,[0,0,-3.2],[1.2,.35,.65],wood);

  block(b,[-4,-.1,0],[.18,.2,7],wood);
  block(b,[4,-.1,0],[.18,.2,7],wood);

  block(b,[-3.1,-.1,-1.7],[1.8,.16,.7],plank);
  block(b,[3.1,-.1,-1.7],[1.8,.16,.7],plank);

  cylinder(b,[0,3.8,0],.12,.15,7.6,dark);

  const sail=group(b);
  const geo=new T.BufferGeometry();

  geo.setAttribute(
   "position",
   new T.Float32BufferAttribute([
    0,7.3,0,
    0,.6,0,
    4.7,1,0
   ],3)
  );

  geo.setIndex([0,1,2]);
  geo.computeVertexNormals();

  add(
   geo,
   new T.MeshStandardMaterial({color,side:T.DoubleSide}),
   sail,[0,0,-.2]
  );

  const bar=block(sail,[2.2,1,0],[4.5,.12,.12],wood);
  bar.rotation.z=-.03;
  sail.visible=false;

  for(let i=0;i<3;i++){
   cylinder(b,[-.6+i*.6,.8,1],.12,.12,.6,dark,6);
  }

  return{mesh:b,sail};
 }

 boats=[
  {name:"YOU",...makeBoat(0xf4e7ca),t:0,lateral:0,speed:0},
  {name:"FRIEND",...makeBoat(0xe83b40),t:0,lateral:3,speed:0},
  {name:"AI TEAM",...makeBoat(0xffd13f),t:.018,lateral:-3,speed:0}
 ];

 function pose(b){
  const t=((b.t%1)+1)%1;
  const p=curve.getPointAt(t);
  const tangent=curve.getTangentAt(t).normalize();

  const side=new T.Vector3(tangent.z,0,-tangent.x);
  const pos=p.clone().addScaledVector(side,b.lateral);

  b.mesh.position.set(
   pos.x,
   .7+Math.sin(performance.now()*.002+b.t*10)*.12,
   pos.z
  );

  b.mesh.rotation.y=Math.atan2(-tangent.x,-tangent.z);
 }

 async function broadcastState(){
  if(!channel||!playing)return;

  const b=boats[0];

  try{
   await channel.send({
    type:"broadcast",
    event:"race-state",
    payload:{
     playerID,
     t:b.t,
     lateral:b.lateral,
     speed:b.speed,
     sail:boostHeld
    }
   });
  }catch(e){}
 }

 function reset(){
  if(!ready){
   $("status").textContent="3D loading. Wait, then press START again.";
   return;
  }

  level=+$("level").value;
  laps=+$("laps").value;

  playing=true;
  paused=false;
  overview=false;
  steer=0;
  sailHeld=false;
  boostHeld=false;
  braking=false;

  boats.forEach((b,i)=>{
   b.t=0;
   b.speed=0;
   b.lateral=i===0?0:(i===1?3:-3);
   b.sail.visible=false;
   pose(b);
  });

  $("boost").classList.remove("active");
  $("menu").style.display="none";
  $("hud").style.display="block";
  $("pause").textContent="PAUSE";

  say(roomCode?"LIVE RACE "+roomCode+" • GO!":"RACE START! Create/join a room for online play.");
 }

 $("start").onclick=reset;

 function hold(id,down,up){
  const el=$(id);

  el.addEventListener("pointerdown",e=>{
   e.preventDefault();
   down();
   try{el.setPointerCapture(e.pointerId)}catch(_){}
  });

  const release=()=>up();

  el.addEventListener("pointerup",release);
  el.addEventListener("pointercancel",release);
  el.addEventListener("lostpointercapture",release);
 }

 hold("left",()=>steer=1,()=>{if(steer===1)steer=0});
 hold("right",()=>steer=-1,()=>{if(steer===-1)steer=0});
 hold("sail",()=>sailHeld=true,()=>sailHeld=false);

 hold("boost",()=>{
  boostHeld=true;
  boats[0].sail.visible=true;
  $("boost").classList.add("active");
  say("பாயைத் தூக்கு டா! FULL SPEED!");
 },()=>{
  boostHeld=false;
  boats[0].sail.visible=false;
  $("boost").classList.remove("active");
 });

 hold("brake",()=>braking=true,()=>braking=false);

 $("pause").onclick=()=>{
  if(!playing)return;
  paused=!paused;
  $("pause").textContent=paused?"RESUME":"PAUSE";
  say(paused?"Race paused":"Race resumed");
 };

 $("map").onclick=()=>overview=!overview;

 function update(dt){
  const me=boats[0];

  me.lateral=T.MathUtils.clamp(
   me.lateral+steer*dt*7,
   -8,
   8
  );

  if(sailHeld){
   me.speed=Math.min(me.speed+dt*2.4,6+level*.7);
  }else{
   me.speed=Math.max(0,me.speed-dt*.7);
  }

  if(boostHeld){
   me.speed=Math.min(me.speed+dt*7,13+level*1.5);
  }

  if(braking){
   me.speed=Math.max(0,me.speed-dt*5);
  }

  me.t+=me.speed*dt/300;

  if(me.t>=laps){
   playing=false;
   me.sail.visible=false;
   $("menu").style.display="flex";
   $("hud").style.display="none";
   $("panel").querySelector("h1").textContent="RACE COMPLETE!";
   $("panel").querySelector(".sub").textContent="MANDAADI CHAMPIONSHIP";
   $("status").textContent="Race complete! Start again for another race.";
   return;
  }

  if(!roomCode||!channel){
   const ai=boats[2];
   const target=[2.6,3.1,3.5][level]+.12;

   ai.speed=T.MathUtils.lerp(ai.speed,target,dt*.6);
   ai.t+=ai.speed*dt/300;
   ai.lateral+=Math.sin(performance.now()*.0006+2)*dt*.2;
  }

  boats.forEach(pose);

  // SMART BOAT AVOIDANCE
  // Nearby boats trigger a sideways slide instead of blocking the player.
  for(let i=1;i<boats.length;i++){
   const other=boats[i];

   if(!other||!other.mesh)continue;

   // In online mode, the AI boat is not treated as a live opponent.
   if(i===2&&roomCode&&channel)continue;

   const dx=me.mesh.position.x-other.mesh.position.x;
   const dz=me.mesh.position.z-other.mesh.position.z;
   const distance=Math.hypot(dx,dz);

   // Start avoiding before the boats physically overlap.
   if(distance<8.5){

    // Move away from the opponent's lane.
    let side=me.lateral>=other.lateral?1:-1;

    // If they are almost in the same lane, choose a consistent side.
    if(Math.abs(me.lateral-other.lateral)<0.8){
     side=(i===1?1:-1);
    }

    const target=T.MathUtils.clamp(
     other.lateral+side*5.2,
     -9.5,
     9.5
    );

    const strength=T.MathUtils.clamp(
     (8.5-distance)/5.0,
     0,
     1
    );

    // Smooth slide to the side, not an instant teleport.
    me.lateral=T.MathUtils.lerp(
     me.lateral,
     target,
     Math.min(1,dt*(2.5+strength*4.5))
    );

    // Do not roll back progress or apply a hard collision stop.
    me.speed=Math.max(me.speed,sailHeld?2.2:0.8);

    pose(me);

    if(distance<5.5){
     say("BOAT AVOID! Sliding past opponent...");
    }
   }
  }

  const now=performance.now();

  if(channel&&now-lastBroadcast>90){
   lastBroadcast=now;
   broadcastState();
  }

  const place=1
   +(boats[1].t>me.t?1:0)
   +(boats[2].t>me.t?1:0);

  $("stats").innerHTML=
   "⛵ YOU<br>SPEED "+
   Math.round(me.speed*20)+
   " km/h<br>PLACE "+place+"/3";

  $("checkpoint").innerHTML=
   "LAP "+(Math.floor(me.t)+1)+"/"+laps+
   "<br>CHECKPOINT "+(Math.floor((me.t%1)*8)+1)+"/8";

  $("wind").innerHTML=roomCode
   ?(remoteSeen?"FRIEND ONLINE":"ROOM "+roomCode+" • WAITING")
   :(boostHeld?"SAIL: UP 🚀":"SAIL: DOWN");
 }

 function animate(){
  requestAnimationFrame(animate);

  const now=performance.now();
  const dt=Math.min((now-last)/1000,.035);
  last=now;

  if(playing&&!paused)update(dt);

  if(messageTimer>0){
   messageTimer-=dt;
   if(messageTimer<=0)notice.style.display="none";
  }

  if(boats.length){
   const me=boats[0];
   const p=me.mesh.position;

   if(overview){
    camera.position.lerp(new T.Vector3(0,210,145),.04);
    camera.lookAt(0,0,0);
   }else{
    const forward=new T.Vector3(0,0,-1).applyQuaternion(me.mesh.quaternion);
    const desired=p.clone().addScaledVector(forward,-19);
    desired.y=12;

    camera.position.lerp(
     desired,
     1-Math.exp(-dt*3)
    );

    camera.lookAt(p.x,2,p.z);
   }
  }

  renderer.render(scene,camera);
 }

 boats.forEach(pose);

 camera.position.set(0,210,145);
 camera.lookAt(0,0,0);

 ready=true;
 $("status").textContent="3D READY • Create/join a room for live multiplayer";

 animate();

 window.addEventListener("resize",()=>{
  camera.aspect=innerWidth/innerHeight;
  camera.updateProjectionMatrix();
  renderer.setPixelRatio(Math.min(devicePixelRatio,1.4));
  renderer.setSize(innerWidth,innerHeight);
 });

}catch(err){
 console.error(err);
 $("status").textContent="3D error: "+err.message+". Reload the page and try again.";
}

})();
</script>
</body>
</html>

