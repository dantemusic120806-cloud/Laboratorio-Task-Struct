# Laboratorio-Task-Struct
# Informe de Laboratorio: Análisis de la Estructura `task_struct` en el Kernel de Linux y Verificación Empírica

## 1. Introducción
En el sistema operativo Linux, cada proceso o hilo de ejecución está representado internamente en el Kernel por un descriptor de proceso denominado *Process Control Block* (PCB). Esta abstracción se implementa a través de la estructura en C `struct task_struct`, localizada en el archivo de cabecera de administración del planificador. Dicha estructura agrupa los atributos esenciales para la gestión de memoria, prioridades de ejecución, estado del ciclo de vida y recursos de entrada/salida de cada tarea en el sistema.

---

## 2. Ubicación y Entorno de Trabajo
* **Entorno:** Linux Mint virtualizado en Oracle VM VirtualBox.
* **Ruta del archivo de cabecera:** `/usr/src/linux-headers-$(uname -r)/include/linux/sched.h`
* **Definición de la estructura:** Se ubica aproximadamente a partir de la línea 797 del archivo `sched.h`.

---

## 3. Identificación y Análisis de 10 Subcampos de `task_struct`

A partir de la inspección del archivo `sched.h`, se seleccionaron e identificaron 10 subcampos clave dentro de `struct task_struct`, integrando las sugerencias de la guía de laboratorio (`mm`, `files`, `signal`, `real_parent`, `cred`):

| Nombre | Tipo C | Subsistema | Propósito |
| :--- | :--- | :--- | :--- |
| **`__state`** | `unsigned int` | Planificador (*Scheduler*) | Almacena el estado actual en el ciclo de vida del proceso (`TASK_RUNNING`, `TASK_INTERRUPTIBLE`, etc.). |
| **`stack`** | `void *` | Gestión de Procesos | Puntero a la base de la pila de ejecución reservada en espacio de kernel (*kernel stack*). |
| **`usage`** | `refcount_t` | Kernel Core | Contador de referencias activas sobre la estructura para evitar su liberación prematura en memoria. |
| **`flags`** | `unsigned int` | Gestión de Procesos | Banderas de atributos a nivel de proceso (p. ej., si es un hilo del kernel o se está finalizando). |
| **`prio`** | `int` | Planificador (*Scheduler*) | Prioridad dinámica actual calculada por el planificador para la asignación de CPU. |
| **`real_parent`** | `struct task_struct *` | Jerarquía de Procesos | Puntero al proceso padre legítimo que creó la tarea mediante `fork()`. |
| **`mm`** | `struct mm_struct *` | Memoria Virtual | Referencia la estructura que gestiona el espacio de dirección virtual (pila, heap, datos). |
| **`files`** | `struct files_struct *` | Sistema de Archivos (`fs`) | Mantiene la tabla de descriptores de archivos, sockets y tubos (*pipes*) abiertos por el proceso. |
| **`signal`** | `struct signal_struct *` | Comunicación IPC | Contiene descriptores de manejo de señales enviadas o pendientes hacia la tarea. |
| **`cred`** | `const struct cred *` | Seguridad y Permisos | Puntero a las credenciales de seguridad del proceso (IDs de usuario UID y de grupo GID). |

---

## 4. Verificación Empírica (`/proc/self/status`)

Para contrastar el modelo teórico de la estructura C con el estado real del sistema, se ejecutó en la terminal la lectura del pseudo-archivo de estado del proceso actual:

```bash
cat /proc/self/status | grep -E "Name|State|Pid|PPid|Uid"
