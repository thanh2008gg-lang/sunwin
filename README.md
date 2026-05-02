<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<title>Tài Xỉu 3D CINEMA</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
    margin:0;
    overflow:hidden;
    background: radial-gradient(circle at center,#3a3a3a,#000);
    font-family: Arial;
}

/* UI */
#top{
    position:absolute;
    width:100%;
    text-align:center;
    top:10px;
    color:#ffd700;
    font-size:26px;
    text-shadow:0 0 25px #ffd700;
}

#score,#result{
    position:absolute;
    width:100%;
    text-align:center;
    font-size:20px;
}

#score{ top:55px; color:#00ffcc; }
#result{ top:85px; }

/* Buttons */
.btn{
    position:absolute;
    bottom:110px;
    padding:14px 30px;
    border:none;
    border-radius:14px;
    font-size:17px;
    color:white;
    box-shadow:0 0 20px rgba(255,255,255,0.5);
}

#tai{ left:20%; background:#00aa00; }
#xiu{ right:20%; background:#cc0000; }

#roll{
    position:absolute;
    bottom:35px;
    left:50%;
    transform:translateX(-50%);
    padding:18px 35px;
    background:#ffd700;
    border:none;
    border-radius:16px;
    font-size:20px;
    box-shadow:0 0 30px #ffd700;
}
</style>
</head>

<body>

<div id="top">🎲 Tài Xỉu 3D CINEMA</div>
<div id="score"></div>
<div id="result"></div>

<button id="tai" class="btn" onclick="choose('tai')">TÀI</button>
<button id="xiu" class="btn" onclick="choose('xiu')">XỈU</button>
<button id="roll" onclick="rollDice()">LẮC</button>

<audio id="rollSound" src="https://www.soundjay.com/misc/sounds/dice-roll-1.mp3"></audio>
<audio id="winSound" src="https://www.soundjay.com/buttons/sounds/button-4.mp3"></audio>

<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>

<script>
// ===== SCENE =====
const scene = new THREE.Scene();

const camera = new THREE.PerspectiveCamera(70,innerWidth/innerHeight,0.1,1000);
camera.position.set(0,2,8);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.outputEncoding = THREE.sRGBEncoding;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.7;
document.body.appendChild(renderer.domElement);

// ===== LIGHT =====
scene.add(new THREE.AmbientLight(0xffffff,1.6));

let keyLight = new THREE.PointLight(0xffffff,2);
keyLight.position.set(5,10,5);
scene.add(keyLight);

let goldLight = new THREE.PointLight(0xffd700,1.8);
goldLight.position.set(-5,5,5);
scene.add(goldLight);

// ===== FLOOR =====
let floor = new THREE.Mesh(
    new THREE.PlaneGeometry(30,30),
    new THREE.MeshStandardMaterial({color:0x222222, roughness:0.3})
);
floor.rotation.x = -Math.PI/2;
scene.add(floor);

// ===== DICE =====
function createTexture(num){
    let c=document.createElement("canvas");
    c.width=256;c.height=256;
    let ctx=c.getContext("2d");

    ctx.fillStyle="#fff";
    ctx.fillRect(0,0,256,256);

    ctx.fillStyle="#000";
    ctx.font="bold 150px Arial";
    ctx.textAlign="center";
    ctx.textBaseline="middle";
    ctx.fillText(num,128,140);

    return new THREE.CanvasTexture(c);
}

function createDice(){
    let mats=[];
    for(let i=1;i<=6;i++){
        mats.push(new THREE.MeshStandardMaterial({
            map:createTexture(i),
            metalness:0.4,
            roughness:0.2
        }));
    }
    return new THREE.Mesh(new THREE.BoxGeometry(),mats);
}

const dice=[];
for(let i=0;i<3;i++){
    let d=createDice();
    d.position.x=(i-1)*2;
    scene.add(d);
    dice.push(d);
}

// ===== GAME =====
let score=1000000;
let bet=null;

function updateUI(){
    scoreEl.innerHTML="💰 "+score.toLocaleString();
}
updateUI();

function choose(t){
    bet=t;
    result.innerHTML="Bạn chọn: "+(t=="tai"?"TÀI":"XỈU");
}

function rollDice(){
    if(!bet){
        alert("Chọn Tài/Xỉu!");
        return;
    }

    rollSound.play();

    // CAMERA CINEMATIC
    camera.position.z=5;
    camera.position.y=3;

    setTimeout(()=>{
        camera.position.set(0,2,8);
    },400);

    let total=0;

    dice.forEach(d=>{
        let v=Math.floor(Math.random()*6)+1;
        total+=v;

        d.rotation.x=Math.random()*Math.PI*6;
        d.rotation.y=Math.random()*Math.PI*6;
    });

    let res= total>=11?"tai":"xiu";

    setTimeout(()=>{
        if(res===bet){
            score+=100000;
            winSound.play();
            result.innerHTML="🎉 Thắng "+total;
        }else{
            score-=100000;
            result.innerHTML="💀 Thua "+total;
        }
        updateUI();
    },800);
}

// ===== LOOP =====
function animate(){
    requestAnimationFrame(animate);

    // chuyển động nhẹ camera
    camera.position.x = Math.sin(Date.now()*0.0005)*0.3;

    renderer.render(scene,camera);
}
animate();

// ===== RESIZE =====
addEventListener("resize",()=>{
    camera.aspect=innerWidth/innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(innerWidth,innerHeight);
});
</script>

</body>
</html>
