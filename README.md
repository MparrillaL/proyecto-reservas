# Plataforma de Reservas — Actividades Culturales de Sevilla

Aplicación web contenerizada para gestionar reservas de **talleres, visitas y eventos**
de una empresa cultural de Sevilla.

> **Actividad:** DAW · UD01 · AEE RA1 — Implanta arquitecturas web analizando y aplicando criterios de funcionalidad.
> **Autor:** Manuel Parrilla Lahoz
> **Rol asumido:** equipo de despliegue.
> **Fecha:** 2026-09-30

---

## 📑 Índice

1. [Descripción del proyecto](#1-descripción-del-proyecto)
2. [Arquitectura](#2-arquitectura)
3. [Estructura del repositorio](#3-estructura-del-repositorio)
4. [Requisitos previos](#4-requisitos-previos)
5. [Instalación y arranque](#5-instalación-y-arranque)
6. [Uso](#6-uso)
7. [Pruebas](#7-pruebas)
8. [Persistencia de datos](#8-persistencia-de-datos)
9. [Diagnóstico y logs](#9-diagnóstico-y-logs)
10. [Parada y limpieza](#10-parada-y-limpieza)
11. [Decisiones de diseño y seguridad](#11-decisiones-de-diseño-y-seguridad)

---

## 1. Descripción del proyecto

El equipo de desarrollo ha entregado una **API**, una **interfaz web** y un
**esquema de base de datos**. El equipo de despliegue (en este caso yo) debe convertir
ese software en un **servicio operativo, mantenible y documentado** ejecutable
en Docker Desktop con un único comando:

```bash
docker compose up --build

## 2. Arquitectura

### Diagrama de arquitectura
![Diagrama de arquitectura](image.png)

### Explicación

El sistema sigue una arquitectura **cliente–servidor en tres capas**,
completamente contenerizada. El **único punto de entrada** es **Nginx**, que
actúa como *reverse proxy* y servidor de contenido estático.

Los tres servicios se ejecutan en **contenedores independientes** y se
comunican a través de una **red interna de Docker** (`reservas-network`):

| Capa | Contenedor | Función | Puerto host |
|------|-----------|---------|-------------|
| Presentación | **Nginx** | Sirve el frontend y redirige `/api/*` a la API | `80` |
| Lógica | **API** | Expone los endpoints REST de reservas | —  |
| Datos | **MySQL** | Almacena los datos sobre un volumen persistente | —  |

**Claves del diseño:**

- Solo Nginx publica un puerto hacia el host → **mínima superficie de ataque**.
- La base de datos **no es accesible desde fuera** del entorno Docker.
- Los datos persisten en el **volumen nombrado `mysql_data`**, por lo que
  sobreviven a la recreación del contenedor de MySQL.

## 3. Estructura del repositorio

proyecto-reservas/
├── api/ # Código de la API (contenedor independiente)
│ ├── src/ # Código fuente de la aplicación
│ ├── Dockerfile # Imagen de la API
│ └── .dockerignore # Exclusiones del contexto de build
├── db/
│ └── init/ # Scripts SQL de inicialización
│ └── 01-schema.sql
├── docs/ # Documentación adicional (diagramas, capturas)
├── nginx/
│ ├── Dockerfile # Imagen de Nginx + SPA
│ └── default.conf # Configuración del reverse proxy
├── tests/
│ └── smoke.sh # Pruebas de humo (HTTP + endpoints)
├── .dockerignore # Exclusiones globales del contexto de build
├── .env.example # Plantilla de variables de entorno (SÍ va a Git)
├── .gitignore # Ignora .env y artefactos locales
├── compose.yaml # Orquestación de los servicios
└── README.md # Este documento

## 4. Requisitos previos

Antes de desplegar la solución, el equipo que la reciba debe disponer del
siguiente software y cumplir una serie de condiciones mínimas en el entorno.

### Software necesario

| Herramienta | Versión mínima | Comando de comprobación |
|-------------|----------------|-------------------------|
| **Docker Desktop** | 4.x (con Compose v2) | `docker --version` |
| **Docker Compose** | v2.x | `docker compose version` |
| **Git** | 2.x | `git --version` |
| **curl** | cualquiera reciente | `curl --version` |

> En Windows se recomienda trabajar dentro de **WSL 2** para evitar problemas
> de rutas y permisos con los volúmenes de Docker.

