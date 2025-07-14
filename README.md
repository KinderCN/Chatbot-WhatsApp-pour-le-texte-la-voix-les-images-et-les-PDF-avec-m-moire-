# Chatbot-WhatsApp-pour-le-texte-la-voix-les-images-et-les-PDF-avec-m-moire-
n super-bot WhatsApp capable de gérer messages écrits 💬, notes vocales 🎙️ (Whisper), images/PDF 📄 (vision + OCR) et de retenir le contexte grâce à LangChain Memory. Le workflow détecte le type de média, extrait/convertit, passe le contenu à GPT-4o, répond de façon cohérente et met à jour la mémoire pour la suite.

{
  "name": "Chatbot WhatsApp 🤖📲 pour le texte, la voix, les images et les PDF avec mémoire 🧠",
  "nodes": [
    {
      "parameters": {
        "updates": [
          "messages"
        ],
        "options": {}
      },
      "id": "573a1e36-e5b0-430c-acab-8f0842a27e02",
      "name": "WhatsApp Trigger",
      "type": "n8n-nodes-base.whatsAppTrigger",
      "position": [
        880,
        1220
      ],
      "webhookId": "d3978cae-2aca-4553-8ac7-ab89068deabc",
      "typeVersion": 1,
      "credentials": {
        "whatsAppTriggerApi": {
          "id": "XYYdUijmsBWgOTRy",
          "name": "WhatsApp OAuth account"
        }
      }
    },
    {
      "parameters": {
        "url": "={{ $json.url }}",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "options": {}
      },
      "id": "ab02021b-3c4f-47b2-96e1-78d9b406f956",
      "name": "Download Image",
      "type": "n8n-nodes-base.httpRequest",
      "position": [
        2300,
        1260
      ],
      "typeVersion": 4.2,
      "credentials": {
        "httpHeaderAuth": {
          "id": "1cwknbfQbaanUzUk",
          "name": "Header Auth account"
        }
      }
    },
    {
      "parameters": {
        "url": "={{ $json.url }}",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "options": {}
      },
      "id": "be088363-3010-462c-a01d-06a10c271e33",
      "name": "Download Audio",
      "type": "n8n-nodes-base.httpRequest",
      "position": [
        2300,
        960
      ],
      "typeVersion": 4.2,
      "credentials": {
        "httpHeaderAuth": {
          "id": "1cwknbfQbaanUzUk",
          "name": "Header Auth account"
        }
      }
    },
    {
      "parameters": {
        "model": {
          "__rl": true,
          "mode": "list",
          "value": "gpt-4o-mini"
        },
        "options": {}
      },
      "id": "9396d069-f975-4094-bfc9-4d154d278c7e",
      "name": "OpenAI Chat Model",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "position": [
        2960,
        1800
      ],
      "typeVersion": 1.2,
      "credentials": {
        "openAiApi": {
          "id": "IleT53idqzgq2A7w",
          "name": "OpenAi account"
        }
      }
    },
    {
      "parameters": {
        "url": "={{ $json.url }}",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "options": {}
      },
      "id": "29df53de-11cd-4946-9008-b64f7e8a2cd2",
      "name": "Download File",
      "type": "n8n-nodes-base.httpRequest",
      "position": [
        2260,
        1540
      ],
      "typeVersion": 4.2,
      "credentials": {
        "httpHeaderAuth": {
          "id": "1cwknbfQbaanUzUk",
          "name": "Header Auth account"
        }
      }
    },
    {
      "parameters": {
        "sessionIdType": "customKey",
        "sessionKey": "=memory_{{ $('WhatsApp Trigger').item.json.contacts[0].wa_id }}",
        "contextWindowLength": 10
      },
      "id": "0463ebfd-9b18-4892-91c7-370c3baf5dbc",
      "name": "Simple Memory",
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "position": [
        3140,
        1800
      ],
      "typeVersion": 1.3
    },
    {
      "parameters": {
        "resource": "media",
        "operation": "mediaUrlGet",
        "mediaGetId": "={{ $('WhatsApp Trigger').item.json.messages[0].document.id }}"
      },
      "id": "f5a16d07-92f1-474c-bd81-e1cf98cdef96",
      "name": "Get File Url",
      "type": "n8n-nodes-base.whatsApp",
      "position": [
        2000,
        1540
      ],
      "webhookId": "280bd5de-32d7-4d8f-93d2-e91e3b0bc161",
      "typeVersion": 1,
      "credentials": {
        "whatsAppApi": {
          "id": "2jO1IVmC3SFBa7bC",
          "name": "WhatsApp account"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "version": 2,
            "leftValue": "",
            "caseSensitive": true,
            "typeValidation": "strict"
          },
          "combinator": "and",
          "conditions": [
            {
              "id": "f52d2aaa-e0b2-45e5-8c4b-ceef42182a0d",
              "operator": {
                "name": "filter.operator.equals",
                "type": "string",
                "operation": "equals"
              },
              "leftValue": "={{ $json.messages[0].document.mime_type }}",
              "rightValue": "application/pdf"
            }
          ]
        },
        "options": {}
      },
      "id": "dfb8ae90-2991-4f86-8164-544169d29a95",
      "name": "Only PDF File",
      "type": "n8n-nodes-base.if",
      "position": [
        1660,
        1500
      ],
      "typeVersion": 2.2
    },
    {
      "parameters": {
        "jsCode": "for (const item of $input.all()) {\n  if (item.binary) {\n    const binaryPropertyNames = Object.keys(item.binary);\n    for (const propName of binaryPropertyNames) {\n      if (item.binary[propName].mimeType === 'audio/mp3') {\n        item.binary[propName].mimeType = 'audio/mpeg';\n      }\n    }\n  }\n}\n\nreturn $input.all();"
      },
      "id": "52d6369a-ad16-40cc-b755-ab297dee72cb",
      "name": "Fix mimeType for Audio",
      "type": "n8n-nodes-base.code",
      "position": [
        3920,
        1580
      ],
      "typeVersion": 2
    },
    {
      "parameters": {
        "operation": "send",
        "phoneNumberId": "470271332838881",
        "recipientPhoneNumber": "={{ $('WhatsApp Trigger').item.json.messages[0].from }}",
        "textBody": "={{ $json.output }}",
        "additionalFields": {}
      },
      "id": "a5ecce8c-23da-4dca-9b5a-308073e4ffc3",
      "name": "Send message",
      "type": "n8n-nodes-base.whatsApp",
      "position": [
        3680,
        1760
      ],
      "webhookId": "23834751-5066-48ba-8e19-549680df2b27",
      "typeVersion": 1,
      "credentials": {
        "whatsAppApi": {
          "id": "2jO1IVmC3SFBa7bC",
          "name": "WhatsApp account"
        }
      }
    },
    {
      "parameters": {
        "operation": "send",
        "phoneNumberId": "470271332838881",
        "recipientPhoneNumber": "={{ $('Input type').item.json.contacts[0].wa_id }}",
        "messageType": "audio",
        "mediaPath": "useMedian8n",
        "additionalFields": {}
      },
      "id": "3c35ca0c-46e3-464a-b531-4e4034abb17b",
      "name": "Send audio",
      "type": "n8n-nodes-base.whatsApp",
      "position": [
        4100,
        1580
      ],
      "webhookId": "d18b2c98-84e4-43cf-a532-0c47d5161684",
      "typeVersion": 1,
      "credentials": {
        "whatsAppApi": {
          "id": "2jO1IVmC3SFBa7bC",
          "name": "WhatsApp account"
        }
      }
    },
    {
      "parameters": {
        "operation": "send",
        "phoneNumberId": "470271332838881",
        "recipientPhoneNumber": "={{ $('WhatsApp Trigger').item.json.messages[0].from }}",
        "textBody": "=Sorry but you can only send PDF files",
        "additionalFields": {}
      },
      "id": "53d41d8d-ffe9-48b6-bb93-d6988dde488d",
      "name": "Incorrect format",
      "type": "n8n-nodes-base.whatsApp",
      "position": [
        2080,
        1840
      ],
      "webhookId": "23834751-5066-48ba-8e19-549680df2b27",
      "typeVersion": 1,
      "credentials": {
        "whatsAppApi": {
          "id": "2jO1IVmC3SFBa7bC",
          "name": "WhatsApp account"
        }
      }
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "c53cd9f9-77c1-4331-98ff-bfc9bdf95a3c",
              "name": "text",
              "type": "string",
              "value": "={{ $('WhatsApp Trigger').item.json.messages[0].text.body }}"
            }
          ]
        },
        "options": {}
      },
      "id": "916d640d-1bda-4e83-a442-6d37ab363fa8",
      "name": "Text",
      "type": "n8n-nodes-base.set",
      "position": [
        2820,
        620
      ],
      "typeVersion": 3.4
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "219577d5-b028-48fc-90be-980f4171ab68",
              "name": "text",
              "type": "string",
              "value": "={{ $json.text }}"
            }
          ]
        },
        "options": {}
      },
      "id": "dd27e49c-d777-4791-bb91-a05ac375e533",
      "name": "Audio",
      "type": "n8n-nodes-base.set",
      "position": [
        2820,
        960
      ],
      "typeVersion": 3.4
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "67552183-de2e-494a-878e-c2948e8cb6bb",
              "name": "text",
              "type": "string",
              "value": "=User request on the image:\n{{ \"Describe the following image\" || $('WhatsApp Trigger').item.json.messages[0].image.caption }}\n\nImage description:\n{{ $json.content }}"
            }
          ]
        },
        "options": {}
      },
      "id": "faa0d2d4-67e4-446f-b95a-0cef6db8d3c0",
      "name": "Image",
      "type": "n8n-nodes-base.set",
      "position": [
        2800,
        1260
      ],
      "typeVersion": 3.4
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "67552183-de2e-494a-878e-c2948e8cb6bb",
              "name": "text",
              "type": "string",
              "value": "=User request on the file:\n{{ \"Describe this file\" || $('Only PDF File').item.json.messages[0].document.caption }}\n\nFile content:\n{{ $json.text }}"
            }
          ]
        },
        "options": {}
      },
      "id": "6dd37597-a260-4aef-acde-718af1678cc8",
      "name": "File",
      "type": "n8n-nodes-base.set",
      "position": [
        2800,
        1540
      ],
      "typeVersion": 3.4
    },
    {
      "parameters": {
        "operation": "send",
        "phoneNumberId": "470271332838881",
        "recipientPhoneNumber": "={{ $('WhatsApp Trigger').item.json.messages[0].from }}",
        "textBody": "=You can only send text messages, images, audio files and PDF documents.",
        "additionalFields": {}
      },
      "id": "b7f9d928-0f14-4f8f-919e-e18242d36f09",
      "name": "Not supported",
      "type": "n8n-nodes-base.whatsApp",
      "position": [
        1320,
        1500
      ],
      "webhookId": "23834751-5066-48ba-8e19-549680df2b27",
      "typeVersion": 1,
      "credentials": {
        "whatsAppApi": {
          "id": "2jO1IVmC3SFBa7bC",
          "name": "WhatsApp account"
        }
      }
    },
    {
      "parameters": {
        "resource": "media",
        "operation": "mediaUrlGet",
        "mediaGetId": "={{ $('WhatsApp Trigger').item.json.messages[0].image.id }}"
      },
      "id": "4e8e670f-b3c7-4733-a037-2aa6873b36a5",
      "name": "Get Image Url",
      "type": "n8n-nodes-base.whatsApp",
      "position": [
        2060,
        1260
      ],
      "webhookId": "280bd5de-32d7-4d8f-93d2-e91e3b0bc161",
      "typeVersion": 1,
      "credentials": {
        "whatsAppApi": {
          "id": "2jO1IVmC3SFBa7bC",
          "name": "WhatsApp account"
        }
      }
    },
    {
      "parameters": {
        "resource": "media",
        "operation": "mediaUrlGet",
        "mediaGetId": "={{ $('WhatsApp Trigger').item.json.messages[0].audio.id }}"
      },
      "id": "628a0baa-d8f2-4ed5-a49f-e9aa880cd9ec",
      "name": "Get Audio Url",
      "type": "n8n-nodes-base.whatsApp",
      "position": [
        2040,
        960
      ],
      "webhookId": "87caa300-7204-47b5-959a-94f4a8fbf8cf",
      "typeVersion": 1,
      "credentials": {
        "whatsAppApi": {
          "id": "2jO1IVmC3SFBa7bC",
          "name": "WhatsApp account"
        }
      }
    },
    {
      "parameters": {
        "resource": "audio",
        "input": "={{ $('Conversational Brain').item.json.output }}",
        "voice": "onyx",
        "options": {}
      },
      "id": "e842e70c-dbcf-44f4-8cc1-035e42b2f4fd",
      "name": "Generate Audio Response",
      "type": "@n8n/n8n-nodes-langchain.openAi",
      "position": [
        3680,
        1580
      ],
      "typeVersion": 1.8,
      "credentials": {
        "openAiApi": {
          "id": "IleT53idqzgq2A7w",
          "name": "OpenAi account"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "version": 2,
            "leftValue": "",
            "caseSensitive": true,
            "typeValidation": "strict"
          },
          "combinator": "and",
          "conditions": [
            {
              "id": "b9d1d759-f585-4791-a743-b9d72951e77c",
              "operator": {
                "type": "object",
                "operation": "exists",
                "singleValue": true
              },
              "leftValue": "={{ $('WhatsApp Trigger').item.json.messages[0].audio }}",
              "rightValue": ""
            }
          ]
        },
        "options": {}
      },
      "id": "a0497f5d-d647-4f50-aee4-035baaf95162",
      "name": "From audio to audio?",
      "type": "n8n-nodes-base.if",
      "position": [
        3400,
        1600
      ],
      "typeVersion": 2.2
    },
    {
      "parameters": {
        "rules": {
          "values": [
            {
              "conditions": {
                "options": {
                  "version": 2,
                  "leftValue": "",
                  "caseSensitive": true,
                  "typeValidation": "strict"
                },
                "combinator": "and",
                "conditions": [
                  {
                    "id": "08fd0c80-307e-4f45-b1de-35192ee4ec5e",
                    "operator": {
                      "type": "string",
                      "operation": "exists",
                      "singleValue": true
                    },
                    "leftValue": "={{ $json.messages[0].text.body }}",
                    "rightValue": ""
                  }
                ]
              },
              "renameOutput": true,
              "outputKey": "Text"
            },
            {
              "conditions": {
                "options": {
                  "version": 2,
                  "leftValue": "",
                  "caseSensitive": true,
                  "typeValidation": "strict"
                },
                "combinator": "and",
                "conditions": [
                  {
                    "id": "b7b64446-f1ea-4622-990c-22f3999a8269",
                    "operator": {
                      "type": "object",
                      "operation": "exists",
                      "singleValue": true
                    },
                    "leftValue": "={{ $json.messages[0].audio }}",
                    "rightValue": ""
                  }
                ]
              },
              "renameOutput": true,
              "outputKey": "Voice"
            },
            {
              "conditions": {
                "options": {
                  "version": 2,
                  "leftValue": "",
                  "caseSensitive": true,
                  "typeValidation": "strict"
                },
                "combinator": "and",
                "conditions": [
                  {
                    "id": "202af928-a324-411a-bf15-68a349e7bf9e",
                    "operator": {
                      "type": "object",
                      "operation": "exists",
                      "singleValue": true
                    },
                    "leftValue": "={{ $json.messages[0].image }}",
                    "rightValue": ""
                  }
                ]
              },
              "renameOutput": true,
              "outputKey": "Image"
            },
            {
              "conditions": {
                "options": {
                  "version": 2,
                  "leftValue": "",
                  "caseSensitive": true,
                  "typeValidation": "strict"
                },
                "combinator": "and",
                "conditions": [
                  {
                    "id": "c63299e9-6069-4bc6-afb9-7beebf6e3d69",
                    "operator": {
                      "type": "object",
                      "operation": "exists",
                      "singleValue": true
                    },
                    "leftValue": "={{ $json.messages[0].document }}",
                    "rightValue": ""
                  }
                ]
              },
              "renameOutput": true,
              "outputKey": "Document"
            }
          ]
        },
        "options": {
          "fallbackOutput": "extra"
        }
      },
      "id": "5f38e898-b4a3-4a80-95cf-b8c7fa6440fb",
      "name": "Input type",
      "type": "n8n-nodes-base.switch",
      "position": [
        1160,
        1180
      ],
      "typeVersion": 3.2
    },
    {
      "parameters": {
        "resource": "audio",
        "operation": "transcribe",
        "options": {}
      },
      "id": "289bac28-6792-459e-b63d-1b00187570c3",
      "name": "Voice-to-Text Conversion",
      "type": "@n8n/n8n-nodes-langchain.openAi",
      "position": [
        2540,
        960
      ],
      "typeVersion": 1.8,
      "credentials": {
        "openAiApi": {
          "id": "IleT53idqzgq2A7w",
          "name": "OpenAi account"
        }
      }
    },
    {
      "parameters": {
        "resource": "image",
        "operation": "analyze",
        "modelId": {
          "__rl": true,
          "mode": "list",
          "value": "gpt-4o-mini",
          "cachedResultName": "GPT-4O-MINI"
        },
        "text": "=You are an advanced image description AI assistant . Your primary function is to provide detailed, accurate descriptions of images submitted through WhatsApp.\n\nCORE FUNCTIONALITY:\n- When presented with an image, you will analyze it thoroughly and provide a comprehensive description in English.\n- Your descriptions should capture both the obvious and subtle elements within the image.\n\nIMAGE DESCRIPTION GUIDELINES:\n- Begin with a broad overview of what the image contains\n- Describe key subjects, people, objects, and their relationships\n- Note significant visual elements such as colors, lighting, composition, and perspective\n- Identify any text visible in the image\n- Describe the setting or environment\n- Mention any notable actions or events taking place\n- Comment on mood, tone, or atmosphere when relevant\n- If applicable, identify landmarks, famous people, or cultural references\n\nRESPONSE FORMAT:\n- Start with \"Image Description:\" followed by your analysis\n- Structure your description in a logical manner (general to specific)\n- Use clear, precise language appropriate for visual description\n- Format longer descriptions with paragraphs to enhance readability\n- End with any notable observations that might require special attention\n\nLIMITATIONS:\n- If the image is blurry, low resolution, or difficult to interpret, acknowledge these limitations\n- If an image contains potentially sensitive content, provide a factual description without judgment\n- Do not make assumptions about elements that cannot be clearly determined\n\nYour descriptions should be informative, objective, and thorough, enabling someone who cannot see the image to form an accurate mental picture of its contents.",
        "inputType": "base64",
        "options": {
          "detail": "auto"
        }
      },
      "id": "c62e7411-edd3-41f2-90b2-77134cd44de0",
      "name": "Image Content Analysis",
      "type": "@n8n/n8n-nodes-langchain.openAi",
      "position": [
        2540,
        1260
      ],
      "typeVersion": 1.8,
      "credentials": {
        "openAiApi": {
          "id": "IleT53idqzgq2A7w",
          "name": "OpenAi account"
        }
      }
    },
    {
      "parameters": {
        "operation": "pdf",
        "options": {}
      },
      "id": "645a914d-de84-42ee-92e8-e091f0a20c74",
      "name": "PDF Text Extraction",
      "type": "n8n-nodes-base.extractFromFile",
      "position": [
        2540,
        1540
      ],
      "typeVersion": 1
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "={{ $json.text }}",
        "options": {
          "systemMessage": "You are an intelligent assistant. Your purpose is to analyze various types of input and provide helpful, accurate responses.\n\nCAPABILITIES:\n- Process and respond to text messages\n- Analyze uploaded files\n- Interpret and describe images\n- Transcribe and understand voice messages\n\nINPUT HANDLING:\n1. For text messages: Analyze the content, understand the intent, and provide a relevant response.\n2. For file analysis: Examine the file content, extract key information, and summarize important points also based on the questions asked.\n3. For image analysis: Describe what you see in the image, identify key elements, and respond to any questions about the image.\n4. For voice messages: Transcribe the audio, understand the message, and respond appropriately.\n\nRESPONSE GUIDELINES:\n- Be concise but thorough\n- Prioritize accuracy over speculation\n- Maintain a professional and helpful tone\n- When uncertain, acknowledge limitations\n- Format responses for easy reading on mobile devices\n- Include actionable information when appropriate\n\nLIMITATIONS:\n- Mention if you're unable to process certain file formats\n- Indicate if an image is unclear or if details are difficult to discern\n- Note if audio quality impacts transcription accuracy\n\nSECURITY & PRIVACY:\n- Do not store or remember sensitive information shared in files, images, or voice notes\n- Do not share personal information across different user interactions\n- Inform users about data privacy limitations when relevant\n\nAnalyze all inputs carefully before responding. Your goal is to provide value through accurate information and helpful assistance."
        }
      },
      "id": "4ec78b40-ab8d-492d-a0d9-29b1e8152c7a",
      "name": "Conversational Brain",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "position": [
        3040,
        1600
      ],
      "typeVersion": 1.8
    }
  ],
  "pinData": {},
  "connections": {
    "File": {
      "main": [
        [
          {
            "node": "Conversational Brain",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Text": {
      "main": [
        [
          {
            "node": "Conversational Brain",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Audio": {
      "main": [
        [
          {
            "node": "Conversational Brain",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Image": {
      "main": [
        [
          {
            "node": "Conversational Brain",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Input type": {
      "main": [
        [
          {
            "node": "Text",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Get Audio Url",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Get Image Url",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Only PDF File",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Not supported",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get File Url": {
      "main": [
        [
          {
            "node": "Download File",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Download File": {
      "main": [
        [
          {
            "node": "PDF Text Extraction",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get Audio Url": {
      "main": [
        [
          {
            "node": "Download Audio",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get Image Url": {
      "main": [
        [
          {
            "node": "Download Image",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Only PDF File": {
      "main": [
        [
          {
            "node": "Get File Url",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Incorrect format",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Simple Memory": {
      "ai_memory": [
        [
          {
            "node": "Conversational Brain",
            "type": "ai_memory",
            "index": 0
          }
        ]
      ]
    },
    "Download Audio": {
      "main": [
        [
          {
            "node": "Voice-to-Text Conversion",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Download Image": {
      "main": [
        [
          {
            "node": "Image Content Analysis",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "WhatsApp Trigger": {
      "main": [
        [
          {
            "node": "Input type",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "OpenAI Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "Conversational Brain",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "From audio to audio?": {
      "main": [
        [
          {
            "node": "Generate Audio Response",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Send message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Fix mimeType for Audio": {
      "main": [
        [
          {
            "node": "Send audio",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Generate Audio Response": {
      "main": [
        [
          {
            "node": "Fix mimeType for Audio",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Voice-to-Text Conversion": {
      "main": [
        [
          {
            "node": "Audio",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Image Content Analysis": {
      "main": [
        [
          {
            "node": "Image",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "PDF Text Extraction": {
      "main": [
        [
          {
            "node": "File",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Conversational Brain": {
      "main": [
        [
          {
            "node": "From audio to audio?",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1"
  },
  "versionId": "ccbad6a2-a63e-4dbd-ab66-974368ac977a",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "16ed17c9c2a3c2f2456fa33a1606b954271b5e6a768b8b2e1c807a5bb7e13121"
  },
  "id": "Mq0T58KLGM0Oitt0",
  "tags": []
}
