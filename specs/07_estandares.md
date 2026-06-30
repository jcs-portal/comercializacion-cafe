# Estándares: Comercialización de Café

## Convenciones de Código

Se siguen las mismas convenciones que `diagnostico-finca-cafe`, `finanzas-cafeteros` y `plan-mejora-calidad-cafe`, para que cualquiera que ya conozca esas apps pueda leer esta sin aprender un estilo nuevo:

| Elemento | Convención | Ejemplo del código actual |
|----------|------------|---------------------------|
| Variables y campos de dominio | camelCase, en español | `costeLogistico`, `margenNetoCOP`, `tipoPersona`, `incotermHabitual` |
| Funciones de utilidad/helpers | camelCase, inglés o español | `fmtCOP()`, `fmtKg()`, `fmtPct()`, `parsearRespuesta()`, `descargarExcel()` |
| Funciones de dominio | camelCase, en español | `cargar()`, `agregar()`, `guardar()` |
| Componentes React | PascalCase | `MisVentas`, `MisClientes`, `MisDatos`, `MiResumen`, `Parametros`, `PanelFormador`, `AppCaficultor`, `Login` |
| Componentes base | PascalCase, nombres cortos en inglés/español | `Campo`, `Boton`, `Tarjeta`, `Titulo`, `KPI` |
| Clave de localStorage | `ccaf_` como prefijo | `ccaf_code`, `ccaf_password` |
| Constante de API | Un solo `API_BASE` al principio del script | `const API_BASE = "https://..."` |
| Paleta de colores | Objeto `C` con nombres semánticos del café | `C.earth`, `C.roast`, `C.bark`, `C.clay`, `C.cream`, `C.leaf`, `C.alert`, `C.warn`, `C.ok` |
| Rutas API backend | kebab-case | `/api/comercializacion/listas-ref`, `/api/admin/comercializacion/consolidado` |
| Archivos | Un único `index.html` — sin build step, sin archivos adicionales de lógica | — |

### Formato de Commits
```
tipo(alcance): descripción breve

Tipos: feat, fix, docs, style, refactor, chore
Ejemplos:
  feat(ventas): añadir campo coste logístico a formulario de venta
  fix(resumen): corregir cálculo de margen neto cuando moneda es USD
  docs(specs): añadir ADR sobre generación Word en cliente
```

## Estructura con Ownership

```
comercializacion-cafe/
├── index.html          ← Frontend (toda la lógica de la app)
├── README.md           ← Arquitecto
└── specs/              ← Arquitecto
    ├── 01_discovery.md
    ├── 02_producto.md
    ├── 03_restricciones.md
    ├── 04_arquitectura.md
    ├── 05_equipo.md
    ├── 06_pipeline.md
    └── 07_estandares.md

finanzas-cafeteros/      (repo existente, NO scaffolding nuevo)
└── app.js               ← Arquitecto, SOLO las rutas /api/comercializacion/*
                           y /api/admin/comercializacion/*
```

## Reglas por Teammate

**Arquitecto**: documenta cada bloque de rutas nuevas con un comentario igual de breve que los existentes en `app.js` (ej. `// ── API: Comercialización`). No reescribe ni reordena código existente — solo añade. No despliega sin aprobación explícita del usuario.

**Frontend**: un solo archivo, sin dependencias locales ni build step. Componentes funcionales pequeños (funciones, no clases). Un único `API_BASE` — nunca hardcodear la URL del backend más de una vez. Las funciones de fetch (`apiGet`, `apiPost`, `apiPut`, `apiDelete`) son las únicas que hacen llamadas HTTP.

## Patrones a Evitar

### `.catch` vacío o silencioso
El patrón que ya causó problemas en apps hermanas:
```js
// MAL — el error desaparece y el estado queda en null sin que el usuario lo sepa
apiGet("/comercializacion/resumen", code).then(setR).catch(() => {});

// BIEN — el error se muestra al usuario
apiGet("/comercializacion/resumen", code)
  .then(setR)
  .catch(e => setError(e.message || "Error al cargar el resumen"));
```

Excepción aceptada (ya existente en el código): el `.catch(() => setListasRef({}))` en la carga inicial de `listas-ref` tiene degradación controlada documentada. Cualquier nueva excepción a esta regla debe documentarse con un comentario explicando por qué es intencional.

### Acceder a propiedades de un estado que puede ser `null`
```js
// MAL — explota si r es null
<div>{r.totalKg}</div>

// BIEN — guard antes de renderizar
if (!r) return <div style={{ padding: 20, color: C.bark }}>Cargando…</div>;
```
El patrón ya está en el código (`if (!r) return <div...>Cargando…</div>`) — mantenerlo en toda función nueva.

### Hardcodear datos de catálogo en el frontend
Las listas de variedades, etapas, tipos de cliente, incoterms y monedas vienen de `GET /api/comercializacion/listas-ref`. No se añaden hardcodeadas en el cliente. Si el backend no está disponible, `catch(() => setListasRef({}))` produce arrays vacíos que los selects muestran como vacíos — el usuario ve que algo falta, no datos inventados.

### Múltiples constantes con la URL del backend
```js
// MAL
const url1 = "https://doc-comite-finanzas-production.up.railway.app/api/comercializacion/finca";
const url2 = "https://doc-comite-finanzas-production.up.railway.app/api/comercializacion/ventas";

// BIEN — una sola constante al inicio del script
const API_BASE = "https://doc-comite-finanzas-production.up.railway.app/api";
// ...y luego: apiGet("/comercializacion/finca", code)
```

## Comandos de Verificación

| Scope | Acción | Cuándo |
|-------|--------|--------|
| Backend (Arquitecto) | Arrancar `node app.js` localmente; probar con curl/fetch al menos una ruta existente (ej. `GET /api/whoami`) y todas las nuevas de `/api/comercializacion/*` | Antes de marcar la tarea como completa y antes de desplegar |
| Frontend | Abrir `index.html` en el navegador (servido localmente: `npx serve .` o `python -m http.server`) y recorrer los dos flujos de `specs/02_producto.md` (caficultor y formador) contra el backend local o de producción | Antes de marcar la tarea como completa |
| Ambos | Revisión en pantalla de móvil (DevTools modo responsive, 375px de ancho como mínimo) | Antes del checkpoint humano |
| Formatos de salida | Descargar Excel y Word; abrir y verificar que las tablas tienen datos correctos | Antes de anunciarlo a los caficultores |

## Cómo Verificar la App (acceso rápido)
1. Abrir `index.html` en Chrome/Firefox (localmente con `npx serve .` o en GitHub Pages).
2. Entrar con código `SA-01` (o el código de prueba acordado con el formador).
3. Verificar tabs: Ventas → añadir una venta; Clientes → añadir un cliente; Resumen → ver KPIs; Parámetros → cambiar y guardar.
4. Cerrar sesión → entrar con código `FORM-SA` → verificar panel del formador y descarga de Excel.

## Definición de Hecho (antes de decir "listo" o "desplegado")

Este proyecto no tiene compilador, tipos ni tests automáticos — esa red de seguridad que detecta errores sola, aquí no existe. Por eso, antes de afirmar que algo está listo, hay que verificar explícitamente cada punto:

1. **Camino feliz probado de verdad**, no solo sintaxis: ejecutar el flujo en un entorno donde el JS realmente corre (navegador real), no solo revisar el código.
2. **Camino infeliz probado a propósito**: ¿qué pasa si una ruta del backend no existe todavía? ¿si la red falla? ¿si el dato viene vacío o null? El mensaje de `parsearRespuesta` ya cubre el caso de ruta no disponible — verificar que funciona.
3. **Ningún `.catch` vacío**: todo fallo de `fetch`/`apiGet`/`apiPost`/`apiPut`/`apiDelete` debe terminar en algo visible para el usuario (mensaje de error), nunca en pantalla que se queda a medias en silencio.
4. **Ningún acceso a propiedades de un estado que puede ser `null`** sin guard previo (`if (!datos) return <div>Cargando…</div>` antes de usar `datos.campo`).
5. Si algo de los puntos 1-4 no se pudo verificar (ej. no había conexión al backend disponible), decirlo explícitamente como "no verificado" — nunca afirmar "listo" dando por hecho que pasó algo que no se comprobó.

## Configuraciones
No se introducen ESLint/Prettier/TypeScript — el proyecto sigue deliberadamente el mismo estilo "sin tooling" que sus tres apps hermanas. La única herramienta añadida respecto a `diagnostico-finca-cafe` es la librería `docx` por CDN, que no requiere ninguna configuración local.
