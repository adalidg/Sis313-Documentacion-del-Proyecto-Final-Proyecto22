# 🚀 Proyecto Final SIS313: FleetTrack - Arquitectura de Alta Disponibilidad y Telemetría

> **Asignatura:** SIS313: Infraestructura, Plataformas Tecnológicas y Redes<br>
> **Semestre:** 1/2026<br>
> **Docente:** Ing. Marcelo Quispe Ortega

## 👥 Miembros del Equipo (Grupo FleetTrack - proyecto 22)

| Nombre Completo | Rol en el Proyecto | Contacto (GitHub/Email) |
| :--- | :--- | :--- |
| Adalid Gutiérrez Torricos | Administrador de Base de Datos (DBA) y Persistencia | @adalidg |
| Dylan Elyazar Barahona Romero | Arquitecto de Persistencia en Standby (Réplica) | @Elyazarrr |
| Matias Jonathan Quispe Martinez | Ingeniero de Redes Perimetrales y Balanceo de Carga | @QuispeMartínezMatias |
| Weimar Genaro Zamorano Hurtado | Desarrollador Backend, Orquestación y Caché | @Genaro007 |

## 🎯 I. Objetivo del Proyecto

> **Objetivo:** Diseñar, configurar y desplegar una infraestructura de red segmentada y de Alta Disponibilidad (HA) para el ecosistema FleetTrack, integrando un clúster de bases de datos MariaDB con replicación Maestro-Bóveda, balanceo de carga estricto Capa 7 mediante HAProxy, enrutamiento inteligente de consultas con ProxySQL, gestión de estado en memoria con Redis y telemetría avanzada, garantizando la resiliencia integral, la tolerancia a fallos y la continuidad operacional del servicio ante caídas críticas de infraestructura.

## 💡 II. Justificación e Importancia

> **Justificación:** Este proyecto resuelve el problema crítico de los Puntos Únicos de Fallo (SPOF) presentes en arquitecturas monolíticas tradicionales. A nivel empresarial, la interrupción del servicio genera pérdidas incalculables; por ello, implementamos una estrategia de "Alta Disponibilidad Híbrida" (T2). En la capa web, garantizamos la atención fluida de clientes mediante un balanceador perimetral. En la capa de datos (la más vulnerable), mitigamos el riesgo de corrupción por "Split-Brain" aislando la Bóveda de Persistencia (Standby) y automatizando su promoción únicamente a través de un script Watchdog orquestado, asegurando así la integridad de la información y reforzando las políticas de continuidad de negocio y seguridad perimetral (T5).

## 🛠️ III. Tecnologías y Conceptos Implementados

### 3.1. Tecnologías Clave

* **HAProxy:** Balanceador de carga perimetral de alta eficiencia que opera en la Capa 7 del modelo OSI, distribuyendo el tráfico HTTP entrante hacia los nodos de la aplicación usando un algoritmo Round-Robin.
* **MariaDB:** Motor de Base de Datos Relacional configurado en una topología Maestro-Esclavo (Bóveda Standby), asegurando la persistencia de los registros y tolerando caídas de hardware.
* **ProxySQL & Watchdog Script:** ProxySQL actúa como un enrutador inteligente de consultas SQL, mientras que el Watchdog monitoriza el clúster de datos para ejecutar un failover automático si el nodo Maestro deja de responder.
* **Node.js:** Entorno de ejecución asíncrono utilizado para construir y servir la lógica de negocio del backend en múltiples nodos paralelos.
* **Redis:** Motor de estructura de datos en memoria, desplegado en un nodo aislado, fundamental para el manejo ultrarrápido de sesiones web y la reducción de la carga en la base de datos principal.
* **Prometheus & Sysbench:** Prometheus (junto a Node-Exporter) extrae métricas de consumo de CPU/RAM, mientras que Sysbench valida la capacidad estocástica del servidor de datos bajo cargas de estrés OLTP.

### 3.2. Conceptos de la Asignatura Puestos en Práctica (T1 - T6)

* ✅ **Alta Disponibilidad (T2) y Tolerancia a Fallos:** Replicación de bases de datos con sincronización de estado, y uso de HAProxy para mitigar la caída de nodos web sin afectar al usuario final.
* ✅ **Seguridad y Hardening (T5):** Aislamiento de los motores de bases de datos limitando su `bind-address` exclusivamente a IPs de la red interna, preveniendo la sordera de red (Error 111) y accesos no autorizados.
* ✅ **Automatización y Gestión (T6):** Desarrollo de scripts en Bash para la ejecución de respaldos físicos automatizados en horarios de bajo tráfico, integrados directamente al Crontab del sistema (`backup_db.sh`).
* ✅ **Balanceo de Carga/Proxy (T3/T4):** Implementación de HAProxy para distribuir las peticiones HTTP a la capa de Node.js, apoyado por ProxySQL para el manejo eficiente del pool de conexiones hacia MariaDB.
* ✅ **Monitoreo (T4/T1):** Configuración de Node-Exporter en los puertos 9100 para garantizar la observabilidad del sistema y reaccionar proactivamente ante cuellos de botella.
* ✅ **Networking Avanzado (T3):** Segmentación lógica de la infraestructura mediante la configuración nativa en Netplan de 4 VLANs (10, 20, 30, 40), separando el tráfico Perimetral, de Aplicación, de Soporte y de Datos.
## 🌐 IV. Diseño de la Infraestructura y Topología

### 4.1. Diseño Esquemático

Incluye un diagrama de la topología final. Muestra claramente la segmentación de red, las IPs utilizadas, y los flujos de tráfico.

> 
| VM/Host | Rol | IP Física | Red Lógica (VLAN) | SO |
| :--- | :--- | :--- | :--- | :--- |
| **VM 195** | Escudo Perimetral (HAProxy) | 192.168.100.195 | VLAN Perimetral | Ubuntu 24.04 Server |
| **VM 197** | Nodo App 1 + ProxySQL + Watchdog | 192.168.100.197 | VLAN App / Control | Ubuntu 24.04 Server |
| **VM 199** | Nodo App 2 (Redundancia Web) | 192.168.100.199 | VLAN App | Ubuntu 24.04 Server |
| **VM 198** | Caché en Memoria Central (Redis) | 192.168.100.198 | VLAN Soporte | Ubuntu 24.04 Server |
| **VM 196** | Base de Datos (Maestro OLTP) | 192.168.100.196 | VLAN Datos | Ubuntu 24.04 Server |
| **VM 200** | Base de Datos (Bóveda Standby) | 192.168.100.200 | VLAN Datos | Ubuntu 24.04 Server |
| **VM 201** | Telemetría Central (Grafana) | 192.168.100.201 | VLAN Gestión | Ubuntu 24.04 Server |

### 4.2. Estrategia Adoptada (Opcional)

* **Estrategia de Desacoplamiento de Recursos:** A diferencia de diseños monolíticos, decidimos aislar el motor de Redis en la VM 198. Esto previene que la caché consuma la memoria RAM necesaria para el procesamiento de hilos de Node.js, logrando un escalamiento horizontal limpio.
* **Estrategia de Failover Conservador:** La promoción de la VM 200 (Bóveda) no es inmediata al primer fallo de red para evitar escenarios de falsos positivos. Se implementó un "Watchdog" en la VM 197 que verifica métricas consistentes antes de realizar el cambio de estado (`read_only=0`), protegiendo la integridad del esquema transaccional.
* * **Centralización de Telemetría (VM 201):** Se destinó un nodo exclusivo para la recolección y visualización de métricas (Prometheus Server / Grafana). Aislar el monitoreo garantiza que, en caso de un ataque de denegación de servicio (DDoS) o colapso por saturación en la capa de aplicación, los dashboards de administración sigan operativos y reportando alertas.

## 📋 V. Guía de Implementación y Puesta en Marcha

### 5.1. Pre-requisitos
* 7 Máquinas Virtuales provisionadas con Ubuntu 24.04 LTS.
* Interfaz de red troncal configurada con etiquetado 802.1Q para soportar el enrutamiento inter-VLAN.
* Acceso con privilegios de superusuario (`sudo/root`) en todos los nodos.

### 5.2. Despliegue (Ejecución de la Automatización)
1.  **Capa de Persistencia (VM 196 y VM 200):** Instalar MariaDB, configurar los `server-id` únicos en `/etc/mysql/mariadb.conf.d/50-server.cnf` y establecer la replicación `CHANGE MASTER TO` en la VM 200.
2.  **Capa de Aplicación y Soporte (VM 197, 198, 199):** Desplegar Node.js con PM2 en las VMs 197/199. Instalar y asegurar Redis en la VM 198 modificando la directiva `bind` para aceptar conexiones de la VLAN App.
3.  **Capa Perimetral (VM 195):** Instalar HAProxy, definir los nodos web en la sección `backend` del archivo de configuración, habilitando las directivas `check` para monitoreo de estado en vivo.
4. **Capa de Observabilidad (VM 201):** Levantar el servidor central de Prometheus para hacer "scraping" a los puertos 9100 de las demás VMs, y enlazar Grafana para la visualización del estado de salud del clúster en tiempo real.

### 5.3. Ficheros de Configuración Clave
* `/etc/haproxy/haproxy.cfg`: Define los algoritmos de balanceo y las métricas de health-check hacia los Nodos Node.js.
* `/opt/fleettrack/backups/backup_db.sh`: Script forense para el volcado relacional y rotación de históricos locales.
* `[Añadir ruta del script Watchdog]`: Archivo de orquestación responsable de alterar las rutas en ProxySQL durante contingencias.
* `/etc/netplan/01-network-manager-all.yaml`: Fichero de declaración de subinterfaces lógicas y VLANs.

**Incluir además los archivos de configuración y software a utilizar dentro del proyecto y organizados en carpetas.**

## ⚠️ VI. Pruebas y Validación

| Prueba Realizada | Resultado Esperado | Resultado Obtenido |
| :--- | :--- | :--- |
| **Ingeniería del Caos (Simulación de Caída VM 196)** | El sistema Watchdog detecta el timeout del Maestro y ordena a ProxySQL redirigir las consultas de escritura hacia la VM 200. | [OK/FALLIDO - Confirmar test de Watchdog] |
| **Prueba de Carga y Ruptura (Sysbench OLTP)** | La base de datos soporta peticiones masivas (1 millón de registros) sin colapsar el daemon y antes de generar encolamientos en ProxySQL. | **OK:** Rendimiento sostenido a 17,581 QPS y 879 TPS a máxima exigencia de CPU. |
| **Validación Perimetral (Balanceo HAProxy)** | Las peticiones HTTP externas son distribuidas de manera equitativa, comprobable apagando la VM 197 y observando la VM 199 asumir la carga. | [OK/FALLIDO - Confirmar test de Matías] |

## 📚 VII. Conclusiones y Lecciones Aprendidas

La implementación de este proyecto demostró que el verdadero desafío de la Alta Disponibilidad no radica únicamente en desplegar los servicios, sino en orquestar su comunicación segura a través de VLANs segmentadas. El aislamiento de los motores de caché (Redis en VM 198) y la asignación de roles estrictos a las bases de datos (VM 196 como Maestro, VM 200 como Bóveda pasiva) probaron ser decisiones arquitectónicas fundamentales para evitar cuellos de botella. 

Adicionalmente, se aprendió que la monitorización pasiva no es suficiente; la aplicación de pruebas de estrés comprobables (como los 17,581 QPS logrados con Sysbench) y la configuración de rutinas automáticas de recuperación ante desastres (cron jobs para respaldos) son requisitos indispensables para que una infraestructura pase de ser un escenario académico a un entorno con grado de producción empresarial confiable.
