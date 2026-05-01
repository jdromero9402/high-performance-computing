# Laboratorio_ML00: Configuración Clúster SPARK

**Pontificia Universidad Javeriana**  
**Computación de alto desempeño**  
**Docente:** John Corredor, PhD

---

**Integrantes:**

| Nombre | Correo |
|---|---|
| Juan Sebastián Bravo Santacruz | bravos.js@javeriana.edu.co |
| Felix David Cordova Garcia | fdcordova@javeriana.edu.co |
| Jonatan Alejandro Gallo Martinez | jonatan.gallo@javeriana.edu.co |
| Jose Alejandro Jaime Lopez | jo.jaime@javeriana.edu.co |
| Josman Alexdy Ramirez Torres | josman.ramirez@javeriana.edu.co |
| Jesús David Romero Melo | jesus-romero@javeriana.edu.co |
| Juan Camilo Torres Peña | torrespjc@javeriana.edu.co |

**Fecha:** 2026/04/30

---

## 1. Objetivo

Configuración un Clúster de Spark con VM Linux Rocky.

---

## 2. Desarrollo del Taller

A continuación, se documenta cada paso del proceso de configuración del clúster Spark. Para cada paso, se ejecutaron los comandos indicados y se registró la evidencia (captura de pantalla) en el espacio habilitado.

---

### 2A — SSH Passwordless (Master → Workers)

Desde el nodo master, se configuró el acceso SSH sin contraseña hacia todos los workers. Esto permitió que el master pueda iniciar procesos en los workers sin requerir autenticación manual.

**Paso 1:** Generar el par de claves SSH en el master:

```bash
$ ssh-keygen
```

**Paso 2:** Copiar la clave pública al master y a cada worker:

```bash
$ ssh-copy-id master@<ip-del-master>
$ ssh-copy-id worker@<ip-del-worker>
```
<img width="426" height="349" alt="image" src="https://github.com/user-attachments/assets/80aad254-5ca5-4031-bbc0-1e496d28cc41" />

---

### 2B — Edición del Fichero /etc/hosts

En TODOS los nodos (master y workers), se editó el fichero `/etc/hosts` para agregar la resolución de nombres de los nodos del clúster.

**Paso 1:** Abrir el fichero hosts con nano:

```bash
$ nano /etc/hosts
```

**Paso 2:** Agregar las siguientes líneas al fichero:

```
################################################
# IP's Master/Workers
################################################
10.43.97.146  cadhead02.javeriana.edu.co     cadhead02
10.43.97.141  cad02-w000.javeriana.edu.co    cad02-w000
10.43.97.135  cad02-w001.javeriana.edu.co    cad02-w001
10.43.97.136  cad02-w002.javeriana.edu.co    cad02-w002
10.43.97.148  cad02-w003.javeriana.edu.co    cad02-w003
10.43.97.145  cadcliente02.javeriana.edu.co  cadcliente02
10.43.97.149  cad02-nfs01.javeriana.edu.co   cad02-nfs01
```
<img width="771" height="400" alt="image" src="https://github.com/user-attachments/assets/8301e7fa-2023-49e3-8288-7a129d14bf98" />

---

### 2C — Creación de la Carpeta Compartida en el Master

En el nodo master, se creó el directorio `/nfs/condor` que fue compartido vía NFS con los workers.

**Paso 1:** Crear el directorio `/nfs/condor`:

```bash
$ cd /
$ sudo mkdir /nfs/condor -p
$ sudo chmod estudiante:estudiante nfs/condor
```

**Paso 2:** Editar `/etc/exports` para compartir la carpeta:

```bash
$ sudo nano /etc/exports

# Agregar la siguiente línea:
/nfs/condor <ip_cliente>(rw,no_root_squash,subtree_check,fsid=0)
```
<img width="542" height="726" alt="image" src="https://github.com/user-attachments/assets/10ebdbe0-76de-4881-92cd-da003598d7fe" />

---

### 2D — Instalación y Configuración del Servicio NFS

Se instaló y activó el servicio NFS en el nodo master para permitir el montaje de la carpeta compartida desde los workers.

**Paso 1:** Instalar las utilidades NFS:

```bash
$ sudo dnf install nfs-utils
```

**Paso 2:** Habilitar, iniciar y verificar el servicio:

```bash
$ sudo systemctl enable nfs-server
$ sudo systemctl start nfs-server
$ sudo systemctl status nfs-server
```
<img width="775" height="361" alt="image" src="https://github.com/user-attachments/assets/48fc0751-0f65-479d-80ec-59727a1c3e94" />

---

### 2E — Montaje del NFS en los Workers

En cada nodo worker, se creó el directorio de montaje y se conectó la carpeta compartida del master.

**Paso 1:** Crear directorio de montaje en el worker:

```bash
$ cd /
$ sudo mkdir -p nfs/condor
```

**Paso 2:** Montar la carpeta compartida del master:

```bash
$ mount 10.43.97.149:/nfs/condor /nfs/condor
```

**Paso 3:** Verificar los sistemas de ficheros montados:

```bash
$ df -h
```
<img width="776" height="389" alt="image" src="https://github.com/user-attachments/assets/bf7b01ed-5ed7-4ff7-88f7-1dc160691422" />

---

### 2F — Instalación de Apache Spark en la Carpeta Compartida

Desde el master, se copió el paquete de Spark a la carpeta compartida y se descomprimió. Esto permitió que todos los nodos accedan a los mismos binarios vía NFS.

**Paso 1:** Copiar el archivo `.tgz` de Spark a `/nfs/condor/apps`:

```bash
$ cp /home/estudiante/spark-3.5.2-bin-hadoop3.tgz /nfs/condor/apps
```

**Paso 2:** Descomprimir y renombrar la carpeta:

```bash
$ cd /nfs/condor/apps
$ tar -zxvf spark-3.5.2-bin-hadoop3.tgz
$ mv spark-3.5.2-bin-hadoop3 Spark
```

**Paso 3:** Instalar Java en master y workers:

```bash
$ sudo dnf install openjdk-8-jdk-headless
$ java -version
```
<img width="770" height="328" alt="image" src="https://github.com/user-attachments/assets/388fab1b-c8b3-4722-b44c-e3f79f52e52e" />

---

### 2G — Configuración de Variables de Entorno (Todos los Nodos)

En TODOS los nodos (master y workers), se configuró las variables de entorno para Spark y Python editando el fichero `.bashrc`.

**Paso 1:** Verificar el estado actual del entorno Spark:

```bash
$ env | grep -i SPARK
```

**Paso 2:** Editar el fichero `.bashrc`:

```bash
$ cd
$ nano .bashrc
```

**Paso 3:** Buscar la línea `# export SYSTEMD_PAGER=` e insertar después:

```bash
export SPARK_HOME="/nfs/condor/Spark"
export SPARK_CONF_DIR="$SPARK_HOME/conf"
export PATH="$SPARK_HOME/bin":$PATH
export PYSPARK_PYTHON="/usr/local/bin/python3"
export PYSPARK_DRIVER_PYTHON="/usr/local/bin/python3"
```

**Paso 4:** Recargar la configuración y verificar:

```bash
$ source .bashrc
$ env | grep -i SPARK
```
<img width="777" height="329" alt="image" src="https://github.com/user-attachments/assets/5e593b6e-e6fa-48d4-8f83-e69cf092bde6" />

---

### 2H — Configuración de Apache Spark

Se configuró los ficheros de entorno de Spark: `spark-env.sh` (dirección IP del master) y `workers` (lista de nodos worker).

**Paso 1:** Ir al directorio `conf` y crear el fichero de configuración:

```bash
$ cd /nfs/condor/apps/Spark/conf
$ cp spark-env.sh.template spark-env.sh
```

**Paso 2:** Editar `spark-env.sh` — buscar la sección de daemons e insertar la IP del master:

```bash
export SPARK_MASTER_HOST=10.43.97.145
export SPARK_WORKER_PORT=8888
export SPARK_WORKER_WEBUI_PORT=8081

export SPARK_DAEMON_MEMORY=1g

export SPARK_WORKER_CORES=4
export SPARK_WORKER_MEMORY=12g
export SPARK_DAEMON_JAVA_OPTS="-Dspark.schedules.mode=FAIR"

export SPARK_EXECUTOR_CORES=2
export SPARK_EXECUTOR_MEMORY=2g

export PYSPARK_PYTHON=/usr/bin/python3
export PYSPARK_DRIVER_PYTHON=/usr/bin/python3
```

**Paso 3:** Crear y editar el fichero `workers`:

```
# A Spark Worker will be started on each of the machines listed below.
# localhost
cad02-w000
cad02-w001
cad02-w002
cad02-w003
cadhead02
```

**Paso 4:** Instalar herramienta para ver procesos Java:

```bash
$ sudo dnf -y install java-1.8.0-openjdk-devel
$ jps
```
<img width="714" height="109" alt="image" src="https://github.com/user-attachments/assets/57ee4e36-af4c-4d7b-81d7-b80c4c1db5e9" />

---

### 2I — Inicio del Servicio Spark y Verificación

Se levantó el clúster Spark y verifique que el master y los workers estén activos tanto por consola como por la interfaz web.

**Paso 1:** Iniciar todos los servicios Spark desde el master:

```bash
$ cd /nfs/condor/Spark/sbin
$ ./start-all.sh
```

**Paso 2:** Verificar los procesos Java activos (debe aparecer Master y Worker):

```bash
$ jps
```

**Paso 3:** Verificar el clúster en el navegador:

```
http://10.43.103.45:8080
```

<img width="783" height="412" alt="image" src="https://github.com/user-attachments/assets/ce4f41a2-a418-448f-97a5-a5d58767fa92" />


---

## 3. Conclusiones

La configuración exitosa de un clúster Apache Spark sobre VMs con Rocky Linux permitió consolidar conocimientos fundamentales en computación de alto desempeño distribuida. A continuación, se presentan las principales conclusiones del laboratorio:

- **SSH sin contraseña** entre el nodo master y los workers es un requisito previo indispensable para el funcionamiento del clúster, ya que Spark depende de este mecanismo para iniciar y coordinar procesos remotos de forma automática durante el lanzamiento del servicio mediante `start-all.sh`.

- **El uso de NFS** como sistema de archivos compartido demostró ser una estrategia eficiente para gestionar los binarios de Spark en un entorno multi-nodo: al instalar Spark una sola vez en `/nfs/condor/apps` y montarlo en todos los workers, se garantiza consistencia en versiones y se simplifica significativamente el mantenimiento del clúster.

- **La correcta resolución de nombres** mediante `/etc/hosts` en todos los nodos resultó crítica para la comunicación interna del clúster, evitando dependencias de un servidor DNS externo y asegurando que el master pueda identificar y contactar a cada worker por nombre de host.

- **La configuración del archivo `spark-env.sh`** permitió ajustar parámetros clave del clúster como la memoria por worker (12g), los cores disponibles (4 por nodo) y el modo de planificación FAIR, lo que demuestra la flexibilidad de Spark para adaptarse a los recursos físicos disponibles en la infraestructura.

- **La verificación a través de la interfaz web** del Spark Master (`http://10.43.97.145:8080`) confirmó que los 5 workers quedaron en estado ALIVE con un total de 20 cores y 60 GB de memoria disponibles para procesamiento distribuido, validando que el clúster quedó operativo y listo para ejecutar cargas de trabajo de machine learning y análisis de datos a gran escala.
