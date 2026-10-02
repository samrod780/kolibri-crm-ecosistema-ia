# Kolibri CRM — Ecosistema de Automatización IA Autónomo

**Entrega Final — Coderhouse, AI Automation**
Autor: Sam Rodriguez

## Descripción del proyecto

Kolibri CRM es el caso de estudio (negocio inventado) usado para diseñar y construir, de punta a punta, un
ecosistema de automatización con IA para generación de contenido con supervisión humana (Human-in-the-Loop).

El sistema combina:

- **n8n** — orquestación del flujo (trigger, RAG, llamada a IA, escritura de datos, notificación).
- **Notion** — capa de datos: piezas de contenido (Centro de Comando), conocimiento de referencia (Base RAG)
  y registro de errores (Log de Errores).
- **Claude API (Anthropic)** — generación del contenido (`claude-sonnet-5`), con contexto RAG inyectado en el prompt.
- **Gmail** — notificación y aprobación humana (HITL) vía "Send and Wait for Response": ninguna pieza se
  publica sin que una persona la apruebe desde el correo.

## Links públicos (modo lectura)

| Base | Link |
|---|---|
| Centro de Comando (Dashboard de control) | https://app.notion.com/p/3e264d4a9063801f9f60ca0eb7dc19fa |
| Base de Conocimiento RAG | https://app.notion.com/p/b926c5d7c7af4a0b84b2b38472d541bb |
| Log de Errores | https://app.notion.com/p/7d3d894ccd7c4bedb5e63abcb2c6af2f |

Las tres bases están compartidas con acceso general "Cualquier usuario con el enlace" (modo lectura).

## Estructura del repositorio

```
repo_kolibri_crm/
├── README.md                              # este archivo
├── entregables/
│   ├── 01_diagrama_arquitectura.pdf       # Entregable 1 — diagrama del flujo completo
│   ├── 02_manual_operativo_datos.pdf      # Entregable 2 — bases de Notion y procedimientos
│   ├── 03_matriz_costos_modelos_ia.pdf    # Entregable 3 — comparativa de costos de modelos IA
│   └── 04_seguridad_resiliencia.pdf       # Entregable 4 — credenciales, errores, continuidad
├── dashboard/
│   └── dashboard_kolibri.html             # Entregable 5 — copia estática de referencia del dashboard
├── evidencia/
│   └── (screenshots de ejecuciones del flujo en n8n — ver sección "Evidencia visual")
└── n8n/
    └── kolibri-crm-workflow.json          # export completo del workflow
```

## Entregable 5 — Dashboard de control

El Dashboard de Control es la base **Centro de Comando** en Notion (link arriba), combinada con la vista de
**Log de Errores** como panel de KPIs/errores. Ambas están publicadas en modo lectura como Shared View pública,
cumpliendo con el formato pedido (enlace público a una vista tipo Notion Shared View).

Adicionalmente, `dashboard/dashboard_kolibri.html` incluido en este repo es una **copia de referencia** del
código de un dashboard vivo construido como artifact en Cowork (se conecta directamente a Notion en cada
carga, sin exponer credenciales) — al abrirla suelta (fuera de Cowork) no va a poder consultar Notion en vivo.

## Evidencia visual del flujo

En `evidencia/` se incluyen capturas de pantalla reales de ejecuciones del workflow en n8n:

1. **Vista general del canvas** — los 9 nodos del flujo completo.
2. **Ejecución exitosa (camino feliz)** — corrida completa sin errores (Succeeded).
3. **Ejecución con error — credencial de Gmail** — "The credential 'Gmail account' needs to be reconnected".
4. **Ejecución con error — camino infeliz** — "Cannot read properties of undefined (reading 'text')" en el
   nodo "Update a database page", con los nodos en rojo resaltados por el sistema de reintentos de n8n.

Estas capturas documentan tanto el camino feliz como el manejo de errores (reintentos + registro en Log de
Errores) descrito en el Entregable 4.

## Resumen del flujo

1. Se carga una `Idea_Semilla` en la base **Centro de Comando** (Notion).
2. El **Notion Trigger** de n8n detecta la fila nueva y arranca el flujo.
3. Se trae el contexto de la **Base RAG** (6 fragmentos de conocimiento de marca/negocio).
4. Un nodo **Code (JavaScript)** arma el prompt combinando la idea semilla + el contexto RAG.
5. **Claude API** (`claude-sonnet-5`) genera el contenido.
6. n8n escribe el resultado en Notion y cambia el estado a "Esperando aprobación".
7. Se envía un correo por **Gmail** ("Send and Wait for Response") — el flujo queda pausado.
8. Al aprobar o rechazar desde el correo, n8n retoma la ejecución y actualiza el estado final en Notion.
9. Si cualquier paso crítico falla (Claude, Notion o Gmail), el error se reintenta 3 veces y, si persiste,
   queda registrado en la base **Log de Errores** en vez de romper todo el flujo.

Ver el diagrama de arquitectura (Entregable 1) para el detalle visual completo.

## Índice de entregables

| # | Entregable | Archivo / Link |
|---|---|---|
| 1 | Diagrama de arquitectura | `entregables/01_diagrama_arquitectura.pdf` |
| 2 | Manual operativo de datos | `entregables/02_manual_operativo_datos.pdf` |
| 3 | Matriz de costos de modelos IA | `entregables/03_matriz_costos_modelos_ia.pdf` |
| 4 | Documentación de seguridad y resiliencia | `entregables/04_seguridad_resiliencia.pdf` |
| 5 | Dashboard de control | Notion Shared View (link arriba) + `dashboard/dashboard_kolibri.html` |
| — | Workflow de n8n | `n8n/kolibri-crm-workflow.json` |
| — | Link en modo lectura a la Base de Datos | Notion — Centro de Comando (link arriba) |
| — | Evidencia visual del flujo | `evidencia/` (ver sección arriba) |

---
Kolibri CRM · Caso de estudio — Coderhouse AI Automation · Entrega Final
