# ⚽ FIFA World Cup 2026 — Visualización 3D Interactiva

Aplicación web autocontenida que visualiza en 3D interactivo los 12 grupos del Mundial 2026 con datos reales actualizados. Diseñada para abrirse directamente en cualquier navegador moderno sin servidor local, sin dependencias de instalación ni APIs con clave privada.

## 🚀 Demo

Abrir `index.html` directamente en el navegador o servir con cualquier servidor estático (GitHub Pages, Netlify, etc.).

## 📁 Estructura del Proyecto

```
mundial_2026/
├── index.html      ← Aplicación completa (~77KB)
├── favicon.png     ← Icono de pestaña (32×32)
└── README.md       ← Este archivo
```

No requiere `package.json`, `node_modules`, build step, servidor local, base de datos ni API keys. **Un solo archivo HTML autocontenido.**

---

## 🛠️ Stack Tecnológico

| Capa         | Tecnología              | Función                          |
|-------------|--------------------------|----------------------------------|
| Rendering   | Three.js r128 (CDN)     | Escena 3D WebGL                  |
| Controles   | OrbitControls custom    | Navegación cámara 3D             |
| Animación   | requestAnimationFrame   | Loop de renderizado 60fps        |
| Banderas    | flagcdn.com API CDN     | Imágenes PNG de banderas         |
| Tipografía  | Google Fonts            | Orbitron + Rajdhani              |
| Favicon     | PNG local (32×32)       | Icono de pestaña                 |
| Responsive  | CSS @media queries      | Adaptación móvil/tablet          |
| Datos       | Objeto JS embebido      | 48 equipos, 72+ partidos         |

**CDN utilizados:**

- `cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`
- `flagcdn.com/w160/{iso2}.png` (banderas)
- `flagcdn.com/w40/{iso2}.png` (banderas pequeñas en tooltips)
- `fonts.googleapis.com` (Orbitron, Rajdhani)

---

## 🏗️ Arquitectura del Archivo

```
index.html (autocontenido, 1698 líneas)
│
├── <head>
│   ├── Meta tags (charset, viewport responsive)
│   ├── Favicon (favicon.png local)
│   ├── Google Fonts (Orbitron, Rajdhani)
│   └── <style> — ~250 líneas de CSS
│       ├── Variables CSS (--primary, --gold, etc.)
│       ├── Loading screen (radial gradient, barra de progreso)
│       ├── UI overlay (glassmorphism, top bar, controles)
│       ├── Tooltip de estadísticas (hover panel)
│       ├── Buscador con autocompletar
│       └── @media queries responsive (768px, 480px, landscape)
│
├── <body>
│   ├── Loading screen (animación de carga)
│   ├── UI overlay (top bar + controles + leyenda + tooltip)
│   ├── Canvas container (Three.js rendering target)
│   └── <script> — ~1400 líneas de JavaScript
│       ├── SECCIÓN 0: Datos del Mundial (objeto MUNDIAL_2026)
│       ├── SECCIÓN 1: Configuración y constantes
│       ├── SECCIÓN 2: Setup Three.js (escena, cámara, luces)
│       ├── SECCIÓN 3: Loading screen (progreso)
│       ├── SECCIÓN 4: Creación de grupos 3D (tarjetas)
│       ├── SECCIÓN 5: Sistema de líderes (banderas destacadas)
│       ├── SECCIÓN 6: Interactividad (raycasting, hover)
│       ├── SECCIÓN 7: UI Controles (botones, buscador)
│       └── SECCIÓN 8: Animation loop (60fps)
│
└── favicon.png (32x32, icono de pestaña)
```

---

## 📊 Estructura de Datos

El objeto `MUNDIAL_2026` contiene toda la información del torneo:

```
MUNDIAL_2026
├── torneoInfo (nombre, sedes, fechas, fuente)
└── grupos (A a L)
    ├── nombre, jornadaActual, totalJornadas, partidosJugados
    ├── equipos[] (4 por grupo = 48 total)
    │   ├── pais, iso2 (código país para bandera CDN)
    │   ├── stats {pj, g, e, p, gf, gc, dg, pts, pos}
    │   └── partidos[] (3 por equipo)
    │       ├── jornada, rival, gf, gc, res (V/E/D)
    │       ├── fecha, ciudad, estadio, cond (local/visitante)
    │       └── estado ("pendiente" si no se jugó)
    └── lider {tipo, equipos[]}
        ├── "unico" → un equipo líder (glow dorado)
        └── "empate" → múltiples equipos (glow azul)
```

**Fuente de datos:** Wikipedia API (`en.wikipedia.org/w/api.php`) + Promiedos.com.ar para resultados en tiempo real.

---

## ⚙️ Motor de Cálculo de Estadísticas

El sistema calcula automáticamente las estadísticas de cada equipo iterando sobre los resultados de los partidos:

**Por cada partido:**

- Si `score = "PENDIENTE"` → no se cuenta
- Si `gf > gc` → victoria (`g++`, `pts += 3`, `res = "V"`)
- Si `gf = gc` → empate (`e++`, `pts += 1`, `res = "E"`)
- Si `gf < gc` → derrota (`p++`, `res = "D"`)

**Ordenamiento de equipos (criterios FIFA):**

1. Puntos (descendente)
2. Diferencia de goles (descendente)
3. Goles a favor (descendente)

**Detección de líder:**

- Si el 1° tiene pts, GD y GF únicos → líder único
- Si otro equipo iguala en los 3 criterios → empate

---

## 🎨 Escena 3D — Renderizado

### Escena

- Fondo oscuro (`#000011`)
- Niebla exponencial (`FogExp2`, densidad 0.008)
- 2000 partículas de estrellas (`PointsMaterial`)

### Iluminación (4 fuentes)

| Tipo              | Color     | Intensidad | Posición        |
|-------------------|-----------|------------|-----------------|
| AmbientLight      | 0x334466  | 0.6        | Global          |
| DirectionalLight  | Blanca    | 0.8        | (10, 20, 15)    |
| PointLight        | Dorada    | 0.5        | (-15, 10, 10)   |
| PointLight        | Azul      | 0.3        | (15, 10, -10)   |

### Cámara

- `PerspectiveCamera` (FOV 60°, aspect ratio dinámico)
- Posición inicial: `(0, 5, 30)` desktop / `(0, 5, 50)` móvil
- Controles orbitales custom (coordenadas esféricas)
- Soporte mouse (drag, wheel) + touch (touchstart/move/end)

### Loop de Renderizado

- `requestAnimationFrame` (60fps)
- Delta time con `THREE.Clock`
- Uniform global para shaders de ondulación
- Resize handler dinámico

---

## 🃏 Visualización de Grupos

### Layout

Cuadrícula **4 columnas × 3 filas** con separación configurable:

| Constante     | Valor | Descripción                 |
|---------------|-------|-----------------------------|
| `CARD_W`      | 5.5   | Ancho de tarjeta            |
| `CARD_H`      | 4.0   | Alto de tarjeta             |
| `CARD_GAP_X`  | 6.5   | Separación horizontal       |
| `CARD_GAP_Y`  | 7.2   | Separación vertical         |

### Tarjeta de Grupo (por cada grupo A-L)

- `PlaneGeometry` (5.5 × 4.0)
- `MeshPhysicalMaterial` (efecto glassmorphism):
  - `metalness: 0.8`, `roughness: 0.2`
  - `transparent: true`, `opacity: 0.4`
  - `transmission: 0.5`
  - `emissive: 0x00ffcc` (borde luminoso)
- `EdgesGeometry` con `LineBasicMaterial` (borde cyan)
- Título del grupo (TextSprite, fuente Orbitron, dorado)
- Jornada badge (ej: "Jornada 3/3")

### Banderas (4 por grupo, 48 total)

- `PlaneGeometry` con segmentos (32×32) para shader de onda
- Textura cargada desde `flagcdn.com/w160/{iso2}.png`
- `MeshStandardMaterial` (roughness 0.4, metalness 0.1)
- **Shader de ondulación personalizado** (`onBeforeCompile`):
  ```glsl
  transformed.z += sin(position.x * 5.0 + time * 3.0) * 0.15;
  ```
- Fallback de color sólido si la imagen no carga
- Disposición 2×2 dentro de la tarjeta
- Nombre del país debajo de cada bandera (TextSprite)
- Puntos actuales debajo del nombre

---

## ⭐ Sistema de Líderes

### Caso 1 — Líder Único (glow dorado)

- Bandera más grande (1.8 × 1.2) sobre la tarjeta
- Halo dorado (`PlaneGeometry` 2.5×1.8, `AdditiveBlending`)
- Pulsación del glow (`opacity = 0.1 + sin(time*2) * 0.08`)
- Levitación (`position.y += sin(time*1.5) * 0.15`)
- Rotación leve (`sin(time*0.3) * 0.2` radianes)
- Nombre del país encima (canvas 512×128, Gadugi, dorado, MAYÚSCULAS)
- Puntos a la derecha (canvas 256×256, Gadugi, dorado, font 900)
- PJ a la derecha (canvas 256×256, Gadugi, blanco, font 400)
- Badge "LÍDER" debajo (TextSprite, dorado)

### Caso 2 — Empate (glow azul)

- Múltiples banderas sobre la tarjeta
- Glow azul plateado (`0xc0c0ff`)
- Rotación alternada (direcciones opuestas)
- Badge "⚖️ EMPATE"

### Detección de Empate

- Compara puntos + diferencia de goles + goles a favor
- Si los 3 criterios son idénticos → empate
- Si alguno difiere → líder único

---

## 🖱️ Interactividad — Raycasting

### Sistema: Three.js Raycaster

**Flujo de hover:**

1. `onMouseMove` calcula coordenadas normalizadas (-1 a +1)
2. Raycaster intersecta contra array de `flagMeshes`
3. Si hay intersección:
   - Bandera escala 1.0 → 1.2 (suavizado en animation loop)
   - Se muestra tooltip HTML con datos del equipo
   - Tooltip se posiciona junto al cursor
4. Si no hay intersección:
   - Bandera vuelve a escala 1.0
   - Tooltip desaparece con fade

### Tooltip (HTML overlay, no 3D)

- Header con bandera pequeña + nombre del país
- Color del header según tipo (dorado/azul/cyan)
- Grupo y posición actual
- Tabla de estadísticas (PJ, G, E, P, GF, GC, DG, Pts)
- Lista de partidos con:
  - Resultado V/E/D con colores (verde/amarillo/rojo)
  - Marcador, fecha, ciudad, estadio
  - Partidos pendientes en gris con "⏳ Pendiente"

### Click en Grupo

- Cámara hace zoom suave hacia el grupo
- Animación con ease in-out cuadrática (1500ms)
- Target se interpola hacia posición del grupo

---

## 🔍 Buscador con Autocompletar

**Componentes:**

- Input de texto con placeholder "🔍 Buscar equipo..."
- Dropdown de resultados con bandera + nombre + grupo

**Lógica:**

1. Al inicio se indexan los 48 países en array `allCountries`
2. Al escribir, filtra por coincidencia parcial (`includes`)
3. Muestra resultados con bandera CDN + nombre + grupo
4. Navegación con flechas ↑↓ del teclado
5. Enter selecciona → zoom al grupo del país
6. Escape cierra el dropdown
7. Click fuera cierra el dropdown

**Validación:**

- Solo muestra países que existen en los datos
- No permite nombres incorrectos
- Búsqueda case-insensitive

---

## 🎛️ UI Controles

### Top Bar (centro superior)

- Título "⚽ FIFA WORLD CUP 2026" (Orbitron, dorado)
- Badge de estado "FASE DE GRUPOS — EN CURSO" (pulsante verde)

### Panel de Controles (izquierda)

- Botón "🏠 Vista General" (reset cámara)
- Buscador con autocompletar
- Botones A-L (filtro por grupo, 4 columnas)
- Botón "⭐ Solo Líderes" (toggle, activo por defecto)

### Leyenda (inferior izquierda)

- 🟡 Glow dorado = Líder único
- 🔵 Glow azul = Empate en primer lugar
- ⚪ Sin glow = Sin partidos / Por iniciar
- Se oculta en móvil (no hay espacio)

---

## ⏳ Loading Screen

- Pantalla completa con gradiente radial oscuro
- Título "FIFA WORLD CUP 2026" (Orbitron, dorado, pulsante)
- Subtítulo "CANADA · MEXICO · USA"
- Barra de progreso (gradiente dorado-anaranjado)
- Mensajes secuenciales:
  1. "Iniciando..."
  2. "Creando Grupo A..." ... "Creando Grupo L..."
  3. "¡Todo listo! FIFA World Cup 2026"
- Fade out al completar (transición 1.5s)
- UI overlay aparece después del loading

---

## 📱 Responsive Design

### Breakpoint 1 — Tablet (`max-width: 768px`)

- Top bar apilado verticalmente
- Controles en la parte inferior (barra horizontal)
- Botones de grupo en flex-wrap
- Leyenda oculta
- Tooltip ancho completo, posición fija abajo
- Cámara más alejada (radius 50 vs 30)

### Breakpoint 2 — Móvil pequeño (`max-width: 480px`)

- Fuentes más pequeñas
- Padding reducido
- Tooltip con scroll si es largo
- Botones más compactos

### Breakpoint 3 — Landscape móvil (`max-height: 500px`)

- Controles en fila horizontal arriba
- Tooltip con más altura disponible

### Touch Support

- `touchstart`/`touchmove`/`touchend` para rotar cámara
- Un solo dedo = rotación
- Resize handler para cambio de orientación

---

## ⚡ Rendimiento y Optimización

- `TextureLoader` con cache implícita del navegador
- `PlaneGeometry` con segmentos solo donde se necesita (ondulación)
- `EdgesGeometry` para bordes (más ligero que otro mesh)
- Canvas textures para texto (evita Geometry pesada)
- `PixelRatio` limitado a 2 (retina sin exceso)
- `FogExp2` para limitar render distance
- `requestAnimationFrame` (no `setInterval`)
- Sin dependencias de build tools (webpack, etc.)
- Archivo único = una sola petición HTTP

---

## 🛡️ Manejo de Errores

- Banderas: fallback a color sólido si `flagcdn.com` falla
- `onerror` en `<img>` del tooltip para ocultar imagen rota
- Texturas con callback de error en `TextureLoader`
- Sin `console.errors` en operación normal

---


