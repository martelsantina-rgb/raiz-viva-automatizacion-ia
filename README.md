# 🌱 Raíz Viva – Ecosistema de Automatización IA

**Entrega Final · Diplomatura AI Automation (Coderhouse)**

Sistema autónomo de atención al cliente para **Raíz Viva**, un vivero online **ficticio** (todos los datos son inventados). Las consultas que llegan por un formulario web se guardan en Notion. n8n las detecta, OpenAI las clasifica y redacta una respuesta, las consultas críticas esperan aprobación humana por Gmail y todo queda registrado en un dashboard.

| Capa | Tecnología |
|---|---|
| Orquestador | **n8n Cloud** |
| Base de datos / memoria | **Notion** (Clientes ↔ Consultas ↔ Log de errores) |
| Procesamiento IA | **OpenAI gpt-4o-mini** (prompt estructurado, salida JSON) |
| Canal de salida | **Gmail** (respuesta al cliente + aprobación HITL + alertas) |

## 📦 Entregables

| Entregable | Ubicación |
|---|---|
| Diagrama de arquitectura + documentación técnica (PDF) | [`Arquitectura-RaizViva.pdf`](./Arquitectura-RaizViva.pdf) |
| Lógica del flujo principal (n8n) | [`raiz-viva-flujo-principal.json`](./raiz-viva-flujo-principal.json) |
| Flujo de manejo global de errores (n8n) | [`raiz-viva-manejador-errores.json`](./raiz-viva-manejador-errores.json) |
| Base de datos en Notion (solo lectura) + Dashboard | **🔗 [Dashboard y base de datos en Notion](https://crawling-cayenne-74e.notion.site/Ra-z-Viva-Ecosistema-de-Automatizaci-n-IA-412b59fd803281f19641eac533ebc36d)** |
| Capturas de evidencia (incluye `07-dashboard-consultas-por-estado.png`) | archivos `01-…` a `07-….png` (en la raíz del repositorio) |

## 🔄 Cómo funciona

1. **Trigger:** se crea una página nueva en la base *Consultas* (desde el formulario web).
2. **Configuración + normalización:** el flujo valida los datos, aplica el filtro anti-bucle y **anonimiza** emails y teléfonos antes de mandar el texto a la IA.
3. **Memoria relacional:** busca al cliente por email; si no existe, lo crea y lo vincula a la consulta.
4. **IA:** gpt-4o-mini devuelve un JSON con la categoría, la prioridad, el sentimiento, un resumen y la respuesta.
5. **Router:**
   - Spam → `Descartado (spam)`
   - Pedido mayorista / reclamo / sentimiento negativo / prioridad alta → **Human-in-the-loop**: email con botones *Aprobar y enviar* / *Rechazar*. El flujo espera la decisión.
   - Resto → respuesta automática.
6. **Salida:** se le envía el email al cliente y en Notion la consulta queda como `Enviado`, con su Thread ID y la fecha.
7. **Resiliencia:** cada falla (datos faltantes, API de IA, JSON inválido, Gmail) se registra en el *Log de errores*, la consulta pasa a `Error` y se alerta al responsable. Las fallas no previstas las atiende el flujo *Manejador global de errores*.

Ciclo de estados: `Pendiente → Procesado por IA → Esperando aprobación → Aprobado por Humano → Enviado` (más `Error`, `Rechazado por Humano` y `Descartado (spam)`).

## ✅ Check de seguridad

- **Bucles infinitos:** el trigger solo toma páginas nuevas, el IF *¿Procesable?* filtra por estado y el contador *Intentos* corta en 3.
- **Tipos de datos:** todos los IF y Switch usan validación estricta (booleano contra booleano, número contra número).
- **Prompt dinámico:** se arma con las variables de *Configuración* (negocio, modelo, max_tokens, temperatura) y con los datos de cada consulta.

## 🧪 Test de estrés

| # | Caso | Resultado | Evidencia |
|---|---|---|---|
| 1 | Camino feliz: consulta minorista | Enviado automáticamente | `01-camino-feliz-*.png` |
| 2 | Pedido mayorista → aprobado | Aprobado por Humano → Enviado | `02-hitl-*.png` |
| 3 | Reclamo → rechazado | Rechazado por Humano | `03-reclamo-*.png` |
| 4 | Camino infeliz: datos incompletos | Error + Log + alerta | `04-datos-faltantes-*.png` |
| 5 | Camino infeliz: falla de la API (modelo inexistente) | Error + Log + alerta | `05-falla-api-*.png` |
| 6 | Spam | Descartado (spam) | `06-spam-*.png` |

## ⚙️ Cómo importar y configurar

1. En n8n: **Workflows → Import from File**, y elegir `raiz-viva-flujo-principal.json` y `raiz-viva-manejador-errores.json`.
2. Crear las credenciales (no están incluidas en los archivos):
   - **Notion API:** crear una integración interna en notion.so/profile/integrations y darle acceso a la página *Raíz Viva – Ecosistema de Automatización IA*.
   - **OpenAI API:** API key de platform.openai.com.
   - **Gmail OAuth2:** iniciar sesión con Google desde n8n.
3. En el nodo **Configuración**, reemplazar `email_responsable` por el email que recibe las aprobaciones.
4. En *Settings* del flujo principal → **Error Workflow** = "Raíz Viva · Manejador global de errores".
5. Activar el flujo (**Active**).

> 🔐 Las API keys no se publican: se guardan solo en el gestor de credenciales de n8n y se ocultan en las capturas.
