# Práctica 3.5-P1 · Noticias climáticas del día

| Campo | Detalle |
|---|---|
| Asignatura | Desarrollo Sustentable (ACD-0908) |
| Carrera | Ingeniería en Sistemas Computacionales |
| Unidad | 3. Escenario socio-cultural |
| Tema | 3.5 Difusión de la conciencia ambiental mediante automatización de la información |
| Herramientas | IA generativa, Make (Webhooks, RSS, Text aggregator, Make AI Agent, Webhook Response), GitHub Pages |

## Descripción

Página web, sin programar el flujo de automatización, donde el usuario elige un tema ambiental. Make busca las cinco noticias más recientes en Google News mediante RSS, un agente de IA las resume y la página muestra el resultado con los enlaces a las fuentes originales.

**Página publicada:** [PEGAR AQUÍ EL ENLACE DE GITHUB PAGES]

## Video de demostración

[Ver el video en YouTube](https://youtube.com/shorts/gcXIWOFMPpk?feature=share)

## Flujo

```
Página web → Make (Webhooks → RSS → Text aggregator → Make AI Agent → Webhook Response) → Página web
```

| # | Módulo | Función |
|---|---|---|
| 1 | Webhooks · Custom webhook | Recibe el campo `tema` enviado por la página (POST, `URLSearchParams`). |
| 2 | RSS · Retrieve RSS feed items | Consulta Google News con `encodeURL(1.tema)` y devuelve máximo 5 noticias. |
| 3 | Tools · Text aggregator | Junta título, fecha y enlace de las noticias en un solo texto, para que la IA se ejecute una sola vez. |
| 4 | Make AI Agent | Resume las noticias usando únicamente la información recibida (conexión Make's AI Provider). |
| 5 | Webhooks · Webhook Response | Devuelve el resumen en texto plano (`text/plain; charset=utf-8`) con `Access-Control-Allow-Origin: *`. |

El escenario se activa con **Immediately as data arrives**.

## Temas de la página

| Bloque | Tema |
|---|---|
| A. Cambio climático | Cambio climático |
| B. Impacto humano | Deforestación en México |
| C. Consumo | Plásticos de un solo uso |
| D. Energía y soluciones | Energías renovables en México |
| E. Local | Sequía en Sinaloa |
| E. Local | Mazatlán y medio ambiente |

## Archivos del repositorio

| Archivo | Contenido |
|---|---|
| [`index.html`](index.html) | Página web con el selector de temas y la conexión al webhook de Make. |
| [`Noticias_Climaticas_Equipo_blueprint.json`](Noticias_Climaticas_Equipo_blueprint.json) | Blueprint del escenario de Make (también incluido abajo). |
| [`Reporte_Practica_3_5.pdf`](Reporte_Practica_3_5.pdf) | Tabla de verificación de fidelidad y respuestas a las preguntas de reflexión. |

## Reporte

La tabla de fidelidad de los resúmenes y las respuestas a las cinco preguntas de reflexión están en el archivo [Reporte_Practica_3_5.pdf](Reporte_Practica_3_5.pdf).

## Blueprint del escenario (JSON)

<details>
<summary>Ver código JSON completo</summary>

```json
{
  "name": "Noticias_Climaticas_Equipo#",
  "flow": [
    {
      "id": 1,
      "module": "gateway:CustomWebHook",
      "version": 1,
      "parameters": {
        "hook": 3008237,
        "maxResults": 1
      },
      "mapper": {},
      "metadata": {
        "designer": {
          "x": 0,
          "y": 0
        },
        "restore": {
          "parameters": {
            "hook": {
              "data": {
                "editable": "true"
              },
              "label": "Noticias_Climaticas_Equipo# Webhook"
            }
          }
        },
        "parameters": [
          {
            "name": "hook",
            "type": "hook:gateway-webhook",
            "label": "Webhook",
            "required": true
          },
          {
            "name": "maxResults",
            "type": "number",
            "label": "Maximum number of results"
          }
        ]
      }
    },
    {
      "id": 2,
      "module": "rss:ActionReadArticles",
      "version": 4,
      "parameters": {
        "include": []
      },
      "mapper": {
        "url": "https://news.google.com/rss/search?q={{encodeURL(1.tema)}}&hl=es-419&gl=MX&ceid=MX:es-419",
        "gzip": true,
        "maxResults": 5
      },
      "metadata": {
        "designer": {
          "x": 300,
          "y": 0
        },
        "restore": {
          "parameters": {
            "include": {
              "label": "Empty"
            }
          }
        },
        "parameters": [
          {
            "name": "include",
            "type": "select",
            "label": "Process RSS fields",
            "multiple": true
          }
        ],
        "expect": [
          {
            "name": "url",
            "type": "url",
            "label": "URL",
            "required": true
          },
          {
            "name": "username",
            "type": "text",
            "label": "User name"
          },
          {
            "name": "password",
            "type": "password",
            "label": "Password"
          },
          {
            "name": "filterDateFrom",
            "type": "date",
            "label": "Date from"
          },
          {
            "name": "filterDateTo",
            "type": "date",
            "label": "Date to"
          },
          {
            "name": "maxResults",
            "type": "number",
            "label": "Maximum number of returned items",
            "required": true
          },
          {
            "name": "gzip",
            "type": "boolean",
            "label": "Request compressed content",
            "required": true
          }
        ]
      }
    },
    {
      "id": 3,
      "module": "util:TextAggregator",
      "version": 1,
      "parameters": {
        "feeder": 2,
        "rowSeparator": "\n"
      },
      "mapper": {
        "value": "Título: {{2.title}}\nFecha: {{2.dateCreated}}\nEnlace: {{2.url}}"
      },
      "metadata": {
        "designer": {
          "x": 600,
          "y": 0
        },
        "restore": {
          "extra": {
            "feeder": {
              "label": "Retrieve RSS feed items [2]"
            }
          },
          "parameters": {
            "rowSeparator": {
              "label": "New row"
            }
          }
        },
        "parameters": [
          {
            "name": "rowSeparator",
            "type": "select",
            "label": "Row separator"
          }
        ],
        "expect": [
          {
            "name": "value",
            "type": "text",
            "label": "Text",
            "multiline": true
          }
        ]
      }
    },
    {
      "id": 4,
      "module": "ai-local-agent:RunLocalAIAgent",
      "version": 0,
      "tools": [],
      "parameters": {
        "makeConnectionId": 10574833
      },
      "mapper": {
        "files": [],
        "message": "Tema: {{1.tema}}\nNoticias:\n{{3.text}}",
        "threadId": "",
        "outputType": "text",
        "tokenLimit": "50",
        "modelConfig": {
          "timeout": "",
          "recursionLimit": "300",
          "aiCompactionEnabled": true,
          "iterationsFromHistoryCount": "10"
        },
        "defaultModel": "medium",
        "systemPrompt": "Eres un divulgador ambiental para alumnos de Desarrollo Sustentable del TecNM.\nRecibirás un tema y una lista de noticias recientes (título, fecha y enlace).\nUsa ÚNICAMENTE la información de esas noticias; no inventes datos ni agregues noticias que no estén en la lista.\nResponde en texto plano (sin asteriscos, sin markdown), en español, con este formato:\n\nRESUMEN DEL DÍA\n[3 a 4 líneas explicando de qué tratan las noticias en conjunto]\n\nNOTICIAS\n1. [título] - [1 línea explicando por qué importa]\n[enlace]\n(repite para cada noticia)\n\n¿QUÉ PUEDO HACER YO?\n[1 acción concreta que un estudiante puede hacer relacionada con el tema]\n\nSi la lista de noticias viene vacía, responde: \"No se encontraron noticias recientes sobre este tema.\"",
        "promptCaching": "none",
        "fallbackEnabled": false
      },
      "metadata": {
        "designer": {
          "x": 900,
          "y": 0
        },
        "restore": {
          "expect": {
            "files": {
              "mode": "chose"
            },
            "outputType": {
              "label": "Text"
            },
            "defaultModel": {
              "mode": "chose",
              "label": "MediumModel: gpt-5-nano. Reasoning: low. A balanced, quick choice for clearly defined tasks needing reliable answers rather than deep analysis"
            },
            "promptCaching": {
              "label": "Automatic (always on)"
            }
          },
          "parameters": {
            "makeConnectionId": {
              "data": {
                "scoped": "true",
                "connection": "ai-provider"
              },
              "label": "Make's AI Provider (default)"
            }
          }
        },
        "parameters": [
          {
            "name": "makeConnectionId",
            "type": "account:ai-provider,openai-gpt-3,anthropic-claude,gemini-ai-q9zyjp,ai-agent-foundry-openai,ai-agent-foundry-non-openai,mistral-ai,cohere,groq,ai-agent-xai,amazon-bedrock,ai-agent-openai-compatible",
            "label": "Connection",
            "required": true
          }
        ],
        "expect": [
          {
            "name": "defaultModel",
            "type": "select",
            "label": "Model",
            "required": true
          },
          {
            "name": "tokenLimit",
            "type": "number",
            "label": "Maximum output length (%)",
            "validate": {
              "max": 100,
              "min": 20
            }
          },
          {
            "name": "promptCaching",
            "type": "select",
            "label": "Prompt caching",
            "validate": {
              "enum": [
                "none"
              ]
            }
          },
          {
            "name": "fallbackEnabled",
            "type": "boolean",
            "label": "Enable fallback connection",
            "required": true
          },
          {
            "name": "systemPrompt",
            "type": "text",
            "label": "Instructions"
          },
          {
            "name": "message",
            "type": "text",
            "label": "Input",
            "required": true
          },
          {
            "name": "files",
            "type": "array",
            "label": "Input files",
            "spec": [
              {
                "name": "fileName",
                "type": "filename",
                "label": "File name",
                "semantic": "file:name"
              },
              {
                "name": "data",
                "type": "buffer",
                "label": "Data",
                "semantic": "file:data"
              }
            ]
          },
          {
            "name": "threadId",
            "type": "text",
            "label": "Conversation ID",
            "validate": {
              "max": 256
            }
          },
          {
            "name": "modelConfig",
            "type": "collection",
            "label": "Model configuration",
            "spec": [
              {
                "name": "recursionLimit",
                "type": "number",
                "label": "Steps per agent call"
              },
              {
                "name": "iterationsFromHistoryCount",
                "type": "number",
                "label": "Maximum conversation history"
              },
              {
                "name": "timeout",
                "type": "number",
                "label": "Step timeout",
                "validate": {
                  "max": 600,
                  "min": 120
                }
              },
              {
                "name": "aiCompactionEnabled",
                "type": "boolean",
                "label": "Enable compaction",
                "required": true
              }
            ]
          },
          {
            "name": "outputType",
            "type": "select",
            "label": "Response format",
            "required": true,
            "validate": {
              "enum": [
                "text",
                "make-schema",
                "udt-schema"
              ]
            }
          }
        ]
      }
    },
    {
      "id": 5,
      "module": "gateway:WebhookRespond",
      "version": 1,
      "parameters": {},
      "mapper": {
        "body": "{{4.response}}",
        "status": 200,
        "headers": [
          {
            "key": "Content-Type",
            "value": "text/plain; charset=utf-8"
          },
          {
            "key": "Access-Control-Allow-Origin",
            "value": "*"
          }
        ]
      },
      "metadata": {
        "designer": {
          "x": 1200,
          "y": 0
        },
        "restore": {
          "expect": {
            "headers": {
              "mode": "chose",
              "items": [
                null,
                null
              ]
            }
          }
        }
      }
    }
  ],
  "metadata": {
    "instant": true,
    "version": 1,
    "designer": {
      "orphans": []
    },
    "scenario": {
      "dlq": false,
      "slots": null,
      "dataloss": false,
      "maxErrors": 3,
      "autoCommit": true,
      "roundtrips": 1,
      "sequential": false,
      "confidential": false,
      "freshVariables": false,
      "autoCommitTriggerLast": true
    }
  },
  "scheduling": {
    "type": "immediately",
    "maximum_runs_per_minute": 100
  },
  "interface": {
    "input": [],
    "output": []
  }
}
```

</details>

### Cómo importarlo en Make

1. En Make, crea un escenario nuevo.
2. Abre el menú de los tres puntos de la parte inferior y elige **Import Blueprint**.
3. Selecciona el archivo `Noticias_Climaticas_Equipo_blueprint.json`.
4. Crea un webhook nuevo en el primer módulo y selecciona la conexión **Make's AI Provider** en el módulo del agente. Los números de webhook y de conexión incluidos en el archivo corresponden a la cuenta original.
5. Guarda, activa el escenario con **Immediately as data arrives** y pega la URL del webhook en la constante `WEBHOOK_URL` de `index.html`.
