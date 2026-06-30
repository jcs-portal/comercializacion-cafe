# Restricciones y Recursos: Comercialización de Café

## Presupuesto
Proyecto social (Ubuntu Café / Codespa), sin presupuesto de desarrollo comercial. Coste objetivo: ~0 adicional — se reutiliza el mismo hosting (GitHub Pages para el frontend, Railway ya pagado para el backend) que las otras apps del ecosistema. Única dependencia nueva respecto a las apps hermanas: librería `docx` vía unpkg CDN (gratuita, cargada en cliente), ya integrada en `index.html`.

## Timeline
Sin fecha límite dura. El `index.html` ya existe y tiene las funcionalidades principales implementadas. La prioridad actual es completar/validar que los endpoints del backend estén desplegados en Railway y que el flujo completo funcione de punta a punta en producción.

## Supervisión
- Responsable: Juan Carlos (mismo operador que supervisa `diagnostico-finca-cafe`, `finanzas-cafeteros` y `plan-mejora-calidad-cafe`).
- Nivel técnico: alto — conoce el código de todas las apps del ecosistema y su despliegue en Railway/GitHub Pages.
- Disponibilidad: la suficiente para checkpoints entre oleadas (proyecto pequeño).

## Preferencias Tecnológicas
- **Confirmado**: mantener el mismo patrón que las apps existentes — frontend single-file HTML + React + Babel (sin build step), backend Express ya desplegado en Railway, sin infraestructura nueva.
- La única dependencia añadida respecto a las apps hermanas es `docx` por CDN (unpkg), para generar el Word del resumen en el cliente. Esta decisión ya está tomada e implementada.
- Evitar introducir dependencias de servidor nuevas para la generación de documentos: el Word se genera íntegramente en el cliente.

## Normativa
| Regulación | Aplica | Implicaciones |
|------------|--------|----------------|
| Protección de datos personales (datos de ventas de pequeños productores) | Sí, de forma informal | Mismo nivel de cuidado que las apps existentes: acceso solo por código, sin recolectar datos adicionales a los estrictamente necesarios. Los datos de ventas (precio, cliente, cantidades) son sensibles comercialmente — acceso restringido al propio caficultor y al formador de su comunidad. |
| Normativa tributaria colombiana | No aplica como obligación de la app | Los datos de identificación fiscal (NIT/cédula) se recogen para que el caficultor pueda usar la app como soporte de sus propias gestiones, no para que la app genere facturas oficiales. |

## Puntos de Atención Especiales

### Backend compartido en producción
Los endpoints `/api/comercializacion/*` y `/api/admin/comercializacion/*` son rutas nuevas en el mismo `app.js` de `finanzas-cafeteros` que ya atiende a caficultores reales. El código del frontend ya gestiona el caso en que la ruta no está disponible mostrando un mensaje claro al usuario (`parsearRespuesta` detecta respuesta no-JSON y muestra: "Esta función todavía no está disponible en el servidor"). Esto es una salvaguarda temporal hasta que el backend esté actualizado.

### Conectividad limitada en campo
Los usuarios operan en zonas rurales de Huila con conectividad intermitente (3G/4G). El frontend carga por CDN (React, ReactDOM, Babel, docx): si el usuario no tiene conexión al abrir la app por primera vez, la carga fallará. No hay service worker ni modo offline — coherente con las apps hermanas.

### Dispositivos de gama media-baja
La app está optimizada para pantallas pequeñas: viewport fijo (`maximum-scale=1`), botones con `min-height: 44px`, fuentes no menores de 16px en inputs (para evitar zoom automático en iOS), `overscroll-behavior: none`, `-webkit-overflow-scrolling: touch`. La librería Babel Standalone (para JSX in-browser) tiene un coste de carga mayor que las apps que no la usan — es la misma decisión que tomaron las apps hermanas y está aceptada.

### Dependencia de `finanzas-cafeteros`
El frontend depende completamente del backend de `finanzas-cafeteros` para todas sus operaciones. Cualquier actualización de ese backend (nuevas rutas, cambios en la autenticación, redespliegue) afecta directamente a esta app. El coordinador del backend es el responsable de mantener el contrato de API estable.
