<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Лучшее место</title>

<!-- CodeMirror -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/codemirror.min.css">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/theme/dracula.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/codemirror.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.16/mode/javascript/javascript.min.js"></script>

<style>
body{
  margin:0;
  font-family:Arial;
  background:#111;
  color:white;
}

/* ===== LOGIN ===== */
#loginScreen{
  height:100vh;
  display:flex;
  align-items:center;
  justify-content:center;
}
.loginBox{
  background:#222;
  padding:25px;
  border-radius:12px;
  text-align:center;
}
.loginBox input{
  display:block;
  width:220px;
  margin:10px auto;
  padding:8px;
}
.loginBox button{
  padding:8px 20px;
  cursor:pointer;
}

/* ===== APP ===== */
#app{
  display:none;
  height:100vh;
}
#editor{
  width:40%;
  display:flex;
  flex-direction:column;
}
.CodeMirror{
  flex:1;
}
#run{
  padding:12px;
  font-size:16px;
  background:#1e88e5;
  color:white;
  border:none;
  cursor:pointer;
}
canvas{
  background:white;
}
</style>
</head>

<body>

<!-- ===== LOGIN ===== -->
<div id="loginScreen">
  <div class="loginBox">
    <h2>Вход</h2>
    <input id="login" placeholder="Логин">
    <input id="password" type="password" placeholder="Пароль">
    <button onclick="enter()">Вход</button>
    <p id="error" style="color:red;"></p>
  </div>
</div>

<!-- ===== ЛУЧШЕЕ МЕСТО ===== -->
<div id="app">
  <div id="editor">
    <textarea id="code"></textarea>
    <button id="run">▶ Run</button>
  </div>
  <canvas id="canvas" width="600" height="600"></canvas>
</div>

<script>
/* ================= LOGIN ================= */
const CORRECT_LOGIN = "admin";
const CORRECT_PASSWORD = "1234";

function enter(){
  if(
    login.value === CORRECT_LOGIN &&
    password.value === CORRECT_PASSWORD
  ){
    loginScreen.style.display = "none";
    app.style.display = "flex";
    initApp();
  }else{
    error.textContent = "Неверный логин или пароль";
  }
}

/* ================= APP ================= */
let editor;
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
let turtle;
let intervals = [];

/* SAFE setInterval */
const _setInterval = window.setInterval;
window.setInterval = (fn,ms)=>{
  const id = _setInterval(fn,ms);
  intervals.push(id);
  return id;
};

/* RESET */
function resetAll(){
  intervals.forEach(id=>clearInterval(id));
  intervals=[];
  ctx.clearRect(0,0,canvas.width,canvas.height);
  turtle={
    x:300,y:300,angle:0,
    pen:true,color:"black",width:2
  };
}

/* TURTLE API */
function applyStyle(){
  ctx.strokeStyle=turtle.color;
  ctx.lineWidth=turtle.width;
}
function goto(x,y){ turtle.x=x+300; turtle.y=300-y; }
function left(a){ turtle.angle-=a; }
function right(a){ turtle.angle+=a; }
function penup(){ turtle.pen=false; }
function pendown(){ turtle.pen=true; }
function forward(d){
  const r=turtle.angle*Math.PI/180;
  const nx=turtle.x+Math.cos(r)*d;
  const ny=turtle.y+Math.sin(r)*d;
  if(turtle.pen){
    applyStyle();
    ctx.beginPath();
    ctx.moveTo(turtle.x,turtle.y);
    ctx.lineTo(nx,ny);
    ctx.stroke();
  }
  turtle.x=nx; turtle.y=ny;
}
function color(c){ turtle.color=c; }
function width(w){ turtle.width=w; }
function randomColor_h(){
  turtle.color=`hsl(${Math.random()*360},100%,50%)`;
}

/* INIT APP */
function initApp(){
  editor = CodeMirror.fromTextArea(code,{
    mode:"javascript",
    theme:"dracula",
    lineNumbers:true
  });

  editor.setValue(localStorage.getItem("code") || `
width(4)

setInterval(()=>{
  goto(0,0)
  randomColor_h()
  forward(120)
  left(12)
},300)
`);

  editor.on("change",()=>{
    localStorage.setItem("code",editor.getValue());
  });

  run.onclick=()=>{
    resetAll();
    try{
      new Function(editor.getValue())();
    }catch(e){
      alert("❌ Ошибка:\\n"+e.message);
    }
  };

  resetAll();
}
</script>

</body>
</html>
