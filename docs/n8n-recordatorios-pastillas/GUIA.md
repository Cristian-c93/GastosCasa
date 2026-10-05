# 💊 Recordatorios de pastillas: Telegram → n8n → Google Calendar

Le escribes a tu bot de Telegram:

> Tomar ibuprofeno cada 8 horas por tres días

y en tu Google Calendar aparecen **9 recordatorios**, uno cada 8 horas. El bot te responde: *"✅ Listo: creé 9 recordatorios de Ibuprofeno"*.

Tu n8n: `https://vps-3c0c0def.vps.ovh.ca` · Zona horaria: **Colombia (America/Bogota)**

---

## 🧠 ¿Cómo funciona? (la idea en simple)

Piensa en una cadena de 5 ayudantes. Cada uno hace una sola cosa y le pasa el trabajo al siguiente:

```
📱 Tú escribes en Telegram
   ↓
1. 👂 Telegram Trigger  → "¡Llegó un mensaje!"
2. 🧮 Entender mensaje  → "Es ibuprofeno, cada 8 h, 3 días = 9 tomas" y arma la lista de horas
3. ❓ ¿Entendí?         → Si el mensaje está mal escrito, te avisa cómo escribirlo
4. 📅 Crear evento      → Pone cada toma en Google Calendar
5. ✅ Confirmar         → Te dice por Telegram que ya quedó listo
```

Cuenta de ejemplo: un día tiene 24 horas. 24 ÷ 8 = 3 tomas por día. 3 tomas × 3 días = **9 tomas**.

---

## ✏️ Lo que TÚ tienes que cambiar con tus datos

El resto lo copias tal cual.

| # | Qué | Dónde se pone | De dónde lo sacas |
|---|-----|---------------|-------------------|
| 1 | Token del bot | n8n → Credencial *Telegram API* | @BotFather (Paso 1) |
| 2 | Client ID y Client Secret | n8n → Credencial *Google Calendar OAuth2 API* | Google Cloud (Paso 2) |
| 3 | URL de redirección | Google Cloud → tu ID de cliente OAuth | La copias de n8n (Paso 3) |
| 4 | Tu Gmail como "usuario de prueba" | Google Cloud → Pantalla de consentimiento | Tu correo |
| 5 | Calendario | Nodo *Crear evento* → Calendar | Tu correo (calendario principal) |
| 6 | (Opcional) Tu número de chat | Nodo *¿Entendí?* | Sale en la primera prueba (Paso 8) |

La zona horaria ya está puesta para Colombia. Si algún día te mudas, cámbiala en 2 lugares: la primera línea del nodo *Entender mensaje* (`const ZONA = ...`) y en ⋯ → Settings → Timezone del workflow.

---

## Paso 0 – Lo que necesitas

- ✅ Tu celular con Telegram
- ✅ Una cuenta de Google (Gmail)
- ✅ Un computador con tu n8n abierto: `https://vps-3c0c0def.vps.ovh.ca`
- ✅ El archivo `workflow.json`, que está en esta misma carpeta
- ⏱️ Entre 30 y 45 minutos. No hay afán, ve paso por paso.

---

## Paso 1 – Crear tu bot de Telegram 🤖

Un "bot" es como un contacto robot de Telegram. Al bot le vas a escribir las pastillas.

1. Abre Telegram en tu celular.
2. Arriba, en la lupa 🔍, busca **BotFather**. Elige el que tiene **la palomita azul ✔️**, porque es el oficial.
3. Toca **Iniciar** (o *Start*).
4. Escribe `/newbot` y envíalo.
5. Te pide un **nombre**. Escribe algo como `Mis Pastillas` y envíalo.
6. Te pide un **usuario**, y este tiene que terminar en `bot`. Por ejemplo: `cristian_pastillas_bot`. Si te dice que ya existe, prueba otro (agrégale números).
7. BotFather te responde con un mensaje largo donde aparece algo así:
   ```
   123456789:AAHk3j...xyz
   ```
   Ese es tu **TOKEN** 🔑. Cópialo y guárdalo en una nota. **Es como la contraseña del bot, no se la muestres a nadie.**
8. En ese mismo mensaje hay un enlace `t.me/tu_bot`. Tócalo y presiona **Iniciar**. Si no lo haces, el bot no podrá escribirte.

---

## Paso 2 – Darle permiso a n8n para usar tu Google Calendar 🔐

Este es el paso **más largo**. Google pide crear un "proyecto" para darle permiso a n8n de escribir en tu calendario. Hazlo en el **computador**.

### 2.1 Crear el proyecto
1. Entra a **https://console.cloud.google.com** con tu Gmail.
2. Si te pide aceptar términos, acéptalos.
3. Arriba a la izquierda hay un botón que dice *"Selecciona un proyecto"*. Haz clic.
4. Clic en **Proyecto nuevo**.
5. Nombre: `n8n-pastillas` → **Crear**.
6. Espera unos segundos y vuelve a hacer clic arriba para **seleccionar** `n8n-pastillas`. Revisa que su nombre aparezca arriba.

### 2.2 Encender el Google Calendar
1. Menú **☰** (las 3 rayitas, arriba a la izquierda) → **APIs y servicios** → **Biblioteca**.
2. En el buscador escribe `Google Calendar API`.
3. Haz clic en el resultado → botón azul **Habilitar**.

### 2.3 La "pantalla de permiso" (pantalla de consentimiento)
Es la ventanita que sale cuando una app te pregunta *"¿Dejas que esta app use tu cuenta?"*.

1. Menú **☰** → **APIs y servicios** → **Pantalla de consentimiento de OAuth**. En algunas cuentas se llama **Google Auth Platform**; si te sale *"Comenzar"*, dale clic.
2. **Nombre de la app:** `n8n`
3. **Correo de asistencia:** tu Gmail.
4. **Público / Tipo de usuario:** elige **Externo**.
5. **Información de contacto:** tu Gmail.
6. Acepta y dale **Crear** o **Guardar**.
7. Busca la sección **Público** (o **Usuarios de prueba**) → **Agregar usuarios** → escribe **tu propio Gmail** → Guardar.
   ⚠️ **No te saltes esto.** Si lo olvidas, Google te dirá "Acceso bloqueado".

### 2.4 Crear la "llave" para n8n
1. Menú **☰** → **APIs y servicios** → **Credenciales** (o en Google Auth Platform: **Clientes**).
2. **+ Crear credenciales** → **ID de cliente de OAuth**.
3. **Tipo de aplicación:** `Aplicación web`.
4. **Nombre:** `n8n`.
5. Baja hasta **URI de redireccionamiento autorizados** → **+ Agregar URI** y pega:
   ```
   https://vps-3c0c0def.vps.ovh.ca/rest/oauth2-credential/callback
   ```
   (En el Paso 3.2 vas a confirmar que n8n te muestre exactamente esta misma dirección. Si te muestra otra, pon la de n8n.)
6. **Crear**.
7. Te aparecen 2 cosas: **ID de cliente** y **Secreto del cliente**. Cópialas en tu nota. 📝 *(El secreto es otra contraseña, no lo compartas.)*

---

## Paso 3 – Guardar las llaves dentro de n8n 🗝️

### 3.1 La llave de Telegram
1. Abre `https://vps-3c0c0def.vps.ovh.ca` e inicia sesión.
2. A la izquierda busca **Credentials** (o entra a *Overview → Credentials*). Clic en **Add credential / Create credential**.
3. Escribe `Telegram` y elige **Telegram API** → Continue.
4. En **Access Token** pega el **token** del Paso 1.
5. **Save**. Si sale un mensaje verde ✅, quedó bien.

### 3.2 La llave de Google
1. Otra vez **Add credential** → escribe `Google Calendar` → elige **Google Calendar OAuth2 API**.
2. Arriba vas a ver **OAuth Redirect URL**. Revisa que sea igual a la que pusiste en el Paso 2.4. Si es diferente, vuelve a Google Cloud y pon esta.
3. Pega el **Client ID** y el **Client Secret** del Paso 2.4.
4. Clic en **Sign in with Google**. Se abre una ventanita:
   - Elige tu cuenta de Gmail.
   - Si sale **"Google no verificó esta app"**, no te asustes: es tu propia app. Clic en **Configuración avanzada** → **Ir a n8n (no seguro)**.
   - Marca todas las casillas de permisos → **Continuar / Permitir**.
5. La ventanita se cierra y n8n dice **Connected** ✅ → **Save**.

---

## Paso 4 – Traer el workflow a n8n 📥

Un *workflow* es la cadena de ayudantes que viste arriba. Ya está hecho en el archivo `workflow.json`, así que solo lo importas.

1. Descarga `workflow.json` a tu computador.
2. En n8n: **+ (Create) → Workflow**.
3. Arriba a la derecha, en el menú **⋯** (tres puntos) → **Import from File...** → elige `workflow.json`.
4. Vas a ver 6 cuadritos (nodos) conectados con flechas. 🎉

> 💡 **¿Prefieres armarlo a mano para entenderlo mejor?** Mira el *Anexo* al final, ahí está cada nodo explicado.

---

## Paso 5 – Ponerle las llaves a cada nodo 🔌

Los nodos que tienen un **triángulo rojo ⚠️** todavía no tienen llave. Haz esto con cada uno:

| Nodo | Doble clic y en **Credential to connect with** elige… |
|------|------|
| **Telegram Trigger** | tu credencial *Telegram account* |
| **Crear evento** | tu credencial *Google Calendar account*. **Además**, en **Calendar** elige tu correo de la lista |
| **Confirmar** | *Telegram account* |
| **No entendí** | *Telegram account* |

Cierra cada nodo con la ✖ o haciendo clic afuera. Al final no debe quedar ningún triángulo rojo.

---

## Paso 6 – Revisar la hora 🕗

1. Menú **⋯** → **Settings**.
2. **Timezone**: debe decir `America/Bogota`. Si no, búscala y elígela.
3. **Save**.

¿Por qué? Tu VPS está en **Canadá**, y si no le dices que estás en Colombia, pondría las pastillas a una hora equivocada.

---

## Paso 7 – Que te suene la alarma 🔔

n8n crea los eventos, pero quien te **avisa** es Google Calendar. Configúralo así:

1. En el computador entra a **calendar.google.com** → **⚙️ (engranaje) → Configuración**.
2. A la izquierda, en *"Configuración de mis calendarios"*, haz clic en **tu nombre**.
3. Busca **Notificaciones de eventos** → **Agregar notificación** → `Notificación` · `0` · `minutos`.
   (Así te suena **justo a la hora**. Si quieres un aviso antes, agrega otra con 5 minutos.)
4. En el celular instala la app **Google Calendar** y, en los ajustes del celular, deja **activadas sus notificaciones**.

---

## Paso 8 – ¡Probar! 🧪

1. En n8n guarda con **Ctrl + S**.
2. Abajo en el centro haz clic en **Test workflow / Execute workflow**. n8n se queda "escuchando" 👂.
3. En tu celular, escríbele a **tu bot**:
   ```
   Tomar ibuprofeno cada 8 horas por tres días
   ```
4. En n8n los nodos se ponen **verdes** ✅ uno por uno.
5. El bot te responde: *"✅ Listo: creé 9 recordatorios de Ibuprofeno"*.
6. Abre Google Calendar: ahí están **💊 Tomar Ibuprofeno (1/9)**, **(2/9)**…
7. Si funcionó: arriba a la derecha pon el interruptor **Inactive → Active** (o **Publish**). 🟢
   Desde ahora funciona **siempre**, aunque cierres n8n y apagues el computador.

> Si fue solo una prueba y quieres borrar esos 9 eventos, bórralos a mano en Google Calendar.

---

## 📝 Cómo escribirle al bot

El bot entiende frases que tengan **"cada X horas"** y **"por Y días"**. Lo demás es opcional:

| Mensaje | Qué hace |
|---|---|
| `Ibuprofeno cada 8 horas por 3 días` | 9 tomas empezando **ahora** |
| `Tomar ibuprofeno cada 8 horas por tres días` | Igual (entiende números escritos en letras) |
| `Amoxicilina cada 12 horas por 7 días desde las 8 pm` | 14 tomas, la primera hoy a las 8 pm (o mañana, si esa hora ya pasó) |
| `Loratadina cada 24 horas durante 2 semanas a partir de las 7:30 am` | 14 tomas diarias a las 7:30 am |
| `hola` | El bot te explica cómo escribirlo |

Consejo: si no escribes *am* o *pm*, usa la hora de 24 horas (por ejemplo, `desde las 20` = 8 pm).

---

## 🆘 Si algo sale mal

| Problema | Solución |
|---|---|
| El **Telegram Trigger** da error de *webhook* o *HTTPS* | Tu n8n necesita saber su dirección pública. En el VPS, donde está la configuración de n8n (archivo `.env` o `docker-compose.yml`), agrega `WEBHOOK_URL=https://vps-3c0c0def.vps.ovh.ca/` y reinicia n8n. |
| Google dice **"Acceso bloqueado"** | Te faltó agregar tu Gmail como usuario de prueba (Paso 2.3, punto 7). |
| Google dice **redirect_uri_mismatch** | La dirección del Paso 2.4 no es **idéntica** a la que muestra n8n (Paso 3.2). Cópiala otra vez, sin espacios. |
| Las pastillas salen a **horas raras** | Revisa la zona horaria (Paso 6) y la primera línea del nodo *Entender mensaje*. |
| Funciona en prueba pero **no cuando está activo** | Revisa que el interruptor esté en **Active** y que **ningún otro workflow** use el mismo bot (Telegram solo deja uno). |
| El bot manda **muchos mensajes de "Listo"** | En el nodo *Confirmar* → pestaña **Settings** → activa **Execute Once**. |
| Cada 7 días Google **se desconecta** | Google Cloud → Pantalla de consentimiento / Público → **Publicar app**. Después vuelve a hacer *Sign in with Google* en n8n. |
| No te suena nada | Revisa el Paso 7 y que el celular no esté en "No molestar". |

---

## 🔒 Extra (opcional): que solo TÚ puedas usar el bot

Cualquiera que encuentre tu bot podría escribirle y llenarte el calendario. Para evitarlo:

1. Haz una prueba (Paso 8) y abre el nodo **Telegram Trigger**. En la salida busca `message → chat → id`, un número como `987654321`. Ese es **tu número de chat**.
2. Abre el nodo **¿Entendí?** → **Add condition**:
   - Valor 1: `{{ $json.chatId }}`
   - Tipo: **Number → is equal to**
   - Valor 2: tu número.
3. Guarda. Ahora los mensajes de otras personas se van por la salida *false* y no crean nada.

---

## 📎 Anexo: armar el workflow a mano (si no lo importaste)

### Nodo 1 – Telegram Trigger
`+` → busca **Telegram** → **On message**. Credential: tu Telegram. **Updates:** `message`.

### Nodo 2 – Code (nombre: `Entender mensaje`)
`+` → **Code**. **Mode:** *Run Once for All Items*. **Language:** JavaScript. Borra lo que trae y pega esto:

```js
// ===== CAMBIA ESTO si vives en otro país =====
const ZONA = 'America/Bogota';
// =============================================

const msg = $input.first().json.message;
const chatId = msg.chat.id;
let texto = (msg.text || '').toLowerCase();

// 1) Cambiar números escritos con letras por cifras ("tres" -> 3)
const numeros = {
  un: 1, uno: 1, una: 1, dos: 2, tres: 3, cuatro: 4, cinco: 5, seis: 6,
  siete: 7, ocho: 8, nueve: 9, diez: 10, once: 11, doce: 12, catorce: 14,
  quince: 15, veinte: 20, treinta: 30,
};
texto = texto.split(/(\s+)/).map(p => numeros[p] ?? p).join('');

// 2) Buscar "cada X horas" y "por Y días" (o "por Y semanas")
const mHoras = texto.match(/cada\s+(\d+)\s*h/);
const mDias = texto.match(/(?:por|durante)\s+(\d+)\s*(d[ií]a|semana)/);
if (!mHoras || !mDias) {
  return [{ json: { error: true, chatId } }];
}
const cadaHoras = parseInt(mHoras[1]);
const dias = parseInt(mDias[1]) * (mDias[2] === 'semana' ? 7 : 1);
if (cadaHoras < 1) {
  return [{ json: { error: true, chatId } }];
}

// 3) Nombre de la pastilla = lo que está antes de "cada"
let pastilla = texto.split('cada')[0].replace(/^\s*(tomar|toma|tomarme)\s+/, '').trim();
pastilla = pastilla.charAt(0).toUpperCase() + pastilla.slice(1);

// 4) Hora de la primera toma: "desde las 8", "a partir de las 7:30 pm".
//    Si no dices nada, empieza AHORA.
const ahora = DateTime.now().setZone(ZONA);
let inicio = ahora.set({ second: 0, millisecond: 0 });
const mIni = texto.match(/(?:desde|a partir de|empezando)\s+(?:a\s+)?(?:las?\s+)?(\d{1,2})(?::(\d{2}))?\s*(am|pm|a\.\s*m\.?|p\.\s*m\.?)?/);
if (mIni) {
  let h = parseInt(mIni[1]);
  const m = parseInt(mIni[2] || '0');
  if (mIni[3] && mIni[3].startsWith('p') && h < 12) h += 12;
  if (mIni[3] && mIni[3].startsWith('a') && h === 12) h = 0;
  inicio = inicio.set({ hour: h, minute: m });
  if (inicio < ahora) inicio = inicio.plus({ days: 1 }); // si esa hora ya pasó, mañana
}

// 5) Armar la lista de tomas (una por cada evento del calendario)
const total = Math.floor((dias * 24) / cadaHoras);
const items = [];
for (let i = 0; i < total; i++) {
  const t = inicio.plus({ hours: i * cadaHoras });
  items.push({
    json: {
      error: false,
      chatId,
      pastilla,
      total,
      titulo: `💊 Tomar ${pastilla} (${i + 1}/${total})`,
      inicio: t.toISO(),
      fin: t.plus({ minutes: 15 }).toISO(),
      primera: inicio.toFormat('dd/MM/yyyy hh:mm a'),
    },
  });
}
return items;
```

### Nodo 3 – If (nombre: `¿Entendí?`)
`+` → **If**. Condición: valor `{{ $json.error }}` · **Boolean → is false**.
- Salida **true** → Nodo 4.
- Salida **false** → Nodo 6.

### Nodo 4 – Google Calendar (nombre: `Crear evento`)
`+` → **Google Calendar** → **Create an event**.
- **Calendar:** tu correo
- **Start:** `{{ $json.inicio }}`
- **End:** `{{ $json.fin }}`
- **Add Field → Summary:** `{{ $json.titulo }}`
- **Add Field → Description:** `Recordatorio creado desde Telegram`

### Nodo 5 – Telegram (nombre: `Confirmar`), después del Nodo 4
**Telegram → Send a text message**.
- **Chat ID:** `{{ $('Entender mensaje').first().json.chatId }}`
- **Text:**
  ```
  ✅ Listo: creé {{ $('Entender mensaje').first().json.total }} recordatorios de {{ $('Entender mensaje').first().json.pastilla }}.
  Primera toma: {{ $('Entender mensaje').first().json.primera }}
  ```
- **Add Field → Append n8n Attribution:** apagado.
- Pestaña **Settings** → **Execute Once: ON** ⚠️ (si no, te llegan 9 mensajes).

### Nodo 6 – Telegram (nombre: `No entendí`), en la salida *false* del Nodo 3
**Telegram → Send a text message**.
- **Chat ID:** `{{ $json.chatId }}`
- **Text:** `No te entendí 😅 Escríbelo así: Ibuprofeno cada 8 horas por 3 días desde las 8 am`

---

⚕️ *Esta herramienta solo te recuerda las tomas. Las dosis y los horarios siempre los decide tu médico.*
