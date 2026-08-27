# NEON CRUSH — Match-3 Arcade 💎

Juego Match-3 estilo Candy Crush en **un solo `index.html`** (GSAP por CDN, sin build).
Incluye Top 5 con nombre de hasta 10 caracteres, modo LOCAL / JSON en la nube / base de datos global.

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
| JSON | archivo JSON en jsonblob.com | **ninguno** | `JSON` |
| GLOBAL | Postgres gratis en Supabase | cuenta gratis | `GLOBAL` |

Con LOCAL funciona igual, pero cada dispositivo ve su propio Top 5.

---

## 🅰️ Opción JSON — ranking mundial sin crear cuentas (2 minutos)

1. Creá el "archivo" JSON. Con curl (Git Bash / Terminal / PowerShell):

   ```bash
   curl -i -X POST https://jsonblob.com/api/jsonBlob \
     -H "Content-Type: application/json" -H "Accept: application/json" -d "[]"
   ```

   Copiá la URL que aparece en la cabecera `Location:` de la respuesta, algo como:
   `https://jsonblob.com/api/jsonBlob/1234567890123456789`

   (Alternativa: entrá a https://jsonblob.com, pegá `[]`, guardalo y copiá la URL del blob.)

2. Abrí `index.html`, buscá al inicio del `<script>` y pegá esa URL:

   ```js
   const JSONBLOB_URL = 'https://jsonblob.com/api/jsonBlob/1234567890123456789';
   ```

3. Listo: el tag del ranking pasa a decir **JSON** y todos los que jueguen con tu link
   ven y alimentan el mismo Top 5.

> Nota honesta: jsonblob es un servicio gratuito sin garantías; quien conozca la URL del
> blob podría editarlo, y dos guardados simultáneos podrían pisarse (rarísimo con pocos
> jugadores). Para algo más robusto usá la opción 🅱️.

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

El tag del ranking pasa a decir **GLOBAL**. (Si completás ambas opciones, JSON gana por
prioridad; dejá `JSONBLOB_URL` vacío para usar Supabase.)

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
- Doble clic / doble toque en un poder → detonarlo.
- Línea de 4 → LINE BOMB · 5+ gemas (línea, L o T) → COLOR BOMB.
- Match de 4+ → **+2 jugadas** · 10s sin tocar → pista automática (−1 jugada).
- Botón **CRUSH** (se carga cada 60s de juego) → desordena todo el tablero.
