# Proxmox-HomeLab
Despliegue de infraestructura virtualizada sobre hardware limitado con Proxmox y Networking avanzado
# 🚀 Proyecto Hyperoot: Home-Lab de Microservicios y Networking Avanzado

## 📖 Introducción
Este proyecto documenta el proceso de conversión de un ordenador portátil ASUS (hardware *commodity*) en un servidor de virtualización de clase empresarial utilizando **Proxmox VE**. El objetivo es construir un entorno seguro y escalable para el despliegue de microservicios (Docker/LXC) y la experimentación con redes avanzadas (MikroTik).

---

## 💻 0. Especificaciones de Hardware (The Metal)

| Componente | Detalle |
| :--- | :--- |
| **Host** | ASUS Laptop |
| **CPU** | x86_64 compatible con virtualización KVM |
| **RAM** | 8GB DDR3 |
| **Almacenamiento 1** | 240GB SSD (Sistema Operativo y VMs críticas) |
| **Almacenamiento 2** | 3TB HDD (Datos y Backups - *Pendiente de integrar*) |

---

## ⚙️ 1. Fase de Instalación del Hypervisor

### 1.1 Preparación y Booteo
Se utilizó una unidad USB booteable con la ISO oficial de **Proxmox VE 9.1**. Se configuró la BIOS del ASUS para arrancar en modo UEFI.

### 1.2 Configuración del Disco (Filesystem)
Se seleccionó el SSD de 240GB como disco de destino. 

**Decisión Técnica:** Se optó por el sistema de archivos **`ext4`**.
> *Justificación:* Dado que el host solo cuenta con 8GB de RAM, `ext4` ofrece un rendimiento excelente con un consumo mínimo de memoria para caché, a diferencia de ZFS que requiere una gestión de RAM más intensiva.

<img width="1599" height="972" alt="image" src="https://github.com/user-attachments/assets/c0f63465-5d49-4344-b51b-a2862815f4e1" />



### 1.3 Configuración de Red de Gestión
Se asignó una IP estática para la administración del servidor.
* **IP:** `192.168.1.100`
* **Hostname:** `hyperoot-server`

**[AQUÍ VA TU CAPTURA DEL RESUMEN FINAL DE INSTALACIÓN - Imagen 1000384104.jpg]**

---

## 🛠️ 2. Fase de Post-Instalación y Optimización (Shell)

Una vez completada la instalación, se accedió a la interfaz web de administración (`https://192.168.1.100:8006`). Los siguientes pasos se realizaron a través de la **Shell** del nodo.

**[AQUÍ VA TU CAPTURA DEL DASHBOARD DE PROXMOX RECIÉN INSTALADO - Imagen 224975b9.jpg]**

### 2.1 Corrección de Repositorios (No-Subscription)
Por defecto, Proxmox intenta conectar a los repositorios de pago ("Enterprise"), lo que genera errores `401 Unauthorized` al intentar actualizar.

Se ejecutó el siguiente comando en la Shell para limpiar los repositorios antiguos y habilitar el repositorio gratuito de la comunidad:

```bash
# Sobreescribir sources.list con repositorios de Debian y Proxmox No-Subscription
echo "deb [http://ftp.debian.org/debian](http://ftp.debian.org/debian) trixie main contrib
deb [http://ftp.debian.org/debian](http://ftp.debian.org/debian) trixie-updates main contrib
deb [http://security.debian.org/debian-security](http://security.debian.org/debian-security) trixie-security main contrib
deb [http://download.proxmox.com/debian/pve](http://download.proxmox.com/debian/pve) trixie pve-no-subscription" > /etc/apt/sources.list

# Eliminar archivo de suscripción enterprise residual
rm -f /etc/apt/sources.list.d/pve-enterprise.list

# Actualizar la lista de paquetes
apt update
