# Análisis de paquetes de red ante incidente de enumeración de usuarios por el servicio de Kerberos

![/imgs/DJNXa3kXUAAsDnW.jpg](/imgs/DJNXa3kXUAAsDnW.jpg)

---

### Entorno y requisitos

El entorno en el que nos encontramos para realizar esta práctica, es un Windows Server 2016 con el servicio de Kerberos activo, en el que se encuentran 3 usuarios registrados en el dominio, también utilizaremos una máquina Linux con el sistema operativo de Parrot, con la aplicación Kerbrute instalada; ambas máquinas estarán conectadas por NAT con la misma interfaz de red para poder realizar el ataque y poder escanear a bajo nivel los paquetes solicitados y enviados con Wireshark, este programa deberá estar instalado en el Windows Server 2016 capturando los paquetes antes de realizar el ataque.

---

### **Realización del ataque**

Tras tener el entorno preparado utilizaremos la herramienta Kerbrute y ejecutaremos el siguiente comando con los siguientes parámetros repetidas veces para que nos salga una gran cantidad de solicitudes en Wireshark, este comando lo que nos permite es enumerar los usuarios registrados al Dominio utilizando el servicio de Kerberos y pasándole una diccionario con los usuarios.

```bash
./kerbrute_linux_amd64 userenum --dc 192.168.68.131 -d Yericorporation.local users.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 12/20/22 - Ronnie Flathers @ropnop

2022/12/20 00:31:54 >  Using KDC(s):
2022/12/20 00:31:54 >  	192.168.68.131:88

2022/12/20 00:31:54 >  [+] VALID USERNAME:	 Daniel@Yericorporation.local
2022/12/20 00:31:54 >  [+] VALID USERNAME:	 Nurimy@Yericorporation.local
2022/12/20 00:31:54 >  [+] SVC_SQLService has no pre auth required. Dumping hash to crack offline:
$krb5asrep$18$SVC_SQLService@YERICORPORATION.LOCAL:c45e503e04bf0fa9fc6183e5a9b550a1$b4440364049c19827bb458a33ab6ea775b4a7d4e9f5a614a54afb6f6376243fadf19c4981b3dd59f8569865a2af950fb67f3306909bcb8819757d50283621597f6fad44c581bee4647433dc808ad8e6713cb03fc996d8d2a020c1a16ae05ee491486c070f5772ddfc477d69ba1bd6ee0421608005809de6cf480d03505c4e4a5249c42f6587c6b6cf1b7fcf38c27aa984bca341ae8a0696bc4ebf09529645f8cc828e5a75a97bf4996a51396168df53d57d03aaa2620bda06b38fab9dcc0d3a354a765e5a33a648eb51d26d6f1872e8c3182035f040a47fcb8022f9abf9a575369a72427bdad23a6b1d4a5eec458fa7e3b83b8e275dee160e1ba5096914ceb40a2662934a64b045a5408db87ad39b315e01788c2
2022/12/20 00:31:54 >  [+] VALID USERNAME:	 SVC_SQLService@Yericorporation.local
```

![/imgs/Enumeración de kerberos ataque.png](/imgs/Enumeracin_de_kerberos_ataque.png)

| Argumentos | Descripción |
| --- | --- |
| userenum | Indica la enumeración de usuarios a un dominio e IP especificados a través de un diccionario con usuarios. |
| -dc | Sirve para indicar la IP del dominio, si se deja en blanco intentará buscarlo a través del DNS |
| -d | Sirve para indicar el nombre del Dominio del Active Directory |
| users.txt | Es un diccionario que contiene los usuarios a enumerar por el servicio de Kerberos |

| Usuarios | Prea-auth |
| --- | --- |
| Daniel@Yericorporation.local | Requiere autentificación previa en el Dominio para iniciar sesión |
| SVC_SQLService@Yericorporation.local | No requiere autentificación previa para iniciar sesión en el Dominio |
| Nurimy@Yericorporation.local | Requiere autentificación previa en el Dominio para iniciar sesión |

---

### **Analizando los paquetes con wireshark**

Una vez capturados los paquetes, nos saldrá los siguientes paquetes recibidos utilizando el servicio KRB5.

![/imgs/paquetes captados.png](/imgs/paquetes_captados.png)

Como podemos visualizar, nos indica por orden los paquetes capturados, el tiempo que ha durado el paquete, la IP que envió el paquete, la IP de destino, el servicio que utilizan los paquetes que en este caso es KRB5 y en la ‘info’ se puedan observar tres respuestas distintas.

| Información | Descripción |
| --- | --- |
| AS-REQ |  Es un mensaje enviado al servicio de autenticación donde solicita los tickets que suelen ser contraseñas. |
| AS_REP | Envía gracias a la solicitud pedida el ticket encriptado comprobándolo con el centro de distribución de claves |
| KRB Error: KRB5KDC_ERR_PREAUTH_REQUIRED | Envía información indicando que el usuario con el cual se probó la enumeración no se ha autenticado. |

---

### AS-REQ

En este primer apartado podemos observar:

- La interfaz del dispositivo que se ha utilizado para filtrar los paquetes ‘\Device\NPF_{2B10…AA21}’
- La interfaz de red utilizada fue Ethernet0.
- La fecha a la que llego el paquete (19, 2002 23:31:53 Hora estándar)
- El tiempo utilizado para la encapsulación (0.2206580000)
- Los bytes enviados (207)
- El protocolo de la interfaz donde indica que ha utilizado la interfaz ethernet, indicando la IP, el protocolo utilizado que en este caso fue UDP y al servicio dirigido que es Kerberos (eth:ethertype:ip:udp:kerberos)

```bash
Frame 24: 207 bytes on wire (1656 bits), 207 bytes captured (1656 bits) on interface \Device\NPF_{2B10E89C-EC33-49CC-8D8C-49647AB9AA21}, id 0
    Section number: 1
    Interface id: 0 (\Device\NPF_{2B10E89C-EC33-49CC-8D8C-49647AB9AA21})
        Interface name: \Device\NPF_{2B10E89C-EC33-49CC-8D8C-49647AB9AA21}
        Interface description: Ethernet0
    Encapsulation type: Ethernet (1)
    Arrival Time: Dec 19, 2022 23:31:54.328064000 Hora est�ndar romance
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1671489114.328064000 seconds
    [Time delta from previous captured frame: 0.220658000 seconds]
    [Time delta from previous displayed frame: 0.220658000 seconds]
    [Time since reference or first frame: 4.220131000 seconds]
    Frame Number: 24
    Frame Length: 207 bytes (1656 bits)
    Capture Length: 207 bytes (1656 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: eth:ethertype:ip:udp:kerberos]
    [Coloring Rule Name: UDP]
    [Coloring Rule String: udp]
```

En este segundo apartado podemos observar:

- La tarjeta de red utilizada que en este caso es ‘Ethernet II’
- Indica el origen el cuál es una máquina ‘VMware_4c:bd:50’
- Indica el destino el cuál es ‘VMware_eb:fc:2e’
- Indica el tipo de IP que en este caso es IPv4

```bash
Ethernet II, Src: VMware_4c:bd:50 (00:0c:29:4c:bd:50), Dst: VMware_eb:fc:2e (00:0c:29:eb:fc:2e)
    Destination: VMware_eb:fc:2e (00:0c:29:eb:fc:2e)
        Address: VMware_eb:fc:2e (00:0c:29:eb:fc:2e)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Source: VMware_4c:bd:50 (00:0c:29:4c:bd:50)
        Address: VMware_4c:bd:50 (00:0c:29:4c:bd:50)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Type: IPv4 (0x0800)
```

En este tercer apartado podemos observar:

- La versión de protocolo utilizada que este caso es la 4 haciendo referencia al IPv4
- La IP de origen ‘192.168.68.134’
- La IP de destino ‘192.168.68.131’
- La longitud de la cabecera enviada (20 bytes)
- La longitud total enviada (193 bytes)
- El Time to Live al ser 64 indica que es una máquina Linux.
- El protocolo utilizado (UDP)
- La verificación de la cabecera que en este caso está deshabilitada.(validation disabled)

```bash
Internet Protocol Version 4, Src: 192.168.68.134, Dst: 192.168.68.131
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
        0000 00.. = Differentiated Services Codepoint: Default (0)
        .... ..00 = Explicit Congestion Notification: Not ECN-Capable Transport (0)
    Total Length: 193
    Identification: 0xaf45 (44869)
    010. .... = Flags: 0x2, Don't fragment
        0... .... = Reserved bit: Not set
        .1.. .... = Don't fragment: Set
        ..0. .... = More fragments: Not set
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 64
    Protocol: UDP (17)
    Header Checksum: 0x808c [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 192.168.68.134
    Destination Address: 192.168.68.131
```

En este cuarto apartado podemos observar:

- El puerto de origen (40216)
- El puerto de destino (88)
- La longitud en bytes (173)
- Cheksum indica el estado de la verificación, que en este caso esta sin verificar (Unverified)
- UDP payload indica la longitud del protocolo UDP (165 bytes)

```bash
User Datagram Protocol, Src Port: 40216, Dst Port: 88
    Source Port: 40216
    Destination Port: 88
    Length: 173
    Checksum: 0x8aba [unverified]
    [Checksum Status: Unverified]
    [Stream index: 10]
    [Timestamps]
        [Time since first frame: 0.000000000 seconds]
        [Time since previous frame: 0.000000000 seconds]
    UDP payload (165 bytes)
```

En este quinto apartado podemos observar:

- msg-type indica el tipo de mensaje que en este caso es (krb-as-req)
- cname-string indica el nombre del usuario ‘Daniel’
- realm indica el dominio que este caso es ‘YERICORPORATION.LOCAL’
- snamestring indica el nombre del usuario en este caso es ‘krbtgt’ y el dominio ‘YERICORPORATION.LOCAL’
- till indica la fecha que se mando el paquete ‘Dec  21, 2022 00:31:53 Hora estándar)
- etype indica el tipo de encriptación que en este caso son tres (eTYPE-AES256-CTS-HMAC-SHA1-96 | eTYPE-AES128-CTS-HMAC-SHA1-96 | eTYPE-ARCFOUR-HMAC-MD5)

```bash
Kerberos
    as-req
        pvno: 5
        msg-type: krb-as-req (10)
        padata: 0 items
        req-body
            Padding: 0
            kdc-options: 00000010
            cname
                name-type: kRB5-NT-PRINCIPAL (1)
                cname-string: 1 item
                    CNameString: Daniel
            realm: YERICORPORATION.LOCAL
            sname
                name-type: kRB5-NT-SRV-INST (2)
                sname-string: 2 items
                    SNameString: krbtgt
                    SNameString: YERICORPORATION.LOCAL
            till: Dec 21, 2022 00:31:53.000000000 Hora est�ndar romance
            nonce: 956098604
            etype: 3 items
                ENCTYPE: eTYPE-AES256-CTS-HMAC-SHA1-96 (18)
                ENCTYPE: eTYPE-AES128-CTS-HMAC-SHA1-96 (17)
                ENCTYPE: eTYPE-ARCFOUR-HMAC-MD5 (23)
```

---

### AS-REP

En este primer apartado podemos observar:

- La interfaz del dispositivo que se ha utilizado para filtrar los paquetes ‘\Device\NPF_{2B10…AA21}’
- La interfaz de red utilizada fue Ethernet0.
- La fecha a la que llego el paquete (19, 2002 23:31:53 Hora estándar)
- El tiempo utilizado para la encapsulación (0.000265000)
- Los bytes enviados (834)
- El protocolo de la interfaz donde indica que ha utilizado la interfaz ethernet, indicando la IP, el protocolo utilizado que en este caso fue UDP y al servicio dirigido que es Kerberos (eth:ethertype:ip:udp:kerberos)

```bash
Frame 22: 834 bytes on wire (6672 bits), 834 bytes captured (6672 bits) on interface \Device\NPF_{2B10E89C-EC33-49CC-8D8C-49647AB9AA21}, id 0
    Section number: 1
    Interface id: 0 (\Device\NPF_{2B10E89C-EC33-49CC-8D8C-49647AB9AA21})
        Interface name: \Device\NPF_{2B10E89C-EC33-49CC-8D8C-49647AB9AA21}
        Interface description: Ethernet0
    Encapsulation type: Ethernet (1)
    Arrival Time: Dec 19, 2022 23:31:53.961082000 Hora est�ndar romance
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1671489113.961082000 seconds
    [Time delta from previous captured frame: 0.000265000 seconds]
    [Time delta from previous displayed frame: 0.000265000 seconds]
    [Time since reference or first frame: 3.853149000 seconds]
    Frame Number: 22
    Frame Length: 834 bytes (6672 bits)
    Capture Length: 834 bytes (6672 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: eth:ethertype:ip:udp:kerberos]
    [Coloring Rule Name: UDP]
    [Coloring Rule String: udp]
```

En este segundo apartado podemos observar:

- La tarjeta de red utilizada que en este caso es ‘Ethernet II’
- Indica el origen el cuál es una máquina ‘VMware_eb:fc:2e’
- Indica el destino el cuál es ‘VMware_4c:bd:50’
- Indica el tipo de IP que en este caso es IPv4

```bash
Ethernet II, Src: VMware_eb:fc:2e (00:0c:29:eb:fc:2e), Dst: VMware_4c:bd:50 (00:0c:29:4c:bd:50)
    Destination: VMware_4c:bd:50 (00:0c:29:4c:bd:50)
    Source: VMware_eb:fc:2e (00:0c:29:eb:fc:2e)
    Type: IPv4 (0x0800)
```

En este tercer apartado podemos observar:

- La versión de protocolo utilizada que este caso es la 4 haciendo referencia al IPv4
- La IP de origen ‘192.168.68.131’
- La IP de destino ‘192.168.68.134’
- La longitud de la cabecera enviada (20 bytes)
- La longitud total enviada (820 bytes)
- El Time to Live al ser 128 indica que es una máquina Windows.
- El protocolo utilizado (UDP)
- La verificación de la cabecera que en este caso está deshabilitada (validation disabled)

```bash
Internet Protocol Version 4, Src: 192.168.68.131, Dst: 192.168.68.134
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
        0000 00.. = Differentiated Services Codepoint: Default (0)
        .... ..00 = Explicit Congestion Notification: Not ECN-Capable Transport (0)
    Total Length: 820
    Identification: 0x7222 (29218)
    000. .... = Flags: 0x0
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 128
    Protocol: UDP (17)
    Header Checksum: 0x0000 [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 192.168.68.131
    Destination Address: 192.168.68.134
```

En este cuarto apartado podemos observar:

- El puerto de origen (88)
- El puerto de destino (33572)
- La longitud en bytes (800)
- Chekcsum indica el estado de la verificación, que en este caso esta sin verificar (Unverified)
- UDP payload indica la longitud del protocolo UDP (792 bytes)

```bash
User Datagram Protocol, Src Port: 88, Dst Port: 33572
    Source Port: 88
    Destination Port: 33572
    Length: 800
    Checksum: 0x0d8c [unverified]
    [Checksum Status: Unverified]
    [Stream index: 9]
    [Timestamps]
    UDP payload (792 bytes)
```

En este quinto apartado podemos observar:

- msg-type indica el tipo de mensaje que en este caso es (krb-as-rep)
- El valor de la pre-autenticación por el protocolo de kerberos (302e302ca0…)
- El Etype-info2-entry indica el tipo de encriptación que utiliza el centro de distribución de claves (eTYPE-AES256-CTS-HMAC-SHA1-96) aparte de indicar el dominio y usuario con el que se solicita la clave (YERICORPORATION.LOCALSVC_SQLService)
- Crealm indica el dominio que este caso es ‘YERICORPORATION.LOCAL’
- CNameString indica el nombre del usuario en este caso ‘SVC_SQService’
- KRB5-NT-SRV-INST (2) Indica la cuenta por defecto que hay en un directorio activo en este caso ‘krbtgt’ y el dominio que en este caso es ‘YERICORPORATION.LOCAL’
- Enc-part indica los datos encriptados que en este caso se utiliza ‘eTYPE-AES256-CTS-HMAC-SHA1-96’
- Kvno indica el número versión de la clave, en este caso al ser 2 significa que el usuario cambió la clave por defecto 1 vez, por eso está se ha incrementado.
- Cipher indica la parte del ticket del usuario cifrada con el hash de la contraseña.

```bash
Kerberos
    as-rep
        pvno: 5
        msg-type: krb-as-rep (11)
        padata: 1 item
            PA-DATA pA-ETYPE-INFO2
                padata-type: pA-ETYPE-INFO2 (19)
                    padata-value: 302e302ca003020112a1251b2359455249434f52504f524154494f4e2e4c4f43414c5356…
                        ETYPE-INFO2-ENTRY
                            etype: eTYPE-AES256-CTS-HMAC-SHA1-96 (18)
                            salt: YERICORPORATION.LOCALSVC_SQLService
        crealm: YERICORPORATION.LOCAL
        cname
            name-type: kRB5-NT-PRINCIPAL (1)
            cname-string: 1 item
                CNameString: SVC_SQLService
        ticket
            tkt-vno: 5
            realm: YERICORPORATION.LOCAL
            sname
                name-type: kRB5-NT-SRV-INST (2)
                sname-string: 2 items
                    SNameString: krbtgt
                    SNameString: YERICORPORATION.LOCAL
            enc-part
                etype: eTYPE-AES256-CTS-HMAC-SHA1-96 (18)
                kvno: 2
                cipher: ff6d774e32e08b30ed92f6065ef2b0b4d551364af145306f32b50c77ef77e0a66e6914b5…
        enc-part
```

---

### KRB Error: KRB5KDC_ERR_PREAUTH_REQUIRED

En este primer apartado podemos observar:

- La interfaz del dispositivo que se ha utilizado para filtrar los paquetes ‘\Device\NPF_{2B10…AA21}’
- La interfaz de red utilizada fue Ethernet0.
- La fecha a la que llego el paquete (19, 2002 23:31:54 Hora estándar)
- El tiempo utilizado para la encapsulación (0.000192000)
- Los bytes enviados (255)
- El protocolo de la interfaz donde indica que ha utilizado la interfaz ethernet, indicando la IP, el protocolo utilizado que en este caso fue UDP y al servicio dirigido que es Kerberos (eth:ethertype:ip:udp:kerberos)

```bash
Frame 27: 255 bytes on wire (2040 bits), 255 bytes captured (2040 bits) on interface \Device\NPF_{2B10E89C-EC33-49CC-8D8C-49647AB9AA21}, id 0
    Section number: 1
    Interface id: 0 (\Device\NPF_{2B10E89C-EC33-49CC-8D8C-49647AB9AA21})
        Interface name: \Device\NPF_{2B10E89C-EC33-49CC-8D8C-49647AB9AA21}
        Interface description: Ethernet0
    Encapsulation type: Ethernet (1)
    Arrival Time: Dec 19, 2022 23:31:54.328491000 Hora est�ndar romance
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1671489114.328491000 seconds
    [Time delta from previous captured frame: 0.000192000 seconds]
    [Time delta from previous displayed frame: 0.000192000 seconds]
    [Time since reference or first frame: 4.220558000 seconds]
    Frame Number: 27
    Frame Length: 255 bytes (2040 bits)
    Capture Length: 255 bytes (2040 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: eth:ethertype:ip:udp:kerberos]
    [Coloring Rule Name: UDP]
    [Coloring Rule String: udp]
```

En este segundo apartado podemos observar:

- La tarjeta de red utilizada que en este caso es ‘Ethernet II’
- Indica el origen el cuál es una máquina ‘VMware_eb:fc:2e’
- Indica el destino el cuál es ‘VMware_4c:bd:50’
- Indica el tipo de IP que en este caso es IPv4

```bash
Ethernet II, Src: VMware_eb:fc:2e (00:0c:29:eb:fc:2e), Dst: VMware_4c:bd:50 (00:0c:29:4c:bd:50)
    Destination: VMware_4c:bd:50 (00:0c:29:4c:bd:50)
        Address: VMware_4c:bd:50 (00:0c:29:4c:bd:50)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Source: VMware_eb:fc:2e (00:0c:29:eb:fc:2e)
        Address: VMware_eb:fc:2e (00:0c:29:eb:fc:2e)
        .... ..0. .... .... .... .... = LG bit: Globally unique address (factory default)
        .... ...0 .... .... .... .... = IG bit: Individual address (unicast)
    Type: IPv4 (0x0800)
```

En este tercer apartado podemos observar:

- La versión de protocolo utilizada que este caso es la 4 haciendo referencia al IPv4
- La IP de origen ‘192.168.68.131’
- La IP de destino ‘192.168.68.134’
- La longitud de la cabecera enviada (20 bytes)
- La longitud total enviada (241 bytes)
- El Time to Live al ser 128 indica que es una máquina Windows.
- El protocolo utilizado (UDP)
- La verificación de la cabecera que en este caso está deshabilitada (validation disabled)

```bash
Internet Protocol Version 4, Src: 192.168.68.131, Dst: 192.168.68.134
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
        0000 00.. = Differentiated Services Codepoint: Default (0)
        .... ..00 = Explicit Congestion Notification: Not ECN-Capable Transport (0)
    Total Length: 241
    Identification: 0x7223 (29219)
    000. .... = Flags: 0x0
        0... .... = Reserved bit: Not set
        .0.. .... = Don't fragment: Not set
        ..0. .... = More fragments: Not set
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 128
    Protocol: UDP (17)
    Header Checksum: 0x0000 [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 192.168.68.131
    Destination Address: 192.168.68.134
```

En este cuarto apartado podemos observar:

- El puerto de origen (88)
- El puerto de destino (60780)
- La longitud en bytes (221)
- Chekcsum indica el estado de la verificación, que en este caso esta sin verificar (Unverified)
- UDP payload indica la longitud del protocolo UDP (213 bytes)

```bash
User Datagram Protocol, Src Port: 88, Dst Port: 60780
    Source Port: 88
    Destination Port: 60780
    Length: 221
    Checksum: 0x0b49 [unverified]
    [Checksum Status: Unverified]
    [Stream index: 11]
    [Timestamps]
        [Time since first frame: 0.000427000 seconds]
        [Time since previous frame: 0.000427000 seconds]
    UDP payload (213 bytes)
```

En este quinto apartado podemos observar:

- msg-type indica el tipo de mensaje que en este caso es (krb-error)
- stime indica la fecha en este caso Dec 19, 2022 23:31:54 Hora estándar romance
- KRB5-NT-SRV-INST (2) Indica la cuenta por defecto que hay en un directorio activo en este caso ‘krbtgt’ y el dominio que en este caso es ‘YERICORPORATION.LOCAL’
- e-data indica la secuencia de los datos de autenticación previa cuando hay un error.
- el padata-value indica los datos de autenticación previa.
- El Etype-info2-entry indica el tipo de encriptación que utiliza el centro de distribución de claves (eTYPE-AES256-CTS-HMAC-SHA1-96 y eTYPE-ARCFOUR-HMAC-MD5) aparte de indicar el dominio y usuario con el que se solicita la clave (YERICORPORATION.LOCALNurimy)
- En pA-ENC-TIMESTAMP, pA-PK-AS-REQ y pA-PK-AS-REP-19 devuelve el valor ‘MISSING’ debido a que el usuario, al no meter haber metido nunca la contraseña para conectarse al Active Directory a través de Kerberos, no puede comparar está con el servicio de autenticación y por lo tanto no puede devolver el ticket ya que no se ha creado.

```bash
Kerberos
    krb-error
        pvno: 5
        msg-type: krb-error (30)
        stime: Dec 19, 2022 23:31:54.000000000 Hora est�ndar romance
        susec: 126285
        error-code: eRR-PREAUTH-REQUIRED (25)
        realm: YERICORPORATION.LOCAL
        sname
            name-type: kRB5-NT-SRV-INST (2)
            sname-string: 2 items
                SNameString: krbtgt
                SNameString: YERICORPORATION.LOCAL
        e-data: 305b3038a103020113a231042f302d3024a003020112a11d1b1b59455249434f52504f52…
            PA-DATA pA-ETYPE-INFO2
                padata-type: pA-ETYPE-INFO2 (19)
                    padata-value: 302d3024a003020112a11d1b1b59455249434f52504f524154494f4e2e4c4f43414c4e75…
                        ETYPE-INFO2-ENTRY
                            etype: eTYPE-AES256-CTS-HMAC-SHA1-96 (18)
                            salt: YERICORPORATION.LOCALNurimy
                        ETYPE-INFO2-ENTRY
                            etype: eTYPE-ARCFOUR-HMAC-MD5 (23)
            PA-DATA pA-ENC-TIMESTAMP
                padata-type: pA-ENC-TIMESTAMP (2)
                    padata-value: <MISSING>
            PA-DATA pA-PK-AS-REQ
                padata-type: pA-PK-AS-REQ (16)
                    padata-value: <MISSING>
            PA-DATA pA-PK-AS-REP-19
                padata-type: pA-PK-AS-REP-19 (15)
                    padata-value: <MISSING>
```

---

### Métodos para mitigar el ataque

- Se deben supervisar las cuentas de servicio para detectar actividad sospechosa a través del análisis de paquetes de red.
- Se deben utilizar las cuentas separadas para distintos servicios y usuarios.
- Se deben cambiar las contraseñas regularmente para evitar que el atacante rompa los hashes para obtener está, impidiendo que a la hora de conseguirlo puedo autentificarse, ya que se crearía un nuevo hash para la contraseña cambiada.
- Se debe llevar un inventario sobre la revisión de las cuentas, la desactivación y la eliminación de estas.
- Se debe asegurar que las cuentas de los usuarios tengan los mínimos privilegios posibles para realizar sus funciones.

![/imgs/fluffysleeping.webp](/imgs/fluffysleeping.webp)

## Contact

[📧 danielruizraposo02@gmail.com](mailto:adalovelace@mail.com)
