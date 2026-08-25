# Despliegue-de-Wazuh-SIEM-y-Detecci-n-de-Eventos-en-Tiempo-Real-en-Windows
# Laboratorio de Ciberseguridad: Implementación de File Integrity Monitoring (FIM) en Wazuh SIEM

## 📌 Descripción del Proyecto
Este proyecto documenta la configuración, resolución de problemas y validación del módulo **File Integrity Monitoring (FIM)** en tiempo real utilizando **Wazuh SIEM**. Se monitoreó un directorio crítico en un endpoint de Windows 10 para detectar adiciones, modificaciones de integridad (checksums) y eliminaciones de archivos.

---

## 🛠️ Arquitectura e Infraestructura del Laboratorio

| Componente | Sistema Operativo | Versión de Wazuh | Nombre de Host / IP |
| :--- | :--- | :--- | :--- |
| **Manager / Server** | Kali Linux | `v4.9.0` | `wazuh.manager` (`192.168.1.36`) |
| **Agent / Endpoint** | Windows 10 Pro | `v4.9.0` | `WIN10-CLI01` |

---

## 🚀 Paso a Paso de la Implementación

### 1. Despliegue y Alineación de Versiones del Agente
Para evitar errores de incompatibilidad (*"Agent version must be lower or equal to manager version"*), se alineó la versión del agente en Windows con la versión del Manager (`v4.9.0`).

```powershell
# Descarga e instalación silenciosa asignando la IP del Manager
Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.0-1.msi" -OutFile "$env:TEMP\wazuh-agent-4.9.0.msi"
msiexec.exe /i "$env:TEMP\wazuh-agent-4.9.0.msi" /q WAZUH_MANAGER="192.168.1.36" WAZUH_AGENT_NAME="WIN10-CLI01"
Start-Service -Name WazuhSvc
```

### 2. Configuración del Monitoreo en Tiempo Real (`syscheck`)
Se editó la configuración principal del agente (`C:\Program Files (x86)\ossec-agent\ossec.conf`) para activar el monitoreo FIM en tiempo real sobre el directorio objetivo:

```xml
<syscheck>
  <disabled>no</disabled>
  <frequency>43200</frequency>
  
  <!-- Monitoreo en tiempo real de directorio crítico -->
  <directories realtime="yes">C:\carpetacritica</directories>
</syscheck>
```

Se reinició el servicio para aplicar los cambios:
```powershell
Restart-Service -Name WazuhSvc
```

---

## 🧪 Pruebas de Validación y Simulación de Eventos

Desde el endpoint `WIN10-CLI01`, se ejecutó una secuencia de prueba en PowerShell para simular el ciclo de vida de un archivo en `C:\carpetacritica`:

```powershell
# 1. Creación de archivo
echo "Prueba de FIM" > C:\carpetacritica	est_fim.txt

# 2. Modificación del contenido (alteración de hashes MD5/SHA256)
Add-Content -Path "C:\carpetacritica	est_fim.txt" -Value "Cambio no autorizado detectado."

# 3. Eliminación del archivo
Remove-Item -Path "C:\carpetacritica	est_fim.txt"
```

---

## 📊 Resultados y Matriz de Alertas Detección SIEM

Wazuh capturó e identificó con éxito los tres eventos en tiempo real dentro del panel **Endpoint Security > File Integrity Monitoring**:

| Evento (`syscheck.event`) | Archivo Afectado | Regla ID | Nivel | Descripción de la Regla |
| :--- | :--- | :--- | :--- | :--- |
| **`added`** | `c:\carpetacritica	est_fim.txt` | `554` | **5** | *File added to the system.* |
| **`modified`** | `c:\carpetacritica	est_fim.txt` | `550` | **7** | *Integrity checksum changed.* |
| **`deleted`** | `c:\carpetacritica	est_fim.txt` | `553` | **7** | *File deleted.* |

---

## 🔑 Conclusiones y Aprendizajes Clave
* **Requisito de Versiones:** Los agentes de Wazuh deben mantener una versión menor o igual a la del Manager ($Manager \ge Agent$).
* **Monitoreo Realtime vs. Schedule:** El parámetro `realtime="yes"` permite la captura inmediata mediante Hooks del kernel de Windows sin esperar los ciclos por defecto (12h).
* **Integridad de Archivos:** Las alertas de modificación (Regla `550`) calculan diferencias en los checksums (MD5, SHA1, SHA256) garantizando trazabilidad ante cambios no autorizados.
README.md
Mostrando README.md.
