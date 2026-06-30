# Especificación de Producto: Comercialización de Café

## Alcance del MVP (estado actual — derivado del código)

La app ya tiene implementadas las siguientes funcionalidades en `index.html`:

### Funcionalidades implementadas
| ID | Funcionalidad | Descripción | Pantalla |
|----|---------------|-------------|---------|
| F1 | Login con código | Reutiliza el código y backend de `finanzas-cafeteros` (`X-Code` + `X-Password` opcional). Sin alta de usuarios nueva. Detecta rol `joven` (caficultor) vs `formador` y enruta a la vista correspondiente. | Login |
| F2 | Mis datos (identidad fiscal) | Datos del caficultor como vendedor: nombre/razón social, tipo de persona (natural/jurídica), identificador fiscal (NIT o cédula), dirección, teléfono/WhatsApp. Se rellenan una sola vez. GET/PUT `/api/comercializacion/finca`. | Tab "Mis datos" |
| F3 | Mis clientes | Catálogo de compradores: nombre, tipo de actor, país, ciudad, etapas que compra (ej. E3/E4), incoterm habitual, condición de pago. Alta, listado y baja. GET/POST/DELETE `/api/comercializacion/clientes`. | Tab "Clientes" |
| F4 | Mis ventas | Registro de ventas: fecha, variedad, cliente (del catálogo propio), etapa de venta (E1-E6), cantidad (kg), precio, moneda (COP/USD/EUR) y coste logístico opcional. Calcula y muestra: ingreso total en COP, margen de producto y margen neto (absoluto y %). GET/POST/DELETE `/api/comercializacion/ventas`. | Tab "Ventas" |
| F5 | Mi resumen | Totales del caficultor: kg vendidos, ingreso total, margen de producto y margen neto. Desglosado por variedad, por mes y por cliente. Descarga en Excel y en Word. GET `/api/comercializacion/resumen`, GET `/api/comercializacion/mis-datos/excel`. | Tab "Resumen" |
| F6 | Parámetros | Valores que el caficultor puede ajustar: tasas de cambio (COP/USD, COP/EUR), rendimientos por etapa (cereza→baba, baba→pergamino, pergamino→excelso, excelso→tostado) y costes de referencia por etapa (E1-E6 en COP/kg). Tienen defaults razonables para Huila. GET/PUT `/api/comercializacion/parametros`. | Tab "Parámetros" |
| F7 | Panel del formador | Vista consolidada de toda la comunidad: kg totales, ingreso total, margen de producto y margen neto; desglose por productor. Descarga de Excel consolidado compatible con Operativo_Cafeteros. GET `/api/admin/comercializacion/consolidado`, GET `/api/admin/comercializacion/excel`. | Vista formador (sin tabs) |
| F8 | Descarga Word | Resumen del caficultor exportado a Word (.docx) generado en el cliente usando la librería `docx` por CDN. Incluye tablas de totales, por variedad, por mes y por cliente. | Tab "Resumen" |

### Funcionalidades diferidas (no están en el código actual)
- Cruce con datos de costos de producción de `finanzas-cafeteros` para calcular rentabilidad real por ciclo de cosecha.
- Comparación entre canales de venta (¿qué habría pasado si hubiera vendido en E5 en lugar de E3?).
- Notificaciones de precios de referencia (FNC, mercados internacionales) para apoyar la negociación.
- Histórico de precios por cliente y por etapa, con visualización de tendencias.
- Integración con `plan-mejora-calidad-cafe` para cruzar acciones de mejora con variación de precios obtenidos.

### Alcance Negativo
- No gestiona la producción de la finca (eso es `diagnostico-finca-cafe` y `finanzas-cafeteros`).
- No genera facturas ni documentos legales de venta.
- No almacena archivos binarios (el Word se genera en el cliente, no en el servidor).
- No crea ni modifica usuarios/códigos (eso ya lo hace `finanzas-cafeteros` vía `/api/admin/codigo`).

## Plataformas
Web móvil, un único `index.html` con React 18 + Babel por CDN, sin build step. Mismo patrón exacto que `diagnostico-finca-cafe` y `plan-mejora-calidad-cafe`. Sin apps nativas — coherente con conectividad limitada y bajo presupuesto.

## Requisitos No Funcionales
- **Rendimiento**: carga rápida en 3G/4G rural; un solo archivo HTML, scripts CDN bien conocidos. La librería `docx` (para generar Word en cliente) se carga por CDN (unpkg) y es el único añadido respecto a las apps hermanas.
- **Disponibilidad**: depende del mismo Railway que ya corre `finanzas-cafeteros`; sin SLA formal (proyecto social).
- **Seguridad**: mismo esquema que las demás apps (`X-Code` + `X-Password` opcional). Los datos de ventas son datos de un solo productor — el caficultor solo ve los suyos; el formador ve el consolidado de su comunidad.
- **Escalabilidad**: grupo pequeño (San Adolfo + Acevedo, decenas de caficultores); no se diseña para volumen.
- **Offline**: no hay modo offline explícito — la app requiere conexión para guardar/leer. Coherente con el resto del ecosistema.

## Integraciones
| Sistema | Tipo | Estado |
|---------|------|--------|
| Backend Railway (finanzas-cafeteros extendido) | API REST: `/api/comercializacion/*` y `/api/admin/comercializacion/*` | Por confirmar si ya desplegado. La app muestra mensaje explicativo si la ruta no está disponible. |
| `GET /api/whoami` | Autenticación y detección de rol | Existente (shared con todas las apps hermanas) |
| `GET /api/comercializacion/listas-ref` | Catálogos de referencia: variedades, etapas, tipos de cliente, incoterms, monedas | Por confirmar |
| Librería `docx` vía unpkg CDN | Generación de Word en cliente | Implementada en frontend |

## Estructura de Tabs (caficultor)

La navegación del caficultor tiene 5 pestañas con el orden siguiente (diseño deliberado):

1. **Ventas** (icono 🧾) — pestaña activa por defecto. El día a día es registrar venta por venta.
2. **Clientes** (icono 👥) — catálogo de compradores. Prerequisito para registrar ventas.
3. **Mis datos** (icono 🪪) — identidad fiscal. Se rellena una sola vez, queda relegada a posición 3.
4. **Resumen** (icono 📈) — KPIs y descarga de informes.
5. **Parámetros** (icono ⚙️) — rendimientos y costes por etapa. Defaults Huila, se cambia solo si se conoce el coste real.

## Modelo de Datos (derivado del código)

### Finca / datos del vendedor
```
{ nombreFinca, tipoPersona ("natural"|"juridica"), identificadorFiscal, direccion, telefono }
```

### Cliente
```
{ id, nombre, tipoActor, pais, ciudad, etapasQueCompra, incotermHabitual, condicionPago, activo }
```

### Venta
```
{ id, fecha, variedad, clienteId, etapa ("E1"-"E6"), cantidad (kg), precio, moneda ("COP"|"USD"|"EUR"), costeLogistico }
// Calculados por el backend y devueltos en la respuesta:
{ ingresoTotalCOP, margenProductoCOP, margenProductoPct, margenNetoCOP, margenNetoPct, nombreCliente }
```

### Parámetros
```
{
  tasaCambioUSD, tasaCambioEUR,
  rendimientos: { cerezaABaba, babaAPergamino, pergaminoAExcelso, excelsoATostado },
  costesReferenciaPorEtapa: { E1, E2, E3, E4, E5, E6 }
}
```

### Resumen (respuesta de `/api/comercializacion/resumen`)
```
{
  totalKg, ingresoTotal, margenProductoTotal, margenProductoPct, margenNetoTotal, margenNetoPct,
  numPedidos, numClientesDistintos,
  porVariedad: { [variedad]: { kg, ingreso, margenNeto } },
  porMes: { [mes]: { kg, ingreso } },
  porCliente: { [clienteId]: { kg, ingreso, nombreCliente } }
}
```

### Consolidado formador (respuesta de `/api/admin/comercializacion/consolidado`)
```
{
  totalKg, ingresoTotal, margenProductoTotal, margenProductoPct, margenNetoTotal, margenNetoPct,
  numProductoresConVentas,
  porProductor: [{ codigo, totalKg, ingresoTotal, margenNetoTotal, numPedidos }]
}
```

## Flujos Principales

### Flujo caficultor (primera vez)
1. Login con código → `GET /api/whoami` (ya existe) → detección de rol `joven`.
2. `GET /api/comercializacion/listas-ref` → catálogos de referencia (variedades, etapas, tipos, incoterms, monedas).
3. Pestaña "Clientes" → `POST /api/comercializacion/clientes` → crear al menos un cliente.
4. Pestaña "Ventas" → `POST /api/comercializacion/ventas` → registrar primera venta.
5. Pestaña "Mis datos" → `PUT /api/comercializacion/finca` → rellenar datos de facturación (una sola vez).

### Flujo caficultor (uso habitual)
1. Login (código guardado en `localStorage` → auto-verificación con `GET /api/whoami`).
2. Pestaña "Ventas" (activa por defecto) → `POST /api/comercializacion/ventas` → añadir venta.
3. Pestaña "Resumen" → `GET /api/comercializacion/resumen` → ver KPIs actualizados.
4. Opcional: descargar Excel (`GET /api/comercializacion/mis-datos/excel`) o Word (generado en cliente).

### Flujo formador
1. Login con código de formador (FORM-SA).
2. `GET /api/admin/comercializacion/consolidado` → panel de la comunidad.
3. Ver desglose por productor.
4. Descargar Excel consolidado (`GET /api/admin/comercializacion/excel`) para Operativo_Cafeteros.
