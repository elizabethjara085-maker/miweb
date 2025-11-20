<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Expedientes Agrarios - Login</title>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage-compat.js"></script>

<style>
/* Reset y estilos generales */
* { box-sizing: border-box; margin: 0; padding: 0; }
body { 
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    padding: 20px;
}
.container { max-width: 1000px; margin: 0 auto; background: white; border-radius: 12px; box-shadow: 0 10px 40px rgba(0,0,0,0.2); overflow: hidden; }
header { background: linear-gradient(135deg, #0b63b6 0%, #0a4d8f 100%); color: white; padding: 30px; text-align: center; position: relative; }
header h1 { font-size: 2em; margin-bottom: 5px; }
header p { opacity: 0.9; font-size: 0.95em; }
.logout-btn { position: absolute; top: 20px; right: 20px; background: rgba(255,255,255,0.2); color: white; border: 1px solid white; padding: 8px 16px; border-radius: 6px; cursor: pointer; font-size: 0.9em; }
.logout-btn:hover { background: rgba(255,255,255,0.3); }
main { padding: 30px; }
.section { background: #f8f9fa; padding: 20px; border-radius: 8px; margin-bottom: 25px; border: 1px solid #e0e0e0; }
.section h2 { color: #0b63b6; margin-bottom: 15px; font-size: 1.3em; display: flex; align-items: center; gap: 10px; }

/* Formularios */
.form-group { margin-bottom: 15px; }
.form-group label { display: block; margin-bottom: 5px; color: #333; font-weight: 500; }
input[type="text"], input[type="password"], input[type="file"] { width: 100%; padding: 12px; border: 2px solid #e0e0e0; border-radius: 6px; font-size: 1em; transition: border-color 0.3s; }
input[type="text"]:focus, input[type="password"]:focus { outline: none; border-color: #0b63b6; }

/* Botones */
.btn { padding: 12px 24px; border: none; border-radius: 6px; cursor: pointer; font-size: 1em; font-weight: 500; transition: all 0.3s; display: inline-block; }
.btn-primary { background: #0b63b6; color: white; width: 100%; }
.btn-primary:hover { background: #094a8f; transform: translateY(-2px); }
.btn-edit { background: #10b981; color: white; padding: 8px 16px; font-size: 0.9em; }
.btn-edit:hover { background: #059669; }
.btn-delete { background: #ef4444; color: white; padding: 8px 16px; font-size: 0.9em; }
.btn-delete:hover { background: #dc2626; }

/* Tablas */
.table-container { overflow-x: auto; }
table { width: 100%; border-collapse: collapse; margin-top: 15px; background: white; }
th { background: #0b63b6; color: white; padding: 12px; text-align: left; font-weight: 600; }
td { padding: 12px; border-bottom: 1px solid #e0e0e0; }
tr:hover { background: #f8f9fa; }
.actions { display: flex; gap: 8px; }
.download-link { color: #0b63b6; text-decoration: none; font-weight: 500; }
.download-link:hover { text-decoration: underline; }
.empty-state, .loading { text-align: center; padding: 40px; color: #666; }
.file-name { font-size: 0.9em; color: #666; margin-top: 5px; }

/* Login */
.login-container { max-width: 400px; margin: 100px auto; background: white; padding: 40px; border-radius: 12px; box-shadow: 0 10px 40px rgba(0,0,0,0.2); }
.login-header { text-align: center; margin-bottom: 30px; }
.login-header h1 { color: #0b63b6; font-size: 2em; margin-bottom: 10px; }
.login-header p { color: #666; }
.error-message { background: #fee; color: #c33; padding: 10px; border-radius: 6px; margin-bottom: 15px; display: none; }
.hidden { display: none !important; }

/* Media queries para móvil */
@media (max-width: 600px) {
    header h1 { font-size: 1.5em; }
    header p { font-size: 0.9em; }
    .btn { padding: 10px; font-size: 0.95em; }
    .login-container { padding: 20px; margin: 50px 10px; }
}
</style>
</head>
<body>

<!-- Pantalla de Login -->
<div id="loginScreen">
    <div class="login-container">
        <div class="login-header">
            <h1>Expedientes Agrarios</h1>
            <p>Acceso al sistema</p>
        </div>
        <div id="errorMessage" class="error-message"></div>
        <form id="loginForm">
            <div class="form-group">
                <label for="loginUsername">Usuario:</label>
                <input type="text" id="loginUsername" required placeholder="Ingrese su usuario">
            </div>
            <div class="form-group">
                <label for="loginPassword">Contraseña:</label>
                <input type="password" id="loginPassword" required placeholder="Ingrese su contraseña">
            </div>
            <button type="submit" class="btn btn-primary">Iniciar Sesión</button>
        </form>
    </div>
</div>

<!-- Pantalla Principal -->
<div id="mainScreen" class="hidden">
    <div class="container">
        <header>
            <button class="logout-btn" onclick="logout()">🚪 Cerrar Sesión</button>
            <h1>📁 Expedientes Agrarios</h1>
            <p id="welcomeMessage">Bienvenido al sistema</p>
        </header>

        <main>
            <div class="section">
                <h2>➕ Agregar / Editar Expediente</h2>
                <form id="expForm">
                    <input type="hidden" id="expId">
                    <div class="form-group">
                        <label for="numero">Número de expediente:</label>
                        <input type="text" id="numero" required placeholder="Ej: EXP-2025-001">
                    </div>
                    <div class="form-group">
                        <label for="nombre">Nombre completo:</label>
                        <input type="text" id="nombre" required placeholder="Ej: Juan Pérez García">
                    </div>
                    <div class="form-group">
                        <label for="archivo">Archivo adjunto:</label>
                        <input type="file" id="archivo" accept=".pdf,.doc,.docx,.jpg,.jpeg,.png">
                        <div id="currentFile" class="file-name"></div>
                    </div>
                    <button type="submit" class="btn btn-primary">💾 Guardar Expediente</button>
                </form>
            </div>

            <div class="section">
                <h2>🔍 Buscar Expediente</h2>
                <input type="text" id="search" placeholder="Buscar por nombre o número de expediente...">
            </div>

            <div class="section">
                <h2>📋 Lista de Expedientes</h2>
                <div id="loading" class="loading">Cargando expedientes...</div>
                <div class="table-container" id="tableContainer" style="display:none;">
                    <table>
                        <thead>
                            <tr>
                                <th>Número</th>
                                <th>Nombre</th>
                                <th>Archivo</th>
                                <th>Acciones</th>
                            </tr>
                        </thead>
                        <tbody id="lista"></tbody>
                    </table>
                </div>
                <div id="emptyState" class="empty-state" style="display:none;">
                    No hay expedientes registrados. Agrega uno para comenzar.
                </div>
            </div>
        </main>
    </div>
</div>

<script>
/* ----------------- Firebase Config ----------------- */
const firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "tu-proyecto.firebaseapp.com",
  projectId: "tu-proyecto",
  storageBucket: "tu-proyecto.appspot.com",
  messagingSenderId: "TU_ID",
  appId: "TU_APP_ID"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
const storage = firebase.storage();

/* ----------------- Sistema de Login ----------------- */
const USERS = { 'INRA': { password: '12345', nombre: 'INRA' } };
let currentUser = null;

document.getElementById('loginForm').addEventListener('submit', function(e){
    e.preventDefault();
    const username = document.getElementById('loginUsername').value.trim();
    const password = document.getElementById('loginPassword').value;
    const errorDiv = document.getElementById('errorMessage');
    if(USERS[username] && USERS[username].password === password){
        currentUser = username;
        showMainScreen();
        loadExpedientes();
    } else {
        errorDiv.textContent = '❌ Usuario o contraseña incorrectos';
        errorDiv.style.display = 'block';
        setTimeout(()=>{ errorDiv.style.display='none'; }, 3000);
    }
});
function showMainScreen(){
    document.getElementById('loginScreen').classList.add('hidden');
    document.getElementById('mainScreen').classList.remove('hidden');
    document.getElementById('welcomeMessage').textContent = `Bienvenido, ${USERS[currentUser].nombre}`;
}
function logout(){
    if(confirm('¿Cerrar sesión?')){
        currentUser = null;
        document.getElementById('loginScreen').classList.remove('hidden');
        document.getElementById('mainScreen').classList.add('hidden');
        document.getElementById('loginForm').reset();
        document.getElementById('expForm').reset();
    }
}

/* ----------------- Cargar Expedientes ----------------- */
async function loadExpedientes(){
    const lista = document.getElementById('lista');
    lista.innerHTML = '';
    const snapshot = await db.collection('expedientes').orderBy('createdAt','desc').get();
    const expedientes = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    if(expedientes.length===0){
        document.getElementById('tableContainer').style.display='none';
        document.getElementById('emptyState').style.display='block';
        return;
    } else {
        document.getElementById('tableContainer').style.display='block';
        document.getElementById('emptyState').style.display='none';
    }
    expedientes.forEach(e=>{
        const tr=document.createElement('tr');
        tr.innerHTML = `
            <td>${e.numero}</td>
            <td>${e.nombre}</td>
            <td>${e.archivoURL ? `<a href="${e.archivoURL}" download="${e.archivoNombre}" class="download-link">${e.archivoNombre}</a>` : 'Sin archivo'}</td>
            <td class="actions">
                <button class="btn btn-delete" onclick="eliminar('${e.id}')">🗑️ Eliminar</button>
            </td>
        `;
        lista.appendChild(tr);
    });
}

/* ----------------- Guardar Expediente ----------------- */
document.getElementById('expForm').addEventListener('submit', async function(e){
    e.preventDefault();
    const numero = document.getElementById('numero').value.trim();
    const nombre = document.getElementById('nombre').value.trim();
    const archivoInput = document.getElementById('archivo');
    let archivoURL="";
    if(archivoInput.files.length>0){
        const file = archivoInput.files[0];
        const storageRef = storage.ref(`expedientes/${file.name}`);
        await storageRef.put(file);
        archivoURL = await storageRef.getDownloadURL();
    }
    await db.collection('expedientes').add({
        numero,
        nombre,
        archivoNombre: archivoInput.files[0]?.name || "",
        archivoURL,
        createdAt: firebase.firestore.FieldValue.serverTimestamp()
    });
    alert('Expediente guardado exitosamente');
    this.reset();
    loadExpedientes();
});

/* ----------------- Eliminar Expediente ----------------- */
async function eliminar(id){
    if(confirm('¿Eliminar este expediente?')){
        await db.collection('expedientes').doc(id).delete();
        loadExpedientes();
    }
}

/* ----------------- Buscar Expedientes ----------------- */
document.getElementById('search').addEventListener('input', loadExpedientes);
</script>
</body>
</html>
