Modelo de Router : TL-MR3420 v5

entrar en la pagina oficial de OpenWrt y descargar el firmware correcto para mi dispositivo 
al descargarlo se le cambia el nombre a "tp_recovery.bin"  guardarlo en una carpeta vacía 
conectar el router al PC en la entrada LAN y configuramos nuestra ip ethernet a estática 
con la siguiente dirección 
IP: 192.168.0.225
mask: 255.255.255.0
Gateway y dns vacíos 

Instalar un servidor TFTP en mi caso use Tftpd64 
donde al abrirlo hay que seleccionar en server interface el que dice 192.168.0.225
y que el directorio sea el mismo donde guardamos en "tp_recovery.bin"

pasos para poner el routern en modo TFTP

Apaga el TL-MR3420.
Mantén presionado el botón RESET.
Sin soltar RESET, enciende el router.
Mantén RESET aproximadamente 6–7 segundos.
El router debería comenzar a solicitar tp_recovery.bin al servidor TFTP.

una vez hecho todo eso cambiamos nuestra ip otra vez a dinámica y entramos al 
navegador sin estar conectado a internet y poner la ip 192.168.1.1
![inicio](https://github.com/Thelsreaper/homelab-/blob/d0bb87c097cfcbd994487cef7f32c98d5a017463/Imagenes%20Lab/inicio.png)

------------------------Ahora pasamos a configurar el router con el nuevo software---------------------------

cambiamos la contraseña del dispositivo porque a iniciar por primera vez no tendrá ninguna 
nos dirigimos a:
System --> administración ->Router Password
ahí cambiamos la contraseña para luego activar el ssh entrando a 

System --> administración -> SSH Access
-------------------------------------------------------
Enable Instance

![Config-ssh](https://github.com/Thelsreaper/homelab-/blob/4e2fec586ee8d8966ca684211bcdd1fd5e23b9c3/Imagenes%20Lab/Config-ssh.png)


-------------------------------------------------------

Luego nos movemos al apartado de SSH-Keys
ahí tenemos que generar nuestra clave ssh en el cmd o terminal 
utilizamos el "ssh-keygen -t ed25519"

ssh-keygen → programa que genera claves SSH.
-t ed25519 → indica que quieres usar el algoritmo Ed25519, que es moderno y rápido.

al generarlo no entrega una clave publica y otra privada usaremos la publica que es la que 
tiene terminación .pub esa misma la pegaremos en SSH-Keys 

----------------------Creación de Vlans y sus configuraciones-----------------------------

Para crear una vlan nos dirigimos a 
Network -> Switch

![Vlans](https://github.com/Thelsreaper/homelab-/blob/0c016cd5dc4b7133abed48d22aec6016b115008a/Imagenes%20Lab/VLans.png)


----------------------Creación de interfaces y sus configuraciones-----------------------------

Para crear las nuevas interfaces o solamente configurarlas no dirigimos a 
Network -> interfaces 
ahí nos dirigimos donde dice "add new interface" 
donde nos pedira el nombre de la interfaz, protocolo y device
por el momento completamos las primeras 2 por ejemplo
Administración
static address 
ipv4 : 192.168.10.1/24


![Interfaces](https://github.com/Thelsreaper/homelab-/blob/d0bb87c097cfcbd994487cef7f32c98d5a017463/Imagenes%20Lab/Interfaces.png)

----------------------Creación de reglas de firewall y otros -----------------------------
Para la creación de reglas de firewall nos dirigimos a 
Network -> firewall 
al ingresas no mostrara el Zone settings del firewall
![Firewall Zones](https://github.com/Thelsreaper/homelab-/blob/d0bb87c097cfcbd994487cef7f32c98d5a017463/Imagenes%20Lab/Firewall%20Zones.png)
después no dirigimos a donde dice traffic Rules donde creares en esencia 2 reglas que serian para el puerto 53 para el DNS y el puerto 67 y 68 para el DHCP 

![Traffic rules](https://github.com/Thelsreaper/homelab-/blob/d0bb87c097cfcbd994487cef7f32c98d5a017463/Imagenes%20Lab/Traffic%20rules.png)
