# Nexus

Sistema de Code Review multi-agente: subes un repositorio de GitHub o pegas
código directamente, especificas qué quieres que se revise (seguridad,
rendimiento, patrones de diseño, buenas prácticas...) y un grafo de agentes
(LangGraph) analiza el código en paralelo con especialistas dedicados a cada
área. Un nodo Router decide qué especialistas activar según lo que pediste;
un Synthesizer combina sus hallazgos en un informe final. Si pides comentar
los hallazgos en un PR de GitHub, el sistema se detiene y espera tu
aprobación explícita antes de publicar nada.

## Levantar todo con Docker (recomendado)

Requisito: [Docker Desktop](https://www.docker.com/products/docker-desktop/).
No hace falta instalar Python, Node ni pnpm en tu máquina — todo corre
dentro de los contenedores.

1. Copia la plantilla de variables de entorno del backend y rellénala con
  tus claves reales:
   Necesitas rellenar `GROQ_API_KEY` (clave de [Groq](https://console.groq.com/keys)),
   `GITHUB_TOKEN` (personal access token de GitHub, con permiso para leer
   repos y comentar en PRs) y `MCP_API_KEY` (cualquier cadena secreta que tú
   elijas — protege la comunicación interna entre el backend y el servidor
   MCP). El resto de valores del `.env.example` ya vienen listos para este
   flujo con Docker.
2. Levanta todo el stack:
  ```
   docker-compose up --build
  ```
   Esto construye y arranca cinco servicios: `postgres`, `redis`, `backend`
   (API FastAPI, puerto 8000), `mcp_server` (herramientas de GitHub para el
   grafo, puerto 8001) y `frontend` (dashboard Next.js, puerto 3000). El
   backend aplica las migraciones de Alembic automáticamente antes de
   arrancar, y espera a que Postgres y Redis estén realmente listos antes de
   intentar conectarse.
3. Abre [http://localhost:3000](http://localhost:3000).

Para pararlo todo: `docker-compose down`. Para reconstruir las imágenes tras
cambiar código o dependencias: `docker-compose up --build` de nuevo.

**Nota sobre** `NEXT_PUBLIC_API_URL`**:** esta variable se incrusta en el
frontend en tiempo de *build*, no se lee al arrancar el contenedor — así
funciona Next.js con cualquier variable `NEXT_PUBLIC_*`. Si vas a acceder al
dashboard desde una URL distinta de `localhost:8000` para el backend, edita
el valor bajo `frontend.build.args` en `docker-compose.yml` y reconstruye
con `docker-compose up --build`.

## Desarrollo local (sin Docker para backend/frontend)

Así es como se ha desarrollado el proyecto día a día — Postgres y Redis
siguen corriendo vía Docker (`docker-compose up postgres redis`), pero
backend y frontend se ejecutan directamente con recarga en caliente.

- **Backend:** `cd backend && uvicorn app.main:app --reload`
- **Frontend:** `cd frontend && pnpm dev`
- **Tests backend:** `cd backend && pytest -v`
- **Tests frontend:** `cd frontend && pnpm test`

Variables de entorno para este flujo: `backend/.env.example` y
`frontend/.env.example` — cópialas a `backend/.env` y `frontend/.env.local`
respectivamente y rellena los valores reales.