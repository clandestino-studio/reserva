# reserva/
[index.html](https://github.com/user-attachments/files/24316157/index.html)
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Admin | Clandestino Studio</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  background:#0f0f0f;
  color:#fff;
  font-family:Arial, sans-serif;
  padding:20px;
}
.panel{max-width:1000px;margin:auto}
h1{color:#e63946;margin-bottom:10px}

.card{
  background:#1f1f1f;
  padding:20px;
  border-radius:14px;
  margin-bottom:20px;
}

input{
  padding:10px;
  border-radius:8px;
  border:none;
  font-size:15px;
}

.barberos{
  display:flex;
  gap:10px;
  flex-wrap:wrap;
}

.barbero{
  padding:10px 16px;
  border-radius:20px;
  cursor:pointer;
  font-size:14px;
  user-select:none;
}

.on{background:#1e7f43}
.off{background:#7f1e1e}

.agenda{
  display:grid;
  grid-template-columns:repeat(auto-fill,minmax(260px,1fr));
  gap:14px;
}

.turno{
  background:#2e2e2e;
  padding:14px;
  border-radius:12px;
}

.turno .fecha{
  font-weight:bold;
  color:#e63946;
  margin-bottom:4px;
}

.turno button{
  margin-top:10px;
  width:100%;
  border:none;
  padding:8px;
  border-radius:8px;
  background:#e63946;
  color:#fff;
  cursor:pointer;
}
</style>
</head>

<body>
<div class="panel">
<h1>Panel Admin</h1>

<div class="card">
<label>📅 Filtrar por fecha (opcional)</label><br><br>
<input type="date" id="fecha">
</div>

<div class="card">
<h3>Bloquear barberos por día</h3>
<p style="font-size:14px;opacity:.8">Elegí una fecha y tocá el barbero</p>
<div class="barberos">
  <div class="barbero on" data-n="Adrian Cerdan">Adrian Cerdan</div>
  <div class="barbero on" data-n="Gonzalo Toledo">Gonzalo Toledo</div>
  <div class="barbero on" data-n="Uriel Gomez">Uriel Gomez</div>
</div>
</div>

<div class="card">
<h3>Agenda de turnos</h3>
<div class="agenda" id="agenda"></div>
</div>
</div>

<script>
const agenda = document.getElementById("agenda");
const fechaInput = document.getElementById("fecha");
const barberos = document.querySelectorAll(".barbero");

let bloqueos = JSON.parse(localStorage.getItem("bloqueos")) || {};

fechaInput.addEventListener("change", cargar);

// Bloqueo de barberos
barberos.forEach(b=>{
  b.onclick = ()=>{
    const fecha = fechaInput.value;
    if(!fecha){
      alert("Primero elegí una fecha");
      return;
    }
    const nombre = b.dataset.n;
    bloqueos[fecha] = bloqueos[fecha] || [];

    const i = bloqueos[fecha].indexOf(nombre);
    if(i > -1){
      bloqueos[fecha].splice(i,1);
      b.classList.remove("off");
      b.classList.add("on");
    }else{
      bloqueos[fecha].push(nombre);
      b.classList.remove("on");
      b.classList.add("off");
    }
    localStorage.setItem("bloqueos", JSON.stringify(bloqueos));
  };
});

function cargar(){
  agenda.innerHTML = "";

  let turnos = JSON.parse(localStorage.getItem("turnos")) || [];

  // 🧹 LIMPIEZA + NORMALIZACIÓN
  turnos = turnos
    .filter(t => t && t.fecha && t.hora && t.nombre && t.barbero)
    .map(t => ({
      ...t,
      id: t.id || crypto.randomUUID()
    }));

  localStorage.setItem("turnos", JSON.stringify(turnos));

  const fechaFiltro = fechaInput.value;
  let visibles = fechaFiltro
    ? turnos.filter(t => t.fecha === fechaFiltro)
    : turnos;

  if(visibles.length === 0){
    agenda.innerHTML = "<p>No hay turnos para mostrar</p>";
    return;
  }

  visibles
    .sort((a,b)=>{
      if(a.fecha !== b.fecha) return a.fecha.localeCompare(b.fecha);
      return a.hora.localeCompare(b.hora);
    })
    .forEach(t=>{
      const d = document.createElement("div");
      d.className = "turno";
      d.innerHTML = `
        <div class="fecha">${t.fecha} · ${t.hora}</div>
        <strong>${t.barbero}</strong><br>
        Cliente: ${t.nombre}
        <button onclick="borrar('${t.id}')">Cancelar turno</button>
      `;
      agenda.appendChild(d);
    });
}

function borrar(id){
  let turnos = JSON.parse(localStorage.getItem("turnos")) || [];
  turnos = turnos.filter(t => t.id !== id);
  localStorage.setItem("turnos", JSON.stringify(turnos));
  cargar();
}

cargar();
</script>
</body>
</html>
[agendar.html](https://github.com/user-attachments/files/24316161/agendar.html)

<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Reservar turno | Clandestino Studio</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{
  background:#111;
  color:#fff;
  font-family:Arial, sans-serif;
  display:flex;
  justify-content:center;
  padding:20px;
}

.card{
  background:#1f1f1f;
  padding:30px;
  border-radius:16px;
  width:100%;
  max-width:420px;
  box-shadow:0 0 20px rgba(0,0,0,.6);
}

h1{
  text-align:center;
  color:#e63946;
}

label{
  margin-top:16px;
  display:block;
}

input, select{
  width:100%;
  padding:12px;
  margin-top:6px;
  border-radius:8px;
  border:none;
  background:#333;
  color:#fff;
  font-size:15px;
}

.horarios{
  display:grid;
  grid-template-columns:repeat(3,1fr);
  gap:10px;
  margin-top:15px;
}

.hora{
  padding:10px;
  border-radius:10px;
  background:#2e2e2e;
  text-align:center;
  cursor:pointer;
}

.hora:hover{background:#444}
.hora.sel{background:#e63946}
.hora.off{
  background:#555;
  opacity:.4;
  pointer-events:none;
}

button{
  width:100%;
  padding:14px;
  margin-top:20px;
  border:none;
  border-radius:10px;
  background:#e63946;
  color:#fff;
  font-size:16px;
  cursor:pointer;
}
</style>
</head>

<body>
<div class="card">
<h1>Reservar turno</h1>

<label>Nombre</label>
<input id="nombre">

<label>Barbero</label>
<select id="barbero">
  <option value="">Seleccionar</option>
  <option>Adrian Cerdan</option>
  <option>Gonzalo Toledo</option>
  <option>Uriel Gomez</option>
</select>

<label>Servicio</label>
<select id="servicio">
  <option value="">Seleccionar</option>
  <option>Corte + Perfilado</option>
  <option>Corte + Barba</option>
  <option>Corte con Diseño</option>
</select>

<label>Fecha</label>
<input type="date" id="fecha">

<label>Hora</label>
<div class="horarios" id="horarios"></div>

<button onclick="reservar()">Confirmar turno</button>
</div>

<script>
const horariosDiv = document.getElementById("horarios");
let horaSeleccionada = null;

const whatsapp = {
  "Adrian Cerdan":"2805053068",
  "Gonzalo Toledo":"2804303088",
  "Uriel Gomez":"2804630763"
};

function generarHorarios(){
  const list=[];
  const inicio=10*60;
  const fin=21*60;
  const dur=45;

  for(let m=inicio; m+dur<=fin; m+=dur){
    const h=String(Math.floor(m/60)).padStart(2,"0");
    const min=String(m%60).padStart(2,"0");
    list.push(`${h}:${min}`);
  }
  return list;
}

function cargarHorarios(){
  horariosDiv.innerHTML="";
  horaSeleccionada=null;

  const fecha=fechaInput.value;
  const barbero=barberoInput.value;
  if(!fecha||!barbero) return;

  const turnos=JSON.parse(localStorage.getItem("turnos"))||[];
  const bloqueos=JSON.parse(localStorage.getItem("bloqueos"))||{};
  const ocupados=turnos.filter(t=>t.fecha===fecha && t.barbero===barbero)
                        .map(t=>t.hora);

  generarHorarios().forEach(h=>{
    const d=document.createElement("div");
    d.className="hora";
    d.textContent=h;

    if(ocupados.includes(h) || (bloqueos[fecha]||[]).includes(barbero)){
      d.classList.add("off");
    }else{
      d.onclick=()=>{
        document.querySelectorAll(".hora").forEach(x=>x.classList.remove("sel"));
        d.classList.add("sel");
        horaSeleccionada=h;
      };
    }
    horariosDiv.appendChild(d);
  });
}

const nombreInput=document.getElementById("nombre");
const barberoInput=document.getElementById("barbero");
const servicioInput=document.getElementById("servicio");
const fechaInput=document.getElementById("fecha");

barberoInput.onchange=cargarHorarios;
fechaInput.onchange=cargarHorarios;

function reservar(){
  if(!nombreInput.value||!barberoInput.value||!servicioInput.value||!fechaInput.value||!horaSeleccionada){
    alert("Completá todo");
    return;
  }

  const turno={
    id:crypto.randomUUID(),
    nombre:nombreInput.value,
    barbero:barberoInput.value,
    servicio:servicioInput.value,
    fecha:fechaInput.value,
    hora:horaSeleccionada
  };

  const t=JSON.parse(localStorage.getItem("turnos"))||[];
  t.push(turno);
  localStorage.setItem("turnos",JSON.stringify(t));

  const tel=whatsapp[turno.barbero];
  const msg=encodeURIComponent(
    `Nuevo turno:\n${turno.nombre}\n${turno.fecha} ${turno.hora}\n${turno.servicio}`
  );

  window.open(`https://wa.me/54${tel}?text=${msg}`,"_blank");
  alert("Turno confirmado");
  location.reload();
}
</script>
</body>
</html>
