<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Franto Delivery</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

<style>
body { font-family: Arial; margin:0; background:#f5f5f5; }
.container { max-width:720px; margin:auto; padding:20px; }

h1 { text-align:center; color:#0a3d91; }

input, select, button {
  width:100%; padding:12px; margin-top:10px;
  border-radius:10px; border:1px solid #ccc;
}

button {
  background:#25D366; color:white; font-weight:bold;
  border:none; font-size:16px;
}

#map { height:300px; border-radius:10px; margin-top:10px; }

.sabores {
  display:grid;
  grid-template-columns: repeat(2,1fr);
  gap:6px;
}

.sabores label {
  background:white; padding:6px;
  border-radius:6px; border:1px solid #ddd;
  font-size:13px;
}

.total {
  font-size:26px;
  font-weight:bold;
  text-align:center;
  margin-top:15px;
}

.info { text-align:center; font-weight:bold; color:#0a3d91; }

.alerta {
  background:#ffe5e5; color:#b00020;
  padding:10px; border-radius:8px;
  text-align:center; margin-top:10px;
}

.error-box {
  background:#ffdddd;
  color:#900;
  padding:10px;
  margin-bottom:10px;
  border-radius:8px;
  display:none;
}

.footer { text-align:center; margin-top:30px; }
</style>
</head>

<body>

<div class="container">

<div id="errorBox" class="error-box"></div>

<h1>🍦 FRANTO DELIVERY</h1>

<input id="nombre" placeholder="Tu nombre">

<select id="tamano">
<option value="5000">1/4 Kg - $5000</option>
<option value="9000">1/2 Kg - $9000</option>
<option value="16000">1 Kg - $16000</option>
<option value="30000">2 Kg Promo - $30000</option>
</select>

<h3>🍦 Cremas</h3>
<div id="crema" class="sabores"></div>

<h3>🍧 Al agua</h3>
<div id="agua" class="sabores"></div>

<input id="direccion" placeholder="Dirección">

<div class="alerta">🚫 Envíos hasta 5 km</div>

<p>📍 Tocá el mapa</p>
<div id="map"></div>

<div class="info" id="info"></div>
<div class="total" id="total">$0</div>

<button onclick="enviar()">Enviar pedido</button>

<div class="footer">
<p>Av. Edén 465 - La Falda</p>
<div id="qr"></div>
</div>

</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<script>

// --- ERROR HANDLER GLOBAL ---
window.onerror = function(msg){
  document.getElementById("errorBox").style.display="block";
  document.getElementById("errorBox").innerText="Error: "+msg;
};

// --- DATOS ---
const WHATSAPP="5491139362003";
const LOCAL={lat:-31.0947,lng:-64.4936};

// SABORES
const CREMA=["Alfavanna","Bon o bon","Bananita Dolca","Cabsha","Crema Irlandesa","Crema Oreo","Crema Franto","Huevo Kinder","Mascarpone c/frutos del bosque","Marroc","Pistacho","Sambayon","Tiramisú","Dulce de Leche","Chocolate","Ferrero Rocher","Chocolate Blanco","Menta Granizada","Frutilla","Coco","Tramontana","Café"];

const AGUA=["Ananá","Arándano","Durazno","Frambuesa","Frutilla","Limón","Limón Primavera","Limón Granizado"];

// CARGAR SABORES
function cargar(lista,id){
  const div=document.getElementById(id);
  lista.forEach(s=>{
    const el=document.createElement("label");
    el.innerHTML=`<input type="checkbox" value="${s}"> ${s}`;
    div.appendChild(el);
  });
}

cargar(CREMA,"crema");
cargar(AGUA,"agua");

// --- MAPA ---
let map,marker=null,envio=0,lat=null,lng=null;

function iniciarMapa(){
 try{
  map=L.map('map').setView([LOCAL.lat,LOCAL.lng],15);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

  map.on("click",e=>{
    const km=dist(LOCAL.lat,LOCAL.lng,e.latlng.lat,e.latlng.lng);

    if(km>5){
      alert("Fuera de zona");
      return;
    }

    envio=Math.round((1500+km*900)*2);

    lat=e.latlng.lat;
    lng=e.latlng.lng;

    if(marker) map.removeLayer(marker);
    marker=L.marker([lat,lng]).addTo(map);

    actualizar();
  });

 }catch(e){
   setTimeout(iniciarMapa,1000);
 }
}

// --- DISTANCIA ---
function dist(a,b,c,d){
 const R=6371;
 const dLat=(c-a)*Math.PI/180;
 const dLon=(d-b)*Math.PI/180;
 const x=Math.sin(dLat/2)**2+
 Math.cos(a*Math.PI/180)*Math.cos(c*Math.PI/180)*
 Math.sin(dLon/2)**2;
 return R*2*Math.atan2(Math.sqrt(x),Math.sqrt(1-x));
}

// --- TOTAL ---
function actualizar(){
 const base=parseInt(document.getElementById("tamano").value);
 document.getElementById("total").innerText="$"+(base+envio);
 document.getElementById("info").innerText=envio?"Envío: $"+envio:"";
}

document.getElementById("tamano").addEventListener("change",actualizar);

// --- ENVIAR ---
function enviar(){

 try{

 const nombre=document.getElementById("nombre").value;
 const direccion=document.getElementById("direccion").value;

 const sabores=[...document.querySelectorAll("input:checked")]
 .map(e=>e.value).join(", ");

 if(!nombre||!direccion||!sabores||!lat){
   alert("Faltan datos");
   return;
 }

 const link=`https://www.google.com/maps?q=${lat},${lng}`;

 const total=document.getElementById("total").innerText;

 const msg=`Pedido Franto 🍦

Nombre: ${nombre}
Tamaño: ${document.getElementById("tamano").selectedOptions[0].text}
Sabores: ${sabores}
Dirección: ${direccion}

Ubicación:
${link}

Envío: $${envio}
Total: ${total}`;

 window.open(`https://wa.me/${WHATSAPP}?text=${encodeURIComponent(msg)}`);

 }catch(e){
   alert("Error al enviar");
 }

}

// --- QR ---
new QRCode(document.getElementById("qr"), {
 text:"https://wa.me/5491139362003",
 width:120,
 height:120
});

// --- INIT ---
iniciarMapa();

</script>

</body>
</html>
