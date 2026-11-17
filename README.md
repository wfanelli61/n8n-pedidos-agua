# Pedidos de agua con tasa BCV — flujo en n8n

Automatización para una embotelladora de agua: el cliente hace su pedido en un formulario, el flujo consulta la **tasa oficial del BCV por API**, calcula el total en dólares y en bolívares, y avisa al negocio por **Telegram**.

```
Formulario de pedido ─► Configuración ─► Tasa BCV (DolarAPI) ──ok──► Calcular total ─► Aviso por Telegram
                                               │ falla (3 intentos)          ▲
                                               ▼                             │
                                     Tasa de respaldo (ER-API) ──ok──────────┘
                                               │ falla (3 intentos)
                                               ▼
                                     Aviso de error por Telegram
```

## Qué conecta

| Plataforma | Para qué |
|---|---|
| Formulario de n8n | El cliente deja nombre, teléfono, garrafones y dirección |
| [DolarAPI](https://ve.dolarapi.com) | Tasa oficial del BCV (fuente principal) |
| [ER-API](https://open.er-api.com) | Tasa USD→VES de respaldo |
| Telegram Bot API | Aviso del pedido con el total calculado |

## Qué pasa cuando algo falla

- Cada consulta de tasa se **reintenta 3 veces**, con 2 segundos entre intentos.
- Si DolarAPI sigue sin responder, el flujo usa **ER-API** y el mensaje lo indica ("se usó la tasa de respaldo").
- Si **las dos** fallan, el pedido no se pierde: llega por Telegram como "Pedido SIN total" con los datos del cliente, para calcularlo a mano.

## Cómo usarlo

1. Levanta n8n con Docker:
   ```bash
   docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n:stable
   ```
2. Importa `flujo-pedidos-agua.json` (menú ⋯ → *Import from file*).
3. Crea un bot con @BotFather y añade su token como credencial de Telegram en los dos nodos de Telegram.
4. En el nodo **Configuración**, pon tu `chatId` y el precio del garrafón (`precioGarrafonUSD`).
5. Activa el flujo y abre la URL del formulario.

Hecho por **Walter Fanelli** · [CV interactivo](https://claude.ai/artifact/UkB6K6txZx1Y8sgyPHD3N4) · [GitHub](https://github.com/wfanelli61)
