<!DOCTYPE html>
<html lang="es">

<head>

<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">

<title>Control de Inventarios</title>

<script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

<link rel="stylesheet"
href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">

<style>

*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:'Segoe UI',sans-serif;
}

body{
background:#f4f6f9;
padding:20px;
}

.container{
max-width:1400px;
margin:auto;
}

header{
background:linear-gradient(135deg,#2563eb,#1e40af);
padding:20px;
border-radius:15px;
color:white;
text-align:center;
margin-bottom:20px;
}

.botones{
display:flex;
flex-wrap:wrap;
gap:10px;
margin-bottom:20px;
}

button{
border:none;
padding:12px 20px;
border-radius:10px;
cursor:pointer;
font-weight:bold;
}

.btn-cargar{
background:#2563eb;
color:white;
}

.btn-nuevo{
background:#10b981;
color:white;
}

.btn-exportar{
background:#f59e0b;
color:white;
}

#buscar{
flex:1;
padding:12px;
border:1px solid #ddd;
border-radius:10px;
}

.tabla{
background:white;
border-radius:15px;
overflow:auto;
box-shadow:0 5px 15px rgba(0,0,0,.1);
}

table{
width:100%;
border-collapse:collapse;
}

th{
background:#1e40af;
color:white;
padding:12px;
}

td{
padding:10px;
text-align:center;
border-bottom:1px solid #eee;
}

tr:hover{
background:#f3f4f6;
}

.editar{
background:#f59e0b;
color:white;
padding:8px 10px;
border-radius:8px;
}

.eliminar{
background:#ef4444;
color:white;
padding:8px 10px;
border-radius:8px;
}

input[type=file]{
display:none;
}

</style>

</head>

<body>

<div class="container">

<header>

<h1>
<i class="fa-solid fa-boxes-stacked"></i>
Sistema de Inventarios
</h1>

</header>

<div class="botones">

<label class="btn-cargar">
<i class="fa-solid fa-file-excel"></i>
Cargar inventarios.xlsx
<input type="file" id="excelFile" accept=".xlsx,.xls">
</label>

<button class="btn-nuevo" onclick="nuevoProducto()">
<i class="fa fa-plus"></i>
Nuevo Producto
</button>

<button class="btn-exportar" onclick="exportarExcel()">
<i class="fa fa-download"></i>
Exportar Backup
</button>

<input
type="text"
id="buscar"
placeholder="Buscar producto..."
onkeyup="filtrar()">

</div>

<div class="tabla">

<table id="tabla">

<thead>

<tr>

<th>Producto</th>
<th>Características</th>
<th>Precio</th>
<th>Existentes</th>
<th>Vendidas</th>
<th>Dañadas</th>
<th>Acciones</th>

</tr>

</thead>

<tbody></tbody>

</table>

</div>

</div>

<script>

let inventario=[];

document
.getElementById("excelFile")
.addEventListener("change", cargarExcel);

function cargarExcel(e){

const archivo=e.target.files[0];

const reader=new FileReader();

reader.onload=function(event){

const data=new Uint8Array(event.target.result);

const workbook=XLSX.read(data,{
type:"array"
});

const hoja=workbook.Sheets[
workbook.SheetNames[0]
];

inventario=
XLSX.utils.sheet_to_json(hoja);

mostrarTabla();

Swal.fire(
'Correcto',
'Inventario cargado correctamente',
'success'
);

};

reader.readAsArrayBuffer(archivo);

}

function mostrarTabla(){

const tbody=
document.querySelector("tbody");

tbody.innerHTML="";

inventario.forEach((p,index)=>{

tbody.innerHTML+=`

<tr>

<td>${p.producto||''}</td>

<td>${p.características||''}</td>

<td>S/ ${p.precio||0}</td>

<td>${p["unidades existentes"]||0}</td>

<td>${p["unidades vendidas"]||0}</td>

<td>${p["unidades dañadas"]||0}</td>

<td>

<button
class="editar"
onclick="editar(${index})">

<i class="fa fa-edit"></i>

</button>

<button
class="eliminar"
onclick="eliminar(${index})">

<i class="fa fa-trash"></i>

</button>

</td>

</tr>

`;

});

}

async function nuevoProducto(){

const {value:data}=await Swal.fire({

title:'Nuevo Producto',

html:`

<input id="p1" class="swal2-input" placeholder="Producto">

<input id="p2" class="swal2-input" placeholder="Características">

<input id="p3" class="swal2-input" placeholder="Precio">

<input id="p4" class="swal2-input" placeholder="Unidades existentes">

<input id="p5" class="swal2-input" placeholder="Unidades vendidas">

<input id="p6" class="swal2-input" placeholder="Unidades dañadas">

`,

preConfirm:()=>{

return{

producto:p1.value,

características:p2.value,

precio:p3.value,

"unidades existentes":p4.value,

"unidades vendidas":p5.value,

"unidades dañadas":p6.value

};

}

});

if(data){

inventario.push(data);

mostrarTabla();

Swal.fire(
'Registrado',
'Producto agregado',
'success'
);

}

}

async function editar(index){

let p=inventario[index];

const {value:data}=await Swal.fire({

title:'Editar Producto',

html:`

<input id="p1" class="swal2-input"
value="${p.producto}">

<input id="p2" class="swal2-input"
value="${p.características}">

<input id="p3" class="swal2-input"
value="${p.precio}">

<input id="p4" class="swal2-input"
value="${p["unidades existentes"]}">

<input id="p5" class="swal2-input"
value="${p["unidades vendidas"]}">

<input id="p6" class="swal2-input"
value="${p["unidades dañadas"]}">

`,

preConfirm:()=>{

return{

producto:p1.value,

características:p2.value,

precio:p3.value,

"unidades existentes":p4.value,

"unidades vendidas":p5.value,

"unidades dañadas":p6.value

};

}

});

if(data){

inventario[index]=data;

mostrarTabla();

Swal.fire(
'Actualizado',
'Registro actualizado',
'success'
);

}

}

function eliminar(index){

Swal.fire({

title:'¿Eliminar producto?',

text:'No se podrá recuperar',

icon:'warning',

showCancelButton:true,

confirmButtonText:'Eliminar'

})

.then(result=>{

if(result.isConfirmed){

inventario.splice(index,1);

mostrarTabla();

Swal.fire(
'Eliminado',
'Producto eliminado',
'success'
);

}

});

}

function filtrar(){

const texto=
document
.getElementById("buscar")
.value
.toLowerCase();

const filas=
document
.querySelectorAll("tbody tr");

filas.forEach(fila=>{

fila.style.display=

fila.innerText
.toLowerCase()
.includes(texto)

? ''

: 'none';

});

}

function exportarExcel(){

if(inventario.length===0){

Swal.fire(
'Atención',
'No existen datos',
'warning'
);

return;
}

const ws=
XLSX.utils.json_to_sheet(
inventario
);

const wb=
XLSX.utils.book_new();

XLSX.utils.book_append_sheet(
wb,
ws,
'Inventario'
);

const ahora=new Date();

const nombre=

`inventarios-${
ahora.getFullYear()
}-${
String(
ahora.getMonth()+1
).padStart(2,'0')
}-${
String(
ahora.getDate()
).padStart(2,'0')
}-${
String(
ahora.getHours()
).padStart(2,'0')
}-${
String(
ahora.getMinutes()
).padStart(2,'0')
}-${
String(
ahora.getSeconds()
).padStart(2,'0')
}.xlsx`;

XLSX.writeFile(
wb,
nombre
);

Swal.fire(
'Exportado',
'Backup generado correctamente',
'success'
);

}

</script>

</body>
</html>
