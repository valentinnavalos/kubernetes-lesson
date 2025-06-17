# Kubernetes: Orquestación de Contenedores y Gestión de Recursos del Sistema
## Guía para Presentación - Sistemas Operativos

**Duración estimada:** 30 minutos  
**Audiencia:** Estudiantes de Sistemas Operativos  
**Enfoque:** Relación entre Kubernetes y conceptos fundamentales de SO

---

## 1. Introducción y Contexto (5 minutos)

### 1.1 ¿Qué son los Contenedores?
Los contenedores son **procesos aislados** que comparten el kernel del sistema operativo host, a diferencia de las máquinas virtuales que virtualizan hardware completo.

**Diferencias clave:**
- **Contenedores:** Aislamiento a nivel de proceso (misma instancia de kernel)
- **VMs:** Aislamiento a nivel de hardware (kernel independiente por VM)

### 1.2 El Problema que Resuelve Kubernetes
- **Gestión manual:** Administrar cientos/miles de contenedores manualmente es impracticable
- **Aislamiento de recursos:** Necesidad de controlar CPU, memoria, I/O por aplicación
- **Distribución:** Ejecutar aplicaciones en múltiples nodos de forma coordinada

### 1.3 Conexión con Sistemas Operativos
Kubernetes utiliza **primitivas del kernel Linux** para:
- **Namespaces:** Aislamiento de vistas del sistema
- **Cgroups:** Control y limitación de recursos

---

## 2. Generalidades de Kubernetes (8 minutos)

### 2.1 ¿Qué es Kubernetes?
Kubernetes es un **orquestador de contenedores** que automatiza:
- Despliegue de aplicaciones
- Escalado automático
- Gestión del estado deseado
- Distribución de carga

### 2.2 Objetos Fundamentales
- **Pod:** Unidad mínima de despliegue (uno o más contenedores que comparten recursos)
- **Service:** Abstracción de red para acceder a pods de forma estable
- **Deployment:** Gestiona el ciclo de vida de pods (actualizaciones, rollbacks, escalado)
- **ConfigMap/Secret:** Gestión de configuración y datos sensibles
- **Namespace:** Aislamiento lógico de recursos dentro del cluster

### 2.3 Arquitectura Principal

#### Plano de Control (Control Plane)
El **Control Plane** es el "cerebro" del cluster que toma decisiones globales y detecta y responde a eventos del cluster.

**Componentes principales:**
- **API Server (kube-apiserver):** 
  - Punto de entrada único para todas las operaciones REST
  - Valida y configura datos para objetos API (pods, services, etc.)
  - Sirve como frontend del estado compartido del cluster
  
- **etcd:** 
  - Base de datos distribuida clave-valor para todo el estado del cluster
  - Único punto de verdad para la configuración y estado
  - Altamente disponible y consistente
  
- **Scheduler (kube-scheduler):** 
  - Decide en qué nodo ejecutar pods sin asignar
  - Considera recursos requeridos, políticas, afinidad/anti-afinidad
  - No ejecuta pods, solo toma decisiones de placement
  
- **Controller Manager (kube-controller-manager):** 
  - Ejecuta controladores que mantienen el estado deseado
  - Deployment Controller, ReplicaSet Controller, Service Controller, etc.
  - Detecta diferencias entre estado actual y deseado

#### Nodos Worker
Los **Worker Nodes** ejecutan las aplicaciones containerizadas y proporcionan el entorno de ejecución.

**Componentes principales:**
- **Kubelet:** 
  - Agente principal que se ejecuta en cada nodo
  - Comunica con el API Server para recibir especificaciones de pods
  - Gestiona el ciclo de vida de contenedores usando el container runtime
  - Reporta estado del nodo y pods al Control Plane
  
- **Container Runtime:** 
  - Software responsable de ejecutar contenedores (Docker, containerd, CRI-O)
  - Implementa la Container Runtime Interface (CRI)
  - Maneja imágenes, creación/destrucción de contenedores
  
- **kube-proxy:** 
  - Proxy de red que mantiene reglas de red en los nodos
  - Implementa el concepto de Service de Kubernetes
  - Maneja forwarding de tráfico y load balancing

---

## 3. Namespaces: Linux vs Kubernetes (8 minutos)

### 3.1 ¿Qué son los Namespaces? - Aclaración Conceptual

**IMPORTANTE:** Existen **DOS tipos diferentes** de namespaces que no deben confundirse:
1. **Namespaces de Linux (Kernel-level):** Mecanismo del SO para aislamiento real
2. **Namespaces de Kubernetes (Logical):** Organización lógica de recursos del cluster

### 3.2 Namespaces de Linux (Kernel-level)
Los namespaces de Linux son una característica del **kernel** que **aísla vistas del sistema** a nivel de proceso:

#### Tipos de Namespaces Linux:
- **PID Namespace:** Aislamiento de IDs de procesos
- **Network Namespace:** Aislamiento de interfaces de red
- **Mount Namespace:** Aislamiento de puntos de montaje
- **UTS Namespace:** Aislamiento de hostname
- **User Namespace:** Aislamiento de UIDs/GIDs
- **IPC Namespace:** Aislamiento de comunicación entre procesos

#### Características Fundamentales:
Los namespaces de Linux son **tablas de mapeo en el kernel** que proporcionan vistas aisladas de recursos globales del sistema.

**Creación y Gestión:**
- **System calls:** `clone()`, `unshare()`, `setns()`
- **Herramientas:** `ip netns`, `unshare`, etc.
- **Integración:** Los contenedores utilizan estos namespaces automáticamente

### 3.4 Namespaces de Kubernetes (Logical-level)
Los namespaces de Kubernetes son **espacios lógicos** que dividen recursos del cluster dentro de la plataforma de orquestación:

#### Diferencias Clave con Namespaces de Linux:
| Aspecto | Namespaces Linux | Namespaces Kubernetes |
|---------|------------------|----------------------|
| **Nivel** | Kernel del SO | Plataforma de orquestación |
| **Aislamiento** | Recursos del sistema (procesos, red, archivos) | Recursos del cluster (pods, services) |
| **Granularidad** | Procesos individuales | Grupos de aplicaciones |
| **Gestión** | System calls (clone, unshare) | API REST de Kubernetes |
| **Persistencia** | Mientras existan procesos | En el estado del cluster (etcd) |

#### Relación entre Ambos:
- **Kubernetes NO crea namespaces de Linux directamente**
- **Cada contenedor** se ejecuta en sus propios namespaces de Linux
- **Un namespace de Kubernetes** puede contener múltiples pods
- **Cada pod** puede tener múltiples contenedores con namespaces Linux independientes

```
Namespace K8s: "produccion"
├── Pod 1
│   ├── Contenedor A → Namespaces Linux (PID, Net, Mount...)
│   └── Contenedor B → Namespaces Linux (PID, Net, Mount...)
└── Pod 2
    └── Contenedor C → Namespaces Linux (PID, Net, Mount...)
```

#### Características:
- **Aislamiento de nombres:** Los recursos deben ser únicos dentro del namespace
- **Cuotas de recursos:** Limitar CPU/memoria por namespace
- **Políticas de red:** Controlar comunicación entre namespaces
- **Control de acceso:** RBAC por namespace

#### Namespaces por Defecto:
```bash
default          # Namespace por defecto para objetos sin especificar
kube-system      # Objetos creados por el sistema Kubernetes
kube-public      # Legible por todos los usuarios
kube-node-lease  # Objetos de lease asociados a cada nodo
```

### 3.5 Ejemplo Interactivo - Comparación Práctica

#### Namespaces de Linux en Acción:
```bash
# Ver namespaces de un proceso
lsns -p $$
# Output:
#        NS TYPE   NPROCS   PID USER COMMAND
# 4026531836 pid        1  1234 user bash
# 4026531837 net        1  1234 user bash

# Crear nuevo network namespace
sudo ip netns add test-namespace
sudo ip netns exec test-namespace bash
ip addr show  # Solo ve loopback, aislado de la red del host
```

#### Namespaces de Kubernetes en Acción:

```bash
# Listar namespaces de Kubernetes (lógicos)
kubectl get namespaces

# Crear un nuevo namespace de Kubernetes
kubectl create namespace desarrollo

# Crear un pod en el namespace específico
kubectl run nginx --image=nginx --namespace=desarrollo

# Listar pods en el namespace
kubectl get pods --namespace=desarrollo

# Configurar namespace por defecto para el contexto actual
kubectl config set-context --current --namespace=desarrollo

# Verificar que cada contenedor tiene sus propios namespaces de Linux
kubectl exec -it nginx -- lsns
# Cada contenedor tendrá diferentes IDs de namespaces Linux
```

#### Demostración de la Relación:
```bash
# 1. Crear pod en namespace K8s
kubectl create namespace demo
kubectl run test-pod --image=busybox --namespace=demo -- sleep 3600

# 2. Ver namespaces Linux del contenedor
kubectl exec -n demo test-pod -- lsns
# Muestra namespaces Linux únicos para este contenedor

# 3. Comparar con otro pod en el mismo namespace K8s
kubectl run test-pod2 --image=busybox --namespace=demo -- sleep 3600
kubectl exec -n demo test-pod2 -- lsns
# Namespaces Linux DIFERENTES aunque estén en el mismo namespace K8s
```

#### Aislamiento de Recursos:
```yaml
# resource-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: desarrollo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "4"
```

---

## 4. Cgroups: Linux vs Kubernetes (8 minutos)

### 4.1 ¿Qué son los Cgroups? - Aclaración Conceptual

**IMPORTANTE:** Al igual que con los namespaces, los cgroups son una **característica del kernel Linux**, NO de Kubernetes.

- **Cgroups de Linux (Kernel-level):** Mecanismo del SO para control de recursos
- **Uso en Kubernetes:** Kubernetes utiliza cgroups para gestionar recursos de pods/contenedores

### 4.2 Cgroups de Linux (Kernel-level)
Los **Control Groups** son una característica del kernel Linux que permite:
- **Limitar recursos:** CPU, memoria, I/O, network
- **Priorizar procesos:** Asignar más recursos a procesos críticos
- **Contabilizar uso:** Monitorear consumo de recursos
- **Aislar procesos:** Prevenir que un proceso afecte a otros

#### Funcionamiento a Bajo Nivel de Cgroups

**Arquitectura Fundamental:**
Los cgroups forman una **jerarquía de directorios** en el filesystem virtual `/sys/fs/cgroup/`. Cada directorio representa un cgroup que puede contener procesos y tener límites de recursos aplicados.

**Gestión de Procesos:**
```bash
# Cada proceso pertenece a un cgroup
cat /proc/self/cgroup
# Muestra: 12:memory:/user.slice/user-1000.slice
#          11:cpu:/user.slice

# Mover proceso a cgroup específico
echo $PID > /sys/fs/cgroup/cpu/mi-grupo/cgroup.procs
```

### 4.3 Jerarquía de Cgroups en el Sistema

#### Sistema Linux Normal:
```
/sys/fs/cgroup/
├── cpu/                    # Control de CPU
│   ├── mi-aplicacion/      # Grupo personalizado
│   │   ├── cpu.cfs_quota_us
│   │   └── tasks           # PIDs en este grupo
│   ├── cpu.cfs_period_us   # Período de scheduling (microsegundos)
│   ├── cpu.shares          # Peso relativo (1024 = peso estándar)
│   └── tasks               # PIDs en el grupo raíz
├── memory/                 # Control de memoria
│   ├── mi-aplicacion/
│   ├── memory.limit_in_bytes    # Límite hard de memoria
│   ├── memory.usage_in_bytes    # Uso actual
│   └── memory.oom_control       # Control del OOM killer
└── devices/                # Control de acceso a dispositivos
    └── devices.list        # Lista de dispositivos permitidos
```

#### Sistema con Kubernetes:
```
/sys/fs/cgroup/
├── cpu/
│   └── kubepods/           # Todos los pods de Kubernetes
│       ├── burstable/      # Pods con límites flexibles
│       │   └── pod<uid>/   # Pod específico
│       │       └── <container-id>/  # Contenedor específico
│       ├── besteffort/     # Pods sin límites
│       └── guaranteed/     # Pods con límites estrictos
├── memory/
│   └── kubepods/           # Misma estructura jerárquica
│       ├── burstable/
│       ├── besteffort/
│       └── guaranteed/
└── devices/
    └── kubepods/
```

#### ¿Cómo se Agrupan los Recursos?

**Por contenedor individual** con jerarquía organizativa:

```
Ejemplo: Pod "web-app" con nginx + backend

/sys/fs/cgroup/memory/kubepods/guaranteed/pod-abc123/
├── container-nginx/
│   ├── memory.limit_in_bytes → 128MB
│   ├── memory.usage_in_bytes → 45MB
│   └── tasks → [PID proceso nginx]
└── container-backend/
    ├── memory.limit_in_bytes → 256MB
    ├── memory.usage_in_bytes → 180MB
    └── tasks → [PID proceso backend]
```

**Niveles de agrupación:**
1. **Contenedor individual:** Cada contenedor = 1 cgroup
2. **Pod:** Agrupa contenedores relacionados
3. **QoS Class:** Agrupa pods según garantías de recursos
4. **Kubernetes global:** Todos los pods bajo `/kubepods/`

**Subsistemas y su Funcionamiento:**

**CPU Subsystem:**
```bash
# Configuración de límites de CPU
echo 100000 > cpu.cfs_period_us    # Período de 100ms
echo 50000 > cpu.cfs_quota_us      # 50% de un core (50ms de cada 100ms)

# Algoritmo CFS (Completely Fair Scheduler)
# quota_per_period = cpu.cfs_quota_us / cpu.cfs_period_us
# throttling_occurs_when: usage > quota_per_period
```

- El kernel utiliza el **Completely Fair Scheduler (CFS)**
- Cuando un proceso excede su cuota, es **throttled** (pausado)
- El scheduler redistribuye tiempo entre cgroups según sus `cpu.shares`

**Memory Subsystem:**
```bash
# Control de memoria
echo 536870912 > memory.limit_in_bytes  # 512MB límite
cat memory.usage_in_bytes               # Ver uso actual
cat memory.stat                         # Estadísticas detalladas
```

- El kernel mantiene contadores por página de memoria
- Cada página está asociada a un memory cgroup
- Al superar límite → **OOM killer** activado
- Páginas pueden ser swappeadas si hay swap disponible

**Block I/O Subsystem:**
```bash
# Limitar velocidad de lectura a 10MB/s en dispositivo 8:0
echo "8:0 10485760" > blkio.throttle.read_bps_device

# Peso relativo para I/O (100-1000, defecto 500)
echo 750 > blkio.weight
```

- Control granular de bandwidth por dispositivo
- Throttling a nivel de bloque del kernel
- Funciona interceptando llamadas al subsistema de bloques

### 4.3 Cgroups v1 vs v2

#### Cgroups v1:
- **Múltiples jerarquías independientes:** Cada subsistema puede tener su propia jerarquía
- **Un subsistema por jerarquía:** CPU y memory pueden estar en árboles diferentes
- **Más complejo de administrar:** Difícil mantener consistencia entre jerarquías

```
# v1: Estructura separada
/sys/fs/cgroup/cpu/app1/         # Jerarquía CPU
/sys/fs/cgroup/memory/app1/      # Jerarquía Memory
/sys/fs/cgroup/blkio/app1/       # Jerarquía Block I/O
```

#### Cgroups v2:
- **Jerarquía unificada:** Todos los controladores en un solo árbol
- **Mejor delegación de permisos:** Subdirectorios pueden ser delegados de forma segura
- **Mejor contabilidad de memoria:** Incluye memoria de kernel, buffers, cache

```
# v2: Estructura unificada
/sys/fs/cgroup/app1/
├── cpu.max                      # Límites de CPU
├── memory.max                   # Límites de memoria
├── io.max                       # Límites de I/O
└── cgroup.controllers           # Controladores disponibles
```

**Delegación Segura:**
```bash
# v2 permite delegar subárboles completos
chown user:group /sys/fs/cgroup/user-apps/
# El usuario puede crear subcgroups sin privilegios root
```

**Transición y Coexistencia:**
- Muchos sistemas ejecutan ambas versiones simultáneamente
- Cgroups v2 montado en `/sys/fs/cgroup/unified/`
- Kubernetes soporta ambas versiones, prefiere v2 cuando está disponible

### 4.6 Relación con el Sistema Operativo

#### Integración con el Kernel:
Los cgroups están **integrados directamente en el scheduler del kernel**:

1. **Kernel Scheduler:** Respeta automáticamente límites de CPU por cgroup
2. **Memory Manager:** Aplica límites de memoria y activa OOM killer
3. **Block I/O Scheduler:** Controla acceso a disco por cgroup
4. **Network Stack:** Puede limitar ancho de banda por cgroup

#### Flujo de Aplicación de Límites:
```
Pod YAML → Kubernetes API → kubelet → Container Runtime → cgroups → Kernel → Enforcement
```

**Ejemplo del flujo:**
1. Defines `limits.memory: 128Mi` en pod YAML
2. Kubernetes API valida y almacena la configuración
3. kubelet recibe la especificación del pod
4. Container runtime (Docker/containerd) crea el contenedor
5. Runtime escribe `134217728` en `/sys/fs/cgroup/memory/.../memory.limit_in_bytes`
6. Kernel automáticamente enforza el límite para ese contenedor

### 4.7 Kubernetes y Cgroups

#### Diferencias Clave:
| Aspecto | Cgroups Linux | Uso en Kubernetes |
|---------|---------------|-------------------|
| **Nivel** | Kernel del SO | Orquestador de contenedores |
| **Gestión** | Manual o scripts | Automática vía API |
| **Organización** | Libre (cualquier jerarquía) | Estructura fija (/kubepods/...) |
| **Scope** | Cualquier proceso | Solo contenedores de pods |

#### Recursos que Kubernetes Gestiona mediante Cgroups:
1. **CPU:**
   - `requests.cpu`: Garantía mínima
   - `limits.cpu`: Límite máximo
   
2. **Memoria:**
   - `requests.memory`: Garantía mínima
   - `limits.memory`: Límite máximo (OOM si se excede)

3. **Almacenamiento:**
   - `ephemeral-storage`: Almacenamiento temporal

#### Aplicación de Límites:
- **CPU:** Throttling (el proceso se pausa)
- **Memoria:** OOM Kill (el proceso se termina)
- **I/O:** Limitación de ancho de banda

### 4.8 Ejemplo Interactivo - Comparación Práctica

#### Cgroups de Linux en Acción:
```bash
# Crear un cgroup manualmente
sudo mkdir /sys/fs/cgroup/memory/mi-app
echo 67108864 | sudo tee /sys/fs/cgroup/memory/mi-app/memory.limit_in_bytes  # 64MB

# Ejecutar proceso en el cgroup
echo $$ | sudo tee /sys/fs/cgroup/memory/mi-app/cgroup.procs
# Ahora este shell tiene límite de 64MB

# Ver uso actual
cat /sys/fs/cgroup/memory/mi-app/memory.usage_in_bytes
```

#### Cgroups en Kubernetes:

#### Definir Recursos en un Pod:
```yaml
# pod-with-resources.yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"      # 0.25 cores
      limits:
        memory: "128Mi"
        cpu: "500m"      # 0.5 cores
```

#### Monitorear Cgroups creados por Kubernetes:
```bash
# Aplicar el pod
kubectl apply -f pod-with-resources.yaml

# Ver recursos asignados
kubectl describe pod resource-demo

# En el nodo, ver cgroups automáticamente creados por Kubernetes
# Encontrar el container ID
crictl ps | grep resource-demo

# Ver la jerarquía que Kubernetes creó automáticamente
ls /sys/fs/cgroup/memory/kubepods/guaranteed/

# Ver cgroups específicos del contenedor
cat /sys/fs/cgroup/memory/kubepods/pod<pod-uid>/<container-id>/memory.limit_in_bytes
cat /sys/fs/cgroup/cpu/kubepods/pod<pod-uid>/<container-id>/cpu.cfs_quota_us

# Comparar: Kubernetes creó esto automáticamente, 
# vs crear cgroups manualmente como en el ejemplo anterior
```

#### Demostrar la Diferencia:
```bash
# 1. Sistema sin Kubernetes: Control manual
sudo mkdir /sys/fs/cgroup/memory/mi-aplicacion
echo 134217728 > /sys/fs/cgroup/memory/mi-aplicacion/memory.limit_in_bytes

# 2. Sistema con Kubernetes: Control automático
kubectl run test --image=nginx --limits=memory=128Mi
# Kubernetes automáticamente crea:
# /sys/fs/cgroup/memory/kubepods/guaranteed/pod-xxx/container-xxx/
# y configura memory.limit_in_bytes = 134217728
```

#### Demostrar Límites de Memoria:
```yaml
# stress-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-stress
spec:
  containers:
  - name: stress
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
    resources:
      limits:
        memory: "100Mi"
```

```bash
# Aplicar y observar OOM Kill
kubectl apply -f stress-test.yaml
kubectl describe pod memory-stress
# Debería mostrar: reason: OOMKilled
```

---

## 5. Cierre y Conclusiones (5 minutos)

### 5.1 Relación con Sistemas Operativos
Kubernetes demuestra cómo conceptos fundamentales de SO se escalan:

1. **Aislamiento:** Namespaces para múltiples aplicaciones
2. **Gestión de recursos:** Cgroups para control fino
3. **Scheduling:** Distribuir cargas de trabajo optimalmente
4. **Monitoreo:** Observabilidad del sistema distribuido

### 5.2 Beneficios del Enfoque de Kubernetes
- **Abstracción:** Los desarrolladores se enfocan en la aplicación
- **Eficiencia:** Mejor utilización de recursos del hardware
- **Escalabilidad:** Gestión de miles de contenedores
- **Resiliencia:** Recuperación automática ante fallos

### 5.3 Casos de Uso Reales
- **Microservicios:** Cada servicio en su propio espacio aislado
- **Entornos multi-tenant:** Aislamiento entre equipos/clientes
- **CI/CD:** Entornos efímeros para testing
- **Machine Learning:** Gestión de recursos para entrenamientos

---

## Recursos y Referencias

### Documentación Oficial:
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Linux Namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)
- [Linux Cgroups](https://www.kernel.org/doc/Documentation/cgroup-v2.txt)


### Comandos de Referencia Rápida:
```bash
# Gestión de namespaces
kubectl get namespaces
kubectl create namespace <name>
kubectl config set-context --current --namespace=<name>

# Gestión de recursos
kubectl describe node <node-name>
kubectl top nodes
kubectl top pods

# Debugging de cgroups
kubectl describe pod <pod-name>
kubectl exec -it <pod-name> -- /bin/bash
``` 