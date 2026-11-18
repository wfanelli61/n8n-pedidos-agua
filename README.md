# Pedidos de agua con tasa BCV — flujo en n8n

Automatización para una embotelladora de agua: el cliente hace su pedido en un formulario, el flujo consulta la **tasa oficial del BCV por API**, calcula el total en dólares y en bolívares, y abre **WhatsApp** con el pedido ya escrito y dirigido al número del negocio. El cliente solo pulsa *Enviar*.

```
Formulario de pedido ─► Configuración ─► Tasa BCV (DolarAPI) ──ok──► Calcular total ──► Abrir WhatsApp con el pedido
                                               │ falla (3 intentos)          ▲                       ▲
                                               ▼                             │                       │
                                     Tasa de respaldo (ER-API) ──ok──────────┘                       │
                                               │ falla (3 intentos)                                  │
                                               ▼                                                     │
                                         Pedido sin tasa ────────────────────────────────────────────┘
```

## Qué conecta

| Plataforma | Para qué |
|---|---|
| Formulario de n8n | El cliente deja nombre, teléfono, garrafones y dirección |
| [DolarAPI](https://ve.dolarapi.com) | Tasa oficial del BCV (fuente principal) |
| [ER-API](https://open.er-api.com) | Tasa USD→VES de respaldo |
| WhatsApp (enlace `wa.me`) | Abre el chat del negocio con el pedido y el total ya escritos |

## Qué pasa cuando algo falla

- Cada consulta de tasa se **reintenta 3 veces**, con 2 segundos entre intentos.
- Si DolarAPI sigue sin responder, el flujo usa **ER-API** y el mensaje lo indica ("tasa de respaldo").
- Si **las dos** fallan, el pedido no se pierde: WhatsApp se abre igual con los datos del cliente y "Total: por confirmar".

## Por qué un enlace de WhatsApp y no una API

Es gratis, no necesita cuenta de desarrollador en Meta y no hay riesgo de que bloqueen el número del negocio. El pedido le llega al dueño como un mensaje normal del cliente, en el WhatsApp que ya usa.

## Cómo usarlo

1. Levanta n8n con Docker:
   ```bash
   docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n:stable
   ```
2. Importa `flujo-pedidos-agua.json` (menú ⋯ → *Import from file*).
3. En el nodo **Configuración**, pon el WhatsApp del negocio en `whatsappNegocio` (formato internacional sin `+`, por ejemplo `584121234567`) y el precio del garrafón en `precioGarrafonUSD`.
4. Publica el flujo y abre `http://localhost:5678/form/pedido-agua`.

Hecho por **Walter Fanelli** · [CV interactivo](https://claude.ai/artifact/UkB6K6txZx1Y8sgyPHD3N4) · [GitHub](https://github.com/wfanelli61)
