# NEON CRUSH — Match-3 Arcade 💎

Juego Match-3 estilo Candy Crush en **un solo `index.html`** (GSAP por CDN, sin build).
Incluye Top 5 con nombre de hasta 10 caracteres, modo LOCAL / JSON en la nube / base de datos global.

### Mecánicas especiales
- **Agujeros negros** (desde 10K): al juntar 3, el sobreviviente **absorbe las partículas de
  todos** los fusionados (+1 por la unión). Con **3 partículas** se vuelve un **AGUJERO MASIVO**:
  un solo toque y forma una **SINGULARIDAD** que absorbe todo a su alrededor.
- **Lunas** (desde 20K): también acumulan partículas al fusionarse; con **4** colapsan en un
  agujero negro.
- **Bombas lineales apiladas**: si una bomba lineal activa a otra de la misma orientación,
  el poder se apila y barre **+2 líneas por bomba extra** (1 → 3 → 5 → 7…). El gráfico dibuja
  un haz por cada línea barrida, ondulando desde el centro hacia los bordes.
- **Regla de activación**: un poder SOLO se detona si lo tocas, o si el bloque que lo contiene
  va a ser destruido (match, explosión, zona de meteorito, etc.). Nunca se activa solo.
- **Meteoritos** (desde 30K): caen del cielo en tormentas, solo combinan de a 5+ y su explosión
  arrasa todo alrededor.

---

## ⚙️ Rendimiento y calidad gráfica (cualquier dispositivo)

El juego detecta el dispositivo la primera vez y elige un preset por sí solo, pero todo se
puede ajustar desde el **⚙️ (tuerca)** arriba a la derecha:

| Preset | Para qué dispositivos |
|---|---|
| **ALTA** | Gama alta (Pixel 9 Pro, PC, iPhone recientes) |
| **MEDIA** | Gama media y iPhone en Safari (quita brillos animados) |
| **MÍNIMA** | Gama baja / teléfonos viejos (sin partículas ni fondo) |

Además hay un botón **⚡ PROBAR MI DISPOSITIVO** que mide los FPS reales de tu GPU durante
medio segundo y aplica el preset ideal automáticamente.

Ajustes individuales: partículas de bloques, partículas de fondo, respiración de gemas,
animación de formas, vibración, destellos, **brillos/neón** (el que más FPS recupera en
iPhone) y **densidad de partículas** (baja/media/alta). Todo se guarda por dispositivo.

> **Nota iPhone/Safari:** lo que más ralentiza no son las partículas sino los *filtros
> animados* (`drop-shadow`, `backdrop-blur`, `hue-rotate`). Por eso el preset MEDIA y el
> toggle "Brillos y neón" los desactivan por completo.

---

## 🏷️ Versionado semántico automático

La versión vive en `index.html` como `const APP_VERSION = 'vX.Y.Z'` y se muestra bajo el
logo y en los ajustes.

- Cada **Pull Request** dispara `.github/workflows/auto-version.yml`, que incrementa el
  **patch** (`v1.1.0 → v1.1.1`) y hace commit en la rama del PR con el **título del PR
  como motivo**.
- Para saltos de **minor** o **major**, edita `APP_VERSION` a mano dentro del PR.
- Solo funciona en PRs desde ramas del mismo repo (los forks no permiten escribir de vuelta).

---

## ⚠️ ¿Por qué el ranking no puede ser un `.txt`/`.json` dentro del repo de GitHub?

GitHub Pages (y Netlify/Vercel) sirven sitios **estáticos**: el navegador que abre tu link
**no puede escribir archivos en tu repositorio**. Para lograrlo habría que incrustar un
*token personal de GitHub* en el código público del juego → cualquiera que abra el sitio
podría robarlo y **borrar o tomar control de tu repo**. Además, si dos jugadores guardan a
la vez, un archivo se sobreescribe con el otro (no hay bloqueos).

Por eso el juego usa una de estas dos alternativas **gratuitas**, que sí logran exactamente
lo que buscás: un "archivo JSON en la nube" o una DB liviana, visibles desde cualquier link.

| Modo | Dónde vive el ranking | Registro | Tag en el juego |
|---|---|---|---|
| LOCAL | `localStorage` de cada jugador | ninguno | `LOCAL` |
| ~~JSON~~ | ~~archivo JSON en jsonblob.com~~ | ~~ninguno~~ | ~~`JSON`~~ ⚠️ bloqueado |
| **GLOBAL** ✅ | Postgres gratis en Supabase | cuenta gratis | `GLOBAL` |

Con LOCAL funciona igual, pero cada dispositivo ve su propio Top 5.

---

## 🅰️ Opción JSON — jsonblob.com (⚠️ BLOQUEADO, usá la 🅱️)

> **⚠️ AVISO:** jsonblob.com está detrás de **Cloudflare y rechaza las llamadas a su
> API** con una página de bloqueo (error 403 "Attention Required"), tanto para leer como
> para guardar. Tus PUT/GET nunca llegan al servicio, así que el ranking no funciona.
> **Pasá directamente a la opción 🅱️ (Supabase)**: es gratis, confiable y ya está
> implementada en el juego.

1. **Creá el blob desde el navegador** (lo más fácil, sin terminal):

   - Entrá a https://jsonblob.com/editor/json
   - En el editor pegá exactamente: `[]`
   - Dale **Save** y copiá la **URL del blob** que te da (algo como
     `https://jsonblob.com/api/jsonBlob/1234567890123456789`).

   Verificá que exista abriendo esa URL en el navegador: debe mostrarte `[]`.
   Si ves una página **404**, el blob no se creó — repetí el paso.

   > Alternativa con curl (macOS / Linux / Git Bash). ⚠️ En **Windows PowerShell** el
   > alias `curl` NO acepta esta sintaxis; usá Git Bash o `curl.exe`:
   >
   > ```bash
   > curl -i -X POST https://jsonblob.com/api/jsonBlob \
   >   -H "Content-Type: application/json" -H "Accept: application/json" -d "[]"
   > ```
   >
   > Copiá la URL de la cabecera `Location:` de la respuesta.

2. Abrí `index.html`, buscá al inicio del `<script>` y pegá esa URL. El juego acepta
   **ambos formatos** (la URL de la API y la del visor `jsonblob.com/<id>`) y la
   normaliza solo:

   ```js
   const JSONBLOB_URL = 'https://jsonblob.com/api/jsonBlob/1234567890123456789';
   ```

3. **Verificá que el blob sea público** (importante: si lo creaste con tu cuenta puede
   ser privado y responder 404 al resto del mundo).

   **Desde la consola del navegador** (la más rápida): abrí el juego, `F12` → pestaña
   *Console*, y ejecutá:

   ```js
   NC_LB_TEST()
   ```

   Si todo anda vas a ver `✔ GET ok`, `✔ PUT ok` y `✔ blob restaurado — TODO FUNCIONA`.
   La prueba escribe una entrada temporal y después deja el blob exactamente como estaba.

   **Con Postman:**

   - **Leer (GET):** creá una request `GET` a
     `https://jsonblob.com/api/jsonBlob/TU_ID` y mandala.
     → debe responder **200 OK** con el contenido (`[]` al principio).
     → si responde **404**, el blob no existe o es privado.
   - **Guardar (PUT):** creá una request `PUT` a la misma URL, en *Headers* agregá
     `Content-Type: application/json` y `Accept: application/json`, y en *Body* →
     *raw* / *JSON* pegá:

     ```json
     [{"name":"POSTMAN","score":9999}]
     ```

     → debe responder **200 OK**. Repetí el GET: ahora tiene que devolver esa entrada.
     (Terminá haciendo otro PUT con `[]` para limpiar tu prueba.)

4. Listo: el tag del ranking pasa a decir **JSON** y todos los que jueguen con tu link
   ven y alimentan el mismo Top 5.

> Nota honesta: jsonblob es un servicio gratuito sin garantías y hoy pide cuenta para
> guardar — si tu blob queda privado el ranking no funciona para el resto (el test de
> arriba lo detecta). Además quien conozca la URL del blob podría editarlo, y dos
> guardados simultáneos podrían pisarse (rarísimo con pocos jugadores).
> Para algo robusto y realmente compartido usá la opción 🅱️ (Supabase).

---

## 🅱️ Opción DB — Supabase (recomendada, 3 minutos)

Base de datos Postgres real, gratis, con inserciones atómicas (nunca se pierden puntajes).
La clave `anon` **es pública por diseño**: las políticas RLS de abajo solo permiten leer e
insertar puntajes, nada más.

1. Creá un proyecto gratis en https://supabase.com
2. Abrí **SQL Editor**, pegá esto y dale *Run*:

   ```sql
   create table neon_scores (
     id bigint generated always as identity primary key,
     name text not null,
     score integer not null,
     created_at timestamptz default now()
   );
   alter table neon_scores enable row level security;
   create policy "leer scores"   on neon_scores for select using (true);
   create policy "insert scores" on neon_scores for insert with check (true);
   ```

3. En **Settings → API** copiá la **URL del proyecto** y la clave **anon public**.
4. Pegalas en `index.html`:

   ```js
   const SUPA_URL = 'https://xxxx.supabase.co';
   const SUPA_KEY = 'eyJhbGciOi...';
   ```

El tag del ranking pasa a decir **GLOBAL**.

> ⚠️ Como jsonblob está bloqueado por Cloudflare, **dejá `JSONBLOB_URL` vacío** y
> completá solo `SUPA_URL` + `SUPA_KEY` para tener el ranking mundial funcionando.

---

## 📡 Sin conexión a internet (modo OFFLINE)

Si el jugador pierde la conexión (o el servicio no responde), el juego automáticamente:

1. Muestra y compara contra su **top 5 local** — la etiqueta del ranking cambia a
   **OFFLINE** en ámbar y la nota explica el estado.
2. Si el puntaje entra en el top 5, se guarda igual y queda **en cola**
   (el botón dice `QUEUED` y se avisa "se sincronizará al volver internet").
3. Cuando vuelve la conexión — evento `online` del navegador o reintento automático
   cada 20 segundos — la cola se **sincroniza sola con el ranking global**: aparece el
   aviso `SYNCED N SCORES → GLOBAL` con explosión de partículas y el Top 5 se refresca.

Sin conexión nadie pierde su puntaje; con conexión, todos comparten el mismo ranking.

---

## 🚀 Publicar y compartir el link

Solo necesitás `index.html` (GSAP y las fuentes se cargan por CDN).

**GitHub Pages (con git):**

```bash
git init
git add index.html README.md
git commit -m "Neon Crush"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/neon-crush.git
git push -u origin main
```

Luego en GitHub: **Settings → Pages → Branch: `main` → Save**.
En ~1 minuto tu juego estará en `https://TU_USUARIO.github.io/neon-crush/`

**Netlify Drop (30 segundos, sin git):** entrá a `https://app.netlify.com/drop`,
arrastrá la carpeta con `index.html` y te da un link público al instante.

---

## 🎮 Controles

- Arrastrar / swipe o tocar dos gemas vecinas → intercambiar.
- Un solo toque en un poder → detonarlo (arrástralo para intercambiarlo).
- Línea de 4 → LINE BOMB · 5+ gemas (línea, L o T) → COLOR BOMB.
- Match de 4+ → **+2 jugadas** · 10s sin tocar → pista automática (−1 jugada).
- Botón **CRUSH** (se carga cada 60s de juego) → desordena todo el tablero.
