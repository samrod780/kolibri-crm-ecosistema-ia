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
└── n8n/
    └── (agregar acá el export del workflow — ver instrucciones abajo)
```

## Entregable 5 — Dashboard de control

El dashboard vive como un **artifact en vivo dentro de Cowork** (se conecta directamente a Notion en cada
carga, sin exponer credenciales). El archivo `dashboard/dashboard_kolibri.html` incluido en este repo es una
**copia de referencia** del código fuente para documentación — al abrirla suelta (fuera de Cowork) no va a
poder consultar Notion en vivo, porque esa conexión segura sólo existe dentro del entorno de Cowork.

## Cómo agregar el export de n8n

El workflow completo (`PE7 - Generacion con RAG (HITL)`) vive en la instancia local de n8n. Para incluirlo en
este repo:

1. Abrir el workflow en n8n.
2. Menú "..." (arriba a la derecha) → **Download**.
3. Guardar el archivo `.json` resultante dentro de la carpeta `n8n/` de este repo, con el nombre
   `kolibri-crm-workflow.json`.

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

| # | Entregable | Archivo |
|---|---|---|
| 1 | Diagrama de arquitectura | `entregables/01_diagrama_arquitectura.pdf` |
| 2 | Manual operativo de datos | `entregables/02_manual_operativo_datos.pdf` |
| 3 | Matriz de costos de modelos IA | `entregables/03_matriz_costos_modelos_ia.pdf` |
| 4 | Documentación de seguridad y resiliencia | `entregables/04_seguridad_resiliencia.pdf` |
| 5 | Dashboard de control | `dashboard/dashboard_kolibri.html` (versión en vivo en Cowork) |
| — | Workflow de n8n | `n8n/kolibri-crm-workflow.json` (agregar manualmente, ver arriba) |

---
Kolibri CRM · Caso de estudio — Coderhouse AI Automation · Entrega Final
