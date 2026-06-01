# Zabbix - Monitoreo de Windows Failover Cluster

Esta solución proporciona una plantilla de Zabbix que crea ítems y disparadores (triggers) básicos para monitorear de manera efectiva el estado general de un clúster de alta disponibilidad de Windows (Windows Failover Cluster).

### 🚀 Características Principales

* **Nodos del Clúster (Nodes):** Monitorea la disponibilidad y el estado de los nodos individuales que conforman el clúster.
* **Recursos Centrales del Clúster (Core Cluster Resources):** Evalúa la salud de los componentes críticos del clúster, tales como el servicio de nombres, el recurso testigo (Witness) y el recurso de calidad de servicio de almacenamiento (Storage QoS Resource).
* **Máquinas Virtuales (Virtual Machines):** Supervisa el estado operativo y el rendimiento de las máquinas virtuales alojadas dentro de la infraestructura del clúster.
* **Configuraciones de Máquinas Virtuales (Virtual Machine Configurations):** Rastrea las configuraciones de las máquinas virtuales para detectar cambios o problemas en su definición.

### 📋 Requisitos Previos

Para asegurar el correcto funcionamiento de esta plantilla, tu entorno debe cumplir con lo siguiente:

* **Servidor Zabbix:** Compatible con la importación de plantillas en formatos modernos (XML, YAML o JSON).
* **Agente:** Zabbix Agent 2 instalado en cada host que desees monitorear.
* **Permisos y Dependencias:** Ejecución de scripts locales habilitada y permisos suficientes para que el agente consulte los recursos del clúster de Windows.

### ⚙️ Guía de Instalación

Sigue estos pasos en el host o servidor que deseas monitorear.

**Paso 1: Copiar los scripts personalizados**

Descarga la carpeta `CustomScripts` desde este repositorio y colócala dentro del directorio de instalación de tu agente Zabbix.
> *Ruta recomendada:* `C:\Program Files\Zabbix Agent 2\CustomScripts`

**Paso 2: Configurar Zabbix Agent 2 (`zabbix_agent2.conf`)**

Abre el archivo de configuración principal de tu agente (por ejemplo, en `C:\Program Files\Zabbix Agent 2\zabbix_agent2.conf`) y realiza las siguientes modificaciones.

1. **Ajustes de rendimiento:** Asegúrate de buscar y modificar estas dos líneas. El tiempo de espera extendido es necesario para que las consultas de PowerShell se completen correctamente:

```ini
   Timeout=30
   UnsafeUserParameters=1
   ```

2. **Parámetros de Usuario (UserParameters):** Añade el siguiente bloque de código al final del archivo. Esto agrupa lógicamente los descubrimientos y estados de los nodos, recursos y máquinas virtuales:

```ini
   # =======================================================
   # MONITOREO DE FAILOVER CLUSTER (NODES, RESOURCES, VMS)
   # =======================================================
   # Teniendo en cuenta que estás utilizando Zabbix Agent 2, si la ubicación está en otro lado se deben hacer las modificaciones necesarias.
   
   # --- Cluster Nodes ---
   UserParameter=cluster.nodes.discovery,powershell -noninteractive -file "C:\Program Files\Zabbix Agent 2\CustomScripts\FailoverClusterNodesDiscovery.ps1"
   UserParameter=cluster.node.state[*],powershell -noninteractive -file "C:\Program Files\Zabbix Agent 2\CustomScripts\FailoverClusterNodeState.ps1" $1
   
   # --- Cluster Resources ---
   UserParameter=cluster.resources.discovery[*],powershell -noninteractive -file "C:\Program Files\Zabbix Agent 2\CustomScripts\FailoverClusterResourcesDiscovery.ps1" $1
   UserParameter=cluster.resource.state[*],powershell -noninteractive -file "C:\Program Files\Zabbix Agent 2\CustomScripts\FailoverClusterResourceState.ps1" $1
   
   # --- Virtual Machine Networks ---
   UserParameter=cluster.vm.network.adapter.discovery,powershell -noninteractive -file "C:\Program Files\Zabbix Agent 2\CustomScripts\FailoverClusterVmNetworkDiscovery.ps1"
   UserParameter=cluster.vm.network.adapter.type[*],powershell -noninteractive -file "C:\Program Files\Zabbix Agent 2\CustomScripts\FailoverClusterVmNetworkType.ps1" $1 $2
   ```

**Paso 3: Reiniciar el Agente**

Para que el agente Zabbix reconozca los nuevos parámetros y scripts, debes reiniciar el servicio en tu host. Puedes hacerlo desde la consola de Servicios de Windows o mediante PowerShell como Administrador:

```powershell
Restart-Service -Name "Zabbix Agent 2"
```

**Paso 4: Importar la plantilla en Zabbix**

1. Ingresa a la interfaz web de tu servidor Zabbix.
2. Navega a **Configuration -> Templates** y haz clic en el botón **Import** en la esquina superior derecha.
3. Elige el formato de plantilla que prefieras descargar de este repositorio (`zbx_export_templates.xml`, `zbx_export_templates.yaml` o `zbx_export_templates.json`) e impórtalo.
4. Una vez importado con éxito, vincula la plantilla al Host correspondiente dentro de Zabbix.

### 📂 Estructura del Repositorio

* **`zbx_export_templates.[xml/yaml/json]`:** Los archivos de la plantilla oficial en múltiples formatos listos para importar en Zabbix.
* **`zabbix_agent2.conf`:** Archivo que contiene el ejemplo con las líneas exactas (UserParameters) que deben agregarse a tu configuración local.
* **/CustomScripts:** Directorio que contiene la lógica para la extracción de datos de los nodos, recursos y máquinas virtuales del clúster mediante PowerShell.

### 🤝 Contribuciones

¡Las mejoras y sugerencias son bienvenidas! Siéntete libre de abrir un Issue o enviar un Pull Request.
