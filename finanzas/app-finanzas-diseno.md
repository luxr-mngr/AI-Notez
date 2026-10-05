# App de finanzas personales - Documento de diseño

Versión 0.1 · 2026-10-05 · Estado: borrador para revisión

## 1. Objetivo

Una app web personal para saber, en cualquier momento, **cuánto puedo gastar
hoy y este mes**. También registra gastos, saldos y metas de ahorro.
Claude puede leer y escribir datos a través de un servidor MCP.

El plan de deuda queda fuera. Se maneja en Excel (`libro-caja-deuda.xlsx`).

## 2. Problema actual

- Registro en Excel, actualizado casi cada semana desde las apps del banco.
- Al registrar tarde, se olvida la descripción del gasto.
- Una app anterior falló porque se olvidaba registrar.
- No hay una vista rápida de "cuánto me queda" en el celular.

## 3. Requisitos (del cuestionario)

| # | Tema | Decisión |
|---|---|---|
| 1 | Registro | Formulario rápido de **varias filas** + registro vía Claude (MCP) |
| 2 | Frecuencia | Variable: al momento, en la noche o cada semana |
| 3 | Vista principal | **Cuánto me queda este mes** y **cuánto puedo gastar por día** hasta el próximo sueldo. Debajo, con menos jerarquía: cuánto debo |
| 4 | Límites | Tope mensual por categoría + monto diario general |
| 5 | Alertas | Solo visuales, tipo semáforo |
| 6 | Umbral | Amarillo al 80%, rojo al 100% |
| 7 | Fechas de pago | Fuera por ahora |
| 8 | Alcance | Presupuesto mensual, saldos de cuentas, metas de ahorro, análisis. **Sin plan de deuda** |
| 9 | Saldos | Sí, con resumen general |
| 10 | Ingresos variables | Sí (sueldo + freelance) |
| 11 | Dólares | Sí. Se convierte a soles con el tipo de cambio del día |
| 12 | Clientes de Claude | Claude Code desktop y app de Claude en iPhone |
| 13 | Permisos de Claude | Registrar gastos, ingresos, pagos y prepagos de deuda. Consultar todo. Conciliar estados de cuenta. Resumen semanal. **No** cambia topes |
| 14 | Confirmación | **Siempre** antes de guardar |
| 15 | Acceso | Solo el usuario, login con Google, **sesión persistente** |
| 16 | Datos sensibles | No guardar números de cuenta ni identificadores bancarios |
| 17 | Exportar | No por ahora (MCP lo cubre) |
| 18 | Infra | Cuenta Cloudflare + dominio `luxrcore.com` |
| 19 | Stack | El más liviano y con menos mantenimiento |
| 20 | Repo | Repositorio nuevo |
| 21 | Plazo | Lo antes posible |

## 4. Conceptos clave

### 4.1 Ciclo de presupuesto

El ciclo va **de un día de sueldo al siguiente**, no del día 1 al 30.
El día de sueldo es configurable (hoy: último día del mes).

### 4.2 Qué cuenta como gasto

| Tipo de movimiento | ¿Cuenta en el presupuesto? | Ejemplo |
|---|---|---|
| Gasto | Sí | Taxi, supermercado, Claude en USD |
| Ingreso | No (suma al disponible) | Sueldo, freelance |
| Pago de tarjeta | **No** | Pago del estado de cuenta Interbank |
| Pago de deuda | No (se muestra aparte) | Cuota o prepago del ExtraCash |
| Transferencia | No | BCP a Interbank |

Regla: el gasto se registra **cuando ocurre**, con el método de pago usado.
Pagar la tarjeta después no es un gasto nuevo. Así no se cuenta dos veces.

### 4.3 Métricas principales

- **Te queda este mes** = suma de topes del ciclo - gastos del ciclo.
- **Hoy puedes gastar** = te queda este mes / días que faltan para el sueldo.
- **Por categoría** = tope - gastado, con su propio semáforo.

### 4.4 Semáforo

| Color | Condición |
|---|---|
| Verde | Menos del 80% del tope |
| Amarillo | Entre 80% y 99% |
| Rojo | 100% o más |

"Hoy puedes gastar" se pone amarillo si el gasto de hoy pasa el 80% del
monto diario y rojo si lo pasa por completo.

## 5. Funcionalidades

### 5.1 Registro rápido (multi-fila)

- Botón "Registrar gastos" abre una tabla compacta. Cada fila es un gasto.
- Botones "+1 fila" y "+5 filas". Se guardan todas juntas.
- Cada nueva fila copia la fecha y el método de pago de la fila anterior.
- En el celular cada fila ocupa una línea. Toca para ver todos los campos.

Campos por fila:

| Campo | Regla |
|---|---|
| Fecha | Hoy por defecto. Editable para fechas pasadas |
| Método de pago | Lista de métodos con su color |
| Categoría | Transporte, Alimentación, Compras, Entretenimiento, Salud y bienestar, Servicios, Otros |
| Empresa | "Ninguna" por defecto. Ej. Luxr |
| Descripción | Corta. Opcional |
| Monto | En soles o dólares (selector PEN/USD) |
| Tipo de cambio | Automático con el del día si es USD. Editable |

### 5.2 Dashboard (pantalla de inicio)

1. **Te queda este mes** (número grande + semáforo).
2. **Hoy puedes gastar** S/ X · faltan N días para el sueldo.
3. Barras por categoría: gastado / tope.
4. Saldo general: suma de cuentas de débito - deuda de tarjetas.
5. Cuánto debo (total, sin fechas). Menor jerarquía.
6. Últimos 5 movimientos.

### 5.3 Métodos de pago

CRUD simple: nombre, tipo (crédito / débito), banco, color, activo.
Sin números de cuenta. Datos iniciales:

| Nombre | Tipo |
|---|---|
| TC Interbank | Crédito |
| TC BCP | Crédito |
| Ahorros Interbank | Débito |
| Ahorros BCP | Débito |

### 5.4 Saldos

- Registro manual de saldo por cuenta y fecha (formulario o vía Claude
  con una captura).
- Resumen general: total en débito, total deuda en tarjetas, neto.

### 5.5 Metas de ahorro

- Nombre, monto objetivo, fecha objetivo, aportes.
- Barra de avance. Ej. "Fondo de emergencia", "Maestría".

### 5.6 Análisis

- Gasto mensual por categoría a través de los meses (barras apiladas).
- Comparación mes actual vs promedio de los últimos 3 meses.
- Gasto por método de pago y por empresa.
- Ingresos vs gastos por mes.

## 6. Integración con Claude (MCP)

Servidor MCP remoto (HTTP) en el mismo Worker. Se agrega como conector
personalizado en claude.ai para usarlo desde todos los dispositivos, y con
`claude mcp add` en Claude Code.

### 6.1 Herramientas

| Herramienta | Tipo | Descripción |
|---|---|---|
| `resumen_mes` | Lectura | Te queda, hoy puedes gastar, estado por categoría |
| `listar_movimientos` | Lectura | Filtros por fecha, categoría, método, empresa |
| `analisis` | Lectura | Gasto por categoría y mes, tendencias |
| `saldos` | Lectura | Saldos por cuenta y resumen general |
| `metas` | Lectura | Avance de metas de ahorro |
| `resumen_semanal` | Lectura | Gasto de la semana, alertas, comparación |
| `preparar_movimientos` | Borrador | Recibe 1 o más gastos, ingresos o pagos. Devuelve una vista previa y un `borrador_id` |
| `preparar_saldo` | Borrador | Saldo de una cuenta en una fecha |
| `preparar_conciliacion` | Borrador | Recibe movimientos extraídos de un estado de cuenta. Devuelve: coinciden, faltan registrar, sobran |
| `confirmar` | Escritura | Guarda un `borrador_id` |
| `descartar` | Escritura | Borra un borrador |

**Confirmación obligatoria:** ninguna herramienta escribe directo. Todo pasa
por borrador → el usuario aprueba → `confirmar`. Los borradores expiran en
24 horas.

### 6.2 Conciliación de estados de cuenta

1. El usuario envía PDF o captura a Claude.
2. Claude extrae las filas y llama a `preparar_conciliacion`.
3. El servidor busca coincidencias por monto exacto y fecha ±2 días.
4. Claude muestra la tabla y pregunta qué agregar.
5. Lo que el usuario aprueba se guarda con `confirmar`.

Esto resuelve el registro semanal sin perder lo anotado en el momento.

## 7. Arquitectura

```
Celular / Compu ──► finanzas.luxrcore.com ──► Cloudflare Access (Google)
                                                   │
                                                   ▼
                                    Cloudflare Worker (Hono, TypeScript)
                                     ├─ Páginas HTML (SSR) + JS mínimo
                                     ├─ API /api/*
                                     └─ MCP /mcp  (OAuth propio, Google)
                                                   │
                                                   ▼
                                           Cloudflare D1 (SQLite)
                                                   ▲
                               Cron diario ────────┘ (tipo de cambio)
```

### 7.1 Stack

| Pieza | Elección | Motivo |
|---|---|---|
| Runtime | Cloudflare Workers | Gratis, sin servidor que mantener |
| Framework | Hono + JSX en servidor | Liviano, sin build de frontend complejo |
| Interacción | JS vanilla mínimo (formulario multi-fila) | Sin framework de cliente |
| Gráficos | Chart.js desde CDN | Una librería, solo en la página de análisis |
| Base de datos | D1 | SQLite gestionado, backups con Time Travel (30 días) |
| MCP | `agents` (McpAgent) + `workers-oauth-provider` | Plantilla oficial de Cloudflare para MCP remoto |
| Deploy | `wrangler deploy` desde el repo nuevo | Un comando |
| Celular | PWA (agregar a pantalla de inicio en iPhone) | Se abre como app |

### 7.2 Autenticación

- **Web:** Cloudflare Access con Google. Solo se permite el email del
  usuario. Duración de sesión: **1 mes** (para evitar logins repetidos).
- **MCP:** Access no sirve para el flujo OAuth de los clientes MCP. El
  endpoint `/mcp` usa su propio OAuth con Google (`workers-oauth-provider`)
  y la misma lista de un solo email.

### 7.3 Tipo de cambio

- Cron diario guarda compra y venta del día (fuente: SUNAT vía API pública).
- Gastos en USD usan la venta del día. Si no hay dato, el último disponible.
- El tipo de cambio queda guardado en cada movimiento.

## 8. Modelo de datos (D1)

```sql
metodos_pago (id, nombre, tipo, banco, color, activo)
categorias   (id, nombre, color, orden)
empresas     (id, nombre)                       -- "Ninguna" por defecto
movimientos  (id, fecha, tipo, metodo_pago_id, categoria_id, empresa_id,
              descripcion, moneda, monto_original, tipo_cambio, monto_pen,
              origen, conciliado, creado_en)
              -- tipo: gasto | ingreso | pago_tarjeta | pago_deuda | transferencia
              -- origen: web | mcp | conciliacion
topes        (ciclo_inicio, categoria_id, monto)
saldos       (id, metodo_pago_id, fecha, saldo)
metas        (id, nombre, objetivo, fecha_objetivo)
aportes_meta (id, meta_id, fecha, monto)
tipo_cambio  (fecha, compra, venta, fuente)
borradores   (id, payload_json, creado_en, expira_en)
config       (clave, valor)                     -- día de sueldo, etc.
```

Sin números de cuenta, tarjeta ni identificadores bancarios en ninguna tabla.

## 9. Fases

| Fase | Contenido | Resultado |
|---|---|---|
| 1. MVP | D1, catálogos, formulario multi-fila, dashboard, semáforo, tipo de cambio, Access, deploy en `finanzas.luxrcore.com` | Registrar y ver "hoy puedes gastar" desde el celular |
| 2. MCP | Servidor MCP, OAuth, herramientas de lectura y borrador/confirmar | Registrar y consultar hablando con Claude |
| 3. Completo | Saldos, metas, análisis, conciliación, resumen semanal | Reemplaza el Excel de gastos |
| 4. Migración | Importar el histórico del Excel actual | Análisis con meses anteriores |

## 10. Riesgos

| Riesgo | Mitigación |
|---|---|
| Olvidar registrar (pasó antes) | Formulario multi-fila rápido + conciliación semanal con Claude |
| Conector MCP no disponible en algún cliente | Validar en Fase 2 en Claude Code desktop y iPhone antes de seguir |
| Sesión de Access expira en el PWA de iPhone | Sesión de 1 mes. Revisar comportamiento real en Fase 1 |
| API de tipo de cambio cae | Usar último valor guardado + edición manual |
| Pérdida de datos | D1 Time Travel + consulta completa vía MCP |

## 11. Preguntas abiertas

1. **Topes iniciales por categoría.** Propuesta a partir del plan actual:

| Categoría | Tope propuesto | Base |
|---|---|---|
| Alimentación | S/ 800 | Super 500 + delivery 300 |
| Transporte | S/ 300 | Taxis |
| Servicios | S/ 1,161.50 hasta dic, S/ 346.50 desde ene | Luz, gas, internet, celular, Interseguro + pensión |
| Compras | S/ 100 | Por definir |
| Entretenimiento | S/ 50 | Por definir |
| Salud y bienestar | S/ 50 | Por definir |
| Otros | S/ 200 | Incluye contador Luxr (S/ 150) |

2. ¿El contador de Luxr va como gasto personal o con empresa "Luxr"?
3. ¿Tienes deuda en la **TC BCP**? No apareció en el plan de deuda.
4. ¿Puedes compartir el Excel actual de gastos para la migración (Fase 4)?
5. ¿El sueldo llega siempre el último día del mes, o el último día hábil?
6. ¿Subdominio preferido? Propuesta: `finanzas.luxrcore.com`.
