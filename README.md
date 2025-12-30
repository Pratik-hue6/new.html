# new.html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>3D Anime Action Game</title>
<style>
  body { margin: 0; overflow: hidden; background: #000; }
  #ui { position: absolute; top: 10px; left: 10px; color: #0ff; font-size: 18px; z-index: 10;}
  #story { position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); color:#ff0; font-size:24px; text-align:center;}
</style>
</head>
<body>

<div id="ui">Score: 0 | XP: 0 | Boss HP: 100%</div>
<div id="story">🚀 Anime RPG Story Mode: Defeat the Boss!</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/loaders/GLTFLoader.js"></script>

<script>
// Scene
const scene = new THREE.Scene();
scene.fog = new THREE.Fog(0x000000, 10, 100);

// Camera + Renderer
const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// Lighting
const ambient = new THREE.AmbientLight(0x0ff, 0.5); scene.add(ambient);
const dirLight = new THREE.DirectionalLight(0xff00ff, 1); dirLight.position.set(10,20,10); scene.add(dirLight);

// Controls
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableZoom = true;
camera.position.set(0, 3, 10);
controls.update();

// Ground
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(60,60),
  new THREE.MeshStandardMaterial({color:0x111111})
);
ground.rotation.x = -Math.PI/2;
scene.add(ground);

// Load Models
const loader = new THREE.GLTFLoader();
let player, boss;

loader.load("player.glb", gltf => {
  player = gltf.scene;
  player.scale.set(1.5, 1.5, 1.5);
  player.position.set(0, 0, 0);
  scene.add(player);
});

loader.load("boss.glb", gltf => {
  boss = gltf.scene;
  boss.scale.set(2, 2, 2);
  boss.position.set(0, 0, -15);
  scene.add(boss);
});

// Projectiles & power-ups
const projectiles = [];
const powerUps = [];

// Stats
let score = 0, xp = 0, bossHP = 100;

// Shoot
function shootProjectile(){
  const geo = new THREE.SphereGeometry(0.2, 8, 8);
  const mat = new THREE.MeshStandardMaterial({color: 0xffff00, emissive: 0xffff00});
  const p = new THREE.Mesh(geo, mat);
  p.position.set(player.position.x, player.position.y+1, player.position.z - 1);
  p.userData.speed = -0.3;
  scene.add(p);
  projectiles.push(p);
}

// Spawn PowerUp
function spawnPowerUp(){
  const geo = new THREE.IcosahedronGeometry(0.4, 0);
  const mat = new THREE.MeshStandardMaterial({color: 0x00ff00, emissive: 0x00ff00});
  const pu = new THREE.Mesh(geo, mat);
  pu.position.set((Math.random()-0.5)*20, 1, -10 - Math.random()*10);
  scene.add(pu);
  powerUps.push(pu);
}

// Keyboard
document.addEventListener("keydown", e => {
  if(e.key==="ArrowLeft") player.position.x -= 0.5;
  if(e.key==="ArrowRight") player.position.x += 0.5;
  if(e.key===" ") shootProjectile();
});

// Game Loop
function animate(){
  requestAnimationFrame(animate);

  // Move projectiles
  for(let i=projectiles.length-1; i>=0; i--){
    const p = projectiles[i];
    p.position.z += p.userData.speed;
    if(boss && boss.position.distanceTo(p.position) < 2){
      bossHP -= 5;
      score += 10; xp += 5;
      scene.remove(p); projectiles.splice(i, 1);
    }
    if(p.position.z < -30){
      scene.remove(p); projectiles.splice(i, 1);
    }
  }

  // Power-ups
  if(Math.random() < 0.003) spawnPowerUp();
  for(let i=powerUps.length-1; i>=0; i--){
    const pu = powerUps[i];
    pu.position.z += 0.02;
    if(player && player.position.distanceTo(pu.position) < 1){
      score += 50; xp += 20;
      scene.remove(pu); powerUps.splice(i,1);
    }
  }

  // Update UI
  document.getElementById("ui").innerText = 
    `Score: ${score} | XP: ${xp} | Boss HP: ${bossHP}%`;

  renderer.render(scene, camera);
}
animate();
</script>

</body>
</html>
