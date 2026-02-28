<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Sistema de Inventario</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

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
}
.card{
  background:white;
  padding:15px;
  border-radius:8px;
  margin-bottom:15px;
  box-shadow:0 2px 5px rgba(0,0,0,.1);
}
input, select, textarea{
  width:100%;
  padding:8px;
  margin:5px 0;
}
button{
  padding:8px 12px;
  margin:5px 0;
  cursor:pointer;
  background:#2563eb;
  color:white;
  border:none;
  border-radius:5px;
}
button:hover{
  background:#1e40af;
}
.hidden{
  display:none;
}
table{
  width:100%;
  border-collapse:collapse;
}
th, td{
  padding:8px;
  border-bottom:1px solid #ddd;
  text-align:left;
}
</style>
</head>

<body>

<header>
<h2>Sistema de Inventario</h2>
</header>

<div class="container">

<div id="loginBox" class="card">
<h3>Login</h3>
<input type="password" id="password" placeholder="Contraseña">
<button onclick="login()">Ingresar</button>
</div>

<div id="mainApp" class="hidden">

<div class="card">
<h3>Crear / Editar Equipo</h3>
<input id="asset_tag" placeholder="Asset ID">
<input id="tipo" placeholder="Tipo">
<input id="estado" placeholder="Estado">
<input id="marca" placeholder="Marca">
<input id="modelo" placeholder="Modelo">
<input id="serial" placeholder="Serial">
<input id="imei" placeholder="IMEI">
<textarea id="notas" placeholder="Notas"></textarea>
<button onclick="saveAsset()">Guardar</button>
<button onclick="deleteAsset()">Eliminar</button>
</div>

<div class="card">
<h3>Asignación</h3>
<input id="as_asset" placeholder="Asset ID">
<input id="as_person" placeholder="Persona">
<input id="as_doc" placeholder="Documento">
<input id="as_loc" placeholder="Ubicación">
<textarea id="as_notes" placeholder="Notas"></textarea>
<button onclick="assignAsset()">Asignar</button>
<button onclick="returnAsset()">Devolver</button>
</div>

<div class="card">
<h3>Inventario</h3>
<table>
<thead>
<tr>
<th>Asset</th>
<th>Tipo</th>
<th>Estado</th>
<th>Asignado</th>
</tr>
</thead>
<tbody id="assetTable"></tbody>
</table>
</div>

<div class="card">
<h3>Exportar / Importar</h3>
<button onclick="exportData()">Exportar</button>
<input type="file" id="importFile" class="hidden">
<button onclick="document.getElementById('importFile').click()">Importar</button>
</div>

<div class="card">
<h3>Detalle</h3>
<div id="detailBox"></div>
<div id="qr"></div>
<button onclick="downloadQR()">Descargar QR</button>
<button onclick="printLabel()">Etiqueta PDF</button>
<button onclick="reportPDF()">Reporte PDF</button>
</div>

</div>

</div>

<script>
let DB = JSON.parse(localStorage.getItem("inventario")) || {assets:[]};
let selected = null;

function saveDB(){
  localStorage.setItem("inventario", JSON.stringify(DB));
}

function login(){
  if(document.getElementById("password").value==="admin"){
    document.getElementById("loginBox").classList.add("hidden");
    document.getElementById("mainApp").classList.remove("hidden");
    render();
  } else{
    alert("Contraseña incorrecta");
  }
}

function saveAsset(){
  let tag = document.getElementById("asset_tag").value;
  if(!tag) return alert("Asset ID requerido");

  let asset = {
    asset_tag: tag,
    tipo: tipo.value,
    estado: estado.value,
    marca: marca.value,
    modelo: modelo.value,
    serial: serial.value,
    imei: imei.value,
    notas: notas.value,
    assigned:false,
    history:[]
  };

  let index = DB.assets.findIndex(a=>a.asset_tag===tag);
  if(index>=0) DB.assets[index]=asset;
  else DB.assets.push(asset);

  saveDB();
  render();
}

function deleteAsset(){
  let tag = asset_tag.value;
  DB.assets = DB.assets.filter(a=>a.asset_tag!==tag);
  saveDB();
  render();
}

function assignAsset(){
  let tag = as_asset.value;
  let asset = DB.assets.find(a=>a.asset_tag===tag);
  if(!asset) return alert("No existe");
  asset.assigned=true;
  asset.history.push({
    person:as_person.value,
    doc:as_doc.value,
    loc:as_loc.value,
    notes:as_notes.value,
    date:new Date().toLocaleString()
  });
  saveDB();
  render();
  receiptPDF(asset);
}

function returnAsset(){
  let tag = as_asset.value;
  let asset = DB.assets.find(a=>a.asset_tag===tag);
  if(!asset) return;
  asset.assigned=false;
  saveDB();
  render();
}

function render(){
  let table="";
  DB.assets.forEach(a=>{
    table+=`<tr onclick="showDetail('${a.asset_tag}')">
    <td>${a.asset_tag}</td>
    <td>${a.tipo}</td>
    <td>${a.estado}</td>
    <td>${a.assigned?"Sí":"No"}</td>
    </tr>`;
  });
  document.getElementById("assetTable").innerHTML=table;
}

function showDetail(tag){
  selected = DB.assets.find(a=>a.asset_tag===tag);
  document.getElementById("detailBox").innerHTML=`
  <b>Asset:</b> ${selected.asset_tag}<br>
  <b>Marca:</b> ${selected.marca}<br>
  <b>Modelo:</b> ${selected.modelo}<br>
  <b>Serial:</b> ${selected.serial}
  `;
  document.getElementById("qr").innerHTML="";
  new QRCode(document.getElementById("qr"), tag);
}

function exportData(){
  let blob = new Blob([JSON.stringify(DB,null,2)],{type:"application/json"});
  let a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download="backup.json";
  a.click();
}

document.getElementById("importFile").addEventListener("change", e=>{
  let reader=new FileReader();
  reader.onload=()=>{
    DB=JSON.parse(reader.result);
    saveDB();
    render();
  };
  reader.readAsText(e.target.files[0]);
});

function downloadQR(){
  let canvas=document.querySelector("#qr canvas");
  let a=document.createElement("a");
  a.href=canvas.toDataURL();
  a.download="qr.png";
  a.click();
}

function printLabel(){
  const {jsPDF}=window.jspdf;
  let doc=new jsPDF();
  doc.text("Etiqueta Equipo",20,20);
  doc.text("Asset: "+selected.asset_tag,20,30);
  doc.save("etiqueta.pdf");
}

function reportPDF(){
  const {jsPDF}=window.jspdf;
  let doc=new jsPDF();
  doc.text("Reporte Equipo",20,20);
  doc.text("Asset: "+selected.asset_tag,20,30);
  doc.text("Marca: "+selected.marca,20,40);
  doc.text("Modelo: "+selected.modelo,20,50);
  doc.text("Serial: "+selected.serial,20,60);
  doc.save("reporte.pdf");
}

function receiptPDF(asset){
  const {jsPDF}=window.jspdf;
  let doc=new jsPDF();
  doc.text("Recibo Asignación",20,20);
  doc.text("Asset: "+asset.asset_tag,20,30);
  doc.text("Asignado a: "+asset.history.at(-1).person,20,40);
  doc.save("recibo.pdf");
}
</script>

</body>
</html>
