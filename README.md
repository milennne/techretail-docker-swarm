# 🐳 TechRetail – Docker Swarm Deployment

> Despliegue de microservicios con orquestación Docker Swarm sobre AWS EC2

---

## 📋 Descripción

Este repositorio contiene la implementación de un clúster **Docker Swarm** para TechRetail, una empresa peruana de comercio electrónico. La arquitectura migra de un servidor monolítico a microservicios contenerizados con alta disponibilidad y escalabilidad horizontal.

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────┐
│              DOCKER SWARM CLUSTER (AWS EC2)         │
│                                                     │
│  ┌───────────────┐  ┌───────────┐  ┌───────────┐   │
│  │    Manager    │  │ Worker 1  │  │ Worker 2  │   │
│  │172.31.40.178  │  │172.31.37.23│ │172.31.43.14│  │
│  └──────┬────────┘  └─────┬─────┘  └─────┬─────┘   │
│         │                 │              │          │
│  ┌──────▼─────────────────▼──────────────▼──────┐   │
│  │          Red Overlay (techretail_net)        │   │
│  └──────┬──────────┬──────────┬──────────┬──────┘   │
│         │          │          │          │          │
│   [frontend]  [backend]  [database]  [cache]        │
│   5 réplicas  2 réplicas  1 réplica   1 réplica     │
└─────────────────────────────────────────────────────┘
```

---

## 🛠️ Servicios

| Servicio | Imagen | Réplicas | Puerto |
|----------|--------|----------|--------|
| Frontend | `nginx:alpine` | 5 | 80 |
| Backend | `node:18-alpine` | 2 | 3000 |
| Database | `mysql:8` | 1 | - |
| Cache | `redis:7-alpine` | 1 | - |
| Visualizer | `dockersamples/visualizer` | 1 | 8080 |

---

## ☁️ Infraestructura AWS

- **Región:** us-east-2 (Ohio)
- **Tipo de instancia:** t3.micro (capa gratuita)
- **OS:** Ubuntu 26.04 LTS
- **Docker:** v29.4.2

| Nodo | Rol | IP Privada | IP Pública |
|------|-----|-----------|-----------|
| ip-172-31-40-178 | Manager (Leader) | 172.31.40.178 | 18.118.49.196 |
| ip-172-31-37-23 | Worker 1 | 172.31.37.23 | 3.20.238.191 |
| ip-172-31-43-14 | Worker 2 | 172.31.43.14 | 3.17.203.104 |

---

## 🚀 Instrucciones de Despliegue

### 1. Prerrequisitos

- 3 instancias EC2 con Ubuntu y Docker instalado
- Puertos abiertos: 22, 80, 2377, 7946 (TCP/UDP), 4789 (UDP), 8080

### 2. Instalar Docker en cada nodo

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
newgrp docker
```

### 3. Inicializar el clúster Swarm (en el Manager)

```bash
docker swarm init --advertise-addr <IP_PRIVADA_MANAGER>
```

### 4. Unir Workers al clúster

```bash
# Ejecutar en cada Worker con el token generado
docker swarm join --token <TOKEN> <IP_MANAGER>:2377
```

### 5. Verificar nodos

```bash
docker node ls
```

### 6. Crear el Secret

```bash
echo "MiPasswordSegura123" | docker secret create db_password -
```

### 7. Desplegar el Stack

```bash
docker stack deploy -c docker-compose.yml techretail
```

### 8. Verificar servicios

```bash
docker stack services techretail
```

### 9. Escalar el frontend

```bash
docker service scale techretail_frontend=5
```

---

## 📊 Comandos de Monitoreo

```bash
# Ver todos los nodos
docker node ls

# Ver servicios del stack
docker stack services techretail

# Ver réplicas del frontend
docker service ps techretail_frontend

# Ver logs del backend
docker service logs techretail_backend

# Ver secrets
docker secret ls
```

---

## 🌐 Acceso

- **Frontend:** http://18.118.49.196
- **Visualizador:** http://18.118.49.196:8080

---

## 📁 Estructura del Repositorio

```
techretail-swarm/
├── docker-compose.yml      # Stack de servicios
├── README.md               # Este archivo
└── capturas/
    ├── cluster-nodos.png
    ├── servicios-desplegados.png
    ├── visualizador.png
    └── escalado-frontend.png
```

---

## 🔐 Seguridad

- Credenciales de BD gestionadas con **Docker Secrets**
- Red overlay con cifrado automático entre nodos
- Grupos de seguridad AWS configurados con mínimos puertos necesarios

---

## 📝 Licencia

Proyecto académico – TECSUP 2026
