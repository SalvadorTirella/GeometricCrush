# NEON CRUSH — Documentación Técnica Completa
### Match-3 Arcade · Motor GSAP · Single-File
**Versión documentada:** v1.4.3 · **Fecha:** 2026

> Documento de referencia integral: diseño visual (UI), experiencia de usuario (UX),
> motor de juego (lógica), sistemas de partículas/animación/audio, rendimiento,
> dificultad, persistencia, base de datos y despliegue. Escrito para que cualquier
> ingeniero pueda entender, mantener y extender el juego sin leer las 3.000+ líneas de código.

---

## ÍNDICE

1.  [Resumen ejecutivo](#1-resumen-ejecutivo)
2.  [Stack tecnológico y decisiones de arquitectura](#2-stack-tecnológico)
3.  [Anatomía del archivo único](#3-anatomía-del-archivo)
4.  [Máquina de estados](#4-máquina-de-estados)
5.  [Diseño visual (UI)](#5-diseño-visual-ui)
6.  [Diseño de experiencia (UX)](#6-diseño-de-experiencia-ux)
7.  [Motor de juego (lógica)](#7-motor-de-juego-lógica)
8.  [Sistema de partículas](#8-sistema-de-partículas)
9.  [Sistema de animaciones (GSAP)](#9-sistema-de-animaciones-gsap)
10. [Sistema de audio (WebAudio)](#10-sistema-de-audio-webaudio)
11. [Rendimiento y ajustes gráficos](#11-rendimiento-y-ajustes-gráficos)
12. [Sistema de dificultad cósmica](#12-sistema-de-dificultad-cósmica)
13. [Persistencia y base de datos](#13-persistencia-y-base-de-datos)
14. [Versionado semántico automático](#14-versionado-semántico-automático)
15. [Despliegue](#15-despliegue)
16. [Cómo se resolvió cada problema](#16-cómo-se-resolvió-cada-problema)

---

## 1. Resumen ejecutivo

**NEON CRUSH** es un juego Match-3 estilo Candy Crush con estética neón/cyberpunk y
temática cósmica, contenido **íntegramente en un solo archivo `index.html`**
(~3.000 líneas). No usa frameworks de juego, ni build step, ni assets externos de
imagen: toda la geometría es **CSS puro** (clip-path, gradientes) y todo el movimiento
es **GSAP sobre el DOM** + un canvas para el fondo.

**Cifras clave:**
- Tablero: **12 filas × 8 columnas** (96 celdas)
- Gemas base: **6** (círculo, cuadrado, triángulo, rombo, hexágono, estrella)
- Gemas desbloqueables: **3** (agujero negro 10K, sol 20K, meteorito 30K)
- Fichas especiales: **4** (bomba lineal H/V, bomba de color, singularidad)
- Movimientos iniciales: **10** (varía según dificultad)
- Dificultades: **3** (Planeta, Sol, Agujero Negro)
- Ranking: **Top 5** con 3 backends (LOCAL / JSON / Supabase)

---

## 2. Stack tecnológico

| Capa | Tecnología | Justificación |
|------|-----------|---------------|
| Estructura | HTML5 + CSS3 + Vanilla JS (ES2020) | Cero dependencias de build, un solo archivo portable |
| Animación | **GSAP 3.12.5** (CDN, con fallback) | Motor de tween más fluido del mercado; timelines, easing `back.out`, stagger |
| Fondo | **Canvas 2D** | ~170 partículas + meteoros a 60 FPS sin castigar el DOM |
| Tipografía | **Orbitron** (display) + **Rajdhani** (cuerpo) — Google Fonts | Orbitron = sci-fi numérico; Rajdhani = legible y condensada |
| Audio | **WebAudio API** (osciladores sintéticos) | Sin archivos de sonido; todo generado en runtime |
| Persistencia | **localStorage** + **Cookie** + **Supabase REST** / **JSONBlob** | Récord local, ranking global, cola offline |
| Hosting | Estático (GitHub Pages / Netlify) | El archivo es 100% autocontenido |

**¿Por qué GSAP sobre DOM y no un framework de juegos (Phaser, PixiJS)?**
- Las gemas son elementos reales del DOM → accesibilidad, eventos táctiles nativos,
  estilos CSS complejos (gradientes, máscaras, recortes) gratis.
- GSAP garantiza 60 FPS con `will-change: transform` y composición en GPU.
- Un solo archivo, sin bundler: el juego se comparte copiando un link.

---

## 3. Anatomía del archivo

El `index.html` está organizado en bandas claramente comentadas:

```
<!DOCTYPE html>
├─ <head>
│   ├─ meta viewport (zoom bloqueado, viewport-fit=cover)
│   ├─ favicon SVG inline (rombo neón)
│   ├─ Google Fonts (Orbitron + Rajdhani)
│   ├─ GSAP CDN + fallback a cdnjs
│   └─ <style> ────────────── TODO EL CSS (~680 líneas)
│       ├─ GLOBAL: tokens CSS, reset
│       ├─ LAYOUT: fondo ambiente, grid, glow, viñeta, scanlines
│       ├─ HUD: logo, chips, botón CRUSH, iconos
│       ├─ BOARD: shell neón, celdas, esquinas, capa FX
│       ├─ TILES: geometría de las 9 gemas + especiales + skins
│       ├─ OVERLAYS: paneles start/game-over, selector cósmico
│       ├─ AJUSTES: drawer de calidad gráfica
│       ├─ RANKING: top 5, input de nombre
│       └─ RESPONSIVE: media queries
├─ <body>
│   ├─ #bg ────────────────── fondo ambiente (grid, glow, canvas, dust)
│   ├─ #app
│   │   ├─ #hud ───────────── barra superior
│   │   └─ #stage > #shell > #board + #fx + esquinas
│   ├─ #overlay ───────────── panel START + panel GAME OVER
│   ├─ #cosmic-overlay ────── selector de dificultad
│   ├─ #settings-back + #settings-panel ── drawer de ajustes
│   └─ <script> ───────────── TODO EL JS (~2.100 líneas)
│       ├─ Constantes: ROWS, COLS, CFG, DIFFS, TYPES, RANKS
│       ├─ Cookie · LB (leaderboard) · Quality (gráficos)
│       ├─ State (máquina de estados) · AudioFX
│       ├─ ParticlePool (partículas) · AmbientFX (fondo canvas)
│       ├─ Anim (animaciones GSAP)
│       ├─ Tile · Board (clases)
│       ├─ Match · Special · Gravity (lógica pura)
│       ├─ UI (render del HUD/overlays)
│       ├─ InputCtl (entrada táctil/ratón)
│       └─ Game (orquestador) + boot()
```

**Principio de diseño:** cada módulo es un objeto literal o clase con una sola
responsabilidad. La lógica pura (Match, Special, Gravity) no toca el DOM; solo
`Anim`, `UI`, `Tile` y `Board` manipulan elementos.

---

## 4. Máquina de estados

El juego nunca permite acciones fuera de contexto gracias a una máquina de estados
explícita y congelada (`Object.freeze`):

```
const State = Object.freeze({
  START, PLAYING, SWAPPING, RESOLVING, CASCADE, GAME_OVER
});
```

| Estado | Qué significa | Entrada del jugador |
|--------|--------------|---------------------|
| `START` | Panel de inicio / dificultad visible | Solo botones |
| `PLAYING` | Tablero activo, esperando acción | **Habilitada** |
| `SWAPPING` | Dos gemas intercambiándose | Bloqueada |
| `RESOLVING` | Matches → destrucción → especiales | Bloqueada |
| `CASCADE` | Gravedad + caída + nuevo match | Bloqueada |
| `GAME_OVER` | Sin jugadas, panel final | Solo botones |

**Flujo de un turno completo:**

```
PLAYING ──(swap válido)──► SWAPPING ──► RESOLVING
   ▲                        (swap inválido → vuelve a PLAYING sin gastar jugada)
   │                        RESOLVING: detecta matches, destruye, forja especiales
   │                        ▼
   └──────── CASCADE ◄──── gravedad: gemas caen, nuevas entran desde arriba
                │          si hay nuevos matches → RESOLVING de nuevo (loop)
                └───────── si no hay más matches → endTurn()
                              ▼
                           ¿quedan jugadas? ──no──► GAME_OVER
                              │sí
                              ▼
                           PLAYING
```

La regla de oro: **toda entrada se ignora si `state !== PLAYING`**. Esto evita que el
jugador mueva gemas a mitad de una cascada y corrompa el tablero.

---

## 5. Diseño visual (UI)

### 5.1 Sistema de tokens (variables CSS)

Toda la paleta vive en `:root` para mantener coherencia:

```css
--bg0:#04050d   /* fondo base casi negro azulado */
--bg1:#0a0d22   /* fondo secundario */
--panel:#0a102b /* superficie de paneles */
--cyan:#00e5ff  /* acento primario (logo, HUD) */
--magenta:#ff2fd2 /* acento secundario (CRUSH, botón jugar de nuevo) */
--lime:#a3ff12  /* jugadas, éxito */
--amber:#ffb300 /* acento dorado UI */
--coral:#ff4d5e /* peligro, jugadas bajas */
--violet:#9d5cff/* acento terciario */
--ink:#dfe8ff   /* texto principal */
--dim:#6f7fb8   /* texto secundario */
--line:#1d2a5c  /* bordes sutiles */
```

### 5.2 Tipografía

- **Orbitron** (600/800/900, itálica para el logo): títulos, números del HUD, logo.
  Es la fuente "sci-fi" por excelencia; la itálica + letter-spacing le dan el aire
  de cabina de nave.
- **Rajdhani** (500/600/700): cuerpo, descripciones, etiquetas. Condensada y muy
  legible incluso en tamaños pequeños con tracking amplio.

El contraste de tamaños es fuerte: logo ~25px vs etiquetas 9.5px con `letter-spacing:.34em`
(mayúsculas espaciadas, estilo "instrumento de nave").

### 5.3 Geometría de las gemas (100% CSS)

Ninguna gema es una imagen. Cada una es un `<div class="shape">` con:
- **Forma**: `border-radius` (círculo) o `clip-path: polygon(...)` (el resto).
- **Piel neón**: dos gradientes superpuestos — un `radial-gradient` blanco en la esquina
  superior-izquierda (brillo especular) sobre un `linear-gradient(160deg, claro, base, oscuro)`.

| Tipo | Forma | Color base | clip-path / radius |
|------|-------|-----------|--------------------|
| 0 Círculo | `border-radius:50%` | `#00e5ff` | — |
| 1 Cuadrado | `border-radius:18%` | `#ff2fd2` | — |
| 2 Triángulo | `polygon(50% 4%, 97% 92%, 3% 92%)` | `#a3ff12` | ▲ |
| 3 Rombo | `polygon(50% 1%, 99% 50%, 50% 99%, 1% 50%)` | `#ffe14d` | ◆ (amarillo estable) |
| 4 Hexágono | 6 puntos | `#ff4d5e` | ⬡ |
| 5 Estrella | 10 puntos | `#9d5cff` | ★ |
| 6 Agujero negro | círculo + **disco de acreción** cónico | `#b06bff` | ● con anillo giratorio |
| 7 Sol | círculo + **corona** cónica | `#ff6f43` | ☀ amarillo-rojizo |
| 8 Meteorito | polígono irregular de 9 vértices | `#b3bac9` (gris) | ☄ con brillo fundido |

**Las gemas avanzadas usan `conic-gradient` + máscara radial** para crear los discos
giratorios del agujero negro y el sol: el gradiente cónico dibuja los "rayos", y una
`mask: radial-gradient(circle, transparent 54%, #000 57%)` recorta el centro para que
solo se vea el anillo. Animados con `@keyframes holeSpin { to { rotate(360deg) } }`.

**Nota de diseño importante:** los colores de las gemas son **fijos e invariables**.
Se eliminó deliberadamente un `hue-rotate` que hacía "derivar" los tonos con el tiempo
(hacía que el rombo se confundiera con el triángulo). Cada gema mantiene su identidad
de color para que el reconocimiento sea instantáneo.

### 5.4 Fichas especiales

| Ficha | Apariencia | Cómo se señala |
|-------|-----------|----------------|
| Bomba lineal H/V | gema normal + **rayas blancas** (`repeating-linear-gradient`) | Rayas horizontales o verticales |
| Bomba de color | esfera oscura + **núcleo cónico multicolor girando** | `.shape.bomb .core` con los 6 colores |
| Singularidad | vórtice cónico violeta/blanco | `.shape.vortex` |
| Agujero masivo | agujero negro con **3+ partículas orbitando** | `.orbit` con `.odot` |

Las fichas especiales tienen un **pulso de caja** (`spPulse`: brillo interno que respira)
y **partículas orbitando** (`.orbit` gira 360° en 2.4s, la bomba de color en 1.4s invertida).

### 5.5 Fondo y atmósfera (capas apiladas)

El fondo es una composición de **7 capas** con z-index controlado:

1. `#bg` — gradiente radial azul profundo (base)
2. `.bg-grid` — rejilla cyan tenue con deriva lenta (16s) y máscara radial
3. `.bg-glow.a/.b` — dos "auroras" radiales (cyan arriba-izq, magenta abajo-der)
4. `#ambient` (canvas) — **~170 partículas de polvo + meteoros** con parallax
5. `.bg-vig` — viñeta oscura en los bordes (enfoca la atención al centro)
6. `.dust` — 6 motas de colores flotando (CSS puro)
7. `#scan` — scanlines CRT sutiles sobre TODO (z-index 70)

**El canvas `#ambient` (AmbientFX):**
- Dibuja ~170 partículas con sprites de glow pre-renderizados (una vez por color,
  luego `drawImage` — mucho más rápido que `shadowBlur` por frame).
- Cada partícula tiene profundidad → **parallax** al mover el puntero.
- **Meteoros** periódicos: línea con gradiente + cabeza brillante que cruza la pantalla.
- Respeta `prefers-reduced-motion` (frame estático) y se pausa con la pestaña oculta.
- La densidad de partículas se ajusta según la **calidad gráfica** elegida.

### 5.6 El tablero: shell neón

- `#shell` — borde con gradiente cyan→violeta→magenta + `drop-shadow` doble (glow).
  Recortado con `clip-path` de esquinas achaflanadas (estilo "panel de nave").
- `#shell-body` — interior oscuro sobre el que vive el tablero.
- `#board` — fondo `#01020a` con viñeta interna profunda (las gemas resaltan por contraste).
- `.corner` × 4 — esquinas con bordes cyan brillantes que sobresalen (detalle sci-fi).
- `.cell` / `.cell.alt` — damero sutil de fondo (blanco 1.6% / cyan 3.2%).
- `#fx` — capa superior donde viven partículas, anillos, haces y textos flotantes.

---

## 6. Diseño de experiencia (UX)

### 6.1 Entrada: ratón y táctil unificados

El módulo `InputCtl` traduce `pointerdown/move/up` a tres gestos:

| Gesto | Detección | Acción |
|-------|-----------|--------|
| **Arrastrar** | `pointerdown` → desplazamiento > 30% de celda | Intercambia con la vecina en esa dirección |
| **Toque-toque** | Dos toques en gemas adyacentes en <340ms (`tapWindow`) | Intercambia ambas |
| **Un toque a un poder** | `tap()` sobre ficha con `special` | **Detona al instante** |

**Decisión UX clave:** los poderes se activan con **un solo toque**, no doble clic.
En iPhone el doble-toque hace zoom de la página (aunque el viewport lo bloquee, es un
gesto conflictivo). Con un toque el poder explota de inmediato, y si el jugador quiere
intercambiar la ficha especial, la **arrastra** (el arrastre sigue funcionando).

**Bloqueo de zoom en iOS** (triple barrera):
1. `viewport`: `maximum-scale=1, user-scalable=no`
2. `touch-action: manipulation` en TODOS los elementos (mata el doble-toque-zoom y el
   retardo de 300ms). `#stage` usa `touch-action:none` para el arrastre libre.
3. Los inputs de texto se fijan en `font-size:16px` (iOS hace zoom al enfocar inputs <16px).

### 6.2 Feedback en cascada (7 capas simultáneas)

Cada evento importante dispara feedback en **múltiples canales a la vez** — esto es lo
que hace que el juego se sienta "vivo":

1. **Partículas** — explosión de chispas del color de la gema
2. **Anillos** — onda expansiva (`ring()`: círculo que crece y se desvanece)
3. **Texto flotante** — `+120`, `COMBO ×3`, `¡COLISIÓN!` que sube y se apaga
4. **Screen shake** — el `#shell` tiembla con amplitud proporcional al evento
5. **Flash** — la pantalla destella en blanco en combos grandes
6. **Sonido** — tono sintetizado (ver §10)
7. **Pop del marcador** — el número de puntos "salta" (`scale 1.34 → 1`)

### 6.3 Micro-interacciones del HUD

- **Chips** (PUNTOS/MEJOR/JUGADAS): brillo cyan interno, números Orbitron.
- **JUGADAS bajas** (≤3): el chip se pone **coral y pulsa** (`pulseLow`) — aviso de peligro.
- **CRUSH**: botón magenta con barra de carga lima. Deshabilitado hasta pasar 60s de juego;
  cuando está listo pulsa (`crushReady`). Al tocarlo, barre todo el tablero.
- **Engranaje (ajustes)**: se enciende magenta cuando el drawer está abierto.
- **Parlante (mute)**: las ondas de sonido se ocultan y aparece una barra diagonal.

### 6.4 Guía anti-estancamiento (pista automática)

Si el jugador no actúa en `idleMs` (10s en dificultad SOL), el juego:
1. Busca un movimiento válido (`Match.findMove`).
2. Hace **rebotar las dos gemas** del combo con anillos y un marco pulsante (`.tile.hint`).
3. Muestra `HINT · −1 JUGADA` y **descuenta una jugada** (penalización).
4. Si las jugadas llegan a 0 por la penalización → GAME OVER.

Esto evita que el jugador se quede "trabado" sin saber qué hacer, pero con un costo que
mantiene la tensión.

---

## 7. Motor de juego (lógica)

### 7.1 Modelo de datos

```
Board.grid = matriz [12][8] de Tile | null
Tile { type:0-8, special:null|'lineH'|'lineV'|'bomb'|'vortex',
       r, c,              // posición en la grilla
       orb:0-4,           // partículas orbitales acumuladas (agujeros/soles)
       void:bool,         // celda deshabilitada (anomalía)
       el, inner, shape } // referencias DOM
```

`type` indexa el array `TYPES` (color + forma). `type < 0` = celda vacía/transitoria.

### 7.2 Generación del tablero

`Board.generate()` → `clear()` + `fill()`, repitiendo hasta que exista al menos un
movimiento válido (`Match.anyMove`). En `fill()`, cada gema se elige con `randType()`
**rechazando** la que formaría un match inmediato de 3 (mira las 2 gemas a la izquierda
y las 2 de arriba). `randType()` devuelve un tipo del **pool activo según el puntaje**:
6 gemas → 7 (agujero negro) a los 10K → 8 (sol) a los 20K. Los meteoritos **no** entran
en el pool: solo caen del cielo en las tormentas.

### 7.3 Detección de matches (`Match.find`)

Barrido lineal de la grilla buscando **corridas** (runs) horizontales y verticales de 3+.
Cada run: `{ cells:[[r,c]...], dir:'H'|'V', len }`. Todas las gemas (incluidos los
meteoritos) matchean con umbral **3**.

### 7.4 Grupos y planes especiales (`Special.plan`)

Los runs que **comparten celdas** se fusionan en **clusters** (formas L, T, +). Se cuentan
las celdas únicas del cluster y se decide el premio según el **tipo** de la gema:

| Cluster | Celdas únicas | Resultado |
|---------|--------------|-----------|
| Gema normal (0-5) | 5+ | **Bomba de color** |
| Gema normal, run recto | exactamente 4 | **Bomba lineal** (H o V según dirección) |
| Agujero negro (6) | 3+ | `keep` — uno sobrevive y **acumula partículas** |
| Sol (7) | 3+ | `keepSun` — colapsan en un **agujero negro** |
| Meteorito (8) | 3+ | `meteorImpact` — colisión con fragmentos |

**Pivote de aparición:** para la bomba de color se elige la celda de **intersección**
(donde se cruzan el run H y el V); si no, la gema intercambiada; si no, el centro del run.

### 7.5 Acumulación de partículas (agujeros negros y soles)

Mecánica distintiva del juego. Cuando se fusionan agujeros negros (o soles):
1. El **sobreviviente absorbe las partículas (`orb`) de TODAS las gemas fusionadas**.
2. Se suma **+1 por la unión**.
3. Se reconstruye su órbita con ese total (`Special.setOrbit`).

- **Agujero negro** con **3+ partículas** → se convierte en **Agujero Masivo**
  (`morph → vortex`): es una **Singularidad clicable** (un toque la activa y absorbe
  todo en un radio 5×5). Ya NO se auto-detona: espera el toque del jugador.
- **Sol** con **4 partículas** → colapsa en un **agujero negro** (`sunToHole`), que
  empieza con 0 partículas.

Ejemplo: agujero con 1 partícula + agujero con 1 + agujero con 0 → se juntan 3 →
`1+1+0+1(unión) = 3` partículas → **Agujero Masivo**.

### 7.6 Bombas lineales apiladas (cadenas)

Cuando una bomba lineal activa a otra de la **misma orientación**, el haz se apila:
- 1 bomba = 1 línea
- 2 bombas = **3 líneas**
- 3 bombas = **5 líneas**
- k bombas = **2k−1 líneas** (`_lineMult`)

El loop `detonateChain` descubre qué especiales detonan, cuenta cuántas bombas de cada
orientación hay, y asigna el multiplicador. Gráficamente se dibuja **un haz por línea**,
ondulando desde el centro hacia los bordes.

### 7.7 Reacción en cadena (`detonateChain`)

Cualquier especial barrido por un match (o alcanzado por otra explosión) **detona**,
nunca desaparece en silencio. Es un BFS sobre los especiales: cada uno genera su zona de
impacto (`specialTargets`), y si dentro hay otro especial, ese también se encola.
- Bomba de color: limpia todas las gemas del color más frecuente.
- Bomba lineal: barre su(s) línea(s).
- Singularidad: absorbe un radio 5×5 (`Anim.suck`: las gemas espiralan hacia el centro).

Los especiales **recién creados** por el match actual están **protegidos** (`protect`)
para que una explosión no se "coma" la recompensa del jugador.

### 7.8 Colisión de meteoritos (`Anim.fragments`)

Cuando matchean 3+ meteoritos, el punto de impacto **explota** y lanza fragmentos:
- 3 meteoritos → **5 fragmentos**
- 4 meteoritos → **8 fragmentos**
- 5+ meteoritos → **12 fragmentos**

Cada fragmento es un pequeño polígono fundido que:
1. Sale despedido hacia arriba desde el impacto (arco balístico con jitter).
2. Cae sobre una **casilla aleatoria y dispersa** (no lineal: se eligen las más
   separadas entre sí para cubrir el tablero).
3. Al impactar, **destruye el bloque** que toca (lo hace estallar).

### 7.9 Gravedad y cascadas (`Gravity.apply`)

Tras cada destrucción, las gemas **caen** para llenar los huecos (animación con rebote
`bounce.out`), y nuevas gemas **entran desde arriba**. Luego se vuelve a buscar matches:
si los hay, se repite el ciclo (destruir → gravedad → match). Este loop es la **cascada**,
y cada iteración aumenta el `depth` (profundidad) que multiplica los puntos y dispara el
texto `COMBO ×N`.

### 7.10 Scoring y combos (`Score`, `Combo`)

```
puntos = celdas × 10 × profundidad + planes × 50 × profundidad
```
Todo se multiplica por el **multiplicador de dificultad** (`DIFFS.mult`: Planeta 1/3,
Sol 2/3, Agujero 1) para que el ranking sea justo entre órbitas. Un match de 4+ otorga
**jugadas extra** (`bonusMoves`). Los rangos al final (`RANKS`) van de `ESTÁTICA` a
`LEYENDA NEÓN` según el puntaje.

---

## 8. Sistema de partículas

Dos motores independientes, cada uno optimizado para su capa:

### 8.1 `ParticlePool` (DOM, para efectos de juego)

**Object pooling** para no generar basura (GC) en pleno juego:
- Al inicio se crean N `<div>` (según densidad: ~340 en calidad alta) y se guardan en un
  array `free`.
- Cada efecto **toma** un div del pool, lo anima con GSAP, y al terminar lo **devuelve**.
- Cero `createElement`/`remove` durante el juego → 60 FPS estables.

Métodos:
| Método | Uso |
|--------|-----|
| `burst(x,y,color,n,power)` | Explosión radial de chispas (matches, impactos) |
| `implode(x,y,color,n)` | Partículas que **convergen** al centro (forja de especiales, agujeros) |
| `confetti(cx,cy)` | Lluvia multicolor (nuevo récord) |
| `spark(x,y,color)` | Chispa individual (estelas de meteoros) |

### 8.2 `AmbientFX` (Canvas, para el fondo)

- Loop con `requestAnimationFrame` y delta-time (independiente del refresh rate).
- Sprites de glow **pre-renderizados** (un offscreen canvas por color) → `drawImage` por
  frame en lugar de `shadowBlur` (que es carísimo).
- Parallax con el puntero, meteoros con estela, y se **detiene** con la pestaña oculta.
- `recount()` ajusta la cantidad de partículas según la **densidad** de calidad elegida.

### 8.3 Anillos y efectos compuestos

- `ring(x,y,color,{from,to,dur})` — onda expansiva (un div con border que escala y se
  desvanece). Se usa en cada explosión, forja de especial y detonación.
- `Anim.beam` — haz de la bomba lineal (un haz por línea apilada).
- `Anim.suck` — succión de la singularidad (las gemas espiralan encogiéndose al centro).
- `Anim.blast` — onda expansiva de impacto.
- `Anim.fragments` — los fragmentos balísticos de los meteoritos.

---

## 9. Sistema de animaciones (GSAP)

Todo el movimiento usa GSAP. Los easings elegidos no son arbitrarios — cada uno comunica
una sensación física:

| Easing | Dónde se usa | Sensación |
|--------|-------------|-----------|
| `back.out(1.7-4)` | Caída de gemas, aparición | Rebote con "overshoot" (peso + elasticidad) |
| `back.in(2.2)` | Destrucción (scale a 0) | Se "chupa" hacia adentro antes de morir |
| `power2/3.in` | Caída libre, succión | Aceleración (gravedad) |
| `power3.out` | Haz de bomba | Arranque explosivo que frena |
| `bounce.out` | Caída inicial del tablero | Rebote real de goma |
| `elastic` | Pulso del marcador | Resorte |

**Patrones recurrentes:**
- **`will-change: transform`** en las gemas → el navegador las promueve a capa GPU.
- Solo se animan `transform` y `opacity` (propiedades que NO disparan reflow/repaint).
- `gsap.timeline()` encadena fases (ej. destrucción: flash → crecer → encoger → fade).
- `stagger` para efectos en cascada sobre múltiples gemas.
- Animaciones de "respiración" (`gemBreath`: scale 1↔1.075 + leve rotación) con duración
  y delay **aleatorios por gema** → el tablero nunca se ve estático ni sincronizado.

---

## 10. Sistema de audio (WebAudio)

Sin archivos: cada sonido es un **oscilador sintetizado** en runtime.

```
tone(freq, {dur, type, vol, slide, delay})
```
- Crea un `OscillatorNode` + `GainNode` con envolvente ADSR rápida
  (ataque 12ms, decaimiento exponencial).
- `slide` hace un `exponentialRampToValueAtTime` → el tono "cae" o "sube" (láser, pop).
- Todo pasa por un `master` gain (0.16) para no saturar.

| Evento | Diseño sonoro |
|--------|--------------|
| Swap | Dos tonos rápidos ascendentes |
| Match (pop) | Tono que sube con la profundidad del combo |
| Bomba/boom | Seno grave (66Hz) + sawtooth que cae + square |
| Bonus | Arpegio ascendente de 3 notas |
| Warn (pista) | Dos cuadrados descendentes |
| thud (meteorito) | Seno grave corto |
| vortex | Tono que cae largo (succión) |

**Gestión del contexto:** los navegadores exigen un gesto del usuario antes de crear el
`AudioContext`. Se crea/resume en el primer toque (`AudioFX.ensure()`). El estado de mute
se persiste en `localStorage` (`nc-muted`).

---

## 11. Rendimiento y ajustes gráficos

### 11.1 El problema: iPhone lento incluso en "mínimo"

El cuello de botella en iOS Safari **no eran las partículas**, sino los **filtros CSS
animados**: `drop-shadow`, `backdrop-filter` (blur) y `hue-rotate` obligan al compositor
a re-rasterizar capas enormes cada frame. Un Pixel 9 Pro los aguanta; un iPhone 12 no.

### 11.2 La solución: objeto `Quality` con 8 dimensiones

```
Quality.s = {
  particles, bg, breath, shapes, shake, flash, glow, density
}
```

| Dimensión | Qué apaga | Implementación |
|-----------|----------|----------------|
| `particles` | Chispas/explosiones | `ParticlePool` no emite |
| `bg` | Canvas de fondo | `AmbientFX.step` no dibuja |
| `breath` | Respiración de gemas | Clase `q-nobreath` pausa la animación |
| `shapes` | Discos giratorios/órbitas | Clase `q-noshapes` pausa animaciones |
| `shake` | Screen shake | `Game.shake` no anima |
| `flash` | Destellos blancos | `flashBoard` no muestra |
| **`glow`** | **Filtros animados (el grande)** | Clase `q-noglow` elimina `drop-shadow`, `backdrop-blur`, `hue-rotate` |
| `density` | Cantidad de partículas | `alta`(1×) / `media`(0.6×) / `baja`(0.3×) |

### 11.3 Presets y auto-detección

Tres presets: **ALTA / MEDIA / MÍNIMA**. En la **primera visita**, `guessTier()` elige uno
según el dispositivo:

```
memoria ≤ 2GB o núcleos ≤ 2 → MÍNIMA
iPhone/iPad                 → MEDIA   (Safari compone filtros muy lento)
memoria ≤ 4GB o núcleos ≤ 4 → MEDIA
otro (gama alta)            → ALTA
```

Además hay un **benchmark real** (`Quality.benchmark`): durante ~0.6s anima 40 esferas con
blur + drop-shadow, mide los FPS reales del compositor, y aplica el preset ideal
(≥52fps→ALTA, ≥38→MEDIA, menos→MÍNIMA). Es el botón "⚡ PROBAR MI DISPOSITIVO".

Toda la configuración se **persiste por dispositivo** en `localStorage` (`nc-gfx`) y se
aplica al instante (clases en `<body>` + `recount()` del canvas).

### 11.4 Otras optimizaciones

- Object pooling de partículas (cero GC en juego).
- Sprites de glow pre-renderizados en el canvas.
- Pausa del canvas con la pestaña oculta (`visibilitychange`).
- `prefers-reduced-motion` respetado globalmente.
- Canvas a resolución 1× en densidad baja (ahorra fill-rate de GPU).

---

## 12. Sistema de dificultad cósmica

Tras pulsar JUGAR, un **selector de dificultad** (pantalla intermedia) presenta tres
cuerpos celestes que reescriben las reglas. Es la puerta de entrada temática al juego.

| | **PLANETA** (fácil) | **SOL** (medio) | **AGUJERO NEGRO** (difícil) |
|---|---|---|---|
| Multiplicador de puntos | **×1/3** | **×2/3** | **×1** |
| Jugadas iniciales | 14 | 10 | 8 |
| Jugadas extra por match 4+ | 3 | 2 | 2 |
| Pista automática (idle) | 12s | 10s | 7s |
| Penalización por pista | −1 jugada | −1 jugada | **−2 jugadas** |
| Meteoritos | Desde el inicio, frecuentes | A los 30K, cada 30s–2min | A los 30K |
| Anomalías (celdas void) | No | Cada 2–3 min | Cada 75–130s (más punitivo) |

**Equidad del ranking:** el multiplicador hace que jugar fácil dé menos puntos, así el
Top 5 es comparable entre dificultades. Un jugador en PLANETA necesita 3× la destreza
(en puntaje bruto) para igualar a uno en AGUJERO NEGRO.

**Anomalías (celdas void):** cada 2–3 minutos (según dificultad), bloques aleatorios se
**deshabilitan**: se vuelven grises con una X, no matchean, no se intercambian y bloquean
su celda. Dificultan el juego de forma dinámica. (`CFG.voidMs`, `.tile.void`).

---

## 13. Persistencia y base de datos

### 13.1 Qué se guarda dónde

| Dato | Almacenamiento | Clave |
|------|---------------|-------|
| Mejor puntaje (personal) | **Cookie** del navegador | `nc-best` (1 año) |
| Configuración gráfica | localStorage | `nc-gfx` |
| Ranking local (offline) | localStorage | `nc-top5` |
| Cola de sincronización | localStorage | `nc-queue` |
| Estado mute | localStorage | `nc-muted` |
| **Ranking global** | **Supabase Postgres** (o JSONBlob) | tabla `neon_scores` |

### 13.2 La base de datos: Supabase (Postgres gestionado)

Se eligió **Supabase** por ser una base de datos SQL real, gratuita, con API REST
automática y sin necesidad de servidor propio. El juego la consume directamente con
`fetch` al endpoint REST (no usa SDK).

**Esquema:**
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

**Seguridad (Row Level Security):** la clave `anon` es pública por diseño, pero las
políticas RLS **solo permiten SELECT e INSERT** — nadie puede borrar ni modificar
registros desde el cliente.

**Operaciones:**
- **Leer Top 5:** `GET /rest/v1/neon_scores?select=name,score&order=score.desc,id.asc&limit=5`
- **Guardar:** `POST /rest/v1/neon_scores` con `{name, score}`

El código detecta el modo según las constantes configuradas:
```
JSONBLOB_URL no vacío      → modo 'JSON'   (ranking en un blob JSON)
SUPA_URL + SUPA_KEY        → modo 'GLOBAL' (Supabase)
ambos vacíos               → modo 'LOCAL'  (solo por dispositivo)
```

### 13.3 Estrategia offline-first (LB)

El módulo `LB` (leaderboard) está diseñado para **no fallar nunca**, con o sin internet:

```
┌─ Al CARGAR el ranking ──────────────────────────────┐
│ con internet  → GET remoto → guarda copia en cache  │
│ sin internet  → lee la cache local (nc-top5)        │
└─────────────────────────────────────────────────────┘
┌─ Al GUARDAR un puntaje ─────────────────────────────┐
│ 1. SIEMPRE actualiza el top 5 local                 │
│ 2. con internet → POST remoto + vacía la cola       │
│    sin internet → lo enCOLA (nc-queue)              │
│ 3. si el blob dio 404 → marca 'fatal' (no encola)   │
└─────────────────────────────────────────────────────┘
┌─ Al VOLVER el internet ─────────────────────────────┐
│ evento 'online' → flush(): envía toda la cola al    │
│ ranking global y avisa "SYNCED N SCORES → GLOBAL"   │
└─────────────────────────────────────────────────────┘
```

Así un jugador sin conexión sigue viendo su ranking local, y su puntaje **se sincroniza
solo con el ranking mundial** apenas recupera internet.

### 13.4 Flujo del Top 5 al terminar

1. `gameOver()` compara el puntaje contra el Top 5 (`LB.qualifies`).
2. Si clasifica, muestra el panel con un **input de nombre** (máx. 10 caracteres,
   contador `7/10`, autocapitalizado).
3. Al guardar (`SAVE` o Enter): si el nombre está vacío el input **tiembla**; si no,
   `LB.save()` persiste y resalta la fila del jugador en verde con `·TÚ`.
4. Medallas 🥇🥈🥉 para los 3 primeros puestos.

### 13.5 Nota sobre JSONBlob (alternativa)

Se soporta un modo `JSON` vía jsonblob.com (un "archivo" JSON en la nube sin registro),
pero el servicio quedó **bloqueado por Cloudflare** (403) para llamadas de API, por lo
que el README recomienda **Supabase**. El código lo detecta: si el blob responde 404,
marca el error como `'missing'` y avisa en pantalla que no guarda globalmente.

---

## 14. Versionado semántico automático

La versión vive en `index.html` como `const APP_VERSION = 'vX.Y.Z'` y se muestra bajo el
logo y en los ajustes.

**Automatización con GitHub Actions** (`.github/workflows/auto-version.yml`):
- Cada **Pull Request** dispara el workflow.
- Lee `APP_VERSION`, incrementa el **patch** (`v1.4.3 → v1.4.4`).
- Hace commit en la rama del PR usando el **título del PR como motivo**.
- Un guard evita que el propio commit del bot re-dispare el workflow (loop infinito).
- Para saltos de **minor** o **major**, se edita la constante a mano dentro del PR.

> Limitación: solo funciona con ramas del mismo repositorio (los forks externos no
> permiten escribir de vuelta en el PR).

---

## 15. Despliegue

El juego es **100% estático**: solo se necesita `index.html` (GSAP y fuentes vienen de CDN).

**Opción A — GitHub Pages (con git):**
```bash
git init
git add index.html README.md NEON_CRUSH_DOCUMENTACION_TECNICA.md
git commit -m "Neon Crush"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/neon-crush.git
git push -u origin main
```
Luego: *Settings → Pages → Branch: main → Save*. El juego queda en
`https://TU_USUARIO.github.io/neon-crush/`

**Opción B — Netlify Drop (sin git, 30 segundos):**
Arrastrar la carpeta con `index.html` a `app.netlify.com/drop` → link público al instante.

---

## 16. Cómo se resolvió cada problema

Bitácora de los desafíos técnicos más relevantes y su solución:

| # | Problema | Solución |
|---|----------|----------|
| 1 | Gemas borrosas (parecían desenfocadas) | Eran los `drop-shadow` difuminando siluetas. Se quitaron y se oscureció el fondo para que resalten por contraste, no por halo. |
| 2 | iPhone lento aun en calidad mínima | No eran las partículas sino los filtros CSS animados (`hue-rotate`, `backdrop-blur`). Se creó el toggle `glow` que los elimina todos. |
| 3 | Zoom de iOS al tocar rápido | Triple barrera: viewport, `touch-action:manipulation` global, e inputs a 16px. |
| 4 | Doble-toque en iPhone hacía zoom al activar poderes | Se cambió la activación a **un solo toque**. |
| 5 | Poderes se activaban solos | Bug: una gema normal sin poder caía en la rama de bomba lineal de `specialTargets` y devolvía toda su fila/columna como zona de explosión. Se corrigió para devolver `[]` si no es especial. |
| 6 | Bombas lineales no sumaban líneas en cadena | El loop de descubrimiento ya calculaba el multiplicador, pero el haz y los targets no lo aplicaban. Se dibujó un haz por línea y se expandieron los targets a `2k−1` líneas. |
| 7 | Rombo cambiaba de color y se confundía con el triángulo | El `hue-rotate` del marco rotaba todos los tonos. Se eliminó y se fijó el rombo en amarillo estable (`#ffe14d`). |
| 8 | Soles explotaban solos (modo difícil) | Se eliminó el mecanismo de "soles inestables"; ahora los soles solo colapsan en agujero negro **al combinarse de a 3**. |
| 9 | Fragmentos de meteorito caían en línea/cluster | Se cambió la selección de objetivos a "las casillas más separadas entre sí" con jitter, para una dispersión real por el tablero. |
| 10 | JSONBlob devolvía 403/404 | El servicio quedó bloqueado por Cloudflare. Se documentó y se recomienda Supabase; el juego detecta el 404 y avisa. |
| 11 | Tablero pegado a los bordes en pantallas chicas | `resize()` reserva el grosor del marco + un margen mínimo de 10px por lado. |
| 12 | Ranking sin conexión se perdía | Estrategia offline-first: cache local + cola de sincronización que se vacía al volver internet. |

---

## Apéndice: Glosario de módulos

| Módulo | Responsabilidad |
|--------|----------------|
| `State` | Máquina de estados congelada |
| `CFG` | Constantes de juego (tablero, tiempos, umbrales de desbloqueo) |
| `DIFFS` | Reglas por dificultad |
| `TYPES` / `RANKS` | Catálogo de gemas y rangos de puntaje |
| `Cookie` | Lectura/escritura de cookies (mejor puntaje) |
| `LB` | Leaderboard (LOCAL/JSON/GLOBAL + cola offline) |
| `Quality` | Ajustes gráficos + auto-detección + benchmark |
| `AudioFX` | Sonidos sintetizados (WebAudio) |
| `ParticlePool` | Partículas DOM con object pooling |
| `AmbientFX` | Fondo canvas (polvo + meteoros + parallax) |
| `Anim` | Todas las animaciones GSAP (swap, destroy, beam, suck, fragments…) |
| `Tile` / `Board` | Modelo y DOM del tablero |
| `Match` | Detección de corridas y movimientos válidos |
| `Special` | Forja de especiales, acumulación de partículas, cadena |
| `Gravity` | Caída de gemas y entrada de nuevas |
| `UI` | Render del HUD, overlays, ranking, ajustes |
| `InputCtl` | Entrada táctil/ratón (arrastrar, toque-toque, detonar) |
| `Game` | Orquestador principal (turnos, cascadas, tormenta, anomalías, CRUSH) |

---
*Documento generado para NEON CRUSH v1.4.3 — un solo archivo, cero dependencias de build,*
*infinitas cascadas.*



