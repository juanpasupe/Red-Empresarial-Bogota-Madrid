# Red empresarial de Bogotá y Madrid

Informe de implementación y verificación del proyecto RCD 262

> **Proyecto final — Redes y Comunicación de Datos 2026-2**  
> **Integrantes:** Juan Pablo Cerquera y Dylan Gómez.

## Archivos del proyecto

- [Simulación de Cisco Packet Tracer](02_Simulacion/Red_Empresarial_Cerquera_Gomez.pkt)
- [Informe técnico en Word](01_Informe/Informe_RCD262_Cerquera_Gomez.docx)
- [Informe técnico en Markdown](01_Informe/Informe_RCD262_Cerquera_Gomez.md)
- [Configuraciones de dispositivos](03_Configuraciones/Configuraciones_dispositivos.txt)
- [Configuraciones de servidores](03_Configuraciones/Configuraciones_servidores.txt)
- [Pruebas DNS, DHCP y SNMP](04_Evidencias/SNMP_DHCP_DNS_verificados.txt)
- [Traza de la simulación](04_Evidencias/Traza_simulacion.txt)
- [Captura del Tracker](04_Evidencias/Tracker_Dashboard.png)

---


Integrantes: Juan Pablo Cerquera y Dylan Gómez. Entorno de prueba: Cisco Packet Tracer 9.0.1. Archivo: Red_Empresarial_Cerquera_Gomez.pkt.

## Propósito y resultado

La solución conecta dos intranets a través de una nube IPv4 y mantiene una DMZ IPv6. Se implementaron VLAN, DHCPv4, direccionamiento IPv6, DNS, web personalizada, filtros de acceso, túneles protegidos con IPsec, gestión SNMP y Employee Tracker. Las pruebas combinaron consultas a los dispositivos, tráfico de aplicación, contadores y reapertura del archivo. El panel del Tracker está en Programming de PC7; la interfaz web opcional del bonus no se desarrolló.

## Decisiones de diseño

El material recibido no incluía el anexo de asignación de X ni valores concretos para los identificadores VLAN. Se adoptaron X=1, VLAN 10 Guest, 20 Internal, 30 ServerPrinter y 99 Native, EIGRP AS100 y dominio jpcdg.net. Son decisiones del proyecto, no asignaciones confirmadas por el docente.

Para resolver el traslape de etiquetas de la figura, Guest agrupa PC1 a PC3 en Bogotá y PC5 y PC8 en Madrid; Internal agrupa PC4 y PC6 a PC7, respectivamente. Se eligió 3560-24PS para MLSW_MAD, coherente con la figura y con una de las referencias del texto; el enunciado también menciona 3650. La VLAN98 se añadió como tránsito exclusivo entre R2_ESP y MLSW_MAD.

## Metodología

Se extrajeron los requisitos y se preparó un plan de direccionamiento y cableado. Se construyó una topología nueva con routers 2811, switches 2960 y switch multicapa 3560. Se configuraron primero enlaces y VLAN, después routing y servicios, y finalmente seguridad y aplicaciones. Las verificaciones se repitieron después de guardar y reabrir. La guía de preguntas de diseño estructurado de clase y la rúbrica externa no estaban adjuntas; este informe responde las preguntas disponibles en el enunciado.

## Direccionamiento de las intranets

Las LAN IPv4 usan /28, máscara 255.255.255.240 y 14 direcciones útiles. Los gateways ocupan .1. Los cuatro pools de usuarios asignan .5 a .7 y reservan el resto para infraestructura y crecimiento. Cada segmento IPv6 usa /64. Los PCs normales emplean SLAAC; PC4 y PC7 requieren IPv6 estable para los sockets.

| Segmento | IPv4 y gateway | Prefijo IPv6 |
| --- | --- | --- |
| BOG Guest | 10.1.10.0/28; .1 | 2001:1200:EA11:10::/64 |
| BOG Internal | 10.1.20.0/28; .1 | 2001:1200:EA11:20::/64 |
| BOG ServerPrinter | 10.1.30.0/28; .1 | 2001:1200:EA11:30::/64 |
| MAD Guest | 10.2.10.0/28; .1 | 2001:1200:EB21:10::/64 |
| MAD Internal | 10.2.20.0/28; .1 | 2001:1200:EB21:20::/64 |
| MAD ServerPrinter | 10.2.30.0/28; .1 | 2001:1200:EB21:30::/64 |

### Servidores y terminales estables

| Equipo o función | Dirección |
| --- | --- |
| DMZ y gateway R1 | 2001:1200:EC11:1::/64; gateway ::1 |
| Web_Server | 2001:1200:EC11:1::10 |
| DNS_Server | 2001:1200:EC11:1::20 |
| Tracker_Server | 2001:1200:EC11:1::30 |
| PC4 y PC7 | 2001:1200:EA11:20::4; 2001:1200:EB21:20::7 |
| DHCP_Server y DNS auxiliar | 10.1.30.2 |
| Printer0 y Printer1 | 10.1.30.3 y 10.1.30.4 |
| Printer2 y Printer3 | 10.2.30.2 y 10.2.30.3 |
| Gestión SW1 SW2 SW3 | 10.1.20.8; 10.1.20.9; 10.1.20.10 |
| Gestión SW4 SW5 | 10.2.20.8; 10.2.20.9 |

Las concesiones DHCP pueden cambiar entre aperturas. Después de la última reapertura PC4 obtuvo 10.1.20.5 y PC7 10.2.20.6; las aplicaciones no dependen de esas IPv4. Sus IPv6 estáticas se comprobaron tras reabrir. Los servidores de la DMZ permanecen sin IPv4.

## Enlaces y enrutamiento

X=1 produce el bloque público 86.106.77.0/24. Se subdividió en /30 para enlaces punto a punto, con máscara 255.255.255.252, dos hosts útiles y saltos de cuatro.

| Enlace | Red IPv4 | Direcciones A y B |
| --- | --- | --- |
| R1_BOG a ISP_BOG | 86.106.77.0/30 | .1 y .2 |
| ISP_BOG a ISP_NET | 86.106.77.4/30 | .5 y .6 |
| ISP_NET a ISP_ESP | 86.106.77.8/30 | .9 y .10 |
| ISP_BOG a ISP_FL | 86.106.77.12/30 | .13 y .14 |
| ISP_FL a ISP_ESP | 86.106.77.16/30 | .17 y .18 |
| ISP_NET a ISP_FL | 86.106.77.20/30 | .21 y .22 |
| ISP_ESP a R2_ESP | 86.106.77.24/30 | .25 y .26 |

El tramo superior utiliza EIGRP100 y el inferior OSPFv2 área0. ISP_BOG redistribuye entre ambos dominios; para EIGRP se aplica métrica semilla y para OSPF se incluyen subredes. Las tablas de routing mostraron rutas D y O y alcance entre los extremos públicos. Se evita un segundo punto de redistribución para reducir posibilidades de bucles.

### VLAN y tránsito

Bogotá usa router on a stick en FastEthernet0/1 de R1_BOG. Madrid enruta mediante las SVI de MLSW_MAD, con ip routing y soporte dual IPv4/IPv6. Los troncales 802.1Q transportan 10,20,30,99 y usan VLAN99 como nativa. La VLAN98 es un enlace de acceso de tránsito: R2 10.2.255.1, MLSW 10.2.255.2, red /30; IPv6 2001:1200:EB21:FFFE::1 y ::2.

### Migración y protección

Tunnel0 transporta IPv4 privada por GRE entre 86.106.77.1 y 86.106.77.26, con 10.255.1.1 y .2/30. Tunnel6 transporta IPv6 sobre IPv4 mediante ipv6ip, con 2001:1200:ED11:100::1 y ::2/64. Rutas IPv6 estáticas alcanzan las LAN remotas y la DMZ. Se eligió encapsulación porque la nube intermedia permanece completamente IPv4; no se requiere NAT64 para estos clientes duales.

IPsec usa autenticación de laboratorio con clave compartida, AES y SHA, y un mapa criptográfico en Serial1/0 de ambos extremos. La ACL110 selecciona el tráfico entre las dos IPv4 de túnel. Se verificaron QM_IDLE, paquetes cifrados y descifrados y ping IPv6. La configuración exportada contiene credenciales exclusivamente de esta simulación académica.

## Servicios y control de acceso

DHCP_Server entrega IPv4, máscara, gateway y DNS en cuatro pools de usuarios. ip helper-address 10.1.30.2 permite servir las VLAN remotas. El pool de fábrica queda acotado a 10.1.30.5 a .7, gateway 10.1.30.1, para evitar asignar la dirección de red o el gateway. SLAAC resuelve la asignación IPv6 de los hosts que no son extremos fijos de aplicaciones.

### DNS y web

DNS_Server publica el registro AAAA www.jpcdg.net hacia 2001:1200:EC11:1::10. Se comprobó con nslookup dirigido al servidor IPv6. Para la resolución automática de los clientes duales se añadió una réplica AAAA en DHCP_Server, accesible por 10.1.30.2 y anunciada mediante DHCPv4. Este servicio adicional se justifica por las dificultades observadas al seleccionar automáticamente el DNS IPv6 en los clientes de Packet Tracer. La página y la DMZ continúan usando IPv6.

La página identifica el proyecto y los integrantes. HTTP y HTTPS están activos en el servidor; las restricciones se aplican en la entrada de cada VLAN en R1_BOG y MLSW_MAD. Las ACL IPv6 bloquean el puerto no autorizado hacia el host web y permiten el resto del tráfico IPv6.

| Origen | HTTP80 | HTTPS443 |
| --- | --- | --- |
| PC1 Guest BOG | Permite | Bloquea |
| PC4 Internal BOG | Bloquea | Permite |
| PC5 Guest MAD | Permite | Bloquea |
| PC7 Internal MAD | Bloquea | Permite |

Las ocho combinaciones se probaron por dirección IPv6. Las cuatro reglas de denegación registraron 12 coincidencias cada una durante el ensayo. También se obtuvo la página por nombre en los cuatro equipos autorizados, incluyendo PC4 después de reabrir. Un timeout aislado no demuestra una ACL: se contrastó con acceso permitido y contadores.

### SNMP

Los agentes usan JPCDG_Read_262 como comunidad RO y JPCDG_Write_262 como RW. La ACL101 permite UDP161 desde la VLAN20 hacia la propia sede, deniega otros orígenes y permite el tráfico restante. Las IP de gestión de switches están en Internal; esto evita el acceso local desde ServerPrinter sin atravesar el filtro. Se probaron GET de sysName y SET de sysLocation desde PC4 y PC7, además de denegaciones Guest y entre sedes.

GET solicita una variable MIB por UDP161 y el agente devuelve una respuesta al puerto del gestor. SET incluye OID, tipo OctetString y valor; el agente valida comunidad y permisos, cambia sysLocation y confirma el resultado. En la traza real PC4 envió GET, SW1_Intranet lo aceptó y devolvió GET response, y PC4 aceptó la respuesta.

## Employee Tracker

Beacon_EMP001 emite por Bluetooth el identificador EMP-001 y una posición estimada simulada cada segundo nominal. Su UUID de servicio es {b648cf61-90ab-42d9-8d0f-0a8126200001}. Se ubicó físicamente cerca de PC4; la posición de los iconos en Logical no determina por sí sola la cobertura de radio.

PC4 ejecuta Tracker_App. El receptor solo activa el envío después de recibir un JSON válido de la baliza. Genera X e Y dentro de 0 a 100 y selecciona aleatoriamente En reposo, En movimiento o Alta actividad. Cada mensaje incluye beacon_id, mac, x, y, estado, timestamp y seq. La identidad y presencia se obtienen por Bluetooth; las coordenadas y la actividad son simuladas, no mediciones de una persona.

Tracker_Server escucha TCP5000 en la DMZ y ejecuta Tracker_Relay. Valida el JSON y sus rangos, confirma a la App y reenvía hacia TCP5001 de PC7. Mantiene una cola limitada a 120 muestras durante una desconexión. Las conexiones se recrean mediante eventos y reintentos. El dashboard conserva el estado del empleado y confirma la recepción al servidor.

### Delimitación y flujo bidireccional

TCP entrega un flujo de bytes, por lo que cada JSON termina con salto de línea. Los receptores acumulan fragmentos hasta completar una línea y pueden procesar varias líneas de una recepción. Así se evita asumir que una llamada a send equivale siempre a una recepción completa. App a Relay y Relay a Dashboard transportan muestras; ACK EMP-001 y ACK vuelven en sentido inverso. Los ACK de aplicación se distinguen de los ACK del transporte TCP.

### Alertas verificadas

Se define la zona restringida como X>=80 e Y>=80. La primera muestra dentro de la zona inicia un temporizador; una muestra fuera lo reinicia. Al superar 60 segundos continuos se imprime ALERTA ZONA >60s EMP-001, una vez por permanencia. También se imprime ALERTA ACTIVIDAD EMP-001 cuando En reposo cambia directamente a Alta actividad.

La secuencia de demostración mantiene las primeras 65 muestras en la zona restringida para reproducir la alerta sin depender de permanecer allí por azar. Después se generan coordenadas en todo el intervalo 0 a 100. El intervalo programado es delay(1000); la carga de Packet Tracer añade tiempo de procesamiento. Se observaron ambas alertas y mensajes recibidos después de reabrir, sin errores en los registros de esa ejecución.

El personal consulta el panel en PC7 > Programming > Tracker_Dashboard (JavaScript) > main.js, en la salida inferior. La visualización es textual en tiempo real. No se reclama el bonus de visualización web.

## Validación y retos resueltos

| Prueba | Resultado observado |
| --- | --- |
| Direccionamiento y DHCP | PC4 y PC7 renovaron direcciones de sus respectivas /28 con gateway y DNS correctos. |
| Routing IPv4 | Rutas D y O; alcance entre extremos públicos. |
| Migración IPv6 | Ping por Tunnel6 y acceso desde Madrid a la DMZ IPv6. |
| VPN | QM_IDLE y contadores IPsec de cifrado y descifrado. |
| ACL web | Cuatro accesos permitidos y cuatro denegados; contadores de las reglas. |
| DNS de DMZ | AAAA resuelto consultando 2001:1200:EC11:1::20. |
| Web por nombre | Carga personalizada desde Guest e Internal de ambas sedes con su protocolo permitido. |
| SNMP local | GET sysName y SET sysLocation desde PC4 y PC7. |
| SNMP no autorizado | Timeout Guest BOG, Guest MAD y gestión entre sedes; ACL101 registró denegaciones. |
| Tracker | Recepción BLE, muestras TCP, ACK, alertas de zona y actividad. |
| Reapertura | Se conservaron IPv6 estáticas, código y DNS; se repitieron pruebas de servicios. |

Las evidencias completas están en Evidencias y en Configuraciones_dispositivos.txt. La traza de simulación conserva decisiones por capa y paquetes reales SNMP y del Tracker. Para analizar SNMP se filtra ese protocolo; para el Tracker se revisan TCP y la encapsulación de los túneles. No se afirma que toda trama de la captura haya sido aceptada: hubo una trama Bluetooth con UUID sin servicio activo durante los ensayos de registro, que se distingue de las muestras TCP aceptadas.

### Retos y decisiones

El switch multicapa requirió la plantilla SDM dual para IPv6. El tránsito se resolvió con SVI98 y se limpiaron vecinos al cambiar la interfaz. El transporte IPv6 por GRE no respondió como se esperaba en el simulador; se separó IPv4 GRE de IPv6 ipv6ip. Las IPv6 configuradas solo mediante ciertos métodos de automatización no siempre persistían: se fijaron con ipv6config y se verificaron tras reabrir. Se cambió el UUID de la baliza para evitar un conflicto de servicio.

Los programas TCP se validaron desde un contexto nuevo al reabrir; editar código mientras corre puede conservar variables del contexto anterior. Se corrigió la creación de conexiones sin reutilizar objetos ya cerrados. Packet Tracer se cerró inesperadamente durante una reapertura inicial; se recuperó el archivo guardado y se completaron aperturas y pruebas posteriores. Antes de cambiar de archivo conviene detener las aplicaciones en ejecución.

### Conclusiones

La segmentación y los filtros separan el acceso web y la administración por función. La encapsulación permite transportar IPv6 sobre la nube IPv4; IPsec protege el tramo entre extremos. Los resultados de aplicación y las respuestas del gestor aportan más evidencia que una configuración escrita. La defensa requiere comprender y repetir las pruebas, incluyendo las denegaciones.

## Contribuciones y uso de inteligencia artificial

Juan Pablo Cerquera y Dylan Gómez declaran que trabajaron juntos presencialmente y coordinaron otras partes por llamada. Las actividades técnicas conjuntas no se individualizaron. La asistencia de IA se describe a continuación.

| Integrante | Contribución registrada o declarada |
| --- | --- |
| Juan Pablo Cerquera | Aportó requisitos y archivos, coordinó solicitudes y preparación de la entrega; declaró trabajo conjunto presencial y por llamada. |
| Dylan Gómez | Participación en sesiones conjuntas presenciales y por llamada, declarada por el equipo; sin desglose técnico individual. |

### Herramienta y alcance

Se utilizó ChatGPT en Codex, con el MCP de Packet Tracer y control de interfaz. La asistencia incluyó diseño, creación de topología, configuración, código JavaScript, diagnóstico, automatización de pruebas y redacción. Fue una intervención amplia; no se describe como simple corrección de estilo. Los resultados técnicos de este informe provienen de pruebas automatizadas y observaciones de Packet Tracer; los integrantes deben revisarlos y reproducirlos durante su preparación.

### Instrucciones exactas consultadas

Se transcriben instrucciones relevantes del chat: «entonces continúa y termina el proyecto con los valores que elegimos, prueba que todo funcione y envíame el archivo .pkt nuevo y el documento actualizado con lo que pide el profesor      , el pkt nuevo cambiale el nombre y pon otra cosa o asi»; «dime, el cisco ya esta todo listo, es que necesito grabar un video probando que todo funcione bien»; «continua donde lo dejaste». La conversación completa puede compartirse como complemento si la plataforma lo permite; aquí no se inventa un enlace de conversación.

Las respuestas se contrastaron con configuraciones, navegador, contadores ACL e IPsec, resultados SNMP, registros del Tracker y reapertura del pkt; un comando aceptado por sí solo no acredita comunicación exitosa.

### Pendientes de la entrega académica

Sin autorización confirmada para otro formato, corresponde publicar la versión Markdown como Wiki en GitHub o Bitbucket y aportar su enlace. También se debe grabar el video de diez minutos con ambas cámaras y presentar los entregables en entradas separadas en Teams. El ZIP no realiza esa publicación. El bonus web del Tracker no está incluido.

### Referencias

[1] Enunciado del curso, «Proyecto final Diseño e Implementación de una Red Empresarial», RCD 262, 2026. Archivo entregado por el docente.

[2] Cisco Networking Academy, «JavaScript API», Packet Tracer Help. https://tutorials.ptnetacad.net/help/default/iot_javascript_api.htm. Consultado durante la implementación.

[3] Cisco Networking Academy, «Layer 7 Models», Packet Tracer Help. https://tutorials.ptnetacad.net/help/default/model_layer7.htm. Consultado durante la validación.

[4] Cisco Networking Academy, «Packet Tracer Extensions API», referencias Device, BluetoothManager y DnsClient, ayuda instalada de Packet Tracer 9.0.1.

## Evidencias visuales de la simulación

Figura 1. Topología exportada del proyecto activo. Los nombres reales de los dispositivos coinciden con el archivo de configuraciones.

Figura 2. Captura de Packet Tracer durante el ensayo de simulación. La traza detallada de solicitudes, respuestas y tránsito está en 04_04_Evidencias/Traza_simulacion.txt; permite distinguir el tráfico SNMP del tráfico de seguimiento y de las tramas Bluetooth transitorias.

## Capturas

![Topología](04_Evidencias/Topologia.png)

![Captura del ensayo](04_Evidencias/Simulacion_TCP_SNMP.png)
