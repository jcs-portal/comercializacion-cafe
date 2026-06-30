# Visión del Proyecto: Comercialización de Café (Ubuntu Café / San Adolfo)

## Resumen Ejecutivo
El programa Ubuntu Café (Codespa, San Adolfo y Acevedo, Huila) ya tiene en producción tres apps que comparten el mismo backend (Railway, `doc-comite-finanzas-production.up.railway.app/api`, autenticación por código de caficultor): **Diagnóstico Finca Café** (indicadores de proceso vs. referencias Cenicafé/FNC/SCA), **Finanzas Cafeteros** (situación económica de la finca) y **Plan de Mejora de Calidad de Café** (plan de acción con seguimiento). Esta app cierra el ciclo añadiendo la dimensión comercial: dónde y a qué precio vende cada caficultor, qué canal le es más rentable y cómo evoluciona el grupo en conjunto.

## Problema

### Situación Actual
Los caficultores de San Adolfo y Acevedo venden su café a través de distintos actores (cooperativa, intermediario local, exportador directo, tostador) y en distintas etapas del proceso (cereza, café en baba, pergamino seco, excelso, tostado). Hoy no existe un registro sistemático de estas ventas: el precio recibido, el cliente, la cantidad, los costes logísticos y el margen resultante se gestionan de memoria o en apuntes en papel. El formador tampoco tiene visibilidad del consolidado del grupo para identificar qué productores tienen canales más rentables ni para negociar colectivamente mejores precios.

### Pain Points
- El caficultor no sabe cuánto margen real obtiene por variedad, por canal ni por etapa de venta — solo el precio bruto que le pagaron.
- No hay comparación entre canales: si la cooperativa paga X/kg de pergamino y el exportador directo paga Y/kg de excelso, no hay herramienta que permita comparar en términos netos.
- La información de clientes (quién compra, qué volúmenes, en qué condiciones) no queda registrada y se pierde cuando cambia el formador o cuando el caficultor no recuerda.
- El formador no puede ver el consolidado de la comunidad: quién vende más, a quién, en qué etapas, con qué márgenes, para tomar decisiones de negociación colectiva.
- Los datos de costos de producción (ya registrados en `finanzas-cafeteros`) no se cruzan con los ingresos de venta para calcular rentabilidad real por tipo de producto.

### Coste del Problema
Los archivos de referencia del programa (Gestion_Comercial_Cafeteros.xlsx, Costos_Cafe_Huila_Cenicafe_FNC.xlsx) muestran que la diferencia entre vender en etapa temprana (cereza, E1) vs. etapa tardía (excelso, E5) puede ser de varios puntos de margen, según el diferencial de transformación asumido. Sin visibilidad de estas diferencias, el caficultor no puede tomar una decisión fundamentada sobre dónde vender ni negociar en igualdad de condiciones con los compradores.

## Solución Propuesta

### Visión
Una app ligera, mobile-first (mismo patrón que las tres existentes), que:
1. Permite al caficultor registrar sus ventas de café: fecha, variedad, cliente, etapa de venta, cantidad (kg), precio y moneda.
2. Calcula automáticamente el margen de producto y el margen neto por venta, usando los rendimientos por etapa y los costes de referencia que el propio caficultor puede ajustar.
3. Mantiene un catálogo de clientes del caficultor (tipo de actor, país, ciudad, etapas que compra, incoterm habitual, condición de pago).
4. Muestra un resumen consolidado del caficultor: totales por variedad, por mes y por cliente, con descarga en Excel y Word.
5. Da al formador un panel consolidado del grupo: kg vendidos, ingresos y márgenes agregados por productor, con descarga en Excel compatible con el Operativo_Cafeteros del programa.

### Propuesta de Valor
Transforma el registro ad-hoc de ventas en un historial estructurado que permite al caficultor y al formador analizar la rentabilidad real por canal y negociar desde datos, reutilizando el mismo backend y la misma autenticación que las demás apps del programa.

## Usuarios Objetivo

### Perfil Principal
Caficultor joven de San Adolfo o Acevedo, participante del proyecto Ubuntu/Codespa. **Registra sus propias ventas, clientes y parámetros** desde su móvil con el mismo código que usa en Finanzas y Diagnóstico (ej. `SA-01`). Alfabetización digital variable, conectividad de campo limitada, dispositivo de gama media-baja.

### Perfil Secundario
**Formador (FORM-SA)**: tiene acceso de **lectura transversal** — ve el consolidado de la comunidad (todos los productores), puede descargar el Excel consolidado compatible con Operativo_Cafeteros, pero no edita los datos de ningún caficultor.

## Contexto de Mercado

### Competencia
Ninguna herramienta específica para este grupo; la alternativa actual son apuntes en papel, memoria y hojas Excel de uso esporádico (Operativo_Cafeteros_PLANTILLA.xlsx, Productor_Simplificado.xlsx) que no se actualizan de forma sistemática.

### Diferenciación
Integración directa con el mismo backend que `finanzas-cafeteros` y `diagnostico-finca-cafe`: el caficultor ya tiene código, ya conoce el acceso, no aprende una herramienta nueva. El cálculo de márgenes por etapa reutiliza los rendimientos del proceso que el programa ya tiene parametrizados.

## Restricciones Identificadas
- Conectividad limitada en campo → mobile-first, archivo único, sin dependencias de build.
- Proyecto social, sin presupuesto → misma infraestructura gratuita (GitHub Pages + Railway ya pagado).
- **Confirmado**: frontend estático independiente en GitHub Pages, backend compartido en Railway (mismo patrón que `diagnostico-finca-cafe` y `plan-mejora-calidad-cafe`).
- **Confirmado**: mismo sistema de autenticación por código (`X-Code` + `X-Password` opcional); sin alta de usuarios nueva.
- **Por confirmar**: si los endpoints de comercialización (`/api/comercializacion/*`) ya están desplegados en el backend Railway o si están pendientes de despliegue (la app muestra un mensaje explicativo al usuario cuando la ruta no está disponible aún).

## Criterios de Éxito (preliminares)
- % de caficultores con al menos una venta registrada tras la primera sesión de capacitación.
- El formador usa el panel consolidado para negociar condiciones con compradores en nombre del grupo.
- Reducción del tiempo que dedica el formador a consolidar datos de ventas manualmente (hoy en Excel).
- Al menos un caficultor identifica que un canal de venta diferente le daría mayor margen y ajusta su estrategia.

## Fuentes Usadas para este Borrador
- `index.html` del proyecto (código fuente completo: endpoints consumidos, campos capturados, lógica de márgenes).
- `README.md` del proyecto.
- Archivos de referencia en el directorio del proyecto: `Gestion_Comercial_Cafeteros.xlsx`, `Costos_Cafe_Huila_Cenicafe_FNC.xlsx`, `Simulacion_Cafeteros_2025.xlsx`, `Productor_Simplificado.xlsx`, `Operativo_Cafeteros_PLANTILLA.xlsx`.
- Specs y código de las apps hermanas: `diagnostico-finca-cafe`, `finanzas-cafeteros`, `plan-mejora-calidad-cafe`.
