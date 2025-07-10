<!-- Zain Arshad – Ultra‑HD 3D Portfolio -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Zain Arshad | 3D Portfolio</title>
  <style>
    body,html{margin:0;padding:0;overflow:hidden;background:#000;color:gold;font-family:"Segoe UI",sans-serif}
    #overlay{position:absolute;top:0;left:0;width:100%;padding:15px 20px;z-index:10;background:linear-gradient(180deg,rgba(0,0,0,.8),transparent);pointer-events:none}
    #overlay h1{margin:0;font-size:2.5rem;letter-spacing:1px}
    #overlay p{margin:4px 0 0;font-size:1rem;opacity:.8}
    #audioToggle{position:absolute;bottom:20px;right:20px;z-index:11;background:rgba(255,215,0,.15);border:1px solid gold;border-radius:50%;width:48px;height:48px;display:flex;align-items:center;justify-content:center;font-size:1.2rem;color:gold;cursor:pointer;backdrop-filter:blur(4px)}
  </style>
</head>
<body>
  <div id="overlay">
    <h1>Zain Arshad – 3D Portfolio ✨</h1>
    <p>Tap / click objects • Orbit to explore</p>
  </div>
  <div id="audioToggle">🔊</div>
  <audio id="bgm" src="https://cdn.pixabay.com/download/audio/2023/03/01/audio_4a8b94a9d8.mp3?filename=ambient-gold-space-139248.mp3" loop></audio>

  <!-- Three.js -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.157.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.157.0/examples/jsm/controls/OrbitControls.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.157.0/examples/jsm/loaders/FontLoader.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.157.0/examples/jsm/geometries/TextGeometry.js"></script>

  <script>
  const scene = new THREE.Scene();
  scene.fog = new THREE.Fog(0x000000, 20, 120);
  const camera = new THREE.PerspectiveCamera(70, innerWidth/innerHeight, 0.1, 1000);
  camera.position.set(0, 2, 18);
  const renderer = new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(innerWidth, innerHeight);
  renderer.setPixelRatio(devicePixelRatio);
  renderer.shadowMap.enabled = true;
  document.body.appendChild(renderer.domElement);
  const controls = new THREE.OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;

  scene.add(new THREE.AmbientLight(0xffd700, 0.3));
  const pLight = new THREE.PointLight(0xffd700, 1.2, 200);
  pLight.position.set(10,15,10);
  pLight.castShadow = true;
  scene.add(pLight);

  function starField(count, size, area){
    const g = new THREE.BufferGeometry();
    const p = new Float32Array(count*3);
    for(let i=0;i<count;i++){
      p[i*3]=(Math.random()-0.5)*area;
      p[i*3+1]=(Math.random()-0.5)*area;
      p[i*3+2]=(Math.random()-0.5)*area;
    }
    g.setAttribute('position', new THREE.BufferAttribute(p,3));
    const m = new THREE.PointsMaterial({color:0xffffff,size});
    return new THREE.Points(g,m);
  }
  scene.add(starField(2000,0.6,400));

  const flowCount=1000;
  const flowGeo = new THREE.BufferGeometry();
  const flowPos = new Float32Array(flowCount*3);
  const flowSpeed = new Float32Array(flowCount);
  for(let i=0;i<flowCount;i++){
    flowPos[3*i]=(Math.random()-0.5)*120;
    flowPos[3*i+1]=Math.random()*80;
    flowPos[3*i+2]=(Math.random()-0.5)*120;
    flowSpeed[i]=Math.random()*0.02+0.005;
  }
  flowGeo.setAttribute('position', new THREE.BufferAttribute(flowPos,3));
  const flowMat = new THREE.PointsMaterial({color:0xffd700,size:0.4});
  const flowPts = new THREE.Points(flowGeo, flowMat);
  scene.add(flowPts);

  const fontLoader = new THREE.FontLoader();
  const sectionData = [
    {text:"ABOUT",  y:3, link:null},
    {text:"EXPERIENCE", y:0.5, link:null},
    {text:"SKILLS", y:-2, link:null},
    {text:"CONTACT", y:-4.5, link:"mailto:Zainhacker7777777@gmail.com"}
  ];
  const textMeshes=[];
  fontLoader.load('https://threejs.org/examples/fonts/helvetiker_bold.typeface.json', font=>{
    const mat = new THREE.MeshStandardMaterial({color:0xffd700,metalness:0.6,roughness:0.2});
    sectionData.forEach((sec,i)=>{
      const geo=new THREE.TextGeometry(sec.text,{font,size:1,depth:0.3,bevelEnabled:true,bevelThickness:0.02,bevelSize:0.02});
      geo.center();
      const mesh=new THREE.Mesh(geo,mat);
      mesh.position.set(i%2===0?-4:4,sec.y,0);
      mesh.userData.link=sec.link;
      mesh.castShadow=true;
      textMeshes.push(mesh);
      scene.add(mesh);
    });
  });

  const raycaster=new THREE.Raycaster();
  const mouse=new THREE.Vector2();
  const clicks=[];
  function spawnDust(pos){
    const g=new THREE.BufferGeometry();
    const count=200;
    const arr=new Float32Array(count*3);
    for(let i=0;i<count;i++){
      arr[3*i]=(Math.random()-0.5)*1;
      arr[3*i+1]=(Math.random()-0.5)*1;
      arr[3*i+2]=(Math.random()-0.5)*1;
    }
    g.setAttribute('position',new THREE.BufferAttribute(arr,3));
    const m=new THREE.PointsMaterial({color:0xffd700,size:0.2,transparent:true,opacity:1});
    const pts=new THREE.Points(g,m);
    pts.position.copy(pos);
    pts.userData.life=60;
    clicks.push(pts);
    scene.add(pts);
  }

  function onClick(e){
    mouse.x=(e.clientX/window.innerWidth)*2-1;
    mouse.y=-(e.clientY/window.innerHeight)*2+1;
    raycaster.setFromCamera(mouse,camera);
    const intersects=raycaster.intersectObjects(textMeshes,true);
    if(intersects.length){
      const obj=intersects[0].object;
      if(obj.userData.link){ window.open(obj.userData.link,'_blank'); }
      spawnDust(intersects[0].point);
    }else{
      const dir=raycaster.ray.direction.clone().multiplyScalar(10);
      const pos=raycaster.ray.origin.clone().add(dir);
      spawnDust(pos);
    }
  }
  window.addEventListener('click',onClick);

  const bgm=document.getElementById('bgm');
  const btn=document.getElementById('audioToggle');
  let playing=false;
  btn.addEventListener('click',()=>{
    if(!playing){bgm.play();btn.textContent='🔈';playing=true;}else{bgm.pause();btn.textContent='🔊';playing=false;}
  });

  function animate(){
    requestAnimationFrame(animate);
    textMeshes.forEach((m)=>{m.rotation.y+=0.005;});
    const p=flowGeo.attributes.position.array;
    for(let i=0;i<flowCount;i++){
      p[3*i+1]-=flowSpeed[i];
      if(p[3*i+1]<-40) p[3*i+1]=40;
    }
    flowGeo.attributes.position.needsUpdate=true;
    for(let i=clicks.length-1;i>=0;i--){
      const sys=clicks[i];
      sys.rotation.y+=0.05;
      sys.userData.life--;
      sys.material.opacity=Math.max(sys.userData.life/60,0);
      if(sys.userData.life<=0){scene.remove(sys);clicks.splice(i,1);}
    }
    controls.update();
    renderer.render(scene,camera);
  }
  animate();

  onresize=function(){
    camera.aspect=innerWidth/innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(innerWidth,innerHeight);
  }
  </script>
</body>
</html>
