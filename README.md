<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Inventario y Asignación de Equipos (QR + Adjuntos + PDF Pro)</title>

  <!-- QR + PDF -->
  <script src="https://cdn.jsdelivr.net/npm/qrcode@1.5.3/build/qrcode.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>

  <style>
    :root{
      --bg:#0b1220; --card:#111a2e; --muted:#93a4c7; --text:#e8f0ff; --line:#223054;
      --brand:#4f8cff; --good:#39d98a; --bad:#ff4d4d; --warn:#ffcc00;
    }
    *{box-sizing:border-box}
    body{margin:0;background:linear-gradient(180deg,#081022, #0b1220); color:var(--text); font:14px/1.4 system-ui, -apple-system, Segoe UI, Roboto, Arial;}
    a{color:var(--brand)}
    .wrap{max-width:1200px;margin:0 auto;padding:22px;}
    .topbar{display:flex;gap:12px;align-items:center;justify-content:space-between;margin-bottom:14px}
    .brand{display:flex;align-items:center;gap:10px}
    .logo{width:36px;height:36px;border-radius:10px;background:rgba(79,140,255,.2);display:grid;place-items:center;border:1px solid rgba(79,140,255,.35)}
    .logo b{color:var(--brand)}
    .card{background:rgba(17,26,46,.9);border:1px solid var(--line);border-radius:16px;padding:14px;box-shadow:0 10px 30px rgba(0,0,0,.25)}
    .grid{display:grid;gap:12px}
    .grid2{grid-template-columns: 1fr 1fr}
    .grid3{grid-template-columns: 1fr 1fr 1fr}
    .row{display:flex;gap:10px;flex-wrap:wrap;align-items:center}
    .muted{color:var(--muted)}
    .pill{display:inline-flex;gap:8px;align-items:center;padding:6px 10px;border-radius:999px;border:1px solid var(--line);background:rgba(255,255,255,.03)}
    .btn{cursor:pointer;border:1px solid var(--line);background:rgba(255,255,255,.04);color:var(--text);padding:9px 12px;border-radius:12px}
    .btn:hover{border-color:rgba(79,140,255,.55)}
    .btn.primary{background:rgba(79,140,255,.18);border-color:rgba(79,140,255,.5)}
    .btn.good{background:rgba(57,217,138,.14);border-color:rgba(57,217,138,.45)}
    .btn.bad{background:rgba(255,77,77,.12);border-color:rgba(255,77,77,.4)}
    .btn.warn{background:rgba(255,204,0,.12);border-color:rgba(255,204,0,.4)}
    .btn.small{padding:6px 9px;border-radius:10px;font-size:12px}
    input, select, textarea{
      width:100%; padding:9px 10px;border-radius:12px;border:1px solid var(--line);
      background:rgba(255,255,255,.03); color:var(--text); outline:none;
    }
    textarea{min-height:74px;resize:vertical}
    label{display:block;margin:8px 0 6px;color:var(--muted);font-size:12px}
    .table{width:100%;border-collapse:collapse;overflow:hidden;border-radius:14px;border:1px solid var(--line)}
    .table th,.table td{padding:10px;border-bottom:1px solid var(--line);vertical-align:top}
    .table th{color:var(--muted);font-weight:600;background:rgba(255,255,255,.02);text-align:left}
    .table tr:hover td{background:rgba(255,255,255,.02)}
    .hidden{display:none !important}
    .sep{height:1px;background:var(--line);margin:12px 0}
    .right{margin-left:auto}
    .mono{font-family:ui-monospace,SFMono-Regular,Menlo,Monaco,Consolas,monospace}
    .status{font-weight:700}
    .status.disponible{color:var(--good)}
    .status.asignado{color:var(--warn)}
    .status.reparacion{color:#8bd3ff}
    .status.baja{color:var(--bad)}
    .toast{position:fixed;bottom:18px;left:50%;transform:translateX(-50%);background:#0e1830;border:1px solid var(--line);
      padding:10px 12px;border-radius:14px;box-shadow:0 10px 30px rgba(0,0,0,.35);max-width:92vw;z-index:9999}
    .kbd{border:1px solid var(--line);padding:2px 6px;border-radius:8px;background:rgba(255,255,255,.03);font-size:12px}
    .mini{font-size:12px;color:var(--muted)}
    .tabs{display:flex;gap:8px;flex-wrap:wrap}
    .tab{padding:8px 10px;border-radius:12px;border:1px solid var(--line);background:rgba(255,255,255,.02);cursor:pointer}
    .tab.active{border-color:rgba(79,140,255,.55);background:rgba(79,140,255,.14)}
    @media (max-width:920px){ .grid2,.grid3{grid-template-columns:1fr} #appGrid{grid-template-columns:1fr !important;} }
  </style>
</head>

<body>
<div class="wrap">

  <div class="topbar">
    <div class="brand">
      <div class="logo"><b>AI</b></div>
      <div>
        <div style="font-weight:800">Inventario & Asignación de Equipos</div>
        <div class="mini">1 archivo (LocalStorage) · QR por equipo · Adjuntos · PDF Presentable</div>
      </div>
    </div>

    <div class="row" id="userBar" style="gap:8px">
      <span class="pill hidden" id="sessionPill"></span>
      <button class="btn small hidden" id="logoutBtn">Salir</button>
      <button class="btn small warn" id="resetBtn" title="Borra datos del navegador">Reset</button>
    </div>
  </div>

  <!-- LOGIN -->
  <div class="card" id="loginCard">
    <div class="row" style="justify-content:space-between;align-items:flex-end">
      <div>
        <div style="font-weight:800;font-size:18px">Login</div>
        <div class="muted">Usuarios por defecto: <span class="mono">admin/admin123</span> y <span class="mono">user/user123</span></div>
      </div>
    </div>
    <div class="sep"></div>
    <div class="grid grid2">
      <div>
        <label>Usuario</label>
        <input id="loginUser" placeholder="admin" autocomplete="username" />
      </div>
      <div>
        <label>Contraseña</label>
        <input id="loginPass" placeholder="admin123" type="password" autocomplete="current-password" />
      </div>
    </div>
    <div class="row" style="margin-top:12px">
      <button class="btn primary" id="loginBtn">Entrar</button>
      <span class="muted">Tip: abre con <span class="kbd">?asset=PC-000001</span> para ir directo a la ficha</span>
    </div>
  </div>

  <!-- APP -->
  <div class="grid hidden" id="appGrid" style="grid-template-columns: 380px 1fr; align-items:start; gap:12px">

    <!-- LEFT -->
    <div class="grid">
      <div class="card">
        <div class="row">
          <div>
            <div style="font-weight:800">Búsqueda rápida</div>
            <div class="mini">Busca por Asset ID o Serial/IMEI</div>
          </div>
        </div>
        <div class="sep"></div>
        <div class="row">
          <input id="searchBox" placeholder="Ej: PC-000123 o SERIAL123" />
          <button class="btn primary" id="searchBtn">Buscar</button>
        </div>
        <div class="mini" style="margin-top:8px">Atajo: <span class="kbd">Enter</span></div>
      </div>

      <!-- ADMIN -->
      <div class="card" id="adminOnlyCard">
        <div class="row">
          <div>
            <div style="font-weight:800">Acciones (ADMIN)</div>
            <div class="mini">Crear equipo, asignar, adjuntos</div>
          </div>
        </div>

        <div class="sep"></div>

        <div class="tabs" id="leftTabs">
          <div class="tab active" data-tab="asset">Equipo</div>
          <div class="tab" data-tab="assign">Asignar</div>
          <div class="tab" data-tab="attach">Adjuntos</div>
        </div>

        <!-- Equipo -->
        <div id="tab-asset" style="margin-top:10px">
          <div class="mini muted">Crea o edita un equipo. El Asset ID debe ser único.</div>
          <label>Asset ID (aidi)</label>
          <input id="a_asset_tag" placeholder="PC-000001" />
          <div class="grid grid2">
            <div>
              <label>Tipo</label>
              <select id="a_tipo">
                <option>Computador</option>
                <option>Celular</option>
                <option>Monitor</option>
                <option>Impresora</option>
                <option>Otro</option>
              </select>
            </div>
            <div>
              <label>Estado</label>
              <select id="a_estado">
                <option value="DISPONIBLE">DISPONIBLE</option>
                <option value="ASIGNADO">ASIGNADO</option>
                <option value="REPARACION">REPARACION</option>
                <option value="BAJA">BAJA</option>
              </select>
            </div>
          </div>
          <div class="grid grid2">
            <div>
              <label>Marca</label>
              <input id="a_marca" placeholder="Dell / Samsung" />
            </div>
            <div>
              <label>Modelo</label>
              <input id="a_modelo" placeholder="Latitude / A54" />
            </div>
          </div>
          <div class="grid grid2">
            <div>
              <label>Serial</label>
              <input id="a_serial" placeholder="SERIAL123" />
            </div>
            <div>
              <label>IMEI (si aplica)</label>
              <input id="a_imei" placeholder="35xxxxxxxxxxxxx" />
            </div>
          </div>
          <label>Observaciones</label>
          <textarea id="a_notas" placeholder="Detalles, estado físico, accesorios..."></textarea>

          <div class="row" style="margin-top:10px">
            <button class="btn good" id="saveAssetBtn">Guardar equipo</button>
            <button class="btn" id="newAssetBtn">Nuevo</button>
            <button class="btn bad right" id="deleteAssetBtn">Eliminar</button>
          </div>
        </div>

        <!-- Asignar -->
        <div id="tab-assign" class="hidden" style="margin-top:10px">
          <div class="mini muted">Asigna un equipo a una persona y ubicación (queda en historial).</div>
          <label>Asset ID</label>
          <input id="as_asset_tag" placeholder="PC-000001" />
          <label>Persona (nombre)</label>
          <input id="as_person" placeholder="Juan Pérez" />
          <label>Documento (opcional)</label>
          <input id="as_doc" placeholder="CC 123..." />
          <label>Ubicación</label>
          <input id="as_loc" placeholder="Sede Centro - Piso 2 - Oficina 201" />
          <label>Notas / accesorios</label>
          <textarea id="as_notes" placeholder="Cargador, mouse, bolso..."></textarea>

          <div class="row" style="margin-top:10px">
            <button class="btn good" id="assignBtn">Asignar + PDF</button>
            <button class="btn warn right" id="returnBtn">Marcar devolución</button>
          </div>
          <div class="mini" style="margin-top:10px">
            “Asignar + PDF” genera un recibo profesional automáticamente.
          </div>
        </div>

        <!-- Adjuntos -->
        <div id="tab-attach" class="hidden" style="margin-top:10px">
          <div class="mini muted">Sube fotos/actas/facturas. Se guardan en el navegador.</div>
          <label>Asset ID</label>
          <input id="att_asset_tag" placeholder="PC-000001" />
          <label>Tipo de adjunto</label>
          <select id="att_type">
            <option value="FOTO_EQUIPO">FOTO_EQUIPO</option>
            <option value="FOTO_SERIAL">FOTO_SERIAL</option>
            <option value="ACTA_ENTREGA">ACTA_ENTREGA</option>
            <option value="ACTA_DEVOLUCION">ACTA_DEVOLUCION</option>
            <option value="FACTURA">FACTURA</option>
            <option value="GARANTIA">GARANTIA</option>
            <option value="OTRO">OTRO</option>
          </select>
          <label>Descripción</label>
          <input id="att_desc" placeholder="Ej: Foto frontal / Factura abril 2025" />
          <label>Archivo (jpg/png/pdf)</label>
          <input id="att_file" type="file" accept=".jpg,.jpeg,.png,.pdf" />
          <div class="row" style="margin-top:10px">
            <button class="btn good" id="uploadAttBtn">Subir adjunto</button>
          </div>
          <div id="attPreview" class="mini" style="margin-top:10px"></div>
        </div>
      </div>

      <!-- Export/Import -->
      <div class="card">
        <div class="row">
          <div>
            <div style="font-weight:800">Datos</div>
            <div class="mini">Exporta / Importa para mover a otro PC</div>
          </div>
        </div>
        <div class="sep"></div>
        <div class="row">
          <button class="btn" id="exportBtn">Exportar JSON</button>
          <button class="btn" id="importBtn">Importar JSON</button>
          <input id="importFile" type="file" accept=".json" class="hidden" />
        </div>
        <div class="mini muted" style="margin-top:10px">
          Esto te sirve como “mini base de datos” portátil (del navegador).
        </div>
      </div>
    </div>

    <!-- RIGHT -->
    <div class="grid">
      <div class="card">
        <div class="row">
          <div>
            <div style="font-weight:800">Equipos</div>
            <div class="mini">Inventario total y estado</div>
          </div>
          <div class="right row" style="gap:8px">
            <select id="filterEstado" style="width:auto">
              <option value="ALL">Todos</option>
              <option value="DISPONIBLE">DISPONIBLE</option>
              <option value="ASIGNADO">ASIGNADO</option>
              <option value="REPARACION">REPARACION</option>
              <option value="BAJA">BAJA</option>
            </select>
            <button class="btn" id="refreshBtn">Refrescar</button>
          </div>
        </div>
        <div class="sep"></div>

        <table class="table" id="assetsTable">
          <thead>
            <tr>
              <th>Asset ID</th>
              <th>Tipo</th>
              <th>Marca/Modelo</th>
              <th>Serial/IMEI</th>
              <th>Estado</th>
              <th>Asignado a</th>
              <th></th>
            </tr>
          </thead>
          <tbody></tbody>
        </table>

        <div class="mini muted" style="margin-top:10px">
          Click en “Ver” para abrir ficha y QR.
        </div>
      </div>

      <div class="card" id="detailCard">
        <div class="row">
          <div>
            <div style="font-weight:800">Ficha del equipo</div>
            <div class="mini">QR, asignación actual, historial y adjuntos</div>
          </div>
          <div class="right row" style="gap:8px">
            <button class="btn primary" id="printLabelBtn">Etiqueta PDF (QR)</button>
            <button class="btn" id="reportBtn">Reporte PDF (Pro)</button>
          </div>
        </div>
        <div class="sep"></div>

        <div id="detailEmpty" class="muted">Busca o selecciona un equipo para ver su ficha.</div>
        <div id="detailBody" class="hidden">
          <div class="grid grid3">
            <div class="card" style="padding:12px">
              <div class="mini muted">Identificación</div>
              <div style="font-weight:800;font-size:16px" id="d_asset_tag" class="mono"></div>
              <div class="mini" id="d_type"></div>
              <div class="mini muted" id="d_make"></div>
              <div class="mini muted" id="d_serial"></div>
              <div class="mini muted" id="d_imei"></div>
              <div class="mini" style="margin-top:6px">Estado: <span id="d_status" class="status"></span></div>
            </div>

            <div class="card" style="padding:12px">
              <div class="mini muted">Asignación actual</div>
              <div id="d_currentAssign" class="mini">—</div>
              <div class="sep"></div>
              <div class="mini muted">Notas del equipo</div>
              <div id="d_notes" class="mini">—</div>
            </div>

            <div class="card" style="padding:12px">
              <div class="mini muted">QR (abre esta ficha)</div>
              <canvas id="qrCanvas" style="width:100%;max-width:220px;background:#fff;border-radius:12px"></canvas>
              <div class="row" style="margin-top:10px">
                <button class="btn small" id="copyLinkBtn">Copiar link</button>
                <button class="btn small" id="downloadQRBtn">Descargar QR</button>
              </div>
              <div class="mini muted" id="d_link" style="margin-top:8px;word-break:break-all"></div>
            </div>
          </div>

          <div class="sep"></div>

          <div class="grid grid2">
            <div>
              <div style="font-weight:800">Historial de asignaciones</div>
              <div class="mini muted">Quién lo tuvo y dónde estuvo</div>
              <div class="sep"></div>
              <table class="table" id="histTable">
                <thead>
                  <tr>
                    <th>Fecha</th>
                    <th>Persona</th>
                    <th>Ubicación</th>
                    <th>Estado</th>
                  </tr>
                </thead>
                <tbody></tbody>
              </table>
            </div>

            <div>
              <div style="font-weight:800">Adjuntos</div>
              <div class="mini muted">Fotos, actas, factura, garantía…</div>
              <div class="sep"></div>
              <table class="table" id="attTable">
                <thead>
                  <tr>
                    <th>Tipo</th>
                    <th>Descripción</th>
                    <th>Fecha</th>
                    <th></th>
                  </tr>
                </thead>
                <tbody></tbody>
              </table>
              <div class="mini muted" style="margin-top:10px">
                Para subir adjuntos: pestaña <b>Adjuntos</b> (ADMIN).
              </div>
            </div>
          </div>

        </div>
      </div>

    </div>
  </div>

</div>

<div id="toast" class="toast hidden"></div>

<script>
/* =========================
   1) Storage + Helpers
========================= */
const LS_KEY = "inv_asg_v1";
const nowISO = () => new Date().toISOString();
const fmtDate = (iso) => {
  if(!iso) return "—";
  const d = new Date(iso);
  return d.toLocaleString("es-CO", {year:"numeric",month:"2-digit",day:"2-digit",hour:"2-digit",minute:"2-digit"});
};
const uid = () => Math.random().toString(36).slice(2) + Date.now().toString(36);

function toast(msg){
  const el = document.getElementById("toast");
  el.textContent = msg;
  el.classList.remove("hidden");
  setTimeout(()=> el.classList.add("hidden"), 2600);
}

function loadDB(){
  const raw = localStorage.getItem(LS_KEY);
  if(raw){
    try{ return JSON.parse(raw); }catch(e){}
  }
  // seed
  const db = {
    users: [
      {id:"u_admin", username:"admin", password:"admin123", role:"ADMIN", name:"Administrador", active:true},
      {id:"u_user", username:"user", password:"user123", role:"USER", name:"Usuario", active:true},
    ],
    assets: [
      {id: uid(), asset_tag:"PC-000001", tipo:"Computador", marca:"Dell", modelo:"Latitude", serial:"SERIAL-001", imei:"", estado:"DISPONIBLE", notas:"Equipo de prueba", created_at: nowISO()},
    ],
    assignments: [],
    attachments: [],
    session: { userId: null }
  };
  saveDB(db);
  return db;
}
function saveDB(db){ localStorage.setItem(LS_KEY, JSON.stringify(db)); }
let DB = loadDB();

function currentUser(){
  const id = DB.session.userId;
  return DB.users.find(u=>u.id===id) || null;
}
function isAdmin(){
  const u = currentUser();
  return u && u.role==="ADMIN";
}
function requireLogin(){
  const u = currentUser();
  if(!u){
    document.getElementById("loginCard").classList.remove("hidden");
    document.getElementById("appGrid").classList.add("hidden");
    document.getElementById("sessionPill").classList.add("hidden");
    document.getElementById("logoutBtn").classList.add("hidden");
    return false;
  }
  document.getElementById("loginCard").classList.add("hidden");
  document.getElementById("appGrid").classList.remove("hidden");

  const pill = document.getElementById("sessionPill");
  pill.textContent = `${u.name} · ${u.role}`;
  pill.classList.remove("hidden");
  document.getElementById("logoutBtn").classList.remove("hidden");

  document.getElementById("adminOnlyCard").classList.toggle("hidden", !isAdmin());
  return true;
}

/* =========================
   2) Core logic
========================= */
function normalizeTag(tag){ return (tag||"").trim().toUpperCase(); }

function getAsset(tag){
  tag = normalizeTag(tag);
  return DB.assets.find(a=>a.asset_tag.toUpperCase()===tag) || null;
}

function currentAssignment(tag){
  tag = normalizeTag(tag);
  const list = DB.assignments
    .filter(x=>x.asset_tag.toUpperCase()===tag)
    .sort((a,b)=> (a.assigned_at>b.assigned_at? -1:1));
  return list.find(x=>!x.returned_at) || null;
}

function assetAssignedTo(tag){
  const asg = currentAssignment(tag);
  if(!asg) return "—";
  return `${asg.person} · ${asg.location}`;
}

function upsertAsset(payload){
  const tag = normalizeTag(payload.asset_tag);
  if(!tag) throw new Error("Asset ID es obligatorio");
  const existing = getAsset(tag);

  if(!existing){
    if(DB.assets.some(a=>a.asset_tag.toUpperCase()===tag)) throw new Error("Asset ID ya existe");
    DB.assets.push({
      id: uid(),
      asset_tag: tag,
      tipo: payload.tipo || "Otro",
      marca: payload.marca || "",
      modelo: payload.modelo || "",
      serial: payload.serial || "",
      imei: payload.imei || "",
      estado: payload.estado || "DISPONIBLE",
      notas: payload.notas || "",
      created_at: nowISO(),
    });
  } else {
    existing.tipo = payload.tipo || existing.tipo;
    existing.marca = payload.marca ?? existing.marca;
    existing.modelo = payload.modelo ?? existing.modelo;
    existing.serial = payload.serial ?? existing.serial;
    existing.imei = payload.imei ?? existing.imei;
    existing.estado = payload.estado || existing.estado;
    existing.notas = payload.notas ?? existing.notas;
  }
  saveDB(DB);
}

function deleteAsset(tag){
  tag = normalizeTag(tag);
  const asset = getAsset(tag);
  if(!asset) return;
  DB.assets = DB.assets.filter(a=>a.asset_tag.toUpperCase()!==tag);
  DB.assignments = DB.assignments.filter(x=>x.asset_tag.toUpperCase()!==tag);
  DB.attachments = DB.attachments.filter(x=>x.asset_tag.toUpperCase()!==tag);
  saveDB(DB);
}

function assignAsset({asset_tag, person, doc, location, notes}){
  if(!isAdmin()) throw new Error("Solo ADMIN puede asignar");
  asset_tag = normalizeTag(asset_tag);
  if(!asset_tag) throw new Error("Asset ID es obligatorio");
  const asset = getAsset(asset_tag);
  if(!asset) throw new Error("No existe ese equipo");
  if(currentAssignment(asset_tag)) throw new Error("Ese equipo ya está ASIGNADO (marca devolución primero)");
  if(!person.trim()) throw new Error("Persona es obligatoria");
  if(!location.trim()) throw new Error("Ubicación es obligatoria");

  DB.assignments.push({
    id: uid(),
    asset_tag,
    person: person.trim(),
    doc: (doc||"").trim(),
    location: location.trim(),
    notes: (notes||"").trim(),
    assigned_at: nowISO(),
    returned_at: null,
    created_by: currentUser().username
  });

  asset.estado = "ASIGNADO";
  saveDB(DB);
}

function returnAsset(asset_tag){
  if(!isAdmin()) throw new Error("Solo ADMIN puede devolver");
  asset_tag = normalizeTag(asset_tag);
  const asset = getAsset(asset_tag);
  if(!asset) throw new Error("No existe ese equipo");
  const asg = currentAssignment(asset_tag);
  if(!asg) throw new Error("Ese equipo no está asignado");
  asg.returned_at = nowISO();
  asset.estado = "DISPONIBLE";
  saveDB(DB);
}

function addAttachment({asset_tag, type, desc, file}){
  if(!isAdmin()) throw new Error("Solo ADMIN puede subir adjuntos");
  asset_tag = normalizeTag(asset_tag);
  const asset = getAsset(asset_tag);
  if(!asset) throw new Error("No existe ese equipo");

  return new Promise((resolve, reject)=>{
    const reader = new FileReader();
    reader.onload = () => {
      const dataUrl = reader.result;
      DB.attachments.push({
        id: uid(),
        asset_tag,
        type,
        desc: (desc||"").trim(),
        filename: file.name,
        mime: file.type || "application/octet-stream",
        dataUrl,
        created_at: nowISO(),
        created_by: currentUser().username
      });
      saveDB(DB);
      resolve(true);
    };
    reader.onerror = () => reject(new Error("No se pudo leer el archivo"));
    reader.readAsDataURL(file);
  });
}

/* =========================
   3) UI Rendering
========================= */
let selectedTag = null;

function statusClass(estado){
  const e = (estado||"").toLowerCase();
  if(e.includes("disp")) return "disponible";
  if(e.includes("asig")) return "asignado";
  if(e.includes("rep")) return "reparacion";
  if(e.includes("baj")) return "baja";
  return "";
}

function renderAssets(){
  const tbody = document.querySelector("#assetsTable tbody");
  tbody.innerHTML = "";

  const filter = document.getElementById("filterEstado").value;
  const list = DB.assets
    .slice()
    .sort((a,b)=> a.asset_tag.localeCompare(b.asset_tag))
    .filter(a => filter==="ALL" ? true : a.estado===filter);

  for(const a of list){
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td class="mono">${a.asset_tag}</td>
      <td>${a.tipo}</td>
      <td>${(a.marca||"—")} ${(a.modelo||"")}</td>
      <td class="mini">${(a.serial||"—")}<br>${(a.imei||"")}</td>
      <td><span class="status ${statusClass(a.estado)}">${a.estado}</span></td>
      <td class="mini">${assetAssignedTo(a.asset_tag)}</td>
      <td style="white-space:nowrap">
        <button class="btn small primary" data-view="${a.asset_tag}">Ver</button>
        ${isAdmin() ? `<button class="btn small" data-edit="${a.asset_tag}">Editar</button>` : ``}
      </td>
    `;
    tbody.appendChild(tr);
  }

  tbody.querySelectorAll("[data-view]").forEach(btn=>{
    btn.addEventListener("click", ()=> openDetail(btn.getAttribute("data-view")));
  });
  tbody.querySelectorAll("[data-edit]").forEach(btn=>{
    btn.addEventListener("click", ()=> loadAssetToForm(btn.getAttribute("data-edit")));
  });
}

function openDetail(tag){
  tag = normalizeTag(tag);
  const asset = getAsset(tag);
  if(!asset){ toast("No existe ese equipo"); return; }
  selectedTag = tag;

  document.getElementById("detailEmpty").classList.add("hidden");
  document.getElementById("detailBody").classList.remove("hidden");

  document.getElementById("d_asset_tag").textContent = asset.asset_tag;
  document.getElementById("d_type").textContent = asset.tipo;
  document.getElementById("d_make").textContent = `${asset.marca || "—"} ${asset.modelo || ""}`.trim();
  document.getElementById("d_serial").textContent = `Serial: ${asset.serial || "—"}`;
  document.getElementById("d_imei").textContent = `IMEI: ${asset.imei || "—"}`;

  const st = document.getElementById("d_status");
  st.textContent = asset.estado;
  st.className = `status ${statusClass(asset.estado)}`;
  document.getElementById("d_notes").textContent = asset.notas || "—";

  const asg = currentAssignment(tag);
  document.getElementById("d_currentAssign").innerHTML = asg
    ? `<b>${asg.person}</b> ${asg.doc?`(${asg.doc})`:``}<br><span class="muted">${asg.location}</span><br><span class="mini muted">Asignado: ${fmtDate(asg.assigned_at)}</span>`
    : `<span class="muted">No asignado</span>`;

  const url = new URL(window.location.href);
  url.searchParams.set("asset", tag);
  const link = url.toString();
  document.getElementById("d_link").textContent = link;

  const canvas = document.getElementById("qrCanvas");
  QRCode.toCanvas(canvas, link, {margin:2, width:220}, (err)=>{
    if(err) console.error(err);
  });

  renderHistory(tag);
  renderAttachments(tag);

  if(isAdmin()){
    document.getElementById("as_asset_tag").value = tag;
    document.getElementById("att_asset_tag").value = tag;
  }
}

function renderHistory(tag){
  tag = normalizeTag(tag);
  const tbody = document.querySelector("#histTable tbody");
  tbody.innerHTML = "";

  const list = DB.assignments
    .filter(x=>x.asset_tag.toUpperCase()===tag)
    .sort((a,b)=> (a.assigned_at>b.assigned_at? -1:1));

  if(list.length===0){
    const tr = document.createElement("tr");
    tr.innerHTML = `<td colspan="4" class="muted">Sin historial</td>`;
    tbody.appendChild(tr);
    return;
  }

  for(const x of list){
    const tr = document.createElement("tr");
    const estado = x.returned_at ? "DEVUELTO" : "ASIGNADO";
    tr.innerHTML = `
      <td class="mini">${fmtDate(x.assigned_at)}${x.returned_at?`<br><span class="muted">Devuelto: ${fmtDate(x.returned_at)}</span>`:``}</td>
      <td>${x.person}<div class="mini muted">${x.doc||""}</div></td>
      <td class="mini">${x.location}</td>
      <td><span class="status ${x.returned_at? "disponible":"asignado"}">${estado}</span></td>
    `;
    tbody.appendChild(tr);
  }
}

function renderAttachments(tag){
  tag = normalizeTag(tag);
  const tbody = document.querySelector("#attTable tbody");
  tbody.innerHTML = "";

  const list = DB.attachments
    .filter(x=>x.asset_tag.toUpperCase()===tag)
    .sort((a,b)=> (a.created_at>b.created_at? -1:1));

  if(list.length===0){
    const tr = document.createElement("tr");
    tr.innerHTML = `<td colspan="4" class="muted">Sin adjuntos</td>`;
    tbody.appendChild(tr);
    return;
  }

  for(const x of list){
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td class="mono mini">${x.type}</td>
      <td class="mini">${(x.desc||"—")}<div class="muted">${x.filename}</div></td>
      <td class="mini">${fmtDate(x.created_at)}</td>
      <td style="white-space:nowrap">
        <button class="btn small primary" data-openatt="${x.id}">Abrir</button>
        ${isAdmin()?`<button class="btn small bad" data-delatt="${x.id}">Borrar</button>`:""}
      </td>
    `;
    tbody.appendChild(tr);
  }

  tbody.querySelectorAll("[data-openatt]").forEach(btn=>{
    btn.addEventListener("click", ()=> openAttachment(btn.getAttribute("data-openatt")));
  });
  tbody.querySelectorAll("[data-delatt]").forEach(btn=>{
    btn.addEventListener("click", ()=> deleteAttachment(btn.getAttribute("data-delatt")));
  });
}

function openAttachment(id){
  const a = DB.attachments.find(x=>x.id===id);
  if(!a) return;
  const w = window.open();
  if(!w){ toast("Permite popups para abrir adjunto"); return; }
  if(a.mime.startsWith("image/")){
    w.document.write(`<title>${a.filename}</title><img src="${a.dataUrl}" style="max-width:100%;height:auto">`);
  } else if(a.mime==="application/pdf"){
    w.document.write(`<title>${a.filename}</title><embed src="${a.dataUrl}" type="application/pdf" width="100%" height="100%"/>`);
  } else {
    w.document.write(`<pre>No se puede previsualizar este tipo. Descarga:</pre><a href="${a.dataUrl}" download="${a.filename}">Descargar</a>`);
  }
}

function deleteAttachment(id){
  if(!isAdmin()) return;
  DB.attachments = DB.attachments.filter(x=>x.id!==id);
  saveDB(DB);
  toast("Adjunto borrado");
  if(selectedTag) renderAttachments(selectedTag);
}

function loadAssetToForm(tag){
  if(!isAdmin()) return;
  const a = getAsset(tag);
  if(!a){ toast("No existe"); return; }
  setLeftTab("asset");
  document.getElementById("a_asset_tag").value = a.asset_tag;
  document.getElementById("a_tipo").value = a.tipo;
  document.getElementById("a_estado").value = a.estado;
  document.getElementById("a_marca").value = a.marca || "";
  document.getElementById("a_modelo").value = a.modelo || "";
  document.getElementById("a_serial").value = a.serial || "";
  document.getElementById("a_imei").value = a.imei || "";
  document.getElementById("a_notas").value = a.notas || "";
  openDetail(tag);
}

/* =========================
   4) Tabs
========================= */
function setLeftTab(key){
  document.querySelectorAll("#leftTabs .tab").forEach(t=>{
    t.classList.toggle("active", t.dataset.tab===key);
  });
  document.getElementById("tab-asset").classList.toggle("hidden", key!=="asset");
  document.getElementById("tab-assign").classList.toggle("hidden", key!=="assign");
  document.getElementById("tab-attach").classList.toggle("hidden", key!=="attach");
}

/* =========================
   5) PDF PRO (Presentable)
========================= */
// Personaliza aquí (nombre empresa, etc.)
const PDF_BRAND = {
  company: "TU EMPRESA / ORGANIZACIÓN",
  nit: "NIT: ____________",
  phone: "Tel: ____________",
  email: "Email: ____________",
  footer: "Documento generado por el Sistema de Inventario y Asignación de Equipos",
};

function pdfDrawHeader(doc, title, rightTextLines = []) {
  const W = doc.internal.pageSize.getWidth();
  doc.setFillColor(14, 24, 48);
  doc.rect(0, 0, W, 86, "F");

  // Logo placeholder
  doc.setFillColor(255, 255, 255);
  doc.roundedRect(40, 22, 48, 48, 10, 10, "F");
  doc.setTextColor(14, 24, 48);
  doc.setFont("helvetica", "bold");
  doc.setFontSize(16);
  doc.text("AI", 56, 54);

  // Empresa
  doc.setTextColor(255, 255, 255);
  doc.setFont("helvetica", "bold");
  doc.setFontSize(14);
  doc.text(PDF_BRAND.company, 100, 40);

  doc.setFont("helvetica", "normal");
  doc.setFontSize(9);
  doc.text(PDF_BRAND.nit, 100, 54);
  doc.text(PDF_BRAND.phone + " · " + PDF_BRAND.email, 100, 66);

  // Right info
  doc.setFont("helvetica", "normal");
  doc.setFontSize(9);
  let y = 38;
  rightTextLines.forEach(line => {
    doc.text(line, W - 40 - doc.getTextWidth(line), y);
    y += 12;
  });

  // Title
  doc.setTextColor(14, 24, 48);
  doc.setFont("helvetica", "bold");
  doc.setFontSize(14);
  doc.text(title, 40, 120);

  doc.setDrawColor(210, 215, 225);
  doc.line(40, 130, W - 40, 130);
}

function pdfSectionTitle(doc, text, y) {
  const W = doc.internal.pageSize.getWidth();
  doc.setFillColor(245, 247, 252);
  doc.roundedRect(40, y, W - 80, 26, 8, 8, "F");
  doc.setTextColor(14, 24, 48);
  doc.setFont("helvetica", "bold");
  doc.setFontSize(11);
  doc.text(text, 52, y + 17);
  return y + 36;
}

function pdfKeyValueGrid(doc, y, items, cols = 2) {
  const pageW = doc.internal.pageSize.getWidth();
  const boxW = (pageW - 80);
  const colW = boxW / cols;
  const rowH = 44;
  const x0 = 40;
  const curY = y;

  doc.setDrawColor(223, 228, 238);
  doc.setTextColor(28, 39, 64);

  for (let i = 0; i < items.length; i++) {
    const col = i % cols;
    const row = Math.floor(i / cols);
    const x = x0 + col * colW;
    const yy = curY + row * rowH;

    doc.setFillColor(255, 255, 255);
    doc.roundedRect(x, yy, colW - 8, rowH - 8, 8, 8, "F");
    doc.roundedRect(x, yy, colW - 8, rowH - 8, 8, 8, "S");

    doc.setFont("helvetica", "bold");
    doc.setFontSize(9);
    doc.setTextColor(90, 105, 130);
    doc.text(items[i].k, x + 10, yy + 16);

    doc.setFont("helvetica", "normal");
    doc.setFontSize(10);
    doc.setTextColor(28, 39, 64);
    const v = String(items[i].v ?? "—");
    const wrapped = doc.splitTextToSize(v, colW - 28);
    doc.text(wrapped, x + 10, yy + 32);
  }

  const rows = Math.ceil(items.length / cols);
  return curY + rows * rowH + 6;
}

function pdfParagraph(doc, y, text, maxW) {
  doc.setFont("helvetica", "normal");
  doc.setFontSize(10);
  doc.setTextColor(28, 39, 64);
  const lines = doc.splitTextToSize(text || "—", maxW);
  doc.text(lines, 52, y);
  return y + lines.length * 12;
}

function pdfFooter(doc, pageNum, totalPages) {
  const W = doc.internal.pageSize.getWidth();
  const H = doc.internal.pageSize.getHeight();
  doc.setDrawColor(223, 228, 238);
  doc.line(40, H - 52, W - 40, H - 52);

  doc.setFont("helvetica", "normal");
  doc.setFontSize(9);
  doc.setTextColor(110, 125, 150);
  doc.text(PDF_BRAND.footer, 40, H - 32);

  const p = `Página ${pageNum} de ${totalPages}`;
  doc.text(p, W - 40 - doc.getTextWidth(p), H - 32);
}

/* RECIBO PRO (con QR y firmas) */
function pdfReciboAsignacion(asset_tag){
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF({unit:"pt", format:"a4"});

  const asset = getAsset(asset_tag);
  const asg = currentAssignment(asset_tag);
  if(!asset || !asg) throw new Error("No hay datos para el recibo");

  const qrData = (document.getElementById("qrCanvas")?.toDataURL?.("image/png")) || null;

  pdfDrawHeader(doc, "RECIBO DE ENTREGA / ASIGNACIÓN", [
    `Fecha: ${fmtDate(asg.assigned_at)}`,
    `Generado por: ${asg.created_by}`,
    `Asset ID: ${asset.asset_tag}`
  ]);

  let y = 150;

  y = pdfSectionTitle(doc, "Datos del equipo", y);
  y = pdfKeyValueGrid(doc, y, [
    {k:"Asset ID", v: asset.asset_tag},
    {k:"Tipo", v: asset.tipo},
    {k:"Marca", v: asset.marca || "—"},
    {k:"Modelo", v: asset.modelo || "—"},
    {k:"Serial", v: asset.serial || "—"},
    {k:"IMEI", v: asset.imei || "—"},
    {k:"Estado", v: asset.estado},
    {k:"Accesorios / notas", v: asg.notes || "—"},
  ], 2);

  y = pdfSectionTitle(doc, "Responsable y ubicación", y);
  y = pdfKeyValueGrid(doc, y, [
    {k:"Persona", v: asg.person},
    {k:"Documento", v: asg.doc || "—"},
    {k:"Ubicación", v: asg.location},
    {k:"Fecha de entrega", v: fmtDate(asg.assigned_at)},
  ], 2);

  // QR bonito
  if(qrData){
    doc.setFillColor(255,255,255);
    doc.roundedRect(400, 470, 155, 175, 12, 12, "F");
    doc.setDrawColor(223,228,238);
    doc.roundedRect(400, 470, 155, 175, 12, 12, "S");
    doc.addImage(qrData, "PNG", 420, 490, 115, 115);
    doc.setFont("helvetica","normal");
    doc.setFontSize(9);
    doc.setTextColor(90,105,130);
    doc.text("Escanea para ver la ficha", 414, 625);
  }

  // Firmas
  const H = doc.internal.pageSize.getHeight();
  const W = doc.internal.pageSize.getWidth();
  doc.setDrawColor(223,228,238);
  doc.roundedRect(40, H - 150, W - 80, 80, 12, 12, "S");

  doc.setFont("helvetica","bold");
  doc.setFontSize(10);
  doc.setTextColor(28,39,64);
  doc.text("Firmas", 52, H - 125);

  doc.setFont("helvetica","normal");
  doc.setFontSize(10);
  doc.text("Entrega: ________________________________", 52, H - 95);
  doc.text("Recibe:  ________________________________", 320, H - 95);
  doc.setFontSize(9);
  doc.setTextColor(90,105,130);
  doc.text("Nombre y firma", 52, H - 75);
  doc.text("Nombre y firma", 320, H - 75);

  pdfFooter(doc, 1, 1);
  doc.save(`Recibo_${asset.asset_tag}_${asg.person.replace(/\s+/g,"_")}.pdf`);
}

/* REPORTE PRO (presentable + QR + historial + adjuntos) */
function pdfReporteEquipo(asset_tag){
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF({unit:"pt", format:"a4"});

  const asset = getAsset(asset_tag);
  if(!asset) throw new Error("No existe ese equipo");

  const asg = currentAssignment(asset_tag);
  const hist = DB.assignments
    .filter(x=>x.asset_tag.toUpperCase()===asset_tag.toUpperCase())
    .sort((a,b)=> (a.assigned_at>b.assigned_at? -1:1));
  const atts = DB.attachments
    .filter(x=>x.asset_tag.toUpperCase()===asset_tag.toUpperCase())
    .sort((a,b)=> (a.created_at>b.created_at? -1:1));

  const url = new URL(window.location.href);
  url.searchParams.set("asset", normalizeTag(asset_tag));
  const link = url.toString();

  const tmpCanvas = document.createElement("canvas");
  QRCode.toCanvas(tmpCanvas, link, {margin:2, width:140}, (err)=>{
    if(err){ toast("Error generando QR"); return; }
    const qrData = tmpCanvas.toDataURL("image/png");

    pdfDrawHeader(doc, "REPORTE DE EQUIPO", [
      `Fecha: ${fmtDate(nowISO())}`,
      `Asset ID: ${asset.asset_tag}`,
      `Estado: ${asset.estado}`
    ]);

    // QR bloque
    doc.setFillColor(255,255,255);
    doc.roundedRect(400, 138, 155, 165, 12, 12, "F");
    doc.setDrawColor(223,228,238);
    doc.roundedRect(400, 138, 155, 165, 12, 12, "S");
    doc.addImage(qrData, "PNG", 422, 156, 110, 110);
    doc.setFont("helvetica","normal");
    doc.setFontSize(9);
    doc.setTextColor(90,105,130);
    doc.text("QR - Ficha del equipo", 428, 286);

    let y = 150;

    // Ficha técnica
    y = pdfSectionTitle(doc, "Ficha técnica", y);
    y = pdfKeyValueGrid(doc, y, [
      {k:"Asset ID", v: asset.asset_tag},
      {k:"Tipo", v: asset.tipo},
      {k:"Marca", v: asset.marca || "—"},
      {k:"Modelo", v: asset.modelo || "—"},
      {k:"Serial", v: asset.serial || "—"},
      {k:"IMEI", v: asset.imei || "—"},
      {k:"Estado", v: asset.estado},
      {k:"Asignación actual", v: asg ? `${asg.person} · ${asg.location} · Desde ${fmtDate(asg.assigned_at)}` : "No asignado"},
    ], 2);

    // Observaciones
    y = pdfSectionTitle(doc, "Observaciones", y);
    y = pdfParagraph(doc, y, asset.notas || "—", doc.internal.pageSize.getWidth() - 120);
    y += 8;

    // Historial (tabla)
    y = pdfSectionTitle(doc, "Historial de asignaciones", y);
    if(hist.length===0){
      y = pdfParagraph(doc, y, "Sin historial.", doc.internal.pageSize.getWidth() - 120);
    } else {
      const startY = y;
      const W = doc.internal.pageSize.getWidth();
      const x1 = 52, x2 = 220, x3 = 420;

      doc.setFillColor(245,247,252);
      doc.roundedRect(40, startY, W-80, 28, 8, 8, "F");

      doc.setFont("helvetica","bold");
      doc.setFontSize(9);
      doc.setTextColor(90,105,130);
      doc.text("Fecha", x1, startY+18);
      doc.text("Persona", x2, startY+18);
      doc.text("Ubicación / Estado", x3, startY+18);

      y = startY + 40;
      doc.setFont("helvetica","normal");
      doc.setFontSize(10);
      doc.setTextColor(28,39,64);

      for(const x of hist){
        const estado = x.returned_at ? `Devuelto ${fmtDate(x.returned_at)}` : "Asignación activa";
        const f = fmtDate(x.assigned_at);
        const persona = `${x.person}${x.doc?` (${x.doc})`:``}`;
        const ub = `${x.location} — ${estado}`;

        const fLines = doc.splitTextToSize(f, 150);
        const pLines = doc.splitTextToSize(persona, 180);
        const uLines = doc.splitTextToSize(ub, W-80- (x3-40) - 12);

        const maxLines = Math.max(fLines.length, pLines.length, uLines.length);
        const rowH = maxLines*12 + 6;

        if(y + rowH > 760){
          doc.addPage();
          pdfDrawHeader(doc, "REPORTE DE EQUIPO (continuación)", [
            `Asset ID: ${asset.asset_tag}`,
            `Fecha: ${fmtDate(nowISO())}`
          ]);
          y = 150;
        }

        doc.setDrawColor(223,228,238);
        doc.roundedRect(40, y-12, W-80, rowH, 8, 8, "S");

        doc.text(fLines, x1, y);
        doc.text(pLines, x2, y);
        doc.text(uLines, x3, y);

        y += rowH + 8;
      }
    }

    // Adjuntos
    y += 8;
    if(y > 720){
      doc.addPage();
      pdfDrawHeader(doc, "REPORTE DE EQUIPO (continuación)", [`Asset ID: ${asset.asset_tag}`]);
      y = 150;
    }

    y = pdfSectionTitle(doc, "Adjuntos registrados", y);
    if(atts.length===0){
      y = pdfParagraph(doc, y, "Sin adjuntos.", doc.internal.pageSize.getWidth() - 120);
    } else {
      for(const a of atts){
        const line = `• ${a.type} — ${a.desc || "—"} — ${a.filename} — ${fmtDate(a.created_at)}`;
        const lines = doc.splitTextToSize(line, doc.internal.pageSize.getWidth() - 120);

        if(y + lines.length*12 > 760){
          doc.addPage();
          pdfDrawHeader(doc, "REPORTE DE EQUIPO (continuación)", [`Asset ID: ${asset.asset_tag}`]);
          y = 150;
        }
        doc.setFont("helvetica","normal");
        doc.setFontSize(10);
        doc.setTextColor(28,39,64);
        doc.text(lines, 52, y);
        y += lines.length*12 + 6;
      }
    }

    // Footer en todas las páginas
    const totalPages = doc.getNumberOfPages();
    for(let p=1; p<=totalPages; p++){
      doc.setPage(p);
      pdfFooter(doc, p, totalPages);
    }

    doc.save(`Reporte_${asset.asset_tag}.pdf`);
  });
}

/* Etiqueta QR (sticker) */
function pdfEtiqueta(asset_tag){
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF({unit:"pt", format:[283,170]}); // ~10x6cm
  const asset = getAsset(asset_tag);
  if(!asset) throw new Error("No existe");

  const url = new URL(window.location.href);
  url.searchParams.set("asset", asset_tag);
  const link = url.toString();

  const tempCanvas = document.createElement("canvas");
  QRCode.toCanvas(tempCanvas, link, {margin:2, width:130}, (err)=>{
    if(err){ toast("Error QR"); return; }
    const qrData = tempCanvas.toDataURL("image/png");

    doc.setFont("helvetica","bold"); doc.setFontSize(14);
    doc.text(asset.asset_tag, 16, 26);
    doc.setFont("helvetica","normal"); doc.setFontSize(10);
    doc.text(`${asset.tipo} · ${asset.marca||"—"} ${asset.modelo||""}`.trim(), 16, 44);
    doc.text(`Serial: ${asset.serial||"—"}`, 16, 60);

    doc.addImage(qrData, "PNG", 145, 18, 120, 120);
    doc.setFontSize(8);
    doc.text("Escanea para ver ficha", 150, 148);

    doc.save(`Etiqueta_${asset.asset_tag}.pdf`);
  });
}

/* =========================
   6) Events
========================= */
document.getElementById("loginBtn").addEventListener("click", ()=>{
  const user = document.getElementById("loginUser").value.trim();
  const pass = document.getElementById("loginPass").value.trim();
  const u = DB.users.find(x=>x.username===user && x.password===pass && x.active);
  if(!u){ toast("Usuario o contraseña incorrecta"); return; }
  DB.session.userId = u.id; saveDB(DB);
  toast(`Bienvenido ${u.name}`);
  boot();
});

document.getElementById("logoutBtn").addEventListener("click", ()=>{
  DB.session.userId = null; saveDB(DB);
  selectedTag = null;
  toast("Sesión cerrada");
  boot();
});

document.getElementById("resetBtn").addEventListener("click", ()=>{
  if(confirm("¿Seguro? Esto borra TODOS los datos guardados en este navegador.")){
    localStorage.removeItem(LS_KEY);
    DB = loadDB();
    toast("Datos reiniciados");
    boot();
  }
});

document.getElementById("refreshBtn").addEventListener("click", ()=>{
  renderAssets();
  if(selectedTag) openDetail(selectedTag);
});

document.getElementById("filterEstado").addEventListener("change", renderAssets);

document.getElementById("searchBtn").addEventListener("click", ()=> {
  const q = document.getElementById("searchBox").value.trim();
  doSearch(q);
});
document.getElementById("searchBox").addEventListener("keydown", (e)=>{
  if(e.key==="Enter") doSearch(e.target.value.trim());
});

function doSearch(q){
  if(!q){ toast("Escribe un Asset ID o Serial/IMEI"); return; }
  const qq = q.trim().toUpperCase();
  const byTag = DB.assets.find(a=>a.asset_tag.toUpperCase()===qq);
  const bySerial = DB.assets.find(a=> (a.serial||"").toUpperCase()===qq || (a.imei||"").toUpperCase()===qq);
  const asset = byTag || bySerial;
  if(!asset){ toast("No encontrado"); return; }
  openDetail(asset.asset_tag);
}

document.querySelectorAll("#leftTabs .tab").forEach(t=>{
  t.addEventListener("click", ()=> setLeftTab(t.dataset.tab));
});

/* Equipo */
document.getElementById("newAssetBtn").addEventListener("click", ()=>{
  ["a_asset_tag","a_marca","a_modelo","a_serial","a_imei","a_notas"].forEach(id=>document.getElementById(id).value="");
  document.getElementById("a_tipo").value="Computador";
  document.getElementById("a_estado").value="DISPONIBLE";
  toast("Formulario limpio");
});

document.getElementById("saveAssetBtn").addEventListener("click", ()=>{
  try{
    upsertAsset({
      asset_tag: document.getElementById("a_asset_tag").value,
      tipo: document.getElementById("a_tipo").value,
      estado: document.getElementById("a_estado").value,
      marca: document.getElementById("a_marca").value,
      modelo: document.getElementById("a_modelo").value,
      serial: document.getElementById("a_serial").value,
      imei: document.getElementById("a_imei").value,
      notas: document.getElementById("a_notas").value,
    });
    toast("Equipo guardado");
    renderAssets();
    const tag = normalizeTag(document.getElementById("a_asset_tag").value);
    if(tag) openDetail(tag);
  } catch(e){
    toast(e.message);
  }
});

document.getElementById("deleteAssetBtn").addEventListener("click", ()=>{
  if(!isAdmin()) return;
  const tag = normalizeTag(document.getElementById("a_asset_tag").value);
  if(!tag){ toast("Pon el Asset ID"); return; }
  if(confirm(`¿Eliminar ${tag}? También se borra historial y adjuntos.`)){
    deleteAsset(tag);
    toast("Equipo eliminado");
    selectedTag = null;
    document.getElementById("detailBody").classList.add("hidden");
    document.getElementById("detailEmpty").classList.remove("hidden");
    renderAssets();
  }
});

/* Asignar */
document.getElementById("assignBtn").addEventListener("click", ()=>{
  try{
    const asset_tag = document.getElementById("as_asset_tag").value;
    assignAsset({
      asset_tag,
      person: document.getElementById("as_person").value,
      doc: document.getElementById("as_doc").value,
      location: document.getElementById("as_loc").value,
      notes: document.getElementById("as_notes").value
    });
    toast("Asignado. Generando PDF...");
    renderAssets();
    openDetail(asset_tag);
    pdfReciboAsignacion(normalizeTag(asset_tag));
  } catch(e){
    toast(e.message);
  }
});

document.getElementById("returnBtn").addEventListener("click", ()=>{
  try{
    const asset_tag = document.getElementById("as_asset_tag").value;
    returnAsset(asset_tag);
    toast("Devolución marcada");
    renderAssets();
    openDetail(asset_tag);
  } catch(e){
    toast(e.message);
  }
});

/* Adjuntos */
document.getElementById("uploadAttBtn").addEventListener("click", async ()=>{
  try{
    const asset_tag = document.getElementById("att_asset_tag").value;
    const type = document.getElementById("att_type").value;
    const desc = document.getElementById("att_desc").value;
    const file = document.getElementById("att_file").files[0];
    if(!file) throw new Error("Selecciona un archivo");

    await addAttachment({asset_tag, type, desc, file});
    toast("Adjunto subido");
    document.getElementById("att_file").value = "";
    document.getElementById("att_desc").value = "";
    document.getElementById("attPreview").innerHTML = "";
    renderAssets();
    openDetail(asset_tag);
  } catch(e){
    toast(e.message);
  }
});

document.getElementById("att_file").addEventListener("change", ()=>{
  const file = document.getElementById("att_file").files[0];
  const box = document.getElementById("attPreview");
  box.innerHTML = "";
  if(!file) return;
  box.innerHTML = `Seleccionado: <b>${file.name}</b> (${Math.round(file.size/1024)} KB)`;
});

/* Ficha acciones */
document.getElementById("copyLinkBtn").addEventListener("click", async ()=>{
  const link = document.getElementById("d_link").textContent;
  try{ await navigator.clipboard.writeText(link); toast("Link copiado"); }
  catch(e){ toast("No se pudo copiar"); }
});

document.getElementById("downloadQRBtn").addEventListener("click", ()=>{
  const canvas = document.getElementById("qrCanvas");
  const a = document.createElement("a");
  a.href = canvas.toDataURL("image/png");
  a.download = `${selectedTag || "QR"}.png`;
  a.click();
});

document.getElementById("printLabelBtn").addEventListener("click", ()=>{
  if(!selectedTag){ toast("Selecciona un equipo"); return; }
  try{ pdfEtiqueta(selectedTag); }catch(e){ toast(e.message); }
});

document.getElementById("reportBtn").addEventListener("click", ()=>{
  if(!selectedTag){ toast("Selecciona un equipo"); return; }
  try{ pdfReporteEquipo(selectedTag); }catch(e){ toast(e.message); }
});

/* Export/Import */
document.getElementById("exportBtn").addEventListener("click", ()=>{
  const data = JSON.stringify(DB, null, 2);
  const blob = new Blob([data], {type:"application/json"});
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "inventario_export.json";
  a.click();
  URL.revokeObjectURL(url);
});

document.getElementById("importBtn").addEventListener("click", ()=>{
  document.getElementById("importFile").click();
});

document.getElementById("importFile").addEventListener("change", async ()=>{
  const file = document.getElementById("importFile").files[0];
  if(!file) return;
  const text = await file.text();
  try{
    const obj = JSON.parse(text);
    if(!obj.assets || !obj.users) throw new Error("JSON inválido");
    DB = obj;
    saveDB(DB);
    toast("Importado correctamente");
    boot();
  } catch(e){
    toast("No se pudo importar: " + e.message);
  }
});

/* =========================
   7) Boot + Deep link (?asset=)
========================= */
function boot(){
  if(!requireLogin()) return;
  renderAssets();
  const params = new URLSearchParams(window.location.search);
  const t = params.get("asset");
  if(t){
    const tag = normalizeTag(t);
    if(getAsset(tag)) openDetail(tag);
  }
}
boot();
</script>
</body>
</html>
