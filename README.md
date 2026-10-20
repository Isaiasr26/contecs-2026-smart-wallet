# Smart Wallet: tu primer agente de IA con Tool Calling

**Taller "Más allá del Chatbot: Construyendo Agentes con Tool Calling y Automatización de Procesos"**
CONTECS 2026 · Facultad de Ingeniería de Sistemas Computacionales (FISC) · Universidad Tecnológica de Panamá
Martes 20 de octubre de 2026 · Ponente: Isaías Reyes ([github.com/Isaiasr26](https://github.com/Isaiasr26))

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Isaiasr26/contecs-2026-smart-wallet/blob/main/Smart_Wallet_Estudiantes.ipynb)

Un chatbot **habla**. Un agente **actúa**. En este taller construyes, desde cero y en Google Colab, un agente de finanzas personales que registra tus gastos en una base de datos real, consulta precios de la bolsa y de criptomonedas, calcula sin equivocarse y te responde por **Telegram, desde tu celular**.

No necesitas experiencia previa con inteligencia artificial: basta con conocimientos básicos de Python.

> **La idea central del taller:** el LLM decide, el código ejecuta.

---

## Qué vas a aprender

| Parte | Tema | Lo que construyes |
|:-:|---|---|
| 1 | Setup: Colab, API Key y Secrets | Tu primera llamada a Gemini, con la clave guardada de forma segura |
| 2 | Por qué un LLM solo no basta | Dos experimentos: un saldo que el modelo no conoce y un interés compuesto que no calcula bien |
| 3 | Tu primera herramienta: Tool Calling | `consultar_saldo_cuenta` y la "radiografía" de lo que pasa por debajo |
| 4 | Memoria persistente con SQLite | Una billetera real con `registrar_movimiento`, `consultar_billetera` y `eliminar_movimiento` |
| 5 | Conexión al mundo real | Precios de Yahoo Finance con plan de respaldo y un simulador de inversión |
| 6 | Contratos con Pydantic | Convertir un texto del Super 99 en datos listos para la base de datos, y tu reto: `calcular_itbms_panama` |
| 7 | Producción en Telegram | Tu agente funcionando en tu celular |

## Arquitectura

```
   TU CELULAR            TU CÓDIGO (el notebook)                     SERVICIOS EXTERNOS
  ┌──────────┐      ┌───────────────────────────────────┐
  │ Telegram │ ---> │ Bot (pyTelegramBotAPI)            │
  │          │ <--- │   |                               │   "¿qué hago?"    ┌──────────────────┐
  └──────────┘      │   v                               │ ----------------> │ Gemini 3.5 Flash │
                    │ Agente (Automatic Function        │ <---------------- │   (el cerebro)   │
                    │         Calling)                  │   function_call   └──────────────────┘
                    │   |                               │
                    │   v                               │
                    │ Herramientas Python (las manos)   │
                    │   ├── SQLite ........ tu billetera │
                    │   ├── yfinance ...... precios ----┼-----------------> Yahoo Finance
                    │   ├── simulador ..... cálculo exacto
                    │   └── Pydantic ...... contratos de datos
                    └───────────────────────────────────┘
```

| Pieza | Tecnología | Para qué sirve |
|---|---|---|
| El cerebro | Gemini 3.5 Flash, con la librería `google-genai` | Entiende lo que pide el usuario y decide qué herramienta usar |
| Las manos | Funciones de Python con *Automatic Function Calling* | Hacen el trabajo exacto: consultar, guardar y calcular |
| La memoria | El historial del chat y SQLite | Recordar la conversación (corto plazo) y los movimientos (largo plazo) |
| Las ventanas | `yfinance` | Traer precios reales de acciones y criptomonedas |
| Los formularios | Pydantic con `response_schema` | Garantizar que los datos lleguen con la forma correcta |
| La interfaz | Telegram, con `pyTelegramBotAPI` | Usar el agente desde el celular |

## Requisitos

- Una **cuenta personal de Gmail**. Las cuentas institucionales pueden tener el acceso bloqueado, y Google exige ser mayor de 18 años para usar la API de Gemini.
- Una **API Key gratuita** de [Google AI Studio](https://aistudio.google.com/apikey). No pide tarjeta.
- **Telegram** instalado en tu celular, para la Parte 7.
- Conocimientos básicos de Python: variables, funciones, listas y diccionarios.

## Inicio rápido

1. Haz clic en **Abrir en Colab** (al inicio de esta página) y luego en **Archivo > Guardar una copia en Drive**.
2. Crea tu API Key en [aistudio.google.com/apikey](https://aistudio.google.com/apikey).
3. En Colab, abre el panel **Secrets** (el ícono de la llave, en la barra izquierda), crea el secreto `GOOGLE_API_KEY`, pega tu clave y activa **Notebook access**.
4. Ejecuta las celdas en orden con `Shift + Enter`.
5. Para la Parte 7, crea tu bot con [@BotFather](https://t.me/BotFather) y guarda su token como el secreto `TELEGRAM_TOKEN`.

El notebook instala todo lo necesario en su primera celda:

```
%pip install -q google-genai pyTelegramBotAPI yfinance pydantic pandas
```

> Si te pierdes o Colab se reinicia, haz clic en la celda donde va la clase y usa **Entorno de ejecución > Ejecutar anteriores** (`Ctrl + F8`).

## Agenda (120 minutos)

| Hora | Bloque |
|:-:|---|
| 3:15 | Bienvenida |
| 3:19 | Teoría: de la era del chatbot a la era de los agentes, qué es un LLM, el agente por dentro, tool calling y automatización |
| 3:48 | Demostración del resultado final |
| 3:51 | Parte 1 · Setup: Colab, API Key y Secrets |
| 4:01 | Parte 2 · Por qué un LLM solo no basta |
| 4:08 | Parte 3 · Tu primera herramienta: Tool Calling |
| 4:21 | Parte 4 · Memoria persistente con SQLite |
| 4:34 | Parte 5 · Conexión al mundo real: Yahoo Finance |
| 4:45 | Parte 6 · Contratos con Pydantic y reto ITBMS |
| 4:57 | Parte 7 · Producción: tu agente en Telegram |
| 5:10 | Cierre y próximos pasos |

## Las herramientas del agente

Cada herramienta es una función normal de Python. Gemini no ve su código: solo lee su **nombre**, sus **type hints** y su **docstring**, y con eso decide cuándo pedir que se ejecute.

| Herramienta | Qué hace | Parte |
|---|---|:-:|
| `consultar_saldo_cuenta(cuenta)` | Devuelve el saldo de una billetera de prueba: efectivo, banco o Yappy | 3 |
| `registrar_movimiento(tipo, categoria, monto, descripcion)` | Guarda un ingreso o un gasto en SQLite | 4 |
| `consultar_billetera()` | Devuelve ingresos, gastos, saldo, gastos por categoría y los últimos movimientos | 4 |
| `eliminar_movimiento(id_movimiento)` | Borra un movimiento por su id | 4 |
| `obtener_precio_activo(ticker)` | Consulta el precio de una acción o criptomoneda en Yahoo Finance, con precios de respaldo si no hay conexión | 5 |
| `simular_inversion(monto_inicial, aporte_mensual, tasa_anual_pct, anios)` | Calcula el interés compuesto con la fórmula exacta | 5 |
| `calcular_itbms_panama(subtotal, porcentaje)` | Calcula el ITBMS y el total de una compra (el reto del taller) | 6 |

**Tasas del ITBMS en Panamá:** 7 % general (incluida la comida en restaurantes), 10 % para bebidas alcohólicas y hospedaje, y 15 % para productos de tabaco.

## Comandos del bot de Telegram

| Mensaje | Qué hace | ¿Usa el LLM? |
|---|---|:-:|
| `/start` o `/ayuda` | Muestra la bienvenida y ejemplos de mensajes | No |
| `/saldo` | Muestra el saldo, los ingresos y los gastos con una consulta directa a SQLite | No |
| `/reset` | Reinicia la conversación; los movimientos siguen guardados en la base de datos | No |
| Cualquier otro texto | Por ejemplo: "Gasté 2 en el Metro y 6.50 en el almuerzo de la fonda" | Sí |

## Seguridad y privacidad

- **Nunca escribas tu API Key ni tu token de Telegram en el código.** Guárdalos en Colab Secrets.
- Todas las consultas SQL usan **placeholders** (`?`) para evitar la inyección SQL.
- Cada herramienta **valida** los datos que recibe del modelo; por ejemplo, rechaza montos negativos.
- En la **capa gratuita** de Gemini, Google puede usar tus prompts para mejorar sus productos. Practica con datos de ejemplo y no escribas números de cuenta ni contraseñas reales.
- El token de Telegram es la contraseña de tu bot. Si se filtra, envía `/revoke` a @BotFather para generar uno nuevo.
- **Limitación conocida:** el bot tiene una sola billetera para todas las personas que le escriben. Para un bot personal funciona bien; para atender a varias personas por separado hay que agregar un `user_id` a la tabla y filtrar por él en cada herramienta. Es el primer reto propuesto después del taller.

## Problemas frecuentes

| Síntoma o error | Solución |
|---|---|
| `SecretNotFoundError` | Crea el secreto con el nombre exacto: `GOOGLE_API_KEY` o `TELEGRAM_TOKEN` |
| `NotebookAccessError` | Activa *Notebook access* en el panel Secrets y vuelve a ejecutar la celda |
| `400 API key not valid` | Copia de nuevo la clave desde AI Studio, sin espacios |
| `403 PERMISSION_DENIED` | Usa una cuenta personal de Gmail |
| `404 NOT_FOUND` (modelo) | Cambia `MODELO` a `"gemini-3.5-flash-lite"` |
| `429 RESOURCE_EXHAUSTED` | Agotaste la cuota: espera 60 segundos |
| `NameError` | Te saltaste una celda: usa **Ejecutar anteriores** (`Ctrl + F8`) |
| Precio con "RESPALDO OFFLINE" | Yahoo Finance limitó la red; el agente sigue funcionando con precios de referencia |
| Telegram `401 Unauthorized` | Copia de nuevo el token desde @BotFather |
| Telegram `409 Conflict` | El bot está corriendo en dos lugares a la vez: deja una sola celda corriendo |

El notebook incluye una tabla más completa en su **Anexo**.

## Notas técnicas

- **Modelo.** El taller usa `gemini-3.5-flash`. Desde septiembre de 2026, Google limita `gemini-2.5-flash` a los proyectos que ya lo usaban, así que una API Key nueva no tiene acceso. Si `gemini-3.5-flash` no está disponible para tu cuenta, usa `gemini-3.5-flash-lite`.
- **Tipos simples en las herramientas.** Usa `str`, `int`, `float` y `bool` en los parámetros y escribe las opciones válidas en el docstring. Con la versión actual de `google-genai`, el modo automático falla si un parámetro se anota con `Literal` o con `Enum`.
- **Solo en Colab.** El notebook lee las claves con `google.colab.userdata`. Para correrlo en Jupyter local, reemplaza esas líneas por variables de entorno, por ejemplo `os.environ["GOOGLE_API_KEY"]`.
- **Datos temporales.** El archivo `smart_wallet.db` vive en la sesión de Colab y se borra cuando el entorno se reinicia.

## Recursos

- [Gemini API: function calling](https://ai.google.dev/gemini-api/docs/function-calling)
- [Gemini API: structured output](https://ai.google.dev/gemini-api/docs/structured-output)
- [Gemini API: modelos y deprecaciones](https://ai.google.dev/gemini-api/docs/deprecations)
- [pyTelegramBotAPI](https://pytba.readthedocs.io)
- [yfinance](https://ranaroussi.github.io/yfinance)
- [Pydantic](https://docs.pydantic.dev)

## Aviso

Proyecto **educativo**. Las respuestas del agente, los precios y las simulaciones de inversión **no constituyen asesoría financiera**. Los precios de las acciones pueden mostrar el cierre del día anterior o tener retraso.
