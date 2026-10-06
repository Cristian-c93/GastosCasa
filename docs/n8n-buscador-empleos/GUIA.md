# 🎯 Buscador de empleos: CV → n8n + Claude → Google Sheets

Subes tu CV una sola vez. Cada mañana n8n busca vacantes **en Quito o remotas a nivel mundial** (LinkedIn, Indeed, Glassdoor, Remotive, RemoteOK y, si quieres, Upwork). Claude lee cada vacante, decide si encajas y **te deja el correo de postulación ya escrito** en una hoja de Google Sheets:

| Cargo | Empresa | Match % | Link | Email | Mail | Preguntas para mí | Estado |
|---|---|---|---|---|---|---|---|
| Data Analyst | Acme | 82 | https://… | jobs@acme.com | Hola equipo de Acme… | • ¿Tienes experiencia con Power BI? | **Sin enviar** |

Tú lees el correo, lo ajustas si quieres, **lo envías tú mismo** y cambias el estado a **Enviado**. Nada se envía solo a las empresas.

Tu n8n: `https://vps-3c0c0def.vps.ovh.ca` · Zona horaria: **Ecuador (America/Guayaquil)**

---

## 🧠 ¿Cómo funciona? (la idea en simple)

Son **2 workflows** (flujos):

### Flujo 1 · "Perfil desde mi CV" (lo usas una vez, o cuando actualices tu CV)

```
📝 Formulario: subes tu CV en PDF + qué buscas
   ↓
1. 📄 Leer PDF               → saca el texto del CV
2. 🤖 Generar perfil (Claude) → arma tu perfil y palabras clave de búsqueda,
                                y anota lo que no sabe de ti como PREGUNTAS
3. 🧾 Preparar fila Perfil    → acomoda los datos
4. 💾 Guardar Perfil          → pestaña "Perfil" de tu hoja
5. ❓ Separar preguntas       → una fila por pregunta
6. 💾 Guardar Preguntas       → pestaña "Preguntas" (tú escribes las respuestas)
```

### Flujo 2 · "Buscar vacantes y redactar correos" (corre solo, lunes a viernes 8:00)

```
⏰ Cada mañana 8:00  (o ▶️ "Probar ahora")
   ↓
1. 📖 Leer Perfil + Leer Preguntas → tu perfil + las preguntas que ya respondiste
2. 🧩 Armar perfil                 → todo en un solo texto para Claude
3. 🔀 Una búsqueda por palabra     → 1 búsqueda por cada palabra clave
   ↓            ↓             ↓            ↓             ↓
 LinkedIn     LinkedIn      Remotive     RemoteOK     Upwork
 · Quito      · Remoto                                (Apify, opcional)
   ↓            ↓             ↓            ↓             ↓
 Normalizar  (cada fuente habla "distinto": aquí todas quedan con el mismo formato)
   ↓
4. 🔗 Unir fuentes
5. 📖 Leer Vacantes guardadas  → para no repetir las que ya tienes
6. 🧹 Quitar repetidas          → máx. 25 vacantes nuevas por día
7. 🤖 Analizar y redactar (Claude) → Match %, por qué encajas, email si aparece,
                                     correo redactado, preguntas para ti
8. 🧾 Preparar fila Vacante
9. ✅ ¿Encaja? (65 %+ y ubicación OK) → descarta lo que no sirve
10. 💾 Guardar en Vacantes       → Estado = "Sin enviar"
11. 📬 Avisarme por Gmail        → "🎯 7 vacantes nuevas donde puedes aplicar"
```

### ¿Y las preguntas sobre mí?

Claude te pregunta cosas de dos formas:
- **Pestaña `Preguntas`**: lo que le faltó al leer tu CV (por ejemplo, tu nivel de inglés o si aceptas trabajo freelance). Escribe tu respuesta en la columna **Respuesta**. Desde la siguiente búsqueda, Claude la usa para todas las vacantes.
- **Columna `Preguntas para mí`** de cada vacante: lo que falta para esa postulación en concreto. Si en el correo ves `[COMPLETAR: …]`, es un dato que Claude no quiso inventar. Si la pregunta sirve para todas las vacantes, cópiala a la pestaña `Preguntas` con su respuesta.

---

## ⚠️ Antes de empezar: LinkedIn y Upwork

- **LinkedIn** no deja que programas externos lean sus ofertas (no tiene API pública y prohíbe el *scraping*). Por eso usamos **JSearch**, un servicio que junta ofertas de Google for Jobs, **LinkedIn**, Indeed, Glassdoor y otros. En tu hoja, la columna **Fuente** te dice de dónde viene cada oferta.
- **Upwork** cerró sus feeds RSS en 2024. La única vía práctica es **Apify** (un servicio de pago por uso). Viene **apagado**; lo activas si quieres proyectos freelance (Paso 9).
- **Correos de contacto:** casi ninguna oferta publica un correo. Claude lo copia solo si aparece en la descripción. Si no hay, la columna dice **"Aplicar por link"**: abres el link y pegas el correo redactado como carta de presentación o en el mensaje de postulación.

---

## ✏️ Lo que TÚ tienes que cambiar con tus datos

| # | Qué | Dónde se pone | De dónde lo sacas |
|---|-----|---------------|-------------------|
| 1 | URL de tu Google Sheet | Los 5 nodos de Google Sheets (los 2 flujos) → *Document* | Paso 1 |
| 2 | API key de Anthropic (Claude) | Credencial *Anthropic* | Paso 2 |
| 3 | Credencial de Google Sheets | Credencial *Google Sheets OAuth2 API* | Paso 3 |
| 4 | API key de RapidAPI (JSearch) | Credencial *Header Auth* | Paso 4 |
| 5 | Tu correo para el aviso | Nodo *Avisarme por Gmail* → *To* | Tu Gmail |
| 6 | (Opcional) Token y actor de Apify | Nodo *Upwork (Apify)* | Paso 9 |

---

## Paso 0 – Lo que necesitas

- ✅ Tu n8n abierto: `https://vps-3c0c0def.vps.ovh.ca`
- ✅ Tu CV en **PDF** (que tenga texto seleccionable, no una foto escaneada)
- ✅ Una cuenta de Google (Gmail)
- ✅ Los archivos de esta carpeta: `workflow-1-perfil.json`, `workflow-2-vacantes.json` y `plantillas/`
- 💳 Una tarjeta para cargar **USD 5** de saldo en Anthropic (alcanza para semanas)
- ⏱️ Entre 45 y 60 minutos

---

## Paso 1 – Crear tu hoja de Google Sheets 📊

1. Entra a **https://sheets.new**. Se abre una hoja en blanco.
2. Ponle de nombre `Buscador de empleos` (arriba a la izquierda).
3. Crea **3 pestañas** (abajo, botón **+**) con estos nombres **exactos**, con tildes y mayúsculas:
   - `Vacantes`
   - `Perfil`
   - `Preguntas`
4. En **cada pestaña**, la **fila 1** son los encabezados. Copia y pega la línea correspondiente en la celda **A1**, luego **Datos → Dividir texto en columnas** (separador: coma). También puedes usar **Archivo → Importar** con los `.csv` de la carpeta `plantillas/` ("Reemplazar hoja actual").

   **Vacantes**
   ```
   Fecha,Fuente,Cargo,Empresa,Ubicación,Match %,Por qué encajo,Link,Email,Asunto,Mail,Preguntas para mí,Estado,Fecha envío,ID
   ```
   **Perfil**
   ```
   Clave,Actualizado,Nombre,Email,Resumen,Cargos objetivo,Keywords,Habilidades,Idiomas,Modalidad,Salario esperado,Perfil JSON
   ```
   **Preguntas**
   ```
   Fecha,Pregunta,Respuesta,Origen
   ```
   ⚠️ Los nombres tienen que ser **idénticos** (n8n busca las columnas por su nombre).

5. **Lista desplegable para el Estado** (en la pestaña `Vacantes`):
   - Selecciona la columna **M** (Estado) → **Datos → Validación de datos → Agregar regla**.
   - Criterio: **Menú desplegable** → opciones `Sin enviar` y `Enviado`. Ponle rojo a *Sin enviar* y verde a *Enviado*.
6. **Para leer cómodo los correos:** selecciona la columna **K** (Mail) → **Formato → Ajuste de texto → Ajustar**. Ensánchala.
7. Copia la **URL** de la hoja (la barra del navegador, algo como `https://docs.google.com/spreadsheets/d/1AbC.../edit`). La usas en el Paso 6.

---

## Paso 2 – La llave de Claude (Anthropic) 🤖

1. Entra a **https://console.anthropic.com** y crea tu cuenta.
2. **Billing / Plans & Billing** → carga **USD 5** de crédito.
3. **API Keys** → **Create Key** → nombre `n8n` → copia la llave (empieza por `sk-ant-…`). Solo se muestra una vez. 🔑
4. En n8n: **Credentials → Add credential** → busca `Anthropic` → pega la llave en **API Key** → **Save** ✅.

> 💰 **Costo:** cada vacante analizada cuesta más o menos USD 0,01 a 0,02. Con 25 vacantes por día, de lunes a viernes, son unos USD 5 a 10 al mes. Para gastar menos, en el nodo *Claude (vacantes)* cambia el modelo a uno más barato (Haiku), o baja `MAX_POR_EJECUCION` en *Quitar repetidas*.

---

## Paso 3 – Conectar Google Sheets y Gmail 🔐

Si ya hiciste la guía de **recordatorios de pastillas**, ya tienes un proyecto en Google Cloud (`n8n-pastillas`). Úsalo.

1. **https://console.cloud.google.com** → selecciona tu proyecto.
2. **☰ → APIs y servicios → Biblioteca** → habilita **Google Sheets API**, **Google Drive API** y **Gmail API** (busca cada una → **Habilitar**).
3. Puedes usar el mismo **ID de cliente OAuth** que ya creaste (misma URI de redirección `https://vps-3c0c0def.vps.ovh.ca/rest/oauth2-credential/callback`). Si no lo tienes, sigue el **Paso 2** de `../n8n-recordatorios-pastillas/GUIA.md`.
4. En n8n: **Add credential → Google Sheets OAuth2 API** → pega Client ID y Client Secret → **Sign in with Google** → permite todo → **Save**.
5. Repite con **Gmail OAuth2** (mismo Client ID y Secret).

---

## Paso 4 – La llave de JSearch (LinkedIn y otros) 🔎

1. Entra a **https://rapidapi.com** y crea tu cuenta (puedes entrar con Google).
2. Busca **JSearch** (de *OpenWeb Ninja* / *letscrape*) → **Pricing** → elige **Basic (gratis)**: 200 búsquedas al mes.
3. En la pestaña **Endpoints**, copia el valor de **X-RapidAPI-Key**.
4. En n8n: **Add credential → Header Auth**:
   - **Name:** `X-RapidAPI-Key`
   - **Value:** tu llave
   - Nombre de la credencial: `JSearch RapidAPI` → **Save**.

> 📏 **Las cuentas:** cada palabra clave hace 2 búsquedas (Quito + Remoto). Con 4 palabras son 8 búsquedas al día; de lunes a viernes, unas 170 al mes. Cabe en el plan gratis. Si agregas más palabras, pásate al plan pagado o cambia el horario a 3 días por semana (Paso 8).

Remotive y RemoteOK **no necesitan llave**.

---

## Paso 5 – Importar los 2 workflows 📥

Repite esto para **cada archivo**:

1. En n8n: **+ (Create) → Workflow**.
2. Menú **⋯** (arriba a la derecha) → **Import from File…** → elige `workflow-1-perfil.json` (luego, en otro workflow nuevo, `workflow-2-vacantes.json`).
3. **Save**.

---

## Paso 6 – Ponerle las llaves y la hoja a cada nodo 🔌

### Flujo 1 · "Empleos 1 · Perfil desde mi CV"

| Nodo | Qué hacer |
|------|-----------|
| **Claude (perfil)** | Credential → tu credencial *Anthropic* |
| **Guardar Perfil** | Credential → *Google Sheets*. En **Document** → *By URL* → pega la URL de tu hoja. **Sheet** → `Perfil` |
| **Guardar Preguntas** | Igual, con la hoja `Preguntas` |

### Flujo 2 · "Empleos 2 · Buscar vacantes y redactar correos"

| Nodo | Qué hacer |
|------|-----------|
| **Leer Perfil** / **Leer Preguntas** / **Leer Vacantes guardadas** / **Guardar en Vacantes** | Credential → *Google Sheets* · **Document** → URL de tu hoja · **Sheet** → la pestaña que dice su nombre |
| **LinkedIn y otros · Quito** y **· Remoto** | **Header Auth** → *JSearch RapidAPI* |
| **Claude (vacantes)** | Credential → *Anthropic* |
| **Avisarme por Gmail** | Credential → *Gmail*. En **To** cambia `PEGA_AQUI_TU_CORREO@gmail.com` por tu correo |

Al final no debe quedar ningún **triángulo rojo ⚠️**, salvo en **Upwork (Apify)**, que está apagado (gris) a propósito.

> 💡 El modelo viene puesto como `claude-sonnet-5-5`. Si en tu cuenta aparece otro nombre, en el nodo Claude cambia **Model** a *From list* y elige uno.

---

## Paso 7 – Primera prueba 🧪

### 7.1 Tu perfil (Flujo 1)
1. Abre el **Flujo 1** → actívalo con el interruptor **Active** (arriba a la derecha).
2. Doble clic en **Formulario CV** → copia la **Production URL** y ábrela en el navegador.
3. Sube tu CV y llena el formulario → **Submit**.
4. En 1 minuto, revisa tu hoja:
   - **Perfil**: una fila con tu resumen y las **Keywords** de búsqueda. 👉 **Revisa las Keywords.** Son las palabras que se van a buscar. Puedes editarlas a mano, separadas por comas. Se usan máximo 4.
   - **Preguntas**: las dudas de Claude. **Responde las que puedas** en la columna *Respuesta*.

> ❗ Si **Leer PDF** sale en rojo con un error de "binary", entra al nodo **Formulario CV**, haz una prueba (**Test step**) y mira en la salida (pestaña *Binary*) cómo se llama el archivo. Pon ese nombre en **Leer PDF → Input Binary Field**.

### 7.2 Las vacantes (Flujo 2)
1. Abre el **Flujo 2** → clic en **Test workflow** (ejecuta desde *Probar ahora*).
2. Mira cómo se pone verde cada nodo ✅. Tarda 1 o 2 minutos, porque Claude analiza vacante por vacante.
3. Revisa la pestaña **Vacantes** de tu hoja 🎉 y tu Gmail.

Si algo falla, haz clic en el nodo rojo y lee el error. Los más comunes están en **Solución de problemas**, más abajo.

---

## Paso 8 – Dejarlo automático ⏰

1. En el **Flujo 2**, clic en el interruptor **Active**.
2. Listo: corre **de lunes a viernes a las 8:00 (hora de Ecuador)**.
3. ¿Otro horario? Doble clic en **Cada mañana 8:00** → *Expression*:
   - `0 8 * * 1-5` → lunes a viernes 8:00
   - `0 8 * * 1,3,5` → lunes, miércoles y viernes
   - `0 7,18 * * *` → todos los días a las 7:00 y a las 18:00

---

## Paso 9 – (Opcional) Upwork con Apify 💼

1. Crea tu cuenta en **https://apify.com** (tiene USD 5 gratis al mes).
2. En **Store** busca `Upwork jobs scraper` → elige uno con buenas reseñas → copia su **ID** (formato `usuario~nombre-actor`, sale en la URL o en la pestaña *API*).
3. **Settings → API & Integrations** → copia tu **API token**.
4. En n8n: **Add credential → Query Auth** → **Name:** `token` · **Value:** tu token de Apify.
5. En el nodo **Upwork (Apify)**:
   - En la URL cambia `PEGA_AQUI_EL_ID_DEL_ACTOR` por el ID del actor.
   - **Query Auth** → tu credencial.
   - Revisa en la página del actor (pestaña *Input*) cómo se llaman sus campos. Si no es `searchQueries`, ajusta el **JSON Body**.
   - Clic derecho en el nodo → **Activate** (deja de estar gris).

> 💡 Si quieres más ofertas **directo de LinkedIn**, puedes hacer lo mismo con un actor de Apify tipo `LinkedIn Jobs Scraper` (duplica el nodo de Upwork). Úsalo con moderación: va contra los términos de uso de LinkedIn.

---

## 📅 Tu rutina diaria (2 minutos por vacante)

1. Te llega el correo **"🎯 N vacantes nuevas donde puedes aplicar"**.
2. Abre tu hoja → pestaña **Vacantes** → filtra **Estado = Sin enviar** y ordena por **Match %**.
3. Por cada vacante:
   1. Abre el **Link** y confirma que la oferta sigue abierta.
   2. Lee el **Mail** y ajústalo. Reemplaza cualquier `[COMPLETAR: …]`.
   3. ¿Hay **Preguntas para mí**? Si la respuesta sirve para todo, agrégala en la pestaña **Preguntas**.
   4. **Envíalo tú:**
      - Si hay **Email** → desde tu Gmail, con el **Asunto** y tu CV adjunto.
      - Si dice **"Aplicar por link"** → postula en la página y usa el mail como carta o mensaje.
   5. Cambia **Estado → Enviado** y pon la **Fecha envío**.

---

## 🛠️ Solución de problemas

| Problema | Solución |
|---|---|
| *"No encontré tu perfil"* | Corre primero el Flujo 1. En la pestaña Perfil, la columna **Clave** debe decir `principal`. |
| Error **403 / 429** en "LinkedIn y otros" | Llave de RapidAPI mal puesta, no te suscribiste al plan Basic de JSearch, o se acabaron las 200 búsquedas del mes. |
| La hoja queda vacía | Puede que ninguna vacante pasara el 65 %. Baja el número en **¿Encaja?** (por ejemplo, a 55) o revisa las **Keywords**. |
| Error *"Could not parse"* en Claude | Pasa a veces. El flujo sigue con las demás vacantes. Si se repite mucho, sube **Maximum Number of Tokens** en el nodo *Claude (vacantes)*. |
| Columnas vacías en la hoja | Los encabezados de la fila 1 no coinciden letra por letra (tildes, `%`, mayúsculas). |
| RemoteOK sin resultados | Su búsqueda usa solo la **primera palabra** de cada keyword en inglés (por ejemplo, `data`, `marketing`). Es normal que a veces no traiga nada. |
| Aparecen vacantes de otra ciudad | Algunas ofertas "Remoto" son solo para EE. UU. Claude intenta descartarlas (`ubicacion_valida`). Si se cuela alguna, ignórala. |

---

## 📎 Anexo – Cada nodo explicado (para armarlo a mano o entenderlo)

### Flujo 1

| # | Nodo (tipo) | Qué hace | Configuración clave |
|---|---|---|---|
| 1 | **Formulario CV** (*n8n Form Trigger*) | Crea una página web con un formulario. Al enviarlo, arranca el flujo. | Campos: `CV` (File, `.pdf`), `Cargos que buscas`, `Modalidad` (Ambos / Solo Quito / Solo remoto), `Nivel`, `Idiomas y nivel`, `Salario esperado (USD/mes)`, `Algo más que deba saber` |
| 2 | **Leer PDF** (*Extract from File*) | Convierte el PDF en texto. | Operation: *Extract From PDF* · Input Binary Field: `CV` |
| 3 | **Generar perfil (Claude)** (*Basic LLM Chain*) | Le manda tu CV a Claude con instrucciones de reclutador. | Prompt: *Define below* · **Require Specific Output Format** activado · System message: "no inventes, convierte lo que falta en preguntas" |
| 3a | **Claude (perfil)** (*Anthropic Chat Model*) | Es "el cerebro" que usa la cadena. Se conecta abajo, en *Chat Model*. | Model `claude-sonnet-5-5` · Temperature 0.4 |
| 3b | **Formato del perfil** (*Structured Output Parser*) | Obliga a Claude a responder con un JSON de forma fija. | Campos: `nombre`, `resumen_profesional`, `cargos_objetivo[]`, `keywords_busqueda[]`, `habilidades_tecnicas[]`, `idiomas[]`, `logros_clave[]`, `preguntas_aclaracion[]`… |
| 4 | **Preparar fila Perfil** (*Code*) | Pasa el JSON a columnas de la hoja. `Clave = principal` para que siempre se actualice la misma fila. | — |
| 5 | **Guardar Perfil** (*Google Sheets*) | Escribe o actualiza tu perfil. | Operation: **Append or Update Row** · Column to match on: `Clave` · Mapping: *Map Automatically* |
| 6 | **Separar preguntas** (*Code*) | Convierte la lista de preguntas en una fila por pregunta. | — |
| 7 | **Guardar Preguntas** (*Google Sheets*) | Agrega las preguntas a la pestaña Preguntas. | Operation: **Append Row** · *Map Automatically* |

### Flujo 2

| # | Nodo (tipo) | Qué hace | Configuración clave |
|---|---|---|---|
| 1 | **Cada mañana 8:00** (*Schedule Trigger*) y **Probar ahora** (*Manual Trigger*) | Arrancan el flujo: uno automático, otro con botón. | Cron `0 8 * * 1-5` · Zona horaria del workflow: `America/Guayaquil` |
| 2 | **Leer Perfil** / **Leer Preguntas** (*Google Sheets*) | Traen tu perfil y tus respuestas. | Operation: **Get Row(s)** · *Leer Preguntas* tiene **Execute Once** y **Always Output Data** (para que no se rompa si la hoja está vacía) |
| 3 | **Armar perfil** (*Code*) | Junta perfil y respuestas en un texto (`perfilTexto`) y saca las palabras clave (máx. 4). | — |
| 4 | **Una búsqueda por palabra** (*Code*) | Crea 1 item por palabra clave. Así, cada nodo HTTP siguiente se ejecuta una vez por palabra. | — |
| 5 | **LinkedIn y otros · Quito** (*HTTP Request*) | Busca en JSearch: `"<palabra> in Quito, Ecuador"`. | GET `https://jsearch.p.rapidapi.com/search` · query, `country=ec`, `date_posted=week` · Header Auth + header `X-RapidAPI-Host` |
| 6 | **LinkedIn y otros · Remoto** (*HTTP Request*) | Busca en JSearch solo remotos. | igual + `remote_jobs_only=true` |
| 7 | **Remotive (remoto)** (*HTTP Request*) | Portal de trabajos 100 % remotos. | GET `https://remotive.com/api/remote-jobs?search=<palabra>&limit=20` |
| 8 | **RemoteOK (remoto)** (*HTTP Request*) | Otro portal remoto. | GET `https://remoteok.com/api?tag=<palabra>` + header `User-Agent` |
| 9 | **Upwork (Apify)** (*HTTP Request*, **desactivado**) | Proyectos freelance de Upwork. | POST `https://api.apify.com/v2/acts/<actor>/run-sync-get-dataset-items` · Query Auth `token` · **Execute Once** |
| — | Todos los HTTP | Si una fuente falla, el flujo sigue con las demás. | Settings → **On Error: Continue** |
| 10 | **Normalizar …** (*Code*, uno por fuente) | Convierte cada respuesta al mismo formato: `id, fuente, cargo, empresa, ubicacion, modalidad, link, descripcion, fecha`. Además limpia el HTML. | **Always Output Data** activado |
| 11 | **Unir fuentes** (*Merge*) | Junta las 5 listas en una. | Mode: **Append** · Number of Inputs: **5** |
| 12 | **Leer Vacantes guardadas** (*Google Sheets*) | Trae lo que ya tienes, para no repetir. | **Execute Once** + **Always Output Data** |
| 13 | **Quitar repetidas** (*Code*) | Quita duplicados (mismo link o mismo cargo+empresa), quita lo que ya está en la hoja, respeta tu modalidad (Solo Quito / Solo remoto) y se queda con máx. 25. | `MAX_POR_EJECUCION = 25` |
| 14 | **Analizar y redactar (Claude)** (*Basic LLM Chain*) | Por cada vacante: puntaje, razón, ubicación válida, email (solo si aparece), asunto, correo y preguntas. | System message con las reglas del correo (mismo idioma de la vacante, 120 a 180 palabras, sin inventar, `[COMPLETAR: …]`) |
| 14a | **Claude (vacantes)** + **Formato del análisis** | Modelo y formato JSON de la respuesta. | Campos: `match_score`, `razon`, `ubicacion_valida`, `email_contacto`, `mail_asunto`, `mail_cuerpo`, `preguntas_para_mi[]` |
| 15 | **Preparar fila Vacante** (*Code*, una vez por item) | Une la vacante con el análisis y arma la fila. **Estado = "Sin enviar"**. | Mode: *Run Once for Each Item* |
| 16 | **¿Encaja? (65 %+ y ubicación OK)** (*Filter*) | Deja pasar solo si `Match % ≥ 65` **y** `ubicacion_valida = true`. | Cambia 65 por lo que quieras |
| 17 | **Quitar campo auxiliar** (*Code*) | Quita `_ubicacion_valida` (no es columna de la hoja). | — |
| 18 | **Guardar en Vacantes** (*Google Sheets*) | Agrega las filas nuevas. | **Append Row** · *Map Automatically* |
| 19 | **Armar aviso** (*Code*) + **Avisarme por Gmail** (*Gmail*) | Te manda la lista de cargos ordenada por Match %. **Solo a ti.** | To: tu correo · Email Type: *Text* |

---

## 🚀 Ideas para mejorarlo después

- **Aviso por Telegram** en vez de Gmail: reemplaza el último nodo por *Telegram → Send Message* (ya tienes el bot de las pastillas).
- **Borrador en Gmail:** cuando cambias el Estado a "Listo para enviar", un tercer flujo (con *Google Sheets Trigger*) crea el **borrador** en tu Gmail con el CV adjunto. Tú solo das clic en Enviar.
- **Seguimiento:** un flujo semanal que te recuerda las vacantes "Enviado" de hace 7 días sin respuesta.
- **Más fuentes:** Computrabajo Ecuador, Multitrabajos o Get on Board, vía Apify o con sus propios RSS.
