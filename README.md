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

<img width="1600" height="943" alt="image" src="https://github.com/user-attachments/assets/4f6f5f62-b78e-4992-b00c-d82b5372e020" />


---

## 🛠️ 2. Fase de Post-Instalación y Optimización (Shell)

Una vez completada la instalación, se accedió a la interfaz web de administración (`https://192.168.1.100:8006`). Los siguientes pasos se realizaron a través de la **Shell** del nodo.

<img width="1360" height="643" alt="image" src="https://github.com/user-attachments/assets/c880d60a-37e3-4f98-af0b-e969efd4400d" />


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

### 2.2 Configuración de Energía (Headless Mode)
​Para operar el servidor con la tapa del portátil cerrada sin que entre en suspensión, se modificó la configuración del servicio systemd-logind.

# Comando para editar el archivo de configuración
nano /etc/systemd/logind.conf

Se aplicaron los cambios reiniciando el servicio:

systemctl restart systemd-logind


🌐 3. Fase de Virtualización de Networking (MikroTik CHR)
​Para gestionar la red del laboratorio y segmentar el tráfico, se desplegó un router virtualizado MikroTik Cloud Hosted Router (CHR).
​### 3.1 Preparación de la Imagen
​Dado que los servidores de MikroTik pueden bloquear descargas directas (wget) desde terminales, se utilizó curl simulando un navegador para descargar la imagen del disco:
# Instalar unzip
apt install unzip -y

# Descargar la imagen estable de MikroTik v7
curl -L -H "User-Agent: Mozilla/5.0" -o chr.zip [https://download.mikrotik.com/routeros/7.12.1/chr-7.12.1.img.zip](https://download.mikrotik.com/routeros/7.12.1/chr-7.12.1.img.zip)

# Descomprimir
unzip chr.zip

### 3.2 Creación de la VM e Importación del Disco
​Se creó una Máquina Virtual (VM ID 100) desde la interfaz web sin disco duro. Luego, se importó la imagen descargada directamente al almacenamiento local de Proxmox:

# Importar el disco a la VM 100
qm importdisk 100 chr-7.12.1.img local-lvm

📢 4. Troubleshooting (Resolución de Problemas Clave)
​Esta sección documenta los desafíos técnicos encontrados y sus soluciones, demostrando capacidad de análisis y corrección.
​### Error 1: KVM host doesn't support requested feature (CPUID)
​Situación: Al intentar iniciar la VM de MikroTik, el arranque fallaba con un error relacionado con características del CPU no soportadas.
​Solución: Se modificó el tipo de procesador emulado en la VM.
​Ir a VM 100 -> Hardware -> Processors.
​Cambiar Type de kvm64 o x86-64-v2-AES a host.
​Explicación: Esto permite que la VM utilice directamente las instrucciones del procesador físico del ASUS, resolviendo la incompatibilidad de emulación.
​### Error 2: Bloqueo de Descarga de MikroTik (HTTP Error 501/403)
​Situación: El comando wget fallaba al intentar descargar la ISO desde el servidor oficial.
Solución: Uso de curl con el parámetro -H "User-Agent: Mozilla/5.0" para simular una petición de navegador estándar.
​📈 5. Estado Actual y Próximos Pasos
​Estado actual
​Servidor Proxmox operativo sobre ASUS Laptop con 8GB RAM.
​Tapa del portátil operativa en modo "ignore" (cerrada).
​VM 100 (MikroTik CHR) encendida y accesible vía Consola.
​Próximos Pasos
​Conexión vía Winbox: Configurar IP de gestión en MikroTik para acceso desde el Lenovo.
​Integración de Almacenamiento: Montar el disco de 2TB para datos.
​Despliegue de Docker: Crear una VM para contenedores (Nginx Proxy Manager, Nextcloud).

