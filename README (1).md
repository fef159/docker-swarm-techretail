# 🐳 TechRetail — Docker Swarm Deployment

> Trabajo 1: Orquestación de contenedores con Docker Swarm  
> Entorno: WSL2 + Docker Desktop 29.4.1

---

## 📋 Descripción

Implementación de un clúster **Docker Swarm** de 3 nodos (1 Manager + 2 Workers) para la empresa ficticia **TechRetail**, una plataforma de comercio electrónico que requiere alta disponibilidad y escalabilidad.

Se desplegaron 5 microservicios orquestados con Docker Swarm usando contenedores Docker-in-Docker (DinD) sobre WSL2.

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────┐
│           Docker Swarm Cluster          │
│                                         │
│  ┌──────────────┐                       │
│  │   MANAGER    │ ← Leader (172.18.0.2) │
│  │  database    │                       │
│  │  frontend    │                       │
│  │  visualizer  │                       │
│  │  backend     │                       │
│  └──────────────┘                       │
│                                         │
│  ┌──────────────┐  ┌──────────────┐     │
│  │   WORKER 1   │  │   WORKER 2   │     │
│  │  frontend x2 │  │  frontend x2 │     │
│  │  backend     │  │  cache       │     │
│  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────┘
```

---

## 🚀 Servicios

| Servicio | Imagen | Réplicas | Puerto |
|---|---|---|---|
| frontend | nginx:alpine | 5 | 80 |
| backend | node:18-alpine | 2 | - |
| database | mysql:8 | 1 | - |
| cache | redis:7-alpine | 1 | - |
| visualizer | dockersamples/visualizer | 1 | 8080 |

---

## ⚙️ Requisitos

- Docker Desktop 29.x o superior
- WSL2 (Ubuntu) o Linux
- 4GB RAM mínimo

---

## 🛠️ Instalación y Despliegue

### 1. Crear la red y los nodos simulados

```bash
# Red para comunicación entre nodos
docker network create --driver bridge swarm-net

# Nodo Manager
docker run -d --privileged \
  --name manager \
  --network swarm-net \
  --hostname manager \
  -p 8080:8080 -p 80:80 \
  docker:dind

# Worker 1
docker run -d --privileged \
  --name worker1 \
  --network swarm-net \
  --hostname worker1 \
  docker:dind

# Worker 2
docker run -d --privileged \
  --name worker2 \
  --network swarm-net \
  --hostname worker2 \
  docker:dind
```

### 2. Inicializar el Swarm

```bash
# Entrar al manager
docker exec -it manager sh

# Obtener la IP del manager
ip addr show eth0 | grep "inet " | awk '{print $2}' | cut -d/ -f1

# Inicializar el Swarm (reemplaza con tu IP)
docker swarm init --advertise-addr 172.18.0.2
```

### 3. Unir los Workers al Swarm

```bash
# En worker1 y worker2 (usar el token generado en el paso anterior)
docker exec -it worker1 sh
docker swarm join --token SWMTKN-1-xxxx 172.18.0.2:2377

docker exec -it worker2 sh
docker swarm join --token SWMTKN-1-xxxx 172.18.0.2:2377
```

### 4. Verificar el clúster

```bash
# En el manager
docker node ls
```

### 5. Crear el Docker Secret

```bash
echo "MiPasswordSegura123" | docker secret create db_password -
docker secret ls
```

### 6. Desplegar el stack

```bash
# Copiar el compose al manager
docker cp docker-compose.yml manager:/docker-compose.yml

# Desplegar
docker exec -it manager sh
docker stack deploy -c docker-compose.yml techretail

# Verificar servicios
docker stack services techretail
```

---

## 📈 Escalado Dinámico

```bash
# Escalar el frontend a 5 réplicas
docker service scale techretail_frontend=5

# Verificar el escalado
docker stack services techretail
```

---

## 🌐 Acceso a los servicios

| Servicio | URL |
|---|---|
| Frontend (nginx) | http://localhost |
| Visualizer | http://localhost:8080 |

---

## 🔐 Seguridad

Las credenciales de la base de datos se gestionan mediante **Docker Secrets**, evitando exponer contraseñas en variables de entorno o archivos de configuración:

```bash
echo "MiPasswordSegura123" | docker secret create db_password -
```

El secret se monta automáticamente en `/run/secrets/db_password` dentro de los contenedores autorizados.

---

## 🗂️ Estructura del repositorio

```
.
├── docker-compose.yml      # Definición de servicios del stack
├── README.md               # Este archivo
└── informe/
    └── Informe_TechRetail_DockerSwarm.pdf
```

---

## ♻️ Gestión del clúster

```bash
# Detener los nodos (sin eliminar)
docker stop manager worker1 worker2

# Reiniciar los nodos
docker start manager worker1 worker2

# Eliminar el stack
docker exec -it manager sh
docker stack rm techretail

# Eliminar todo
docker rm -f manager worker1 worker2
docker network rm swarm-net
```

---

## 📊 Criterios de Evaluación Cumplidos

| Criterio | Puntaje | Estado |
|---|---|---|
| Configuración correcta del clúster Swarm | 20% | ✅ |
| Despliegue funcional de todos los servicios | 25% | ✅ |
| Implementación de réplicas y escalado | 20% | ✅ |
| Uso correcto de Secrets y Configs | 10% | ✅ |
| Informe técnico y documentación | 15% | ✅ |
| Video demostrativo | 10% | ✅ |

---

## 👤 Autor

Trabajo desarrollado como parte del curso de Computación en la Nube / DevOps.
