<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tài Xỉu VIP MAX</title>

<style>
body {
    margin: 0;
    background: radial-gradient(circle, #222, #000);
    color: white;
    text-align: center;
    font-family: Arial;
}

h1 { margin-top: 20px; }

.balance {
    font-size: 20px;
    margin: 10px;
}

/* Dice */
.scene {
    display: inline-block;
    perspective: 600px;
    margin: 10px;
}

.cube {
    width: 70px;
    height: 70px;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 1s;
}

.face {
    position: absolute;
    width: 70px;
    height: 70px;
    background: white;
    border: 2px solid black;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 25px;
}

.front  { transform: rotateY(0deg) translateZ(35px); }
.back   { transform: rotateY(180deg) translateZ(35px); }
.right  { transform: rotateY(90deg) translateZ(35px); }
.left   { transform: rotateY(-90deg) translateZ(35px); }
.top    { transform: rotateX(90deg) translateZ(35px); }
.bottom { transform: rotateX(-90deg) translateZ(35px); }

/* Buttons */
button {
    padding: 10px 15px;
    margin: 5px;
    border: none;
    border-radius: 8px;
    font-size: 16px;
}

.tai { background: red; color: white; }
.xiu { background: blue; color: white; }
.roll { background: green; color: white; }

/* Input */
input {
    padding: 8px;
    border-radius: 6px;
    border: none;
    width: 120px;
    text-align: center;
}

/* Result */
.result {
    margin-top: 15px;
    font-size: 22px;
}

/* History */
.history {
    margin-top: 20px;
    max-height: 120px;
    overflow-y: auto;
    font-size: 14px;
}

/* Leaderboard */
.leaderboard {
    margin-top: 20px;
    font-size: 14px;
}
</style>

</head>

<body>

<h1>🎰 Tài Xỉu VIP MAX</h1>

<div class="balance">💰 Số dư: <span id="money">0</span></div>

<!-- Dice -->
<div class="scene"><div id="cube1" class="cube"></div></div>
<div class="scene"><div id="cube2" class="cube"></div></div>
<div class="scene"><div id="cube3" class="cube"></div></div>

<br>

<input type="number" id="betAmount" placeholder="Nhập tiền cược">

<br>

<button class="tai" onclick="bet('tai')">Tài</button>
<button class="xiu" onclick="bet('xiu')">Xỉu</button>

<br>
<button class="roll" onclick="roll()">Lắc</button>

<div class="result" id="result"></div>

<div class="history" id="history"></div>

<div class="leaderboard">
    <h3>🏆 Top tiền cao nhất</h3>
    <div id="top"></div>
</div>

<!-- 🔊 Nhạc -->
<audio id="bgMusic" loop>
    <source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3">
</audio>

<audio id="sound" src="https://www.soundjay.com/misc/sounds/dice-roll-1.mp3"></audio>

<script>
let money = localStorage.getItem("money") ? parseInt(localStorage.getItem("money")) : 1000;
let historyData = JSON.parse(localStorage.getItem("history")) || [];
let top = JSON.parse(localStorage.getItem("top")) || [];
let choice = "";

document.getElementById("money").innerText = money;

const faces = ["⚀","⚁","⚂","⚃","⚄","⚅"];

function createCube(id){
    let cube = document.getElementById(id);
    cube.innerHTML = `
    <div class="face front">${faces[0]}</div>
    <div class="face back">${faces[1]}</div>
    <div class="face right">${faces[2]}</div>
    <div class="face left">${faces[3]}</div>
    <div class="face top">${faces[4]}</div>
    <div class="face bottom">${faces[5]}</div>
    `;
}

createCube("cube1");
createCube("cube2");
createCube("cube3");

function bet(type){
    choice = type;
    result.innerHTML = "Bạn chọn: " + type.toUpperCase();
}

function randomDice(){
    return Math.floor(Math.random()*6)+1;
}

function rollCube(cube){
    let x = Math.random()*720;
    let y = Math.random()*720;
    cube.style.transform = `rotateX(${x}deg) rotateY(${y}deg)`;
}

function updateTop(){
    top.push(money);
    top.sort((a,b)=>b-a);
    top = top.slice(0,5);
    localStorage.setItem("top", JSON.stringify(top));

    let html="";
    top.forEach((t,i)=>{
        html += `<div>${i+1}. 💰 ${t}</div>`;
    });
    document.getElementById("top").innerHTML = html;
}

function loadHistory(){
    let h = document.getElementById("history");
    historyData.forEach(item=>{
        let div = document.createElement("div");
        div.innerText = item;
        h.appendChild(div);
    });
}

loadHistory();
updateTop();

function roll(){
    let bet = parseInt(document.getElementById("betAmount").value);

    if(choice===""){
        alert("Chọn Tài hoặc Xỉu!");
        return;
    }

    if(!bet || bet<=0){
        alert("Nhập tiền cược!");
        return;
    }

    if(bet>money){
        alert("Không đủ tiền!");
        return;
    }

    sound.play();
    bgMusic.play();

    let d1 = randomDice();
    let d2 = randomDice();
    let d3 = randomDice();

    rollCube(cube1);
    rollCube(cube2);
    rollCube(cube3);

    let total = d1+d2+d3;

    setTimeout(()=>{
        let outcome = total>=11 ? "tai" : "xiu";
        let text="";

        if(outcome===choice){
            money += bet;
            text = "🎉 Thắng +" + bet + " ("+total+")";
        } else {
            money -= bet;
            text = "💀 Thua -" + bet + " ("+total+")";
        }

        document.getElementById("money").innerText = money;
        result.innerHTML = text;

        historyData.unshift(text);
        localStorage.setItem("history", JSON.stringify(historyData));
        localStorage.setItem("money", money);

        let div = document.createElement("div");
        div.innerText = text;
        document.getElementById("history").prepend(div);

        updateTop();
    },1000);
}
</script>

</body>
</html>