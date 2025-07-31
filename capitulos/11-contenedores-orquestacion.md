# Capítulo 11: Contenedores y Orquestación

## 11.1 Fundamentos de Contenedores

### 11.1.1 ¿Qué es un contenedor?

Un contenedor es una unidad estándar de software que empaqueta código y todas sus dependencias para que la aplicación se ejecute de manera rápida y confiable de un entorno informático a otro. A diferencia de las máquinas virtuales, los contenedores virtualizan el sistema operativo en lugar del hardware.

**Características principales:**
- Aislamiento de procesos y recursos
- Portabilidad entre entornos
- Ligereza (comparten el kernel del host)
- Consistencia en todos los entornos (desarrollo, pruebas, producción)

**Diferencias entre contenedores y máquinas virtuales:**

| Aspecto | Contenedores | Máquinas Virtuales |
|---------|-------------|-------------------|
| Virtualización | A nivel de SO | A nivel de hardware |
| Tamaño | Megabytes | Gigabytes |
| Tiempo de inicio | Segundos | Minutos |
| Eficiencia | Alta (bajo overhead) | Menor (más overhead) |
| Aislamiento | Procesos aislados | Completo |
| Portabilidad | Alta | Limitada |

### 11.1.2 Tecnologías de contenedores

#### Docker

Docker es la tecnología de contenedores más popular que revolucionó el desarrollo y despliegue de software.

**Componentes clave de Docker:**
- **Docker Engine**: El runtime que ejecuta contenedores
- **Docker CLI**: Interfaz de línea de comandos
- **Docker Hub**: Registro público de imágenes
- **Dockerfile**: Script para construir imágenes
- **Docker Compose**: Herramienta para definir aplicaciones multi-contenedor

#### Alternativas a Docker

- **Podman**: Compatible con Docker pero sin daemon centralizado
- **containerd**: Runtime de contenedores utilizado por Kubernetes
- **LXC/LXD**: Contenedores a nivel de sistema operativo
- **rkt (Rocket)**: Alternativa enfocada en la seguridad

### 11.1.3 Conceptos fundamentales

#### Imágenes y contenedores

- **Imagen**: Plantilla de solo lectura con instrucciones para crear un contenedor
- **Contenedor**: Instancia ejecutable de una imagen

#### Capas y sistema de archivos

Las imágenes Docker utilizan un sistema de archivos por capas:
- Cada instrucción en un Dockerfile crea una capa
- Las capas son inmutables y se comparten entre imágenes
- Los contenedores añaden una capa superior de escritura

#### Registros y repositorios

- **Registro**: Servicio que almacena imágenes (ej: Docker Hub, GitHub Container Registry)
- **Repositorio**: Colección de imágenes relacionadas con diferentes tags

## 11.2 Trabajando con Docker

### 11.2.1 Instalación y configuración

**Instalación en sistemas principales:**

```bash
# Windows/Mac: Descargar Docker Desktop
# Linux:
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

**Verificación de instalación:**

```bash
docker --version
docker info
```

### 11.2.2 Comandos básicos de Docker

**Gestión de imágenes:**

```bash
# Descargar una imagen
docker pull nginx:latest

# Listar imágenes
docker images

# Eliminar una imagen
docker rmi nginx:latest

# Construir una imagen desde un Dockerfile
docker build -t miapp:1.0 .
```

**Gestión de contenedores:**

```bash
# Crear y ejecutar un contenedor
docker run -d -p 8080:80 --name miservidor nginx

# Listar contenedores en ejecución
docker ps

# Listar todos los contenedores
docker ps -a

# Detener un contenedor
docker stop miservidor

# Iniciar un contenedor detenido
docker start miservidor

# Eliminar un contenedor
docker rm miservidor
```

**Ejecutar comandos en contenedores:**

```bash
# Ejecutar un comando en un contenedor en ejecución
docker exec -it miservidor bash

# Ver logs de un contenedor
docker logs miservidor

# Seguir logs en tiempo real
docker logs -f miservidor
```

### 11.2.3 Creando Dockerfiles

Un Dockerfile es un script con instrucciones para construir una imagen Docker.

**Estructura básica:**

```dockerfile
# Imagen base
FROM node:16-alpine

# Establecer directorio de trabajo
WORKDIR /app

# Copiar archivos de dependencias
COPY package*.json ./

# Instalar dependencias
RUN npm install

# Copiar el resto del código
COPY . .

# Exponer puerto
EXPOSE 3000

# Comando a ejecutar al iniciar el contenedor
CMD ["npm", "start"]
```

**Instrucciones comunes:**

- `FROM`: Imagen base
- `WORKDIR`: Directorio de trabajo
- `COPY`/`ADD`: Copiar archivos al contenedor
- `RUN`: Ejecutar comandos durante la construcción
- `ENV`: Definir variables de entorno
- `EXPOSE`: Documentar puertos de escucha
- `VOLUME`: Montar puntos de volumen
- `CMD`/`ENTRYPOINT`: Comandos a ejecutar al iniciar

**Mejores prácticas:**

1. Utilizar imágenes oficiales y específicas (con tag)
2. Minimizar el número de capas
3. Utilizar .dockerignore para excluir archivos innecesarios
4. Ordenar comandos por frecuencia de cambio (menos frecuentes primero)
5. Usar imágenes multi-stage para reducir el tamaño final

**Ejemplo de build multi-stage:**

```dockerfile
# Etapa de build
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Etapa de producción
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 11.2.4 Docker Compose

Docker Compose permite definir y ejecutar aplicaciones multi-contenedor.

**Estructura básica de docker-compose.yml:**

```yaml
version: '3.8'

services:
  webapp:
    build: ./app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      - db
      - redis
    volumes:
      - ./app:/app
      - /app/node_modules

  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=usuario
      - POSTGRES_PASSWORD=contraseña
      - POSTGRES_DB=miapp
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:6-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

**Comandos básicos:**

```bash
# Iniciar servicios
docker-compose up -d

# Ver logs
docker-compose logs

# Detener servicios
docker-compose down

# Reconstruir servicios
docker-compose up -d --build
```

### 11.2.5 Redes en Docker

**Tipos de redes:**

1. **bridge**: Red predeterminada para contenedores
2. **host**: Elimina el aislamiento de red entre el contenedor y el host
3. **none**: Desactiva la red
4. **overlay**: Conecta múltiples daemons Docker (usado en Swarm)
5. **macvlan**: Asigna MAC a contenedores para que parezcan dispositivos físicos

**Comandos para gestionar redes:**

```bash
# Crear red
docker network create mired

# Listar redes
docker network ls

# Conectar contenedor a red
docker network connect mired micontenedor

# Desconectar contenedor
docker network disconnect mired micontenedor

# Inspeccionar red
docker network inspect mired

# Eliminar red
docker network rm mired
```

### 11.2.6 Volúmenes en Docker

Los volúmenes permiten persistir datos y compartirlos entre contenedores.

**Tipos de volúmenes:**

1. **Volúmenes nombrados**: Gestionados por Docker
2. **Volúmenes bind**: Montajes desde el host
3. **tmpfs**: Almacenamiento en memoria

**Comandos para gestionar volúmenes:**

```bash
# Crear volumen
docker volume create mivolumen

# Listar volúmenes
docker volume ls

# Inspeccionar volumen
docker volume inspect mivolumen

# Eliminar volumen
docker volume rm mivolumen

# Eliminar volúmenes no utilizados
docker volume prune
```

**Ejemplo de uso de volúmenes:**

```bash
# Volumen nombrado
docker run -v mivolumen:/data nginx

# Bind mount
docker run -v $(pwd)/config:/etc/nginx/conf.d nginx

# tmpfs
docker run --tmpfs /tmp nginx
```

## 11.3 Fundamentos de Orquestación de Contenedores

### 11.3.1 ¿Qué es la orquestación de contenedores?

La orquestación de contenedores automatiza el despliegue, gestión, escalado y redes de contenedores.

**Principales funciones:**
- Programación de contenedores en nodos
- Equilibrio de carga
- Descubrimiento de servicios
- Escalado automático
- Auto-reparación
- Actualizaciones progresivas (rolling updates)
- Gestión de secretos y configuraciones

### 11.3.2 Plataformas de orquestación

#### Kubernetes

Kubernetes (K8s) es el orquestador de contenedores más popular y completo.

**Componentes principales:**
- **Master**: Controla el cluster
  - API Server
  - Scheduler
  - Controller Manager
  - etcd (almacén de configuración)
- **Nodos**: Ejecutan los contenedores
  - Kubelet
  - Kube-proxy
  - Container Runtime (Docker/containerd)

#### Alternativas a Kubernetes

- **Docker Swarm**: Solución de orquestación integrada con Docker
- **Amazon ECS**: Servicio de contenedores gestionado de AWS
- **Azure Container Instances**: Solución serverless para contenedores
- **Nomad**: Orquestador ligero de HashiCorp

## 11.4 Kubernetes en Profundidad

### 11.4.1 Arquitectura de Kubernetes

**Plano de control (Control Plane):**
- **API Server**: Punto de entrada para las solicitudes
- **etcd**: Base de datos clave-valor para configuración
- **Scheduler**: Asigna pods a nodos
- **Controller Manager**: Gestiona controladores del sistema
- **Cloud Controller Manager**: Integración con proveedores de nube

**Plano de datos (Data Plane):**
- **Nodos**: Máquinas físicas o virtuales
- **Kubelet**: Agente que ejecuta pods
- **Kube-proxy**: Gestiona redes y reglas de enrutamiento
- **Container Runtime**: Ejecuta contenedores (Docker, containerd, etc.)

### 11.4.2 Objetos básicos de Kubernetes

#### Pods

El Pod es la unidad mínima desplegable en Kubernetes, contiene uno o más contenedores.

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:1.19
    ports:
    - containerPort: 80
```

#### ReplicaSets

Garantiza que un número específico de réplicas de un pod se estén ejecutando.

```yaml
# replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
```

#### Deployments

Proporciona actualizaciones declarativas para Pods y ReplicaSets.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "0.5"
            memory: "512Mi"
          requests:
            cpu: "0.2"
            memory: "256Mi"
```

#### Services

Expone pods como servicios de red.

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP  # Puede ser ClusterIP, NodePort, LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

#### Ingress

Gestiona el acceso externo a los servicios dentro del clúster.

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: miapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

#### ConfigMaps y Secrets

Permiten separar la configuración y secretos del código.

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgres://localhost:5432/db"
  api_host: "api.example.com"
```

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  db_password: cGFzc3dvcmQ=  # base64 encoded "password"
  api_key: c2VjcmV0a2V5      # base64 encoded "secretkey"
```

### 11.4.3 Herramientas para trabajar con Kubernetes

#### kubectl

CLI para interactuar con clústeres Kubernetes.

**Comandos básicos:**

```bash
# Crear recursos desde archivo YAML
kubectl apply -f deployment.yaml

# Listar recursos
kubectl get pods
kubectl get deployments
kubectl get services

# Describir un recurso
kubectl describe pod mi-pod

# Ver logs
kubectl logs mi-pod

# Ejecutar comandos en un pod
kubectl exec -it mi-pod -- bash

# Eliminar recursos
kubectl delete deployment web-deployment
```

#### Helm

Gestor de paquetes para Kubernetes que facilita la instalación de aplicaciones.

```bash
# Añadir repositorio
helm repo add bitnami https://charts.bitnami.com/bitnami

# Buscar charts
helm search repo nginx

# Instalar un chart
helm install mi-web bitnami/nginx

# Listar instalaciones
helm list

# Actualizar instalación
helm upgrade mi-web bitnami/nginx --values valores.yaml

# Desinstalar
helm uninstall mi-web
```

#### k9s

Interfaz terminal para gestionar clústeres Kubernetes.

```bash
# Instalar k9s
brew install k9s  # macOS
choco install k9s # Windows

# Ejecutar
k9s
```

### 11.4.4 Patrones de despliegue en Kubernetes

#### Despliegue azul-verde (Blue-Green)

Mantiene dos versiones idénticas (azul y verde) y cambia el tráfico entre ellas.

```yaml
# Servicio con selector apuntando al deployment "blue"
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
    version: blue
  ports:
  - port: 80
    targetPort: 80
```

#### Despliegue canario (Canary)

Envía un pequeño porcentaje de tráfico a la nueva versión antes del despliegue completo.

```yaml
# Implementación con Istio para despliegue canario
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: web-canary
spec:
  hosts:
  - web-service
  http:
  - route:
    - destination:
        host: web-service
        subset: v1
      weight: 80
    - destination:
        host: web-service
        subset: v2
      weight: 20
```

#### Despliegue A/B Testing

Similar al canario pero con el propósito de probar características con diferentes usuarios.

## 11.5 Despliegue en entornos de producción

### 11.5.1 Kubernetes gestionado vs auto-administrado

**Servicios gestionados:**
- Amazon EKS (Elastic Kubernetes Service)
- Google GKE (Google Kubernetes Engine)
- Azure AKS (Azure Kubernetes Service)
- DigitalOcean Kubernetes

**Ventajas de servicios gestionados:**
- Plano de control gestionado
- Actualizaciones automáticas
- Integración con servicios cloud
- Soporte

**Auto-administrado:**
- Mayor control
- Posibilidad de personalización
- Independencia del proveedor
- Potencialmente más económico

### 11.5.2 Consideraciones para producción

#### Alta disponibilidad

- **Control plane**: Múltiples réplicas
- **etcd**: Clúster con quórum (3, 5 o 7 nodos)
- **Nodos**: Distribuidos en múltiples zonas
- **Pod Disruption Budgets**: Garantizan disponibilidad durante mantenimientos

```yaml
# PDB para garantizar disponibilidad
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2  # O maxUnavailable: 1
  selector:
    matchLabels:
      app: web
```

#### Monitorización

**Herramientas populares:**
- **Prometheus**: Monitorización y alertas
- **Grafana**: Visualización de métricas
- **Loki**: Agregación de logs
- **Jaeger/Zipkin**: Trazabilidad distribuida

#### Seguridad

**Mejores prácticas:**
- Utilizar RBAC (Role-Based Access Control)
- Implementar Network Policies
- Escanear imágenes por vulnerabilidades
- Utilizar PodSecurityPolicies/PodSecurityStandards
- Cifrar secrets
- Implementar service mesh (Istio/Linkerd) para mTLS

```yaml
# Network Policy para restricción de tráfico
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 5432
```

#### Backup y recuperación

**Estrategias:**
- Backup de etcd
- Velero para copias de seguridad de recursos
- Backup de volúmenes persistentes

```bash
# Backup del cluster con Velero
velero backup create backup-1 --include-namespaces=default,web
```

### 11.5.3 Estrategias de CI/CD para contenedores

**Herramientas populares:**
- GitHub Actions
- GitLab CI
- Jenkins
- ArgoCD (GitOps)
- Flux (GitOps)

**Flujo típico de CI/CD:**
1. Construir y probar la aplicación
2. Construir y escanear imagen Docker
3. Publicar imagen a un registro
4. Actualizar manifiestos Kubernetes
5. Aplicar cambios al clúster

**Ejemplo con GitHub Actions:**

```yaml
# .github/workflows/ci-cd.yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      
      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          push: true
          tags: usuario/app:latest,usuario/app:${{ github.sha }}
      
      - name: Set up Kube config
        uses: azure/k8s-set-context@v1
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
      
      - name: Update Kubernetes resources
        run: |
          sed -i "s|image: usuario/app:.*|image: usuario/app:${{ github.sha }}|" k8s/deployment.yaml
          kubectl apply -f k8s/
```

### 11.5.4 Service Mesh

Service mesh proporciona gestión, observabilidad y seguridad para microservicios.

**Soluciones populares:**
- Istio
- Linkerd
- Consul Connect
- AWS App Mesh

**Capacidades principales:**
- Descubrimiento de servicios
- Enrutamiento avanzado
- Balanceo de carga inteligente
- Seguridad (mTLS)
- Observabilidad
- Resiliencia (Circuit breaking, retries)

**Ejemplo con Istio:**

```yaml
# Virtual Service para control de tráfico
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews-route
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

## 11.6 Patrones avanzados y optimizaciones

### 11.6.1 Microservicios con contenedores

**Principios de microservicios:**
- Servicios pequeños y enfocados
- Despliegue independiente
- Aislamiento de fallos
- Escalado individualizado

**Consideraciones para microservicios:**
- API Gateway
- Service discovery
- Comunicación (REST, gRPC, eventos)
- Consistencia de datos

### 11.6.2 Optimización de imágenes Docker

**Estrategias:**
- Usar imágenes base mínimas (alpine, distroless)
- Multi-stage builds
- Agrupar comandos RUN para reducir capas
- Limpiar caché de paquetes
- Comprimir archivos estáticos

**Ejemplo de optimización:**

```dockerfile
# Sin optimizar
FROM node:16
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]

# Optimizado
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### 11.6.3 Auto-scaling en Kubernetes

**Tipos de auto-scaling:**
- **Horizontal Pod Autoscaler (HPA)**: Escala pods
- **Vertical Pod Autoscaler (VPA)**: Ajusta recursos de pods
- **Cluster Autoscaler**: Escala nodos

```yaml
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```

### 11.6.4 Operadores de Kubernetes

Los operadores automatizan tareas complejas extendiendiendo la API de Kubernetes.

**Casos de uso comunes:**
- Bases de datos (PostgreSQL, MongoDB, etc.)
- Sistemas de mensajería (Kafka, RabbitMQ)
- Monitorización (Prometheus)
- Service mesh (Istio)

**Ejemplo: Operator para PostgreSQL (Zalando Operator):**

```yaml
apiVersion: acid.zalan.do/v1
kind: postgresql
metadata:
  name: postgres-db
spec:
  teamId: "myteam"
  volume:
    size: 10Gi
  numberOfInstances: 3
  users:
    myapp: []
  databases:
    myapp: myapp
  postgresql:
    version: "13"
```

## 11.7 Tendencias y futuro

### 11.7.1 Serverless Containers

Plataformas que permiten ejecutar contenedores sin gestionar la infraestructura:

- AWS Fargate
- Azure Container Instances
- Google Cloud Run
- Knative

### 11.7.2 WebAssembly y contenedores

WebAssembly (WASM) emerge como alternativa ligera a los contenedores:
- Más compacto
- Arranque casi instantáneo
- Ejecución segura
- Portable

### 11.7.3 eBPF y contenedores

eBPF (extended Berkeley Packet Filter) permite programar el kernel:
- Observabilidad avanzada
- Seguridad de red
- Aceleración de networking

### 11.7.4 GitOps

GitOps utiliza Git como fuente única de verdad para infraestructura:
- Declarativo
- Versionado
- Procesos de aprobación
- Auditoría

Herramientas:
- ArgoCD
- Flux
- Jenkins X

## 11.8 Conclusión

Los contenedores y su orquestación han transformado radicalmente el desarrollo y despliegue de software moderno, proporcionando consistencia, portabilidad y escalabilidad. Docker simplificó la creación y distribución de aplicaciones, mientras que Kubernetes estableció el estándar para orquestar contenedores a escala.

La adopción de estas tecnologías requiere nuevos conocimientos y prácticas, pero los beneficios en términos de eficiencia, resiliencia y agilidad son sustanciales. A medida que el ecosistema madura, las herramientas y prácticas continúan evolucionando para abordar desafíos como seguridad, observabilidad y gestión de la complejidad.

Ya sea que estés comenzando con contenedores o buscando optimizar tu infraestructura existente basada en Kubernetes, entender estos fundamentos y patrones avanzados te ayudará a aprovechar al máximo estas tecnologías transformadoras.
