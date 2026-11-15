# Agente Cliente Google ADK para Gestión de Tareas (`agentemcp`)

Agente conversacional desarrollado en **Java 21** con **Google Agent Development Kit (`google-adk 0.6.0`)** e impulsado por **Gemini (`gemini-3-flash-preview`)**. Se conecta como cliente MCP vía **Streamable HTTP** al servidor [`demomcp`](https://github.com/BryanMedrano/demomcp) (`http://localhost:8088/mcp`) para administrar tareas pendientes y completadas en lenguaje natural desde una interfaz web interactiva.

---

## Arquitectura y Funcionamiento

- **Host MCP (`AdkWebServer`):** Servidor web embebido en el puerto `8080` que gestiona las sesiones conversacionales, la interfaz gráfica de chat y la inspección de eventos (`Events`, `Trace` y `State`).
- **Cliente MCP (`McpToolset`):** Conecta mediante `StreamableHttpServerParameters` a `http://localhost:8088/mcp`, negocia el protocolo JSON-RPC 2.0 y descubre automáticamente las **9 herramientas** expuestas por el servidor `demomcp`.
- **Agente Inteligente (`TareaAgent`):** Implementado con `LlmAgent` y el modelo `gemini-3-flash-preview`, recibe instrucciones de sistema para consultar, contar, crear, actualizar y eliminar tareas solicitando confirmación previa antes de operaciones destructivas.

---

## Herramientas MCP Consumidas (`demomcp`)

El agente descubre y orquesta dinámicamente las siguientes herramientas del backend:

- `listarTodas` — Lista todas las tareas registradas en MySQL.
- `listPendientes` — Consulta las tareas pendientes (`completado = false`).
- `listarCompletadas` — Consulta las tareas finalizadas (`completado = true`).
- `contarTareas` — Obtiene el conteo total de registros.
- `buscarPorTitulo` — Filtra tareas por coincidencia parcial en el título.
- `crearTarea` — Crea nuevas tareas validando fecha en formato ISO `yyyy-MM-dd`.
- `updateTarea` — Modifica el título o la fecha de una tarea existente.
- `marcarCompletado` — Cambia el estado de una tarea a completada.
- `eliminarTarea` — Elimina una tarea por su identificador.

---

## Requisitos Previos

- **Java JDK 21** (con la variable de entorno `JAVA_HOME` configurada y agregada al `PATH`).
- **Servidor MCP [`demomcp`](https://github.com/BryanMedrano/demomcp) ejecutándose** previamente en `http://localhost:8088/mcp` con MySQL activo.
- **Clave API de Google AI Studio** con acceso habilitado a `gemini-3-flash-preview`.
- **Variable de entorno `GOOGLE_API_KEY`** configurada en el sistema o sesión de terminal con la clave de Google AI Studio.
- **Puerto `8080` libre** en el equipo local (`localhost:8080`).
- **Acceso a Internet** para comunicarse con la API de Google Gemini y descargar las dependencias Maven (`google-adk`, `google-adk-dev`, `httpclient5`, `rxjava`) en la primera compilación.

---

## Configuración y Ejecución

1. **Clonar el repositorio:**
   ```powershell
   git clone https://github.com/BryanMedrano/agentemcp.git
   cd agentemcp
   ```

2. **Configurar la variable de entorno con la API Key de Gemini:**
   ```powershell
   $env:GOOGLE_API_KEY = "TU_API_KEY_DE_GOOGLE_AI_STUDIO"
   ```

3. **Compilar e instalar el proyecto en `target/`:**
   ```powershell
   .\mvnw.cmd clean install
   ```
   *(Este paso es obligatorio antes de arrancar `AdkWebServer`, ya que el servidor carga los agentes compilados desde `--adk.agents.source-dir=target`).*

4. **Levantar la interfaz web de Google ADK:**
   ```powershell
   .\mvnw.cmd exec:java@runAdkWebServer
   ```
   El servidor web iniciará en `http://localhost:8080`.

5. **Probar el agente en el navegador:**
   - Abre `http://localhost:8080`.
   - Selecciona **`TareaAgent`** en el selector superior izquierdo.
   - Asegúrate de mantener el interruptor **Token Streaming** **apagado** para visualizar correctamente las llamadas a herramientas MCP síncronas.
   - Envía consultas como:
     - *"Hola, muestra las tareas pendientes y dime cuántas hay en total"*
     - *"Crea una tarea llamada Validar agente ADK con fecha 2026-11-16"*

---
