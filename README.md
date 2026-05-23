# 🧠 White-Box Cognitive Architecture (WBCA)
## A Domain-Agnostic 4D Temporal Cognitive Graph Framework for AI Agents and MCP

[![GitHub License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![MariaDB](https://img.shields.io/badge/Database-MariaDB%20%2F%20MySQL-orange.svg)](https://mariadb.org/)
[![Model Context Protocol](https://img.shields.io/badge/Protocol-MCP-green.svg)](https://modelcontextprotocol.io/)

---

## 🌟 ¿Qué es WBCA?
Las arquitecturas RAG vectoriales tradicionales operan como cajas negras opacas, consumen recursos excesivos en llamadas de embeddings y sufren de **ceguera temporal absoluta**: no entienden el pasado, borran el historial ante cambios y saturan la ventana de contexto de los LLM con prosa humana redundante (ruido sintáctico).

La **White-Box Cognitive Architecture (WBCA)** es un **framework cognitivo universal y agnóstico al dominio (Domain-Agnostic)**. Mapea el conocimiento de manera determinista en un **Grafo Temporal de 4 Dimensiones (4D-TCG)** persistido en MariaDB y consumido quirúrgicamente por agentes de IA mediante el protocolo **MCP (Model Context Protocol)**. 

El framework es completamente portable y adaptable a cualquier dominio de conocimiento (ej. *desarrollo de software, ERPs, finanzas, automatización de infraestructura, o gestión de conocimiento personal*), abstrayendo las entidades como nodos generales y las relaciones como aristas temporalizadas bajo transaccionalidad ACID.

Este repositorio contiene la especificación, los planos y la guía de implementación paso a paso (*HOWTO*) estructurada en las tres fases evolutivas de nuestro trayecto de desarrollo real.

---

## 🚀 La Evolución del Framework (Fases)

Para llegar a la robustez transaccional actual de MariaDB, este framework recorrió un viaje evolutivo empírico impulsado por dolores operacionales reales al trabajar con agentes de IA de pair programming (como *Antigravity* y *OpenClaw*):

| Fase | Infraestructura | Almacenamiento | Enfoque Cognitivo | Limitación / Dolor |
| :--- | :--- | :--- | :--- | :--- |
| **Fase A** | Multi-Dispositivo (PC / Laptop) | Google Drive + JSON | Descentralizado con Firmas (`whoami`) | Latencia de Drive y colisiones JSON |
| **Fase B** | Multi-Dispositivo | Markdown Superdenso (LLM-Wiki) | Inyección Machine-to-Machine (M2M) | Latencia de lectura y parseo de disco |
| **Fase C** *(El Desmadre)* | Servidor MCP 4D | MariaDB (Transaccional SCD) | Quórum 4D Expandido + Fast-Path | Ninguna (Operación en producción) |

---

## 🏗️ Características Principales (Fase C - Actual)
* **Quórum Expandido de 4 Agentes:** Un proceso de debate cognitivo con roles especializados y agnósticos al dominio:
  * 🕵️‍♂️ **Agente A (Analista):** Extracción atómica de entidades y conceptos.
  * 🔗 **Agente B (Compilador):** Vinculación de aristas semánticas puras.
  * 🛡️ **Agente C (CISO/Consistencia):** Auditoría estricta de reglas de negocio y consistencia lógica.
  * ⏳ **Agente D (El Historiador):** Mapeo de deltas de transición temporal.
* **FinOps Fast-Path vs Deep-Debate:** Clasificación inicial de confianza (Gemini Flash). Si la confianza estructural es alta y no hay colisiones históricas en la base de datos, el sistema realiza una inserción directa sin debate, **ahorrando hasta un 90% en costos de tokens**.
* **Dimensión Temporal de Primera Clase (SCD Tipo 2):** Nodos y aristas versionados en MariaDB (`version`, `valid_from`, `valid_to`, `is_active`) para viajar en el tiempo y diagnosticar fallas analizando deltas históricos.
* **De-compilación Cognitiva On-Demand:** Traducción del grafo de alta densidad optimizado para máquinas a prosa comprensible por humanos a través de herramientas MCP.

---

## 📁 Estructura del Repositorio
```
├── README.md                # Este archivo de bienvenida y descripción general
└── HOWTO.md                 # Guía técnica e histórica con los scripts Python integrados
```

---

## 🛠️ ¿Cómo Empezar?
1. Dirígete a la guía detallada de implementación en [HOWTO.md](HOWTO.md) para explorar la evolución de los scripts y ver los ejemplos de mapeo para diferentes dominios de conocimiento.
2. Despliega el esquema relacional en MariaDB utilizando el script DDL provisto.
3. Levanta tu propio servidor MCP para inyectar conocimiento libre de ruido a tus agentes de IA (como OpenClaw).

---

## 🤝 Contribuir y Comunidad
Esta arquitectura es de código abierto y libre distribución. Si tienes ideas para optimizar el Quórum de agentes, mejorar la indexación en MariaDB o expandir las herramientas del servidor MCP, siéntete libre de abrir un *Pull Request* o reportar un *Issue*. 

*¡Hagamos que el razonamiento agentico sea transparente, rápido, barato y de Caja Blanca!*
