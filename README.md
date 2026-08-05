# Trivia HCA — pantalla de preguntas automática

Sitio estático para GitHub Pages pensado como cartelera viva del colegio:
corre trivias solas en las TVs, intercala videos promocionales y plaquetas
de comunicación (como las pantallas de los ómnibus), y **todas las pantallas
—y cualquiera que entre al sitio— ven lo mismo al mismo tiempo**.

## Estructura

Todo vive junto en la raíz del repositorio, sin carpetas:

```
index.html       → el player (lo que se ve en la pantalla)
editor.html      → formulador de preguntas + calendario + export
preguntas.csv    → banco de preguntas
calendario.json  → qué preguntas rotan cada día
config.json      → temática del evento + pausas (videos/plaquetas)
logo.png         → logo institucional (azul; el player lo pasa a blanco)
foto.jpg, promo.mp4, …  → las imágenes y videos que subas, sueltos acá
```

Si algún día usás un logo a color, sacá la clase `logo-blanco` de los
`<img>` en `index.html` (el filtro que lo pinta de blanco).

## Publicar en GitHub Pages

1. Creá un repositorio y subí todos estos archivos a la raíz.
2. En **Settings → Pages**, elegí la rama `main` y carpeta `/ (root)`.
3. Abrí `https://TU-USUARIO.github.io/TU-REPO/` en la pantalla y poné el
   navegador en pantalla completa (F11 o modo kiosco).

Si `preguntas.csv` no existe o no carga, el player entra en **MODO
DEMO** con 12 preguntas de ejemplo (es lo que pasa al abrir el archivo
suelto, sin servidor).

## Flujo de actualización

1. Abrí el sitio → botón **✎** → editor → **Traer datos** (carga lo que
   está en la base de Drive).
2. Editá preguntas, calendario, temática y pausas; subí imágenes o videos.
3. **⬆ Publicar en Drive**. Las pantallas revisan las fuentes cada 5
   minutos y toman los cambios solas — no hace falta tocar las TVs ni GitHub.

El ZIP queda como respaldo (y para subir el sitio la primera vez). El editor
además guarda un borrador local por si cerrás sin publicar.

## Formato de `preguntas.csv`

| Columna | Contenido |
|---|---|
| `id` | Número único (lo maneja el editor) |
| `categoria` | Texto libre: Física, Astronomía… |
| `pregunta` | El enunciado |
| `opcion_a` … `opcion_d` | C y D pueden quedar vacías |
| `correcta` | `A`, `B`, `C` o `D` |
| `explicacion` | Dato curioso que aparece con la respuesta |
| `imagen` | Nombre del archivo (ej. `pendulo.jpg`) o link de Drive (opcional) |
| `video` | Nombre de un mp4 corto y mudo, o link de Drive (opcional) |
| `activa` | `1` visible, `0` la saca de rotación sin borrarla |

Codificación UTF-8; los campos con comas o comillas van entre comillas
(el editor ya lo resuelve al exportar).

## Formato de `calendario.json`

```json
{
  "default": { "modo": "todas" },
  "dias": {
    "2026-08-03": { "categoria": "Física" },
    "2026-08-14": { "ids": ["11", "12"] }
  }
}
```

Cada día puede fijar **una categoría** o **una lista de ids**. Los días sin
entrada rotan todas las preguntas activas. El player revisa la fecha al
terminar cada vuelta, así el cambio de día ocurre solo, sin recargar.

## Parámetros del player (URL)

| Parámetro | Efecto | Ejemplo |
|---|---|---|
| `tiempo` | Segundos para responder | `?tiempo=20` |
| `reveal` | Segundos que queda la respuesta | `?reveal=6` |
| `splash` | Pantalla de marca cada N preguntas (0 = nunca) | `?splash=10` |
| `mezclar` | `0` respeta el orden del CSV | `?mezclar=0` |
| `categoria` | Fuerza una categoría e ignora el calendario | `?categoria=Física` |
| `csv` | Usa otro banco de preguntas | `?csv=feria.csv` |

Se combinan: `index.html?tiempo=20&categoria=Física`

## Colores institucionales

Toda la paleta vive en las variables CSS del inicio de `index.html` y
`editor.html` (bloque `:root`, comentado como **PALETA HCA**). Si tenés los
códigos exactos del manual de marca, se cambian ahí y pinta todo el sitio.

## Tips de kiosco

- El escudo sale de `logo.png` (ya incluido).
- Videos: mp4 cortos, sin audio y livianos (GitHub limita archivos a 100 MB,
  pero para que cargue rápido conviene mucho menos).
- El player pide "wake lock" para que la pantalla no se apague, si el
  navegador lo permite; si no, desactivá la suspensión en el dispositivo.

## Pantalla "en vivo" (sincronizada)

No hay servidor: la sincronía se logra porque cada dispositivo calcula, según
la **hora del día**, qué punto exacto del ciclo (pregunta, segundo del timer,
pausa) corresponde ahora. Dos requisitos:

- La hora del dispositivo tiene que estar bien (todas las TVs y celulares la
  ajustan solos por internet, así que en la práctica alcanza con eso).
- El orden "aleatorio" se sortea con una semilla que sale de la fecha: todas
  las pantallas barajan igual y el mazo cambia solo cada día.

Con `?sync=0` una pantalla queda en **modo libre** (arranca desde el
principio y va a su ritmo), útil para probar o para un aula.

## Temática y pausas — `config.json`

```json
{
  "evento": {
    "kicker": "MES DE LA",
    "linea1": "CIENCIA",
    "conector": "y la",
    "linea2": "TECNOLOGÍA",
    "pie": "EN EL HCA"
  },
  "pausas": {
    "cada": 4,
    "items": [
      { "tipo": "splash",   "duracion": 6 },
      { "tipo": "video",    "duracion": 20, "src": "promo.mp4" },
      { "tipo": "plaqueta", "duracion": 10, "kicker": "NOVEDADES",
        "titulo": "Feria de Ciencias", "texto": "Del 14 al 18/9 en el gimnasio.",
        "imagen": "afiche.jpg" }
    ]
  }
}
```

- **evento**: el título de la portada. `linea1` sale en dorado, `linea2` en
  celeste; los campos vacíos no se muestran. Se edita cómodo desde la pestaña
  **Pantalla** del editor, con vista previa.
- **pausas**: cada `cada` preguntas se intercala un ítem de la lista, rotando
  en orden. Tipos: `splash` (portada), `video` (mp4 **mudo**, se corta a los
  segundos indicados) y `plaqueta` (kicker + título + texto y/o imagen; si
  solo hay imagen, va a pantalla completa).

## La base de datos en Drive (así se maneja todo)

Todo el contenido —preguntas, calendario, temática, pausas— vive en una
**Google Sheet llamada `trivia-db`** dentro de tu carpeta de Drive, junto a
las fotos y videos. El script la **crea solo** la primera vez, con hojas
`preguntas`, `calendario`, `config` y `pausas` ya armadas.

GitHub solo aloja el sitio; el contenido nunca más pasa por ahí. El flujo
diario es 100 % web:

1. Entrás al sitio (`index.html`): ves la pantalla en vivo.
2. Movés el mouse o tocás → aparece el botón **✎** → abrís el editor.
3. Editás y tocás **⬆ Publicar en Drive**. En ≤5 minutos todas las
   pantallas recargan solas.

También podés editar la Sheet `trivia-db` a mano desde Drive (celular
incluido) — vale igual, es la misma base.

### El Apps Script (una sola vez)

En [script.google.com](https://script.google.com) → **Nuevo proyecto**,
pegá esto entero. Cambiá la `CLAVE` por una tuya (es la contraseña de
edición: quien la tenga puede publicar).

```js
const CARPETA   = '1WmOYoRS_oaBfK0EdBZuLRWZ-EibfzVxU';
const CLAVE     = 'cambia-esta-clave';
const NOMBRE_DB = 'trivia-db';

const COLUMNAS = ['id','categoria','pregunta','opcion_a','opcion_b',
  'opcion_c','opcion_d','correcta','explicacion','imagen','video','activa'];

/* ---------- LECTURA: el player y el editor leen de acá ---------- */
function doGet(){
  const db = obtenerDB();
  const salida = {archivos:{}, textos:{}};
  const it = DriveApp.getFolderById(CARPETA).getFiles();
  while (it.hasNext()){
    const f = it.next();
    if (f.getMimeType() !== 'application/vnd.google-apps.spreadsheet')
      salida.archivos[f.getName()] = f.getId();
  }
  salida.textos['preguntas.csv']   = csvDeHoja(db.getSheetByName('preguntas'));
  salida.textos['calendario.json'] = JSON.stringify(leerCalendario(db));
  salida.textos['config.json']     = JSON.stringify(leerConfig(db));
  return json(salida);
}

/* ---------- ESCRITURA: el botón "Publicar" del editor ---------- */
function doPost(e){
  let d = {};
  try { d = JSON.parse(e.postData.contents); }
  catch(err){ return json({ok:false, error:'JSON inválido'}); }
  if (d.clave !== CLAVE) return json({ok:false, error:'clave incorrecta'});
  const db = obtenerDB();

  if (d.accion === 'guardar'){
    if (Array.isArray(d.preguntas)) guardarPreguntas(db, d.preguntas);
    if (d.calendario) guardarCalendario(db, d.calendario);
    if (d.config)     guardarConfig(db, d.config);
    return json({ok:true});
  }
  if (d.accion === 'subir' && d.nombre && d.base64){
    const carpeta = DriveApp.getFolderById(CARPETA);
    const viejos = carpeta.getFilesByName(d.nombre);
    while (viejos.hasNext()) viejos.next().setTrashed(true);
    const blob = Utilities.newBlob(Utilities.base64Decode(d.base64),
      d.mime || 'application/octet-stream', d.nombre);
    return json({ok:true, id: carpeta.createFile(blob).getId()});
  }
  return json({ok:false, error:'acción desconocida'});
}

/* ---------- La base: se crea sola si no existe ---------- */
function obtenerDB(){
  const carpeta = DriveApp.getFolderById(CARPETA);
  const it = carpeta.getFilesByName(NOMBRE_DB);
  if (it.hasNext()) return SpreadsheetApp.open(it.next());
  const ss = SpreadsheetApp.create(NOMBRE_DB);
  DriveApp.getFileById(ss.getId()).moveTo(carpeta);
  const p = ss.getSheets()[0]; p.setName('preguntas');
  p.getRange(1,1,2,COLUMNAS.length).setValues([COLUMNAS,
    [1,'Ciencia','¿Primera pregunta de la base?','Sí','Todavía no','','',
     'A','¡Editá esta fila o publicá desde el editor web!','','',1]]);
  const c = ss.insertSheet('calendario');
  c.getRange(1,1,1,3).setValues([['fecha','categoria','ids']]);
  const g = ss.insertSheet('config');
  g.getRange(1,1,6,2).setValues([['kicker','MES DE LA'],['linea1','CIENCIA'],
    ['conector','y la'],['linea2','TECNOLOGÍA'],['pie','EN EL HCA'],
    ['pausas_cada','4']]);
  const a = ss.insertSheet('pausas');
  a.getRange(1,1,2,7).setValues([
    ['tipo','duracion','kicker','titulo','texto','imagen','src'],
    ['splash',6,'','','','','']]);
  return ss;
}

/* ---------- Conversiones ---------- */
function csvDeHoja(h){
  return h.getDataRange().getValues().map(fila =>
    fila.map(c => {
      const s = String(c == null ? '' : c);
      return /[",\n]/.test(s) ? '"' + s.replace(/"/g,'""') + '"' : s;
    }).join(',')).join('\n');
}
function leerCalendario(db){
  const dias = {};
  db.getSheetByName('calendario').getDataRange().getValues().slice(1)
    .forEach(f => {
      const fecha = f[0] instanceof Date
        ? Utilities.formatDate(f[0], Session.getScriptTimeZone(), 'yyyy-MM-dd')
        : String(f[0]||'').trim();
      if (!fecha) return;
      const cat = String(f[1]||'').trim();
      const ids = String(f[2]||'').split(',').map(x=>x.trim()).filter(Boolean);
      dias[fecha] = cat ? {categoria:cat} : (ids.length ? {ids:ids} : {});
    });
  return {default:{modo:'todas'}, dias:dias};
}
function leerConfig(db){
  const kv = {};
  db.getSheetByName('config').getDataRange().getValues()
    .forEach(f => kv[String(f[0]).trim()] = String(f[1]==null?'':f[1]).trim());
  const items = db.getSheetByName('pausas').getDataRange().getValues().slice(1)
    .filter(f => String(f[0]).trim())
    .map(f => ({tipo:String(f[0]).trim(), duracion:+f[1]||8,
      kicker:String(f[2]||''), titulo:String(f[3]||''), texto:String(f[4]||''),
      imagen:String(f[5]||''), src:String(f[6]||'')}));
  return {
    evento:{kicker:kv.kicker||'', linea1:kv.linea1||'', conector:kv.conector||'',
            linea2:kv.linea2||'', pie:kv.pie||''},
    pausas:{cada:+kv.pausas_cada||0, items:items}
  };
}
function guardarPreguntas(db, lista){
  const h = db.getSheetByName('preguntas');
  h.clearContents();
  const filas = [COLUMNAS].concat(lista.map(p => COLUMNAS.map(c => p[c] ?? '')));
  h.getRange(1,1,filas.length,COLUMNAS.length).setValues(filas);
}
function guardarCalendario(db, cal){
  const h = db.getSheetByName('calendario');
  h.clearContents();
  const filas = [['fecha','categoria','ids']];
  Object.keys(cal.dias||{}).sort().forEach(fecha => {
    const r = cal.dias[fecha]||{};
    filas.push([fecha, r.categoria||'', (r.ids||[]).join(',')]);
  });
  h.getRange(1,1,filas.length,3).setValues(filas);
}
function guardarConfig(db, cfg){
  const e = (cfg && cfg.evento) || {};
  const pa = (cfg && cfg.pausas) || {};
  const g = db.getSheetByName('config');
  g.clearContents();
  g.getRange(1,1,6,2).setValues([['kicker',e.kicker||''],['linea1',e.linea1||''],
    ['conector',e.conector||''],['linea2',e.linea2||''],['pie',e.pie||''],
    ['pausas_cada', pa.cada==null?0:+pa.cada]]);
  const a = db.getSheetByName('pausas');
  a.clearContents();
  const filas = [['tipo','duracion','kicker','titulo','texto','imagen','src']]
    .concat((pa.items||[]).map(it => [it.tipo||'splash', +it.duracion||8,
      it.kicker||'', it.titulo||'', it.texto||'', it.imagen||'', it.src||'']));
  a.getRange(1,1,filas.length,7).setValues(filas);
}
function json(obj){
  return ContentService.createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
```

Después: **Implementar → Nueva implementación → Aplicación web**, ejecutar
como **vos**, acceso **Cualquier persona**. Autorizá y copiá la URL `/exec`.

En el editor → **Pantalla → Carpeta de media**: pegá la URL, tocá *Probar
conexión* (la primera consulta crea la base `trivia-db` sola), poné tu
clave de edición y listo. La clave queda solo en tu navegador, nunca en el
repositorio. Al **⬆ Publicar** se escriben las hojas; al subir una imagen o
video desde el editor, va directo a la carpeta (hasta 30 MB; más pesado,
subilo a mano a Drive).

**Nota de seguridad**: la clave viaja en la petición y frena la edición
casual, alcanza para una cartelera escolar; no protege secretos. Si alguna
vez se filtra, cambiala en el script y reimplementá.

## Multimedia con link directo de Drive## Multimedia con link directo de Drive## Multimedia con link directo de Drive

En cualquier campo de imagen o video (preguntas y pausas) podés **pegar el
link de Drive tal cual** en vez de subir el archivo al repo. Requisitos:

1. En Drive: **Compartir → Cualquier persona con el enlace → Lector**.
2. Copiá el link (`https://drive.google.com/file/d/…/view?usp=sharing`) y
   pegalo en el campo. El sitio lo convierte solo a una URL directa.

Notas:
- Las **imágenes** se sirven redimensionadas a 1920 px (perfecto para TV,
  carga rápida).
- Los **videos** tienen que ser livianos (ideal < 50 MB): los archivos muy
  grandes disparan la pantalla intermedia de Drive y no se reproducen — en
  ese caso subilos al repo con el editor.
- Ventaja: cambiás el archivo en Drive y la pantalla lo toma sin tocar el
  repo. Desventaja: si Drive está caído o el permiso cambia, esa media no
  aparece (la pregunta sigue funcionando igual, sin la imagen).

## Parámetro nuevo

| Parámetro | Efecto |
|---|---|
| `?sync=0` | Modo libre: la pantalla no se sincroniza con las demás |
