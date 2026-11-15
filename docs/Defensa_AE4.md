# Guion de defensa · Gym Control AE4

## 0:00 a 3:00 · Recorrido funcional por teclado

Prepará XAMPP, una cuenta de socio, una cuenta de administrador y el catálogo importado. Abrí la portada. Usá Tab para enfocar controles y Enter para activarlos. Escribí el texto de los formularios cuando corresponda.

1. Activá **Saltar al contenido** y señalá el contorno visible. Entrá por **Ver planes** (CU01).
2. Compará los planes en tabla, pasá a tarjetas y buscá `renovacion` (CU02). Abrí el detalle del Plan Mensual (CU03).
3. Elegí **Seleccionar plan**, ingresá si corresponde y comprobá que el plan se mantenga seleccionado. Completá dirección y teléfono, seleccioná Efectivo y cargá `UCP10`. Confirmá y activá **Ver mi membresía** (CU04). Mostrá el total y la vigencia.
4. Si queda tiempo, mostrá el alta y baja de un plan de prueba con la cuenta administradora. Ensayá el cambio de cuenta para evitar demoras.

## 3:00 a 5:00 · Evidencia

- Abrí **Lighthouse** en DevTools o el reporte guardado. Mostrá la calificación y aclará qué estado de la interfaz se auditó.
- En **Network → Fetch/XHR**, revisá la respuesta de `GET planes.php` y de `POST membresias.php`. Identificá URL, método, código y cabecera JSON. Mostrá un 401 desde una ventana sin sesión.

## 5:00 a 12:00 · Fundamentación individual

Cada integrante debe poder abrir el código y explicar estos recorridos.

### Cómo evolucionó la AE1

La portada, la tabla, las tarjetas, la ficha y el formulario siguen el mismo recorrido. `public/index.html` conserva las estructuras semánticas y `public/css/styles.css` reutiliza los estilos responsive. `public/js/app.js` cambia de vista con el fragmento de URL, solicita JSON y construye el contenido sin recargar el documento. CU01 a CU03 leen `planes`. CU04 usa la sesión para identificar al socio y guarda una membresía con la asociación `usuario_id` y `plan_id`.

### Cómo actúan las protecciones OWASP

- **PDO.** Abrí `models/PlanModel.php` y `models/Modelo.php`. `:id` ocupa el lugar de un valor. `prepare()` prepara la sentencia y `execute()` asocia los datos. `Conexion::obtener()` desactiva la emulación y pide excepciones. En `api/http.php` el límite captura `PDOException`, registra el SQLSTATE y responde 500 con mensaje genérico.
- **BCRYPT.** En `api/usuarios.php`, validá el flujo desde el objeto JSON hasta `password_hash(..., PASSWORD_BCRYPT)`. La base guarda el resultado en `clave_hash VARCHAR(255)`. En `api/login.php`, `password_verify()` usa el hash guardado. La respuesta expone solo los datos de cuenta necesarios.
- **Sesión y acceso.** En `api/sesion.php`, explicá los parámetros de cookie y `session_start()`. `session_regenerate_id(true)` reemplaza el identificador al ingresar. La función protegida consulta `$_SESSION` y devuelve 401 si falta una sesión válida. El rol determina si corresponde un 403. Las escrituras autenticadas comparan el token CSRF con `hash_equals()`. Para el historial, el ID sale de la sesión y llega al `WHERE usuario_id = :usuario`.
- **XSS.** En `public/js/app.js`, ubicá `elemento()` y `actualizarDetalle()`. `textContent` asigna texto, de modo que el contenido de un nombre con etiquetas se muestra de forma literal. Los enlaces se arman con rutas internas e identificadores de la API.

### Qué característica ISO/IEC 25010 requirió más cambios

La adecuación funcional exigió trasladar el formulario visual a una operación con persistencia. La asignación ahora toma el precio de la base, calcula vigencia y total, fija el socio desde la sesión y guarda todo dentro de una transacción. Un identificador de solicitud permite repetir un intento sin duplicar la membresía. Mostrá `MembresiaModel::asignar()` y el resultado en el historial.
