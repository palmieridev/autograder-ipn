# AutoGrader IPN

Plataforma de calificación automática de código para el Instituto Politécnico Nacional.

Los profesores crean tareas de programación, los alumnos envían su código y el sistema lo ejecuta en contenedores aislados, evalúa los resultados contra casos de prueba y asigna una calificación — todo de forma automática.

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| Backend | Go + Fiber |
| Frontend | Astro + TypeScript |
| Base de datos | PostgreSQL + PGMQ |
| Contenedores | Podman |
| Queries | sqlc + golang-migrate |

## Flujo de calificación

```mermaid
sequenceDiagram
    actor A as Alumno
    participant F as Frontend (Astro)
    participant B as Backend (Go/Fiber)
    participant Q as Cola PGMQ
    participant C as Contenedor Podman
    participant DB as PostgreSQL

    A->>F: Envía código
    F->>B: POST /submissions
    B->>DB: Guarda envío (pendiente)
    B->>Q: Encola trabajo de ejecución
    Q->>B: Worker toma trabajo
    B->>C: Ejecuta código en sandbox
    C-->>B: stdout / stderr / exit code
    B->>B: Compara salida vs casos de prueba
    B->>DB: Guarda calificación
    A->>F: Consulta resultado
    F->>B: GET /submissions/{id}
    B->>DB: Lee calificación
    DB-->>F: Resultado + detalle
```

## Arquitectura general

```mermaid
graph TD
    subgraph Cliente
        FE[Frontend Astro]
    end

    subgraph Servidor
        API[API Go/Fiber]
        W[Workers]
    end

    subgraph Datos
        DB[(PostgreSQL)]
        MQ[PGMQ]
    end

    subgraph Ejecución
        P1[Contenedor Podman]
        P2[Contenedor Podman]
        Pn[Contenedor Podman]
    end

    FE -->|REST| API
    API --> DB
    API --> MQ
    MQ --> W
    W --> P1
    W --> P2
    W --> Pn
    P1 -->|resultado| W
    P2 -->|resultado| W
    Pn -->|resultado| W
    W --> DB
```

## Fases del proyecto

**Fase 1** — Servidor centralizado. Un solo servidor ejecuta los contenedores de forma local.
**Fase 2** — Agentes distribuidos. Las computadoras de los profesores actúan como nodos de ejecución con fallback al servidor central.
