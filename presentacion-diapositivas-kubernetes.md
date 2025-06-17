# Kubernetes: Orquestación de Contenedores y Gestión de Recursos
## Guía de Diapositivas para Presentación (30 minutos)

---

## DIAPOSITIVA 1: Portada
**Título:** 
- Kubernetes: Orquestación de Contenedores y Gestión de Recursos del Sistema

**Subtítulo:**
- Relación con Sistemas Operativos

**Ilustración sugerida:**
- Logo de Kubernetes + Logo de Linux
- Iconos de contenedores conectados

**Presentadores:** [Nombres]  
**Materia:** Sistemas Operativos  
**Duración:** 30 minutos

---

## DIAPOSITIVA 2: Agenda
**Título:** ¿Qué veremos hoy?

**Puntos clave:**
- Contenedores vs VMs
- ¿Qué es Kubernetes?
- Namespaces: Linux vs Kubernetes  
- Cgroups: Linux vs Kubernetes
- Relación con Sistemas Operativos

**Ilustración sugerida:**
- Timeline o roadmap visual con iconos

---

## DIAPOSITIVA 3: Contenedores vs Máquinas Virtuales
**Título:** El Punto de Partida

**Puntos clave:**
- Contenedores = Procesos aislados
- Comparten kernel del host
- VMs = Hardware virtualizado

**Ilustración sugerida:**
- Diagrama lado a lado:
  - VM: Host → Hypervisor → Guest OS → App
  - Container: Host → Container Runtime → App
- Mostrar que containers comparten kernel

---

## DIAPOSITIVA 4: El Problema sin Kubernetes
**Título:** ¿Por qué necesitamos un orquestador?

**Puntos clave:**
- Gestión manual de miles de contenedores
- Control de recursos (CPU, memoria)
- Distribución en múltiples servidores
- Recuperación ante fallos

**Ilustración sugerida:**
- Caos: múltiples servidores con contenedores desordenados
- Flechas mostrando problemas de gestión

---

## DIAPOSITIVA 5: ¿Qué es Kubernetes?
**Título:** El Orquestador de Contenedores

**Puntos clave:**
- Automatiza despliegue
- Escalado automático
- Gestiona estado deseado
- Distribuye carga

**Ilustración sugerida:**
- Kubernetes como "director de orquesta"
- Contenedores organizados y gestionados

---

## DIAPOSITIVA 6: Arquitectura de Kubernetes
**Título:** Control Plane vs Worker Nodes

**Puntos clave:**
**Control Plane:**
- API Server (entrada única)
- etcd (estado del cluster)
- Scheduler (decide dónde ejecutar)
- Controller Manager (mantiene estado)

**Worker Nodes:**
- kubelet (agente)
- Container Runtime (ejecuta contenedores)
- kube-proxy (red)

**Ilustración sugerida:**
- Diagrama de arquitectura con Control Plane arriba y Worker Nodes abajo
- Flechas mostrando comunicación

---

## DIAPOSITIVA 7: Objetos Fundamentales
**Título:** Los Bloques Básicos

**Puntos clave:**
- **Pod:** Unidad mínima (1+ contenedores)
- **Service:** Acceso estable a pods
- **Deployment:** Gestiona ciclo de vida
- **Namespace:** Aislamiento lógico

**Ilustración sugerida:**
- Iconos representando cada objeto
- Pod conteniendo contenedores
- Service apuntando a múltiples pods

---

## DIAPOSITIVA 8: CONCEPTO CLAVE - Dos Tipos de Namespaces
**Título:** ⚠️ Aclaración Importante

**Puntos clave:**
- **Namespaces de Linux** → Kernel del SO
- **Namespaces de Kubernetes** → Organización lógica
- ¡SON DIFERENTES!
- No confundir

**Ilustración sugerida:**
- Diagrama con dos columnas claramente separadas
- Kernel de Linux vs Plataforma Kubernetes
- Iconos distintivos para cada tipo

---

## DIAPOSITIVA 9: Namespaces de Linux
**Título:** Aislamiento a Nivel Kernel

**Puntos clave:**
- **PID:** IDs de procesos
- **Network:** Interfaces de red  
- **Mount:** Puntos de montaje
- **User:** UIDs/GIDs
- **IPC:** Comunicación entre procesos

**Ilustración sugerida:**
- Contenedor mostrando diferentes "vistas" aisladas
- Proceso viendo PID 1 dentro, pero PID real diferente fuera

---

## DIAPOSITIVA 10: Namespaces de Kubernetes
**Título:** Organización Lógica del Cluster

**Puntos clave:**
- Aislamiento de nombres de recursos
- Cuotas de recursos por namespace
- Control de acceso (RBAC)
- Ejemplos: `default`, `kube-system`

**Ilustración sugerida:**
- Cluster dividido en secciones lógicas
- Cada sección con pods y servicios
- Etiquetas mostrando diferentes namespaces

---

## DIAPOSITIVA 11: Relación entre Ambos Namespaces
**Título:** ¿Cómo se Relacionan?

**Puntos clave:**
- Kubernetes NO crea namespaces de Linux
- Cada contenedor → namespaces Linux propios
- Un namespace K8s → múltiples pods
- Cada pod → múltiples contenedores

**Ilustración sugerida:**
- Diagrama jerárquico:
  ```
  Namespace K8s: "producción"
  ├── Pod 1
  │   ├── Container A → Linux namespaces
  │   └── Container B → Linux namespaces  
  └── Pod 2
      └── Container C → Linux namespaces
  ```

---

## DIAPOSITIVA 12: CONCEPTO CLAVE - Cgroups
**Título:** Control de Recursos del Sistema

**Puntos clave:**
- **También del kernel Linux**, no de Kubernetes
- Kubernetes los **utiliza**
- Limitan: CPU, memoria, I/O
- Cuentan y priorizan recursos

**Ilustración sugerida:**
- Kernel Linux con cgroups como "controladores"
- Kubernetes como "usuario" de estos controladores

---

## DIAPOSITIVA 13: Jerarquía de Cgroups
**Título:** Organización de Recursos

**Sin Kubernetes:**
- `/sys/fs/cgroup/cpu/mi-app/`
- Gestión manual

**Con Kubernetes:**
- `/sys/fs/cgroup/cpu/kubepods/guaranteed/pod-xyz/container-abc/`
- Gestión automática

**Ilustración sugerida:**
- Árbol de directorios mostrando jerarquía
- Comparación lado a lado: manual vs automático

---

## DIAPOSITIVA 14: Agrupación por Contenedor
**Título:** Cada Contenedor = 1 Cgroup

**Niveles de agrupación:**
1. **Contenedor individual** → cgroup específico
2. **Pod** → agrupa contenedores relacionados  
3. **QoS Class** → guaranteed, burstable, besteffort
4. **Kubernetes global** → `/kubepods/`

**Ilustración sugerida:**
- Pirámide mostrando niveles de agrupación
- Ejemplos concretos en cada nivel

---

## DIAPOSITIVA 15: Integración con el Kernel
**Título:** ¿Cómo Aplica el SO los Límites?

**Componentes del kernel:**
- **Scheduler** → respeta límites CPU
- **Memory Manager** → aplica límites memoria
- **I/O Scheduler** → controla acceso disco

**Flujo:**
YAML → K8s → kubelet → Runtime → cgroups → Kernel

**Ilustración sugerida:**
- Diagrama de flujo con flechas
- Kernel como "executor" final de límites

---

## DIAPOSITIVA 16: Ejemplo Práctico
**Título:** Límites en Acción

**Código ejemplo:**
```yaml
resources:
  limits:
    memory: "128Mi"
    cpu: "500m"
```

**Resultado en el sistema:**
```
/sys/fs/cgroup/memory/.../memory.limit_in_bytes
→ 134217728
```

**Ilustración sugerida:**
- Flujo visual: YAML → cgroup → enforcement
- Números reales mostrando conversión

---

## DIAPOSITIVA 17: Demostración en Vivo
**Título:** Veamos cómo Funciona

**Comandos a mostrar:**
```bash
# Crear namespace K8s
kubectl create namespace demo

# Crear pod con límites  
kubectl run nginx --limits=memory=64Mi

# Ver cgroups creados automáticamente
ls /sys/fs/cgroup/memory/kubepods/
```

**Ilustración sugerida:**
- Terminal/consola preparada
- Comandos listos para ejecutar

---

## DIAPOSITIVA 18: Beneficios del Enfoque
**Título:** ¿Por qué es Valioso?

**Para desarrolladores:**
- Abstracción de la infraestructura
- Foco en la aplicación

**Para el sistema:**
- Mejor utilización de recursos
- Gestión de miles de contenedores
- Recuperación automática

**Ilustración sugerida:**
- Comparación antes/después
- Desarrollador enfocado en código vs infraestructura

---

## DIAPOSITIVA 19: Casos de Uso Reales
**Título:** ¿Dónde se Usa?

**Ejemplos:**
- **Microservicios** → cada servicio aislado
- **Multi-tenant** → equipos separados  
- **CI/CD** → entornos de testing
- **Machine Learning** → gestión de recursos intensivos

**Ilustración sugerida:**
- Iconos representando cada caso de uso
- Empresas conocidas que usan Kubernetes

---

## DIAPOSITIVA 20: Relación con Sistemas Operativos
**Título:** Kubernetes como Extensión del SO

**Conceptos escalados:**
- **Aislamiento** → namespaces para aplicaciones
- **Gestión recursos** → cgroups distribuidos
- **Scheduling** → distribución óptima de cargas
- **Monitoreo** → observabilidad distribuida

**Ilustración sugerida:**
- SO tradicional → SO distribuido (Kubernetes)
- Paralelismo de conceptos

---

## DIAPOSITIVA 21: Resumen Ejecutivo
**Título:** Puntos Clave para Recordar

**Conceptos fundamentales:**
- Namespaces: Linux (kernel) ≠ Kubernetes (lógico)
- Cgroups: Linux (kernel) → Kubernetes los utiliza
- Kubernetes = Orquestador que usa primitivas del SO
- Escalabilidad de conceptos de SO a nivel distribuido

**Ilustración sugerida:**
- Checklist con puntos principales
- Iconos de verificación

---

## DIAPOSITIVA 22: Preguntas y Discusión
**Título:** ¿Preguntas?

**Temas para profundizar:**
- Diferencias específicas entre namespaces
- Casos de uso de cgroups
- Arquitectura de Kubernetes
- Relación con otros orquestadores

**Ilustración sugerida:**
- Signo de interrogación grande
- Iconos invitando a la participación

---

## DIAPOSITIVA 23: Referencias y Recursos
**Título:** Para Seguir Aprendiendo

**Documentación:**
- kubernetes.io/docs/
- Linux namespaces man pages
- Cgroups kernel documentation

**Labs prácticos:**
- Kubernetes playground
- Minikube local

**Ilustración sugerida:**
- QR codes a recursos
- Logos de plataformas de aprendizaje

---

## NOTAS PARA PRESENTADORES:

### Timing sugerido:
- Diapositivas 1-4: 5 minutos (Introducción)
- Diapositivas 5-7: 5 minutos (Kubernetes básico)  
- Diapositivas 8-11: 8 minutos (Namespaces)
- Diapositivas 12-16: 8 minutos (Cgroups)
- Diapositiva 17: 2 minutos (Demo)
- Diapositivas 18-21: 2 minutos (Cierre)

### Consejos de presentación:
- **Diapositiva 8 y 12:** Enfatizar que son conceptos del SO, no de K8s
- **Diapositiva 17:** Tener terminal preparado previamente
- **Interacción:** Preguntar si hay dudas en conceptos clave
- **Ejemplos:** Usar analogías del mundo real cuando sea posible 