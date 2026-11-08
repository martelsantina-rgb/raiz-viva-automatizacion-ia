# Raíz Viva - Entrega final AI Automation

Proyecto final de la diplomatura de AI Automation de Coderhouse.

Elegí como caso un vivero online inventado, Raíz Viva, que recibe consultas de clientes por un formulario web. La idea es que las consultas se clasifiquen y se respondan solas con IA, pero que las más delicadas (pedidos mayoristas y reclamos) esperen mi aprobación antes de que le llegue algo al cliente. Todos los datos son ficticios.

## Herramientas que usé

- n8n Cloud como orquestador
- Notion como base de datos (tablas Clientes, Consultas y Log de errores, relacionadas entre sí)
- OpenAI con el modelo gpt-4o-mini para clasificar y redactar las respuestas
- Gmail para responder a los clientes, pedir la aprobación y avisar cuando hay errores

## Cómo funciona

1. Cuando entra una consulta nueva en Notion, n8n la detecta.
2. Se revisa que tenga un email válido y un mensaje. Si falta algo, no se manda a la IA y se registra como error.
3. Antes de mandar el texto a la IA se reemplazan los emails y teléfonos que haya en el mensaje, para no compartir datos personales.
4. Se busca el cliente por email y, si no existe, se crea.
5. La IA devuelve la categoría, la prioridad, el sentimiento, un resumen y una respuesta propuesta. Todo se guarda en Notion.
6. Según la categoría:
   - si es spam, se descarta;
   - si es un pedido mayorista, un reclamo, tiene sentimiento negativo o prioridad alta, me llega un mail para aprobar o rechazar la respuesta;
   - si es una consulta común, se responde automáticamente.
7. Cuando se envía la respuesta, la consulta queda como "Enviado" en Notion.

Si algo falla (datos incompletos, la API de OpenAI, la respuesta de la IA o el envío del mail), queda registrado en la tabla Log de errores, la consulta pasa a "Error" y me llega un aviso por mail. Además hay un segundo flujo que avisa si se corta el flujo principal por algún error no previsto.

## Archivos

- `Arquitectura-RaizViva.pdf`: diagrama de arquitectura y documentación (estructura de datos, costos, seguridad y dashboard)
- `raiz-viva-flujo-principal.json`: flujo principal de n8n
- `raiz-viva-manejador-errores.json`: flujo de manejo de errores
- Capturas de las pruebas: archivos que empiezan con `01-` a `07-`

Base de datos y dashboard en Notion (solo lectura):
https://crawling-cayenne-74e.notion.site/Ra-z-Viva-Ecosistema-de-Automatizaci-n-IA-412b59fd803281f19641eac533ebc36d

## Pruebas que hice

1. Consulta común (potus): se respondió sola. Capturas `01-`.
2. Pedido mayorista: me llegó el mail de aprobación, lo aprobé y se envió la respuesta. Capturas `02-`.
3. Reclamo: lo rechacé y no se le envió nada al cliente. Capturas `03-`.
4. Datos incompletos (email mal escrito y mensaje muy corto): fue por la ruta de error. Capturas `04-`.
5. Falla de la API: puse un modelo que no existe para forzar el error. Hizo 3 reintentos y después fue por la ruta de error. Capturas `05-`.
6. Spam: quedó descartado. Capturas `06-`.

El dashboard con el estado de todas las consultas está en las capturas `07-`.

Las API keys no están en los archivos. Quedan guardadas en las credenciales de n8n.
