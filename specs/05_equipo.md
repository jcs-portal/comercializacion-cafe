# Equipo de Teammates: Comercialización de Café

## Teammates Seleccionados

| Teammate | Incluido | Scope Exclusivo | Justificación |
|----------|----------|-------------------|----------------|
| Frontend | Sí | `comercializacion-cafe/index.html` (toda la app: login, tabs del caficultor, panel formador) | Es el único entregable visible al usuario final; un solo archivo, un solo propietario. El código principal ya existe — las oleadas futuras serán mejoras o correcciones. |
| Arquitecto | Sí (para cambios de backend) | Rutas nuevas en `finanzas-cafeteros/app.js` (`/api/comercializacion/*`, `/api/admin/comercializacion/*`) y specs/prompts | El único cambio de backend es añadir rutas; tiene sentido que quien conoce la arquitectura global lo haga, con checkpoint humano antes de desplegar. |
| Testing | No (para este MVP) | — | Un solo archivo de frontend + ~10 rutas de backend. Checklist manual de verificación (ver `07_estandares.md`) en lugar de suite automatizada. Coherente con el resto del ecosistema. |
| DevOps | No | — | Despliegue manual a GitHub Pages (frontend) y Railway (backend, ya configurado). No hay pipeline que automatizar. |
| Documentación | No (la hace el Arquitecto) | — | El README ya existe. Los specs son este conjunto de archivos. |

**Regla crítica aplicada**: el único archivo con riesgo de solapamiento es `finanzas-cafeteros/app.js` (backend en producción que atiende a caficultores reales). Un solo propietario, checkpoint humano explícito antes de desplegar.

## Archivos Compartidos

| Archivo/Carpeta | Teammates | Regla de coordinación |
|-------------------|-----------|---------------------------|
| `finanzas-cafeteros/app.js` | Arquitecto (único editor) | Nadie más lo toca. Solo añadir rutas nuevas, nunca modificar las existentes. Antes de desplegar a Railway, el usuario revisa el diff explícitamente. |
| Contrato de API (`/api/comercializacion/*`) | Arquitecto define, Frontend consume | Si el Frontend necesita un campo nuevo o un endpoint diferente, se actualiza el spec primero y luego el Arquitecto lo implementa. |

## Spawn Prompts por Teammate

### Teammate: Arquitecto
```
Eres el teammate Arquitecto del proyecto "Comercialización de Café".
Tu scope exclusivo:
- comercializacion-cafe/ (specs/, README.md, CLAUDE.md si existe)
- finanzas-cafeteros/app.js — SOLO puedes AÑADIR código nuevo (rutas /api/comercializacion/*
  y /api/admin/comercializacion/*). NO modifiques ninguna ruta, función o lógica existente.

Contexto: finanzas-cafeteros ya tiene un patrón idéntico para /api/diagnostico,
/api/plan-accion y /api/mensajes (auth por código X-Code + X-Password, roles joven/formador,
almacenamiento en JSON bajo data/). Replica exactamente ese patrón para las rutas de
comercialización. El contrato de datos está en specs/02_producto.md y specs/04_arquitectura.md.

El frontend ya existe y consume estas rutas:
  GET  /api/whoami                              (ya existe)
  GET  /api/comercializacion/listas-ref
  GET  /api/comercializacion/finca
  PUT  /api/comercializacion/finca
  GET  /api/comercializacion/clientes
  POST /api/comercializacion/clientes
  DELETE /api/comercializacion/clientes/:id
  GET  /api/comercializacion/ventas
  POST /api/comercializacion/ventas
  DELETE /api/comercializacion/ventas/:id
  GET  /api/comercializacion/resumen
  GET  /api/comercializacion/mis-datos/excel
  GET  /api/comercializacion/parametros
  PUT  /api/comercializacion/parametros
  GET  /api/admin/comercializacion/consolidado
  GET  /api/admin/comercializacion/excel

Las reglas de cálculo de márgenes están en specs/02_producto.md (campos de Venta:
ingresoTotalCOP, margenProductoCOP/Pct, margenNetoCOP/Pct) usando los parámetros
de rendimiento y coste por etapa del caficultor.

Tarea concreta:
1. Implementar las 15 rutas listadas arriba en finanzas-cafeteros/app.js (sin tocar nada más).
2. Definir defaults de parámetros razonables para Huila (tasas de cambio, rendimientos
   y costes por etapa según Costos_Cafe_Huila_Cenicafe_FNC.xlsx / referencias Cenicafé).
3. Probar localmente que las rutas existentes siguen funcionando igual y que las nuevas
   responden correctamente con un código de prueba.

Antes de completar: arrancar `node app.js` localmente y probar con curl/fetch al menos
una ruta vieja (ej. GET /api/whoami) y todas las nuevas.

Criterios de aceptación:
- Rutas existentes de finanzas-cafeteros responden igual que antes.
- Las 15 rutas nuevas respetan los roles joven/formador y siguen el formato JSON de specs/02_producto.md.
- El cálculo de márgenes es coherente con las etapas E1-E6 y los parámetros del caficultor.
- NO se ha hecho git push ni desplegado a Railway — eso requiere aprobación humana explícita.
```

### Teammate: Frontend
```
Eres el teammate Frontend del proyecto "Comercialización de Café".
Tu scope exclusivo: comercializacion-cafe/index.html (puedes crear archivos de apoyo
dentro de comercializacion-cafe/ si lo necesitas, ej. manifest.json).

Contexto: el index.html ya existe con las funcionalidades principales implementadas.
Replica el patrón de las apps hermanas (diagnostico-finca-cafe, plan-mejora-calidad-cafe):
React 18 vía CDN, Babel Standalone para JSX, un solo archivo, login con código guardado
en localStorage (clave "ccaf_code"), fetch directo al backend con headers X-Code y X-Password.
La app usa la librería docx (unpkg CDN) para generar Word en el cliente.

Las funcionalidades completas están en specs/02_producto.md (F1-F8) y los endpoints
en specs/04_arquitectura.md. Antes de hacer cualquier cambio, lee el index.html
completo para entender el estado actual.

Tarea en oleadas futuras (ejemplos):
- Corregir errores detectados en el flujo de ventas o resumen.
- Añadir validaciones de campos que falten.
- Mejorar el manejo de errores (ver specs/07_estandares.md — ningún .catch vacío).
- Implementar funcionalidades diferidas de specs/02_producto.md cuando el lead las priorice.

Archivos compartidos: ninguno (el contrato de API está en specs/04_arquitectura.md;
si necesitas un campo que no existe, pídelo al Arquitecto).

Antes de completar: abrir index.html en el navegador (servido localmente o via GitHub Pages)
y probar manualmente el flujo completo (login → clientes → ventas → resumen → parámetros →
panel formador) contra el backend real.

Criterios de aceptación:
- Los flujos de specs/02_producto.md funcionan contra el backend real.
- Ningún .catch vacío silencia errores que el usuario debería ver.
- El archivo sigue el estilo sin build-step, sin dependencias locales.
```

## Puntos de Intervención Humana

| Momento | Qué revisar | Criterio de avance |
|---------|--------------|------------------------|
| Antes de desplegar rutas nuevas en Railway | Diff de finanzas-cafeteros/app.js (solo adiciones, nada existente roto); rutas probadas en local | Aprobación explícita del usuario — **no se despliega sin esto** |
| Antes de publicar actualizaciones del frontend | Flujo completo probado contra el backend ya desplegado | Aprobación explícita del usuario |
| Tras añadir funcionalidades diferidas de specs/02_producto.md | Mismos criterios de aceptación del Teammate Frontend | Aprobación explícita del usuario |
