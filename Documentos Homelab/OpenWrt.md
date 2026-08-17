# Instalación y Configuración de OpenWrt en TL-MR3420 v5

## 1. Flasheo inicial mediante TFTP 

1. **Descarga del firmware**: Entra en la página oficial de OpenWrt y descarga el firmware correcto para tu dispositivo (TL-MR3420 v5).
2. **Renombra el archivo**: Cámbiale el nombre a `tp_recovery.bin` y guárdalo en una carpeta vacía.
3. **Configuración de red del PC**:
   - Conecta el router al PC en el puerto **LAN**.
   - Configura tu tarjeta Ethernet con una **IP estática** con los siguientes datos:
     - **IP**: `192.168.0.225`
     - **Máscara**: `255.255.255.0`
     - **Puerta de enlace (Gateway) y DNS**: vacíos.
4. **Servidor TFTP**: Instala un servidor TFTP (en mi caso usé **Tftpd64**). Al abrirlo:
   - En **Server interface**, selecciona la que dice `192.168.0.225`.
   - En **Directory**, elige la misma carpeta donde guardaste `tp_recovery.bin`.
5. **Poner el router en modo TFTP** (sigue este orden estrictamente):
   - Apaga el TL-MR3420.
   - Mantén presionado el botón **RESET**.
   - Sin soltar RESET, enciende el router.
   - Mantén RESET presionado durante **6 o 7 segundos**.
   - El router comenzará a solicitar automáticamente el archivo `tp_recovery.bin` al servidor TFTP. Espera a que termine la transferencia.
6. **Finalizada la instalación**: Vuelve a cambiar la IP de tu PC a **modo dinámico (DHCP)**.
7. **Acceso al router**: Abre el navegador (sin necesidad de estar conectado a Internet) y escribe la IP `192.168.1.1`. Deberías ver la pantalla de inicio de OpenWrt.
   
![inicio](https://github.com/Thelsreaper/homelab-/blob/d0bb87c097cfcbd994487cef7f32c98d5a017463/Imagenes%20Lab/inicio.png)

## 2. Configuración inicial del nuevo software

Al iniciar sesión por primera vez, el router no tiene contraseña. Es obligatorio establecer una para habilitar la seguridad y el acceso SSH.

### 2.1 Cambiar la contraseña del router
- Ve a **System → Administration → Router Password**.
- Establece una contraseña robusta y **guarda los cambios** (`Save & Apply`).

### 2.2 Activar el servidor SSH
- Ve a **System → Administration → SSH Access**.
- Marca la casilla **Enable Instance** para activar el servicio SSH.
- Haz clic en **Save & Apply**.
### 2.3 Añadir tu clave pública SSH (para acceso sin contraseña)
1. Abre tu terminal (CMD o PowerShell en Windows, o tu terminal en Linux/Mac).
2. Genera un par de claves con el algoritmo Ed25519 (moderno y seguro):
   ```bash
   ssh-keygen -t ed25519
Esto creará dos archivos: id_ed25519 (clave privada)
y id_ed25519.pub (clave pública).

Copia el contenido del archivo id_ed25519.pub.

En la interfaz web de OpenWrt, ve a la sección SSH-Keys y pega el contenido de tu clave pública.

Haz clic en Save & Apply.

![Config-ssh](https://github.com/Thelsreaper/homelab-/blob/4e2fec586ee8d8966ca684211bcdd1fd5e23b9c3/Imagenes%20Lab/Config-ssh.png)


## 3. Creación de VLANs
Las VLANs nos permiten segmentar la red en subredes lógicas independientes.

### 1. Ve a Network → Switch.

Aquí verás los puertos físicos del router (LAN1, LAN2, LAN3, LAN4 y la CPU).

Para crear una nueva VLAN (ej. VLAN 10 para "Administración"):

Haz clic en Add (o modifica una existente).
Asigna un VLAN ID, por ejemplo 10.
Configura los puertos:
Puertos LAN (hacia dispositivos finales): Ponlos en modo untagged (esto significa que los dispositivos conectados ahí no necesitan saber de VLANs).
Puerto CPU (el que va al procesador): Ponlo en modo tagged (para que el router pueda gestionar el tráfico de esta VLAN internamente).
Haz clic en Save & Apply.
Nota: Si quieres varias VLANs (ej. 10, 20, 30), repite este proceso asignando diferentes puertos untagged para cada una. El puerto CPU siempre debe ir tagged en todas ellas.

![Vlans](https://github.com/Thelsreaper/homelab-/blob/0c016cd5dc4b7133abed48d22aec6016b115008a/Imagenes%20Lab/VLans.png)


## 4. Creación de Interfaces (asociadas a las VLANs)
Ahora debemos crear una interfaz lógica para cada VLAN, asignarle una IP y decirle qué dispositivo físico (la VLAN) debe usar.

Ve a Network → Interfaces.

Haz clic en Add new interface.

Rellena los campos:

Name of the new interface: Ponle un nombre descriptivo, por ejemplo Administracion.

Protocol of the new interface: Elige Static address (IP fija).

Device: Debes seleccionar el dispositivo que corresponde a tu VLAN creada. Normalmente será algo como eth0.10 (si tu VLAN tiene ID 10) o br-lan.10. Asegúrate de elegir el que acabamos de crear.

Haz clic en Submit.

Ahora configura la IP:

IPv4 address: Pon la IP que quieras para el router en esa red, por ejemplo **192.168.10.1**.

IPv4 netmask: **255.255.255.0** (o /24).

IPv4 gateway: Déjalo vacío (el router es la puerta de enlace de esta red).

DNS servers: Puedes dejar vacío o poner **8.8.8.8** para pruebas, aunque normalmente se hereda de la WAN.

Haz clic en Save & Apply.

![Interfaces](https://github.com/Thelsreaper/homelab-/blob/d0bb87c097cfcbd994487cef7f32c98d5a017463/Imagenes%20Lab/Interfaces.png)

## 5. Configuración del servidor DHCP para las nuevas VLANs
Para que los dispositivos conectados a la VLAN 10 obtengan IP automáticamente, debemos activar el DHCP en esa interfaz.

Ve a Network → DHCP and DNS.

Desplázate hacia abajo hasta la sección DHCP Server y haz clic en la pestaña de tu nueva interfaz (ej. Administracion) o en Add si no aparece.

Marca Ignore interface en desmarcado (para que NO ignore, es decir, que sí sirva DHCP).

Configura un rango de IPs (pool):

Start: **192.168.10.100**

Limit: 150 (esto dará desde .100 hasta .250).

Lease time: Déjalo por defecto (12h).

Haz clic en Save & Apply.


## 6. Configuración del Firewall (Reglas y Zonas)
Para que la nueva VLAN pueda navegar a Internet y recibir DHCP/DNS correctamente, debemos asignarla a la zona de seguridad adecuada y permitir el tráfico.

### 6.1 Asignar la nueva interfaz a una Zona
Ve a Network → Firewall.

En la pestaña Zones:

Busca la zona lan (que suele ser la zona confiable por defecto) y haz clic en Edit.

En Covered networks, asegúrate de que esté marcada tu nueva interfaz (ej. Administracion). Si no está, agrégala.

La opción Allow forward to destination zones debe tener marcada wan.

La opción Allow forward from source zones puede tener marcada wan (o dejarlo solo en lan si quieres aislamiento, pero para Internet necesita salir a wan).

Masquerading: Márcalo (esto es el NAT, necesario para que los dispositivos internos salgan a Internet con la IP pública del router).

Haz clic en Save & Apply.

### 6.2 Crear reglas de tráfico (para DHCP y DNS)
Las reglas de firewall permiten el paso de paquetes específicos. El puerto 53 (DNS) y los puertos 67/68 (DHCP) deben estar abiertos dentro de la zona LAN (y no hacia la WAN).

Ve a Network → Firewall → Traffic Rules.

Regla para DHCP (puertos 67 y 68):

Haz clic en Add.

Name: Allow DHCP.

Protocol: UDP.

Source zone: lan (o la que contenga tus VLANs).

Destination zone: Device (input).

Destination port: 67-68.

Action: accept.

Haz clic en Save & Apply.

Regla para DNS (puerto 53):

Haz clic en Add.

Name: Allow DNS.

Protocol: UDP y TCP (o solo UDP).

Source zone: lan.

Destination zone: Device (input).

Destination port: 53.

Action: accept.

Haz clic en Save & Apply.
![Firewall Zones](https://github.com/Thelsreaper/homelab-/blob/d0bb87c097cfcbd994487cef7f32c98d5a017463/Imagenes%20Lab/Firewall%20Zones.png)

![Traffic rules](https://github.com/Thelsreaper/homelab-/blob/d0bb87c097cfcbd994487cef7f32c98d5a017463/Imagenes%20Lab/Traffic%20rules.png)


## 7. ADVERTENCIA  (Cambio de IP de acceso)
Si has modificado la IP de la interfaz LAN (por ejemplo, de **192.168.1.1** a **192.168.10.1**), dejarás de acceder al router por la IP antigua. A partir de ese momento, para seguir configurando o administrando el router, deberás:

Configurar la IP de tu PC en el rango de la nueva red (ej. **192.168.10.2** con máscara **255.255.255.0**), o poner tu PC en modo DHCP (si ya configuraste el servidor DHCP en esa VLAN).

