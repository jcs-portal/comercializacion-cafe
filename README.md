# Comercialización de Café

Frontend de la app de comercialización del programa Ubuntu Café (San Adolfo y
Acevedo, Huila). Un único `index.html` (React + Babel por CDN, sin build) que
consume la API de [`finanzas-cafeteros`](https://github.com/jcs-portal/finanzas-cafeteros)
desplegada en Railway.

Se entra con el mismo código que en Finanzas Cafeteros y Diagnóstico de Finca
(ej. `SA-01`). El productor registra finca, clientes y ventas; el formador ve
el consolidado de la comunidad.

Mismo patrón de despliegue que `diagnostico-finca-cafe` y
`plan-mejora-calidad-cafe`: frontend estático en GitHub Pages, backend
compartido en Railway.
