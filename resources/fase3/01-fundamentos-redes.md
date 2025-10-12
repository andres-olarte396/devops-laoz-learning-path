# Fase 3: Fundamentos de Redes

Para entender cómo los contenedores, los servicios en la nube y las aplicaciones distribuidas se comunican, es esencial tener una base sólida de los principios de redes (networking).

---

<a name="modelo-osi"></a>
## 1. Modelo OSI y TCP/IP

Son modelos conceptuales que estandarizan las funciones de un sistema de telecomunicaciones o de computación en capas.

### **Modelo OSI (Open Systems Interconnection)**

Tiene 7 capas, y es un modelo de referencia más teórico.
1. **Física:**Transmisión de bits (cables, fibra óptica).
2. **Enlace de Datos:**Direccionamiento físico (MAC), switches.
3. **Red:**Direccionamiento lógico (IP), enrutamiento, routers.
4. **Transporte:**Conexión extremo a extremo, control de flujo (TCP, UDP).
5. **Sesión:**Gestión de diálogos entre computadoras.
6. **Presentación:**Formateo de datos, encriptación.
7. **Aplicación:**Protocolos de alto nivel (HTTP, FTP, SMTP).

### **Modelo TCP/IP**

Es un modelo más práctico y es el que se utiliza en Internet. Tiene 4 capas.

1. **Acceso a la Red:**Combina las capas Física y de Enlace de OSI.
2. **Internet:**Equivalente a la capa de Red de OSI (IP, ICMP).
3. **Transporte:**Equivalente a la capa de Transporte de OSI (TCP, UDP).
4. **Aplicación:**Combina las capas de Sesión, Presentación y Aplicación de OSI (HTTP, DNS, FTP).

En DevOps, la mayoría de las veces te preocuparás por las capas de **Aplicación, Transporte e Internet**.

---

<a name="ip-dns"></a>
## 2. Direcciones IP, Subredes y DNS

### **Direcciones IP (Internet Protocol)**

Una dirección IP es un identificador único para una máquina en una red.
- **IPv4:**El más común (ej: `192.168.1.10`). Son 4 números de 0 a 255.
- **IPv6:**Más moderno y largo, para acomodar el creciente número de dispositivos (ej: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`).
- **IP Pública vs. Privada:**
  - **Pública:**Única en todo Internet. Asignada por tu proveedor de servicios (ISP).
  - **Privada:**Usada dentro de una red local (LAN). Rangos comunes: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.

### **Subredes (Subnetting)**

Es la práctica de dividir una red IP grande en redes más pequeñas o "subredes". Esto ayuda a organizar y asegurar la red.
- **Máscara de subred:**Define qué parte de la IP es la red y qué parte son los hosts (ej: `255.255.255.0` o `/24`).

### **DNS (Domain Name System)**

El DNS es como la "guía telefónica" de Internet. Traduce nombres de dominio legibles para humanos (ej: `www.google.com`) a direcciones IP numéricas (ej: `142.250.184.196`).

**Tipos de registros DNS comunes:**
- **A:**Mapea un nombre de dominio a una dirección IPv4.
- **AAAA:**Mapea un nombre de dominio a una dirección IPv6.
- **CNAME (Canonical Name):**Crea un alias de un nombre de dominio a otro.
- **MX (Mail Exchange):**Especifica los servidores de correo para un dominio.

---

<a name="protocolos"></a>
## 3. Puertos, Protocolos y Firewalls

### **Protocolos de Transporte**

- **TCP (Transmission Control Protocol):**
  - **Orientado a la conexión:**Establece una conexión (un "handshake") antes de enviar datos.
  - **Fiable:**Garantiza que todos los paquetes lleguen en orden y sin errores.
  - **Ejemplos:**HTTP/S, FTP, SSH.

- **UDP (User Datagram Protocol):**
  - **Sin conexión:**Simplemente envía los paquetes.
  - **No fiable (pero rápido):**No garantiza la entrega ni el orden. Es más veloz.
  - **Ejemplos:**DNS, streaming de video, videojuegos.

### **Puertos (Ports)**

Un puerto es un número (0-65535) que identifica un proceso o servicio específico en una máquina. Cuando te conectas a un servidor, te conectas a su IP y a un puerto específico.

**Puertos comunes:**
- **22:**SSH (Secure Shell)
- **80:**HTTP (Web)
- **443:**HTTPS (Web seguro)
- **53:**DNS
- **3306:**MySQL
- **5432:**PostgreSQL

### **Firewalls**

Un firewall es un dispositivo de seguridad que monitorea el tráfico de red entrante y saliente y decide si permitirlo o bloquearlo basándose en un conjunto de reglas de seguridad.

En DevOps, a menudo configurarás **grupos de seguridad**en la nube, que actúan como firewalls virtuales para tus servidores, permitiendo tráfico solo en los puertos que necesites (ej: permitir tráfico en el puerto 443 para un servidor web).

---

<a name="redes-nube"></a>
## 4. Redes en la Nube (Cloud Networking)

Los proveedores de nube (AWS, Azure, GCP) ofrecen servicios para que construyas tu propia red privada y aislada en la nube.

- **VPC (Virtual Private Cloud):**Tu propia sección de red privada y aislada en la nube de un proveedor. Tienes control total sobre ella.
- **Subnets:**Divisiones de tu VPC. Puedes tener subredes públicas (accesibles desde Internet) y privadas (no accesibles directamente).
- **Internet Gateway:**Permite que los recursos en tus subredes públicas se comuniquen con Internet.
- **NAT Gateway (Network Address Translation):**Permite que los recursos en tus subredes privadas inicien conexiones a Internet (ej: para descargar actualizaciones) sin permitir que Internet inicie conexiones hacia ellos.
- **Security Groups / Network Security Groups (NSGs):**Actúan como firewalls a nivel de instancia (servidor virtual) para controlar el tráfico entrante y saliente.
- **NACLs (Network Access Control Lists):**Actúan como firewalls a nivel de subred, añadiendo otra capa de seguridad.
