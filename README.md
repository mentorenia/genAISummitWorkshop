![Python](https://img.shields.io/badge/Python-3.11-blue?style=for-the-badge&logo=python)
![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker)
![MCP](https://img.shields.io/badge/Protocol-MCP-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Production%20Grade-green?style=for-the-badge)

> **Workshop Oficial - GenAI Summit 2026**
>
> * Nexus Agent: Arquitectura de Agentes Auditables en Producción*

---

## 📖 Propósito del Proyecto

La mayoría de los agentes de IA fallan en producción por tres razones:
* ❌ **Indeterminismo:** Respuestas impredecibles o alucinaciones.
* ❌ **Costes descontrolados:** Bucles infinitos que queman presupuesto.
* ❌ **Opacidad:** Cajas negras inauditables para negocio o legal.

**Nexus Agent** es una implementación de referencia que resuelve estos problemas mediante un enfoque de **Defensa en Profundidad**:

1.  **Validación Estricta:** Uso de `Pydantic` para garantizar contratos de datos a la entrada y salida.
2.  **Estándar de Herramientas:** Implementación del **Model Context Protocol (MCP)** para desacoplar la lógica del LLM de las integraciones (CRM, Docs).
3.  **Gobernanza Financiera:** Un orquestador con presupuesto (€) en tiempo real y logs de auditoría estructurados.
4.  **Despliegue Inmutable:** Containerización con Docker para despliegue serverless (Cloud Run).



---

## Estructura del Proyecto

Este repositorio sigue una arquitectura hexagonal simplificada para separar la infraestructura, el dominio y las interfaces.

```text
nexus-agent/
├── 📂 .devcontainer/       # Configuración para GitHub Codespaces (Entorno Efímero)
├── 📂 data/                # SIMULACIÓN DEL ENTORNO CORPORATIVO
│   ├── crm.db              # SQLite simulando un CRM Enterprise
│   ├── setup_data.py       # Script para resetear la simulación
│   └── 📂 knowledge/       # Base de conocimiento (Archivos Markdown para RAG)
│
├── 📂 src/                 # CÓDIGO FUENTE
│   ├── 📂 core/            # EL CEREBRO (Gobernanza)
│   │   ├── agent.py        # BudgetedOrchestrator: El bucle de ejecución controlado
│   │   └── audit.py        # AuditLogger: Sistema de trazas y logs estructurados
│   │
│   ├── 📂 models/          # LOS CONTRATOS (Módulo 1)
│   │   ├── incoming.py     # Sanitización de inputs (Emails, Webhooks)
│   │   └── decision.py     # Estructura determinista de salida (NexusDecision)
│   │
│   ├── 📂 tools/           # LAS MANOS (Módulo 2 - MCP)
│   │   ├── crm.py          # Lógica de acceso a BBDD (Read-Only)
│   │   ├── knowledge.py    # Lógica de lectura de archivos segura
│   │   └── registry.py     # Definiciones JSON Schema para el LLM
│   │
│   ├── main.py             # LA PUERTA (API FastAPI)
│   └── config.py           # Gestión de Secretos (.env)
│
├── 📂 infra/               # DESPLIEGUE (Módulo 4)
│   └── Dockerfile          # Definición inmutable del entorno de ejecución
│
├── .env.example            # Plantilla de variables de entorno
├── requirements.txt        # Dependencias congeladas
└── README.md               # Este documento
```

## Stack Tecnológico

- Python 3.11
- Pydantic V2. Convierte el caos probabilístico del LLM en objetos Python validados. Si el LLM alucina un campo, Pydantic detiene la ejecución.
- Anthropic SDK. Control a bajo nivel del modelo Claude 3.5 Sonnet. Preferimos SDKs crudos a frameworks abstractos (como LangChain) para tener control total del bucle.
- MCP (Model Context Protocol. Permite escribir herramientas una vez y usarlas con cualquier modelo o cliente (Claude Desktop, Cursor, etc.).
- FastAPI. Expone el agente como un microservicio asíncrono de alto rendimiento.
- Rich. Proporciona logs visuales en consola para depurar latencia y costes en tiempo real.
- Docker. Garantiza despliegues serverless en cualquier máquina.

## Requisitos Previos
1. Docker Desktop instalado y corriendo.
2. Python 3.10+ instalado.
3. Una API Key de Anthropic (con créditos disponibles)

## Instalación 

1. Clonar el repositorio
``git clone https://github.com/mentorenia/genAISummitWorkshop.git
cd genAISummirWorkshop``

2. Crear entorno virtual 
``python -m venv venv
source venv/bin/activate``  

En Windows: 
``venv\Scripts\activate``

3. Instalar dependencias
``pip install -r requirements.txt``

4. Configurar entorno
``cp .env.example .env``

Edita el archivo .evn y añade tu ANTRHOPIC_API_KEY. 

### Inicializar la Simulación

Antes de correr el agente, necesitamos crear el "Mundo Falso" (Base de datos CRM y Archivos).
``python data/setup_data.py``


Salida esperada: 
``✅ CRM Simulado creado... 
✅ Base de Conocimiento creada...``

## Uso del Taller

### Fase 1: Ejecución CLI (Modo desarrollo)
Para ver los logs de dinero y trazas en tiempo real
``python -m src.cli``
Esto lanzará un prompt interactivo donde podrás enviar emails simulados y ver cómo el agente "piensa", gasta dinero y consulta herramientas.

### Fase 2: Ejecución API (Modo producción)
Para levantar el servidor FastAPI
* ``uvicorn src.main:app --reload``
Accede a la documentación automática en: 
``http://localhost:8000/docs``

### Fase 3: Despliegue con Docker (Simulación Cloud)
Para construir la imagen
``docker build -t nexus-agent -f infra/Dockerfile``

# Correr el contenedor (inyectando la key)
``docker run -p 8080:8080 -e ANTHROPIC_API_KEY=tu-key nexus-agent``

## Escenarios de Prueba (Los "Stress Tests")
Durante el taller, probaremos la robustez del sistema con estos inputs:
1. Un email solicitando información de soporte (Tool: Knowledge Base).
2. Un email pidiendo el estatus VIP (Tool: CRM).
3. El Ataque de Inyección: Un email intentando leer archivos del sistema (../../etc/passwd). -> El sistema debe bloquearlo.
4. Un email pidiendo una tarea infinita. -> El BudgetLimiter debe matar el proceso.

## 📝 Roadmap del Taller (3 Horas)
00:00 - 01:00 | Arquitectura & Contratos: Diseño del sistema y validación con Pydantic.
01:00 - 02:00 | Manos a la Obra (MCP): Implementación de tools (CRM/Docs) y servidor MCP.
02:00 - 03:00 | Control & Despliegue: Implementación del Orquestador con presupuesto, logs de auditoría y containerización final.
