

***Poner una IP a una VLAN***
Se usa para acceso remoto
```go
switch(config)# line vty 0
switch(config-line)# password [pass]
switch(config-line)# login
switch(config)# interface vlan [numero]
switch(config-if)# ip address [ip] [mask]
```

***Establecer un default gateway***
Usado para acceder remotamente si esta en otra red
```go
router(config)# ip dafault-gateway [ip]
```

***Cambiar el modo de operacion (full/half duplex)***
Se hace sobre una interfaz
```go
switch(config-if)# duplex full --> full duplex
switch(config-if)# duplex half --> half duplex
switch(config-if)# speed [numero] --> 10/100/1000 (Ethernet/fast/gigabit), por defecto es "auto"
```

***Cambiar contraseñas de un switch***
1. Conectarse por terminal (consola)
2. Desconectar el cable de alimentacion electrica
3. Presione el boton "Mode" y enchufe el cable 
4. Suelte el boton de "Mode" hasta que el led "systled" parpadee en ambar y se quede en verde fijo, al soltar el boton, parpadeea en verde (Cisco 2960)
	1. Suelte el boton "Mode" despues de 5 segundos cuando el led de estado (stat), cuando suelte el boton "mode", el indicador systled parpadea en ambar (Cisco 2950)

```go
switch: flash_init
switch: load_helper --> no obligatorio
switch: rename flash:config.text flash:config.old
switch: boot
switch> enable
switch# rename flash:config.old flash:config.text
switch# copy flash:config.text system:running-config
switch# configure terminal
switch(config)# enable secret [pass]
```


***Configurar SSH en Switches***
```go
switch(config)# ip domain-name [nombre]
switch(config)# crypto key generate rsa
switch(config)# username [usuario] secret [pass] --> USUARIO CON EL QUE INICIARAN EN EL SWITCH
switch(config)# line vty [numero-numero]
switch(config-line)# transport input ssh
switch(config-line)# login local
switch(config)# ip ssh version 2
```
***Configurar seguridad de puertos basada en MAC***
Esto permite que solo la MAC asignada pueda acceder a la interfaz
```go
--> GESTION DE ACCIONES <--
//Acciones que se haran cuando ocurra una violacion de seguridad
switch(config-if)# switchport port-security violation [protect | restrict | shutdown]
switch(config-if)# switcport port-security mac-address [MAC]

--> MODO DINAMICO <--
switch(config-if)# siwitchport port-security mac-address sticky

--> MAXIMO DE DIRECCIONES MAC PERMITIDAS <--
switch(config-if)# switchport port-security maximum [numero]

--> MOSTRAR LAS DIRECCIONES REGISTRADAS <--
switch# show port-security address

--> SOLUCION DE ERRORES <--
//Port-security not enabled on interface
switch(config-if)#switchport port-security

//Command rejected is a dynamic port
switch(config-if)#switchport mode access
```

***VLANs***
```go
--> CREAR VLAN <--
switch(config)# vlan [vlan-id]
switch(config-vlan)# name [nombre] --> OPCIONAL

--> ASIGNAR PUERTO A UNA VLAN <--
switch(config)# interface [intefaz fast o giga]
switch(config-if)# switchport mode access
switch(config-if)# switchpot access vlan [vlan-id]

--> AGRUPAR VARIAS INTERFACES PARA LA CONFIGURACION <--
switch(config)# interface range [interfaz]0/1-10

--> REDIRIJIR A UNA VLAN DE VOZ <--
//Redirije el trafico de telefonia a la vlan que se indique (sin importar si la propia interfaz ya se encuentra en una VLAN)
switch(config-if)# mls qos trust cos 
switch(config-if)# switch voice vlan [vlan-id] 

--> CONFIGURAR ENLACES TRONCALES <--
//por defecto se usa la VLAN 1 para enlace troncales
switch(config)# interface [interfaz]
switch(config-if)# switchport mode trunk
switch(config-if)# switchport trunk native vlan [vlan-id]
switch(config-if)# switchport trunk allowed vlan [vlan-id,vlan-id] //indica las VLAN que pasaran por el enlace troncal

--> CONFIGURAR ROAS (Router-on-a-Stick) <--
//se usa para comunicacion entre vlans diferentes
router(config)# interface [interfaz.[vlan-id]] //ejemplo: g0/0/0.1
router(config-subif)# encapsulation dot1q [vlan-id]
router(config-subif)# ip address [ip] [mask]

--> FACTORES A CONSIDERAR PARA VLANS TRUNK y SUBINTERFACES  <--
/*
1. Las VLAN troncales se configuran en todas las interfaces de los switches que se conectan entre si, ej: la interfaz que conecta el switch1 con el switch2

2. La VLANs troncales deben de apuntar a las mismas VLANs en todos los switches

3. Las VLANs creadas en la topologia se deben de crear en todos los switches, no importa que no tengan un puerto asignado en modo de acceso ni una interfaz

4. La VLAN nativa debe ser la misma en todos los switches

5. La interfaz que se conecta al router tambien se debe indicar como trunk tomando en cuenta todo lo anterior

6. En la configuracion de las subinterfaces en el router, es importante indicar el mismo ID de la VLAN en la subinterfaz y cuando se habilita el encapsulamiento, ej: En la topologia se configuro la "vlan 2", entonces en el router se hara referencia de esta forma cuando se cree la subinterfaz "router(config)# interface g0/0/0.2", y posteriormente en el encapsulamiento debe ser el mismo ID de la VLAN: encapsulation dot1q 2, ambos numeros van de acuerdo a la vlan de los switches y de la topologia en general (la VLAN troncal no es necesaria indicarla)
*/

--> ELIMINAR LA CONFIGURACION DE TODAS LAS VLANS <---
switch# delete flash:vlan.dat
```
***VTP***
```go
--> CONFIGURAR VTP (SERVIDOR) <--
router(config)# vtp mode server
router(config)# vtp domain [nombre]
router(config)# vtp password [pass]

--> CONFIGURAR VTP (CLIENTE) <--
router(config)# vtp mode client
router(config)# vtp domain [nombre]
router(config)# vtp password [pass]

--> FACTORES A CONSIDERAR <--
/*
1. El dominio y password deben ser iguales
2. Las VLANs se configuran en el servidor
3. Las interfaces deben de estar configuradas en modo troncal
4. Para las VLAN extendidas se debe de habilitar el modo transparente
*/*
--> COMANDOS DE AYUDA <---
switch# show vtp status
```

***DTP***
```go
--> MODOS DTP EN UNA INTERFAZ <--
router(config-if)# swichport mode access
router(config-if)# swichport mode trunk //enlace troncal estatico sin negociacion
router(config-if)# swichport mode dynamic auto //escucha solicitudes DTP para crear el troncal pero no inicia la negociacion
router(config-if)# swichport mode dynamic desirable //inicia la negociacion para crear el troncal

--> FACTORES A CONSIDERAR EN DTP <--
/*
RESULTADO DE LAS DIFERENTES COMBINACIONES DE MODOS
1. access + access = access
2. access + trunk = inconsistencia
3. trunk + trunk = trunk
4. trunk + auto = access
5. trunk + desirable = trunk
6. auto + auto = access
7. auto + desirable = trunk
8. desirable + desirable = trunk
*/

--> IDENTIFICAR EL MODO DTP <--
router# show dtp interface [interfaz]
```

***Switching L3***
Usado para router intervlans
```go
//esta configuracion solo aplica en el MLS (multilayer switch).
MLS(config)# interface vlan [vlan-id]
MLS(config-if)# ip address [dir-gateway] [mask]

--> AGREGAR IP A UNA INTERTAZ GIGA O FAST <--
MLS(config)# interface [interfaz]
MLS(config-if)# no switchport
MLS(config-if)# ip address [ip] [mask]

--> PONER UNA INTERFAZ COMO TRONCAL <--
MLS(config)# interfaz [interfaz]
MLS(config-if)# switchport trunk encapsulation dot1q
MLS(config-if)# switchport mode trunk
MLS(config-if)# switchport trunk native vlan [vlan-id]
MLS(config-if)# switchport trunk allowed vlan [vlan-id...n]

--> FACTORES A CONSIDERAR <--
/*
1. Las VLANs que se desean enrutar se deben de crear con el comando "vlan" desde configuracion global
2. Al agregar una IP a a la VLAN para poder hacer el enrutamiento, siempre debe ser la direccion de gateway correspondiente a la VLAN
*/
```

