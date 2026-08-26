# 💎 NEON CRUSH — Match-3 Arcade

Juego Match-3 estilo Candy Crush en **un solo archivo `index.html`** (GSAP por CDN, sin assets ni build obligatorio).
8×8 · 6 gemas neón · Line Bombs · Color Bombs · combos en cadena · botón CRUSH · Top 5 compartido.

---

## 🚀 Subirlo y compartir el link (3 opciones)

El juego es 100% estático: **basta con servir `index.html`**.

### Opción A — GitHub Pages (con git)
```bash
git init
git add index.html README.md
git commit -m "Neon Crush"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/neon-crush.git
git push -u origin main
```
Luego en GitHub: **Settings → Pages → Branch: main → (root) → Save**.
En ~1 minuto tu juego estará en `https://TU_USUARIO.github.io/neon-crush/`.

### Opción B — Netlify Drop (la más rápida, sin git)
1. Entra a https://app.netlify.com/drop
2. Arrastra la carpeta que contiene `index.html`.
3. Te da un link público al instante (y puedes cambiarle el nombre).

### Opción C — Vercel
```bash
npm i -g vercel
vercel          # acepta los defaults; framework: Other
```

---

## 🏆 Ranking TOP 5 compartido (para todos los jugadores)

Por defecto el ranking es **LOCAL** (cada navegador guarda su propio Top 5).
Para que el Top 5 sea **global** (el mismo para todos los que jueguen tu link), usa
[Supabase](https://supabase.com) gratis — toma ~3 minutos:

### 1. Crea el proyecto
1. Regístrate gratis en https://supabase.com → **New project** (contraseña y región cualquiera).
2. En el menú lateral: **SQL Editor → New query**, pega esto y dale **Run**:

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

### 2. Copia tus credenciales
En **Project Settings → API** copia:
- **Project URL** (ej. `https://xxxx.supabase.co`)
- **anon public key** (la `anon` / `public`, NO la `service_role`)

### 3. Pégalas en el juego
Abre `index.html` y busca (arriba del script):

```js
const SUPA_URL = '';
const SUPA_KEY = '';
```

Déjalas así:

```js
const SUPA_URL = 'https://xxxx.supabase.co';
const SUPA_KEY = 'eyJhbGciOi...';
```

Listo: la etiqueta del ranking cambiará de **LOCAL** a **GLOBAL** y todos los
jugadores verán y alimentarán el mismo Top 5. Si el servicio se cae o no hay
internet, el juego sigue funcionando con el ranking local.

> ℹ️ La clave `anon` es pública por diseño (se expone en el navegador) y las
> políticas RLS de arriba solo permiten leer e insertar scores.

---

## 🎮 Reglas rápidas
- **25 jugadas** · match de 4+ da **+2 jugadas** · combos en cascada multiplican puntos.
- **5 gemas** (línea de 5, o 3+3 en L/T) → **Color Bomb** · **4 en línea** → **Line Bomb**.
- **Doble clic/toque** en un especial lo detona; los especiales **se encadenan**.
- **CRUSH** (se carga cada 60 s de juego): desordena todo el tablero.
- 10 s sin jugar → el juego te muestra un combo y te descuenta 1 jugada.
- Récord personal en **cookie**; Top 5 con nombre de **máx. 10 caracteres**.
