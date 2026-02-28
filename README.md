<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Inventario Online con Dashboard</title>

<!-- Firebase -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getFirestore, collection, addDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

window.firebaseModules = { initializeApp, getFirestore, collection, addDoc, onSnapshot };
</script>

<!-- Charts -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:#f4f6f9;
}
header{
  background:#1f2937;
  color:white;
  padding:15px;
  text-align:center;
}
.container{
  padding:20px;
  max-width:1100px;
  margin:auto;
}
.card{
  background:white;
  padding:15px;
  margin-bottom:15px;
  border-radius:8px;
  box-shadow:0 2px 6px rgba(0,0,0,.1);
}
.grid{
  display:grid;
  grid-template-columns:1fr 1fr;
  gap:15px;
}
input, select{
  width:100%;
  padding:8px;
  margin:5px 0;
}
button{
  padding:8px 12px;
  background:#2563eb;
  color:white;
  border:none;
  border-radius:5px;
  cursor:pointer;
}
button:hover{
  background:#1e40af;
}
table{
  width:100%;
  border-collapse:collapse;
}
th, td{
  padding:8px;
  border-bottom:1px solid #ddd;
}
.stat{
  font-size:28px;
  font-weight:bold;
}
.stats{
  display:grid;
  grid-template-columns:repeat(auto-fit,minmax(180px,1fr));
  gap:15px;
}
.stat-box{
  background:#2563eb;
  color:white;
  padding:15px;
  border-radius:10px;
  text-align:center;
}
</style>
</head>

<body>

<header>
<h2>Sistema de Inventario Online</h2>
</header>

<div class="container">

<!-- DASHBOARD -->
<div class="card">
<h3>Dashboard</h3>
<div class="stats">
  <div class="stat-box">
    Total Equipos
    <div class="stat" id="totalEquipos">0</div>
  </div>
  <div class="stat-box">
    Disponibles
    <div class="stat" id="disponibles">0</div>
  </div>
  <div class="stat-box">
    Asignados
    <div class="stat" id="asignados">0</div>
  </div>
</div>
</div>

<!-- FORM -->
<div class="card">
<h3>Registrar Equipo</h3>
<input id="asset" placeholder="Asset ID">
<input id="tipo" placeholder="Tipo">
<select id="estado">
  <option value="DISPONIBLE">DISPONIBLE</option>
  <option value="ASIGNADO">ASIGNADO</option>
  <option value="REPARACION">REPARACIÓN</option>
</select>
<input id="marca" placeholder="Marca">
<input id="modelo" placeholder="Modelo">
<button onclick="guardarEquipo()">Guardar Equipo</button>
</div>

<!-- TABLA -->
<div class="card">
<h3>Inventario</h3>
<table>
<thead>
<tr>
<th>Asset</th>
<th>Tipo</th>
<th>Estado</th>
<th>Marca</th>
<th>Modelo</th>
</tr>
</thead>
<tbody id="tabla"></tbody>
</table>
</div>

<!-- GRAFICAS -->
<div class="card">
<h3>Estadísticas</h3>
<div class="grid">
  <canvas id="chartEstado"></canvas>
  <canvas id="chartTipo"></canvas>
</div>
</div>

</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getFirestore, collection, addDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

/* 🔥 PEGA TU CONFIGURACIÓN AQUÍ 🔥 */
const firebaseConfig = {
  apiKey: "AQUI_TU_CONFIG",
  authDomain: "AQUI_TU_CONFIG",
  projectId: "AQUI_TU_CONFIG",
  storageBucket: "AQUI_TU_CONFIG",
  messagingSenderId: "AQUI_TU_CONFIG",
  appId: "AQUI_TU_CONFIG"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const ref = collection(db, "assets");

let chartEstado, chartTipo;

window.guardarEquipo = async ()=>{
  await addDoc(ref,{
    asset:asset.value,
    tipo:tipo.value,
    estado:estado.value,
    marca:marca.value,
    modelo:modelo.value
  });
  asset.value=tipo.value=marca.value=modelo.value="";
};

onSnapshot(ref, snap=>{
  let html="";
  let estados={}, tipos={};
  let total=0, disp=0, asig=0;

  snap.forEach(doc=>{
    const a=doc.data();
    total++;
    if(a.estado==="DISPONIBLE") disp++;
    if(a.estado==="ASIGNADO") asig++;

    html+=`<tr>
      <td>${a.asset}</td>
      <td>${a.tipo}</td>
      <td>${a.estado}</td>
      <td>${a.marca}</td>
      <td>${a.modelo}</td>
    </tr>`;

    estados[a.estado]=(estados[a.estado]||0)+1;
    tipos[a.tipo]=(tipos[a.tipo]||0)+1;
  });

  tabla.innerHTML=html;
  totalEquipos.textContent=total;
  disponibles.textContent=disp;
  asignados.textContent=asig;

  if(chartEstado) chartEstado.destroy();
  if(chartTipo) chartTipo.destroy();

  chartEstado=new Chart(document.getElementById("chartEstado"),{
    type:"pie",
    data:{labels:Object.keys(estados),datasets:[{data:Object.values(estados)}]}
  });

  chartTipo=new Chart(document.getElementById("chartTipo"),{
    type:"bar",
    data:{labels:Object.keys(tipos),datasets:[{data:Object.values(tipos)}]}
  });
});
</script>

</body>
</html>
