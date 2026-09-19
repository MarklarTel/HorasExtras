<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<meta name="theme-color" content="#1769aa">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<title>Registro de Horas Extra</title>
<style>
:root{--blue:#1769aa;--blue2:#0d4f86;--green:#16a673;--green2:#087653;--bg:#f3f8fc;--text:#18344f;--muted:#71869a;--line:#d8e6f1;--red:#ef4444}
*{box-sizing:border-box}body{margin:0;background:linear-gradient(#1769aa 0 185px,var(--bg) 185px);font-family:system-ui,-apple-system,Segoe UI,Roboto,sans-serif;color:var(--text)}
.app{max-width:720px;margin:auto;padding-bottom:28px}
header{color:#fff;padding:24px 18px 30px}header h1{font-size:27px;margin:0}header p{margin:5px 0 0;opacity:.9}
.card{background:#fff;margin:0 12px 14px;padding:18px;border:1px solid var(--line);border-radius:22px;box-shadow:0 8px 24px #0c4d7a12}
h2{margin:0 0 15px;font-size:20px}.grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
label{display:block;font-size:13px;font-weight:800;color:var(--muted);margin-bottom:6px}
input{width:100%;padding:13px;border:1px solid #cbdceb;border-radius:13px;background:#fbfdff;font-size:16px;color:var(--text)}
.full{grid-column:1/-1}
.result{margin-top:15px;padding:15px;border-radius:17px;background:#e5faf2;border:1px solid #b7ead8;display:flex;justify-content:space-between;gap:14px}
.result strong{display:block;font-size:26px;color:var(--green2);margin-top:2px}
.result small{color:var(--muted)}
button{border:0;border-radius:14px;padding:13px 16px;font-weight:800;font-size:15px}
.save{width:100%;margin-top:14px;background:var(--green);color:#fff;font-size:17px}
.total{display:flex;align-items:center;justify-content:space-between;background:linear-gradient(135deg,#eaf5ff,#f8fcff)}
.total small{display:block;color:var(--muted);font-weight:800}.total strong{display:block;font-size:28px;color:var(--blue)}
.reset{background:#e8f3ff;color:var(--blue);font-size:13px}
.table-wrap{overflow:auto}.history{width:100%;min-width:590px;border-collapse:collapse;font-size:14px}
.history th{background:var(--blue2);color:#fff;padding:11px;text-align:left}.history td{padding:10px;border-bottom:1px solid #e5edf4}
.action{padding:7px 9px;font-size:13px;margin-right:4px}.edit{background:#e7f1ff;color:var(--blue)}.del{background:#ffe9e9;color:#c62828}
.note{font-size:13px;line-height:1.5;color:var(--muted)}
.empty{text-align:center;color:var(--muted);padding:22px}
@media(max-width:480px){.grid{grid-template-columns:1fr}.full{grid-column:auto}.result{flex-direction:column}}
</style>
</head>
<body>
<div class="app">
<header>
<h1>🕒 Registro de Horas Extra</h1>
<p>Tu tiempo también cuenta 💪</p>
</header>

<section class="card">
<h2>📋 Nuevo registro</h2>
<div class="grid">
<div><label>Día</label><input id="fecha" type="date"></div>
<div><label>Hora de salida</label><input id="hora" type="time"></div>
<div class="full"><label>OBRA</label><input id="obra" type="text" placeholder="Ej. Obra X"></div>
</div>
<div class="result">
<div><small>HORAS EXTRA DE HOY</small><strong id="extra">0 h 00 min</strong></div>
<div><small>HORARIO HABITUAL</small><strong id="horario" style="font-size:16px">—</strong></div>
</div>
<button class="save" id="guardar">💾 Guardar registro</button>
</section>

<section class="card total">
<div><small>TOTAL ACUMULADO</small><strong id="total">0 h 00 min</strong></div>
<button class="reset" id="reset">↻ Reiniciar total</button>
</section>

<section class="card">
<h2>📑 Historial</h2>
<div class="table-wrap">
<table class="history">
<thead><tr><th>Fecha</th><th>Salida</th><th>Obra</th><th>Extra</th><th></th></tr></thead>
<tbody id="lista"></tbody>
</table>
</div>
<div id="vacio" class="empty">Todavía no hay registros.</div>
</section>

<section class="card note">
ℹ️ <b>Horario habitual:</b> lunes a jueves 07:30–16:15 · viernes 07:30–13:45.
El cálculo de horas extra se hace automáticamente.
</section>
</div>

<script>
const KEY="horas_extra_registros";
let registros=JSON.parse(localStorage.getItem(KEY)||"[]");
const $=id=>document.getElementById(id);

function minutosExtra(fecha,hora){
 if(!fecha||!hora)return 0;
 const d=new Date(fecha+"T00:00:00"), dia=d.getDay();
 if(dia===0||dia===6)return 0;
 const limite=dia===5?825:975;
 const [h,m]=hora.split(":").map(Number);
 return Math.max(0,h*60+m-limite);
}
function formato(min){
 return `${Math.floor(min/60)} h ${String(min%60).padStart(2,"0")} min`;
}
function actualizar(){
 const f=$("fecha").value,h=$("hora").value;
 $("extra").textContent=formato(minutosExtra(f,h));
 if(f){
  const dia=new Date(f+"T00:00:00").getDay();
  $("horario").textContent=dia===5?"07:30–13:45":dia>=1&&dia<=4?"07:30–16:15":"Fin de semana";
 }
}
function escapeHtml(x){return String(x).replace(/[&<>"']/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#039;"}[c]));}
function guardar(){
 const fecha=$("fecha").value,hora=$("hora").value,obra=$("obra").value.trim();
 if(!fecha||!hora){alert("Introduce el día y la hora de salida.");return;}
 registros.push({fecha,hora,obra,min:minutosExtra(fecha,hora)});
 localStorage.setItem(KEY,JSON.stringify(registros));
 $("obra").value="";render();actualizar();
}
function render(){
 const lista=$("lista");lista.innerHTML="";
 let total=0;
 const orden=registros.map((r,i)=>({...r,i})).sort((a,b)=>(b.fecha+b.hora).localeCompare(a.fecha+a.hora));
 $("vacio").style.display=orden.length?"none":"block";
 orden.forEach(r=>{
  total+=r.min;
  const tr=document.createElement("tr");
  tr.innerHTML=`<td>${new Date(r.fecha+"T00:00:00").toLocaleDateString("es-ES")}</td><td>${r.hora}</td><td>${escapeHtml(r.obra)}</td><td><b>${formato(r.min)}</b></td><td><button class="action edit" onclick="editar(${r.i})">✏️</button><button class="action del" onclick="eliminar(${r.i})">🗑️</button></td>`;
  lista.appendChild(tr);
 });
 $("total").textContent=formato(total);
}
function editar(i){
 const r=registros[i];
 $("fecha").value=r.fecha;$("hora").value=r.hora;$("obra").value=r.obra;
 registros.splice(i,1);localStorage.setItem(KEY,JSON.stringify(registros));render();actualizar();
 window.scrollTo({top:0,behavior:"smooth"});
}
function eliminar(i){
 if(confirm("¿Eliminar este registro?")){registros.splice(i,1);localStorage.setItem(KEY,JSON.stringify(registros));render();}
}
$("guardar").onclick=guardar;
$("reset").onclick=()=>{if(confirm("¿Borrar todos los registros?")){registros=[];localStorage.removeItem(KEY);render();}};
$("fecha").onchange=actualizar;$("hora").oninput=actualizar;
$("fecha").value=new Date().toISOString().slice(0,10);
actualizar();render();
</script>
</body>
</html>