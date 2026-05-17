# Tienda de Alimentos para Perritos

## Introducción

Este proyecto fue desarrollado para la Evaluación Parcial N°2 de la asignatura Introducción a Herramientas DevOps.

La solución implementa una aplicación web CRUD para la gestión de productos de una tienda de alimentos para mascotas, utilizando contenedores Docker y automatización CI/CD en AWS.

La arquitectura fue separada en tres servicios independientes:

- Frontend
- Backend
- Base de Datos

Cada componente fue dockerizado y desplegado en instancias EC2 independientes utilizando Amazon Elastic Container Registry (ECR).

---

## Objetivos del Proyecto

- Implementar contenedores Docker para cada servicio.
- Utilizar Docker Compose para integración local.
- Configurar despliegue automatizado mediante GitHub Actions.
- Utilizar Amazon ECR como registro de imágenes Docker.
- Desplegar servicios en instancias AWS EC2.
- Configurar reglas de acceso mediante Security Groups.
- Validar comunicación entre frontend, backend y base de datos.
- Implementar persistencia de datos utilizando volúmenes Docker.

---

## Arquitectura de la Solución

La arquitectura implementada considera separación de responsabilidades por servicio.

```txt
Cliente
   │
   ▼
Frontend (EC2 + Docker + Nginx)
   │
   ▼
Backend API (EC2 + Docker + Node.js)
   │
   ▼
Base de Datos (EC2 + Docker + MySQL)
```

---

## Tecnologías Utilizadas

### Infraestructura y DevOps

- Docker
- Docker Compose
- GitHub Actions
- AWS EC2
- Amazon ECR
- AWS IAM
- Security Groups

### Backend

- Node.js
- Express.js
- MySQL2

### Frontend

- HTML5
- CSS3
- JavaScript
- Nginx

### Base de Datos

- MySQL 8

---

## Estructura del Proyecto

```txt
tienda-perritos/
│
├── .github/
│   └── workflows/
│       ├── cicd-tienda-frontend.yml
│       ├── cicd-tienda-backend.yml
│       └── cicd-tienda-db.yml
│
├── frontend/
│   ├── Dockerfile
│   ├── app.js
│   ├── index.html
│   └── nginx.conf
│
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
│
├── db/
│   ├── Dockerfile
│   └── init.sql
│
├── docker-compose.yml
└── README.md
```

---

## Dockerización de Servicios

### Frontend

El frontend fue dockerizado utilizando Nginx para servir archivos estáticos.

#### Dockerfile

```dockerfile
FROM nginx:alpine

RUN rm -rf /usr/share/nginx/html/*

COPY index.html app.js /usr/share/nginx/html/

COPY nginx.conf /etc/nginx/conf.d/nginx.conf

EXPOSE 80
```

---

### Backend

El backend fue dockerizado utilizando Node.js y Express.

#### Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3001

CMD ["node", "server.js"]
```

---

### Base de Datos

La base de datos fue dockerizada utilizando MySQL 8.

#### Dockerfile

```dockerfile
FROM mysql:8

COPY init.sql /docker-entrypoint-initdb.d/
```

---

## Integración Local con Docker Compose

Para integración y pruebas locales se utilizó Docker Compose.

Los servicios integrados fueron:

- frontend
- backend
- db

Además, se configuraron:

- redes Docker
- variables de entorno
- dependencias entre servicios
- volúmenes persistentes

---

## Configuración Docker Compose

### Servicios definidos

#### Base de Datos

- Imagen MySQL personalizada.
- Persistencia mediante volumen Docker.
- Variables de entorno para configuración automática.

#### Backend

- API REST ejecutándose en Node.js.
- Conexión a la base de datos mediante variables de entorno.
- Exposición del puerto 3001.

#### Frontend

- Aplicación estática servida con Nginx.
- Comunicación hacia backend desplegado.

---

## Persistencia de Datos

La persistencia de la base de datos se implementó mediante volúmenes Docker.

```yaml
volumes:
  db_data:
```

Esto permite conservar la información aunque el contenedor sea detenido o recreado.

El volumen utilizado corresponde a un named volume administrado por Docker.

---

## Infraestructura AWS

La solución fue desplegada utilizando tres instancias EC2 independientes.

| Servicio      | Nombre de Instancia |
| ------------- | ------------------- |
| Frontend      | ep2-frontend        |
| Backend       | ep2-backend         |
| Base de Datos | ep2-db              |

### Configuración utilizada

| Recurso           | Configuración     |
| ----------------- | ----------------- |
| Tipo de instancia | t3.micro          |
| Sistema operativo | Amazon Linux 2023 |
| Región AWS        | us-east-1         |

---

## Configuración de Security Groups

### Frontend

Puertos configurados:

| Puerto | Protocolo | Descripción           |
| ------ | --------- | --------------------- |
| 80     | HTTP      | Acceso web            |
| 443    | HTTPS     | Acceso seguro         |
| 22     | SSH       | Administración remota |

---

### Backend

Puertos configurados:

| Puerto | Protocolo | Descripción           |
| ------ | --------- | --------------------- |
| 3001   | TCP       | API REST              |
| 22     | SSH       | Administración remota |

---

### Base de Datos

Puertos configurados:

| Puerto | Protocolo | Descripción           |
| ------ | --------- | --------------------- |
| 3306   | MySQL     | Acceso desde Backend  |
| 22     | SSH       | Administración remota |

El puerto 3306 fue restringido utilizando el Security Group asociado al backend.

---

## Amazon Elastic Container Registry (ECR)

Se crearon tres repositorios privados en Amazon ECR para almacenar las imágenes Docker del proyecto.

| Repositorio     |
| --------------- |
| tienda-frontend |
| tienda-backend  |
| tienda-db       |

Las imágenes fueron publicadas automáticamente utilizando GitHub Actions.

---

## Automatización CI/CD

La automatización del despliegue fue implementada mediante GitHub Actions.

### Workflows implementados

| Workflow                 |
| ------------------------ |
| cicd-tienda-frontend.yml |
| cicd-tienda-backend.yml  |
| cicd-tienda-db.yml       |

### Flujo de despliegue

```txt
Push rama deploy
        ↓
Construcción imagen Docker
        ↓
Push imagen hacia ECR
        ↓
Despliegue automático en EC2
        ↓
Actualización de contenedores
```

---

## GitHub Secrets Configurados

### Credenciales AWS

```txt
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
AWS_REGION
```

### Instancias EC2

```txt
EC2_FRONTEND_INSTANCE_ID
EC2_BACKEND_INSTANCE_ID
EC2_DB_INSTANCE_ID
```

### Repositorios ECR

```txt
ECR_REGISTRY
ECR_REPO_URL_FRONTEND
ECR_REPO_URL_BACKEND
ECR_REPO_URL_DB
```

---

## Validaciones Realizadas

### Verificación de contenedores

```bash
docker ps
```

### Verificación Backend

```bash
curl http://localhost:3001/api/health
```

### Verificación API de productos

```bash
curl http://localhost:3001/api/productos
```

### Verificación Frontend

```bash
curl http://localhost
```

---

## Endpoints Implementados

### Health Check

```http
GET /api/health
```

### Obtener productos

```http
GET /api/productos
```

### Obtener producto por ID

```http
GET /api/productos/:id
```

### Crear producto

```http
POST /api/productos
```

### Actualizar producto

```http
PUT /api/productos/:id
```

### Eliminar producto

```http
DELETE /api/productos/:id
```

---

## Ejecución Local del Proyecto

Esta sección describe los pasos necesarios para ejecutar y validar el proyecto en un entorno local utilizando Docker y Docker Compose.

### Requisitos previos

Antes de ejecutar el proyecto se debe contar con:

- Git instalado.
- Docker instalado y funcionando correctamente.
- Repositorio clonado localmente.

### Clonar repositorio

```bash
git clone https://github.com/Nachovn12/tienda-perritos.git
cd tienda-perritos
```

### Levantar servicios con Docker Compose

```bash
docker compose up --build -d
```

### Ver contenedores activos

```bash
docker ps
```

Deben visualizarse los siguientes contenedores:

```txt
tienda-frontend
tienda-backend
tienda-db
```

### Validar Frontend

Abrir en navegador:

```txt
http://localhost
```

### Validar Backend

```bash
curl http://localhost:3001/api/health
```

### Validar productos desde la API

```bash
curl http://localhost:3001/api/productos
```

### Detener servicios

```bash
docker compose down
```

### Detener servicios y eliminar volumen local

```bash
docker compose down -v
```

Nota: el comando anterior elimina también el volumen de la base de datos, por lo que se pierde la información persistida localmente.

---

## Resultados Obtenidos

Durante el desarrollo del proyecto se logró:

- Dockerizar correctamente los tres servicios.
- Implementar persistencia de datos mediante volúmenes Docker.
- Publicar imágenes Docker en Amazon ECR.
- Automatizar despliegues mediante GitHub Actions.
- Desplegar servicios en AWS EC2.
- Configurar comunicación entre frontend, backend y base de datos.
- Validar funcionamiento completo de la aplicación desplegada.

---

## Integrantes

- Ignacio Valeria
- Benjamín Flores

---

## Conclusión

Durante el desarrollo de este proyecto se logró implementar una solución basada en contenedores Docker, integrando frontend, backend y base de datos en una arquitectura funcional desplegada en AWS.

Además, se configuró un flujo automatizado de integración y despliegue continuo mediante GitHub Actions y Amazon ECR, permitiendo actualizar los servicios automáticamente en las instancias EC2 utilizadas durante la evaluación.

El proyecto permitió aplicar de forma práctica los contenidos revisados en la asignatura, especialmente en contenedorización, automatización CI/CD, despliegue cloud y configuración de servicios en entornos DevOps.
