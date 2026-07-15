# Arquitectura Técnica: Comercialización de Café

## Visión General

```mermaid
graph LR
    subgraph "Frontend (este proyecto)"
        F[comercializacion-cafe<br/>index.html — React + Babel vía CDN<br/>+ docx vía CDN (generación Word en cliente)]
    end
    subgraph "Backend existente (finanzas-cafeteros) — EXTENDER, no recrear"
        B[app.js Express<br/>+ rutas /api/comercializacion/*<br/>+ rutas /api/admin/comercializacion/*]
        D[(data/users/<codigo>.json → finca — Ficha de Productor compartida<br/>data/comercializacion/<br/>clientes/<codigo>.json<br/>ventas/<codigo>.json<br/>parametros/<codigo>.json)]
    end
    subgraph "Apps hermanas (sin tocar)"
        DX[diagnostico-finca-cafe]
        FZ[finanzas-cafeteros UI]
        PM[plan-mejora-calidad-cafe]
    end
    F -- "fetch + X-Code + X-Password" --> B
    DX -- "fetch + X-Code" --> B
    FZ -- "fetch + X-Code" --> B
    PM -- "fetch + X-Code" --> B
    B --> D
```

## Stack Tecnológico

### Frontend (este proyecto)
| Capa | Tecnología | Justificación |
|------|------------|----------------|
| UI | React 18 vía CDN (cdnjs), un solo `index.html` | Mismo patrón exacto que `diagnostico-finca-cafe` y `plan-mejora-calidad-cafe`: sin build step, fácil de mantener, despliegue trivial. |
| JSX | Babel Standalone 7.23.2 vía CDN (cdnjs) | Permite JSX inline sin compilación previa. Coste: ~140 kB adicionales en carga. Aceptado en todas las apps hermanas. |
| Generación de documentos | `docx` 9.7.1 vía CDN (unpkg) | Generación de Word en cliente. Única diferencia respecto a las apps hermanas (que no tienen esta librería). Coste: ~400 kB adicionales. Justificado porque evita un endpoint de backend y almacenamiento de archivos en el servidor. |
| Estilos | CSS inline + `<style>` en el mismo archivo | Coherente con el resto del ecosistema. |
| Estado/datos | `fetch` directo al backend + `localStorage` para código y contraseña de sesión | Claves: `ccaf_code`, `ccaf_password`. No se necesita gestor de estado externo. |

### Backend (extensión del existente — no se crea uno nuevo)
| Capa | Tecnología | Justificación |
|------|------------|----------------|
| Servidor | Express (ya existe en `finanzas-cafeteros/app.js`) | Ya soporta auth por código (`X-Code` + `X-Password`), CORS y el patrón de rutas que se replica para comercialización. |
| Persistencia | `finca` reutiliza `data/users/<codigo>.json` (mismo objeto que `finanzas-cafeteros`, `diagnostico-finca-cafe` y `plan-mejora-calidad-cafe`); clientes/ventas/parámetros propios de comercialización en `data/comercializacion/<tipo>/<codigo>.json` | Cero infraestructura nueva; la Ficha de Productor (`finca`) es deliberadamente el mismo campo en las 4 apps para no duplicar datos fundamentales — ver ADR-003 en `finanzas-cafeteros/specs/04_arquitectura.md`. |
| Auth | Middleware `auth`/`formador` ya existentes | El caficultor ve y edita solo sus propios datos; el formador ve el consolidado de su comunidad. |
| Listas de referencia | `GET /api/comercializacion/listas-ref` | Devuelve catálogos usados en selects: variedades, etapas (E1-E6), tipos de actor, incoterms, monedas. Estructura por confirmar en el backend. |

### Infraestructura
| Componente | Servicio | Justificación |
|------------|----------|----------------|
| Hosting frontend | GitHub Pages (`jcs-portal.github.io/comercializacion-cafe` — por confirmar URL exacta) | Gratis, sin servidor propio, coherente con el resto del ecosistema. |
| Hosting backend | Railway (`doc-comite-finanzas-production.up.railway.app`) — ya existente | No se crea un segundo backend; se extienden las rutas del actual. |
| CI/CD | Ninguno nuevo — despliegue manual (`git push` al repo de Pages) | Proyecto demasiado pequeño para justificar pipeline; igual que las apps hermanas. |

## Endpoints Consumidos (derivados del código)

### Autenticación (compartida con todas las apps)
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/whoami` | Verifica código, devuelve `{ code, role }`. Role `joven` → vista caficultor; role `formador` → panel formador. |

### Catálogo (caficultor)
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/comercializacion/listas-ref` | Catálogos: variedades, etapas E1-E6, tipos de cliente, incoterms, monedas. Carga al inicio junto con el login. |

### Datos del productor (caficultor)
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/comercializacion/finca` | Alias de `/api/finca`: Ficha de Productor compartida (nombre, tipo persona, NIT/cédula, dirección, teléfono, vereda, altitud, variedad, hectáreas). Si ya se rellenó desde Finanzas Cafeteros o Diagnóstico de Finca, aquí aparece completa. |
| PUT | `/api/comercializacion/finca` | Actualiza el mismo objeto `finca` — el cambio se ve también en las otras 3 apps. |

### Clientes (caficultor)
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/comercializacion/clientes` | Lista de clientes del caficultor. Devuelve `{ clientes: [...] }`. |
| POST | `/api/comercializacion/clientes` | Crea un nuevo cliente. Body: `{ nombre, tipoActor, pais, ciudad, etapasQueCompra, incotermHabitual, condicionPago, activo }`. |
| DELETE | `/api/comercializacion/clientes/:id` | Borra un cliente por ID. |

### Ventas (caficultor)
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/comercializacion/ventas` | Lista de ventas del caficultor (historial abierto — crece con cada venta registrada). Devuelve `{ ventas: [...] }` con márgenes calculados. Cada venta incluye `registradoEn` y, si se editó, `actualizadoEn`. |
| POST | `/api/comercializacion/ventas` | Registra una venta. Body: `{ fecha, variedad, clienteId, etapa, cantidad, precio, moneda, costeLogistico }`. |
| PUT | `/api/comercializacion/ventas/:id` | Edita una venta ya registrada (mismo body que POST). El caficultor puede corregir un dato mal introducido sin borrar y recrear la venta. |
| DELETE | `/api/comercializacion/ventas/:id` | Borra una venta por ID. |

### Resumen y descarga (caficultor)
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/comercializacion/resumen` | KPIs agregados: totales, por variedad, por mes, por cliente. |
| GET | `/api/comercializacion/mis-datos/excel` | Descarga Excel del caficultor (responde con `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`). |

### Parámetros (caficultor)
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/comercializacion/parametros` | Tasas de cambio, rendimientos por etapa y costes de referencia por etapa. |
| PUT | `/api/comercializacion/parametros` | Actualiza los parámetros. |

### Panel formador
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/admin/comercializacion/consolidado` | Totales de la comunidad + desglose `porProductor`. |
| GET | `/api/admin/comercializacion/excel` | Excel consolidado compatible con Operativo_Cafeteros. |

## ADRs

### ADR-001: Frontend como repo estático independiente, no integrado en `finanzas-cafeteros/public/`
- **Estado**: Aceptada
- **Contexto**: `finanzas-cafeteros` sirve su propia UI desde `public/`, pero `diagnostico-finca-cafe` y `plan-mejora-calidad-cafe` son repos y despliegues totalmente separados. La cadena de apps del programa ya tiene este patrón establecido.
- **Decisión**: El frontend vive en su propio repo `comercializacion-cafe`, desplegado en GitHub Pages con el mismo patrón.
- **Consecuencias**: (+) Ownership claro; cambios en esta app no afectan las otras. (+) Se puede desplegar/actualizar de forma independiente. (−) Cuatro frontends que deben mantener coherencia visual a mano (no comparten componentes).

### ADR-002: Reutilizar el backend de `finanzas-cafeteros` en vez de crear uno nuevo
- **Estado**: Aceptada
- **Contexto**: Ya existe un backend Express con auth por código que sirve a tres frontends. Crear uno independiente duplicaría auth, patrón de datos y despliegue.
- **Decisión**: Añadir rutas nuevas (`/api/comercializacion/*`, `/api/admin/comercializacion/*`) al `app.js` existente, siguiendo el patrón exacto de `/api/diagnostico`, `/api/plan-accion`, etc.
- **Consecuencias**: (+) Cero infraestructura nueva, reutiliza auth y roles. (−) Dependencia de un repo externo en producción; cualquier cambio en `finanzas-cafeteros/app.js` debe coordinarse.

### ADR-003: Generación de Word en el cliente (librería `docx` por CDN), no en el servidor
- **Estado**: Aceptada
- **Contexto**: El usuario necesita un resumen descargable en Word. El backend ya tiene `docx` como dependencia (la usa `plan-mejora-calidad-cafe`), pero añadir un endpoint de generación de archivo implicaría almacenamiento temporal y un paso de descarga desde el servidor.
- **Decisión**: La librería `docx` se carga por CDN en el cliente (`unpkg`). El Word se genera en memoria en el navegador del usuario y se descarga directamente.
- **Consecuencias**: (+) Cero rutas de backend nuevas para esto; cero almacenamiento de archivos en el servidor. (+) Coherente con el patrón "sin build step, sin servidor adicional". (−) Coste extra de carga (~400 kB de CDN). (−) El resultado depende de las capacidades del navegador del usuario. (−) Babel Standalone + React + docx suman tiempo de carga apreciable en conexiones lentas.

### ADR-004: Detección de backend no disponible con mensaje explicativo
- **Estado**: Aceptada
- **Contexto**: Los endpoints de comercialización pueden no estar aún desplegados en Railway cuando el frontend ya está publicado en GitHub Pages.
- **Decisión**: La función `parsearRespuesta` detecta que la respuesta no es JSON (lo que ocurre cuando Express devuelve un 404 HTML) y muestra un mensaje claro al usuario: "Esta función todavía no está disponible en el servidor. Pide que se despliegue el backend actualizado." Esto evita errores sin sentido en pantalla.
- **Consecuencias**: (+) El usuario no ve un error técnico oscuro. (+) El formador o coordinador sabe exactamente qué acción tomar. (−) No hay reintentos automáticos ni manejo offline.

### ADR-005: Etapas de venta E1-E6 como modelo de datos central
- **Estado**: Aceptada
- **Contexto**: El café en Huila se puede vender en distintos estados de procesamiento: cereza (E1), café en baba (E2), pergamino húmedo (E3), pergamino seco (E4), excelso (E5), tostado (E6). Los costes y rendimientos varían por etapa.
- **Decisión**: El modelo de venta usa una etapa `E1`-`E6` como campo obligatorio. Los parámetros del caficultor permiten definir el coste de referencia por etapa, lo que permite al backend calcular el margen de producto (precio de venta menos coste de la etapa). El margen neto descuenta además el coste logístico.
- **Consecuencias**: (+) El modelo es coherente con la taxonomía que ya usa el programa (Operativo_Cafeteros_PLANTILLA). (+) Permite comparar rentabilidad entre canales en la misma base. (−) La parametrización de etapas y rendimientos tiene defaults Huila — si no se ajustan, los márgenes calculados son aproximaciones.

## Estructura de Archivos y Ownership

| Carpeta/Archivo | Propietario |
|------------------|-------------|
| `comercializacion-cafe/index.html` | Frontend |
| `comercializacion-cafe/specs/`, `README.md` | Arquitecto |
| `finanzas-cafeteros/app.js` (solo rutas `/api/comercializacion/*` y `/api/admin/comercializacion/*`) | Arquitecto — **coordinar antes de editar; backend en producción** |
| Resto de `finanzas-cafeteros/` | Fuera de scope — no tocar |
