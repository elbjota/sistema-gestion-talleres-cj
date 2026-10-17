# Sistema de gestión para talleres por Telegram

**Automatización para pequeños negocios · Piloto con CJ Car Solutions (Trujillo, Perú) · 2026**

Bot de Telegram construido con **n8n + Google Sheets + IA (Gemini)**. El dueño de un taller de planchado y pintura **cotiza, controla trabajos, inventario y cobros, y entrega notas de venta en PDF** escribiéndole o hablándole al bot desde el celular.

![n8n](https://img.shields.io/badge/n8n-workflow%2027%20nodos-EA4B71?logo=n8n&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-Bot%20API-26A5E4?logo=telegram&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google%20Sheets-base%20de%20datos-34A853?logo=googlesheets&logoColor=white)
![Gemini](https://img.shields.io/badge/IA-Gemini-8E75B2?logo=googlegemini&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-Node.js-F7DF1E?logo=javascript&logoColor=black)
![Tests](https://img.shields.io/badge/pruebas-92%2F92%20pasan-15803D)

📄 **[Ver el caso de estudio completo (PDF)](docs/caso-de-estudio.pdf)**

| Cotización lista en Telegram | PDF que recibe el cliente |
|---|---|
| ![Cotización en Telegram](img/tg-06-cotizacion-pdf.png) | ![PDF de cotización](img/cotizacion-001-ejemplo.png) |

---

## El problema

CJ Car Solutions llevaba sus cotizaciones, trabajos, compras de material y cobros con plantillas sueltas y una hoja de cálculo que había crecido sin orden:

- **Cotizaciones lentas:** se armaban a mano, con formato variable y sin un registro ordenado.
- **Inventario poco confiable:** había movimientos duplicados, montos guardados como texto, compras cargadas a un "trabajo falso" y stock buscado por nombre.
- **No se sabía cuánto dejaba cada trabajo:** el material usado no se asociaba a cada trabajo.
- **Cobros desordenados:** los adelantos y saldos se llevaban de memoria, y el tablero contaba como ingreso trabajos que aún no se cobraban.

> El reto: la solución tenía que usarse **con las manos ocupadas, en medio del taller y sin capacitación**. Por eso la interfaz es Telegram, con botones y notas de voz.

## La solución

| Módulo | Qué hace |
|---|---|
| **Cotizaciones** | Paso a paso con botones o en una sola frase; trabajos, productos o ambos; IGV a elección (sumar, incluido o sin IGV); descuento; PDF con la marca del taller |
| **Trabajos** | Pasar de cotización a trabajo, estados, material usado por trabajo y **ganancia por trabajo** |
| **Inventario** | Compras, consumos y ajustes por conteo; SKU automático; alerta de stock mínimo y capital invertido |
| **Cobros** | Adelanto y saldo, y lo que queda por cobrar |
| **Documentos** | Cotización, nota de venta (por Telegram o correo) y reporte interno en PDF |
| **IA (Gemini)** | Entiende texto libre y notas de voz ("compré cinco lijas a cuatro soles") y **siempre pide confirmar** antes de guardar |
| **Roles y licencias** | Dueño y trabajador (el trabajador no ve montos); claves de activación por taller para revenderlo |

## Arquitectura

![Arquitectura](img/arquitectura.png)

**Decisiones clave:**
- **La IA solo interpreta.** Gemini convierte la frase en JSON; las reglas (IDs, stock, montos, permisos) las aplica un motor en JavaScript con pruebas. Un agente de IA que escribía directo en la hoja generaba duplicados.
- **Nada se guarda sin confirmar** (✅ Guardar · ✏️ Corregir · ❌ Cancelar).
- **Sin duplicados:** se descartan los `update_id` repetidos de Telegram y cada escritura lleva una clave de idempotencia.
- **Apps Script como API** de la hoja, para no tener credenciales OAuth de Google en n8n; también envía los correos.
- **PDF en JavaScript puro**, porque el n8n administrado no permite instalar librerías.
- **"Nota de venta", no boleta:** una boleta válida en Perú es electrónica (SUNAT).

## El bot en acción

| Nueva cotización | El bot guía y suma |
|---|---|
| ![Nueva cotización](img/tg-02-nueva.png) | ![Pasos de la cotización](img/tg-03-pasos.png) |
| **IGV a elección** | **Resumen antes del PDF** |
| ![IGV](img/tg-04-igv.png) | ![Resumen](img/tg-05-resumen.png) |

### IA con control

| Cotización en una frase | Compra en lenguaje natural (cancelada: no se guarda nada) |
|---|---|
| ![IA texto libre](img/tg-07-ia-texto-libre.png) | ![Compra con IA](img/tg-11-compra-ia.png) |

### Gestión del taller

| Inventario (`/stock`) | Ficha del trabajo y nota de venta |
|---|---|
| ![Stock](img/tg-08-stock.png) | ![Ficha de trabajo](img/tg-10-ficha-nota.png) |

### Documentos PDF

| Cotización | Nota de venta | Reporte interno |
|---|---|---|
| [![Cotización](img/cotizacion-001-ejemplo.png)](docs/cotizacion-001-ejemplo.pdf) | [![Nota de venta](img/nota-de-venta-ejemplo.png)](docs/nota-de-venta-ejemplo.pdf) | [![Reporte interno](img/reporte-interno-ejemplo.png)](docs/reporte-interno-ejemplo.pdf) |

## Workflow en n8n (27 nodos)

![Workflow n8n](img/n8n-workflow.png)

## Tecnologías

- **n8n** (autoalojado en Hostinger), desplegado por API con scripts propios
- **JavaScript / Node.js:** motor de conversación, gestión, IA, PDF y montos en céntimos
- **92 pruebas automáticas** con `node --test`, todas pasan
- **Google Sheets + Apps Script** como base de datos, tablero y API
- **Google Gemini** (`gemini-3.5-flash-lite`) con salida en JSON
- **Telegram Bot API** (texto, voz, botones y documentos) y **Gmail**

## Estado

- ✅ **Piloto con CJ Car Solutions** (octubre de 2026): cotizaciones, inventario, trabajos, cobros y documentos probados con el bot real.
- ✅ **IA validada en vivo:** 7 de 8 frases reales se interpretaron bien a la primera; en la otra, el bot completó los datos y pidió confirmar.
- ⏳ **Pendiente:** prueba en vivo del correo, conteo físico del inventario y PDF de inventario de varias páginas.
- 🔧 **Siguiente mejora:** procesar en orden los mensajes de un mismo chat. Al preparar este caso encontré que dos mensajes muy seguidos pueden leer el mismo estado.

## Mi rol

Me encargué del proyecto completo: levanté las necesidades del taller, diseñé la arquitectura y la hoja de cálculo, desarrollé el bot con apoyo de asistentes de IA, escribí las pruebas, lo desplegué en n8n y lo validé en vivo con el dueño. También documenté la operación y el modelo para llevarlo a otros talleres.

---

> 🔒 **Repositorio vitrina:** el código fuente, las claves, las URLs internas y los identificadores de la hoja no se publican. Puedo hacer una demostración en una entrevista.
>
> Las capturas son del bot real conectado a una réplica local del workflow, con clientes y datos de prueba.

**Bryan Jeremy Espinoza Herrera** · BJOTA Systems · Trujillo, Perú
