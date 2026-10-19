# EtherCAT fue diseñado para la velocidad, no para resistir un ataque
### Por qué un protocolo sin autenticación puede seguir siendo seguro

En automatización industrial, EtherCAT se eligió por velocidad y determinismo, no por seguridad. Es un protocolo de hace más de veinte años, diseñado cuando la red de planta se asumía aislada. Esa suposición ya no se sostiene, y vale la pena mirar qué significa en términos de seguridad.

## Por qué EtherCAT es tan rápido

Un maestro envía una única trama que recorre todos los esclavos en cadena. Cada esclavo lee y escribe sus datos al vuelo, sin reenviar nada, y la trama vuelve al maestro. Los relojes distribuidos sincronizan a todos los nodos con precisión de microsegundos. Es ese diseño el que le da la velocidad: no hay negociación por nodo, solo una trama que pasa una vez.

## Qué garantiza y qué no

Ese diseño optimiza determinismo, no protección. El protocolo base no incluye autenticación de nodos ni cifrado de la trama. Trabaja en capa 2 de Ethernet, lo que significa dos cosas: no cruza un router sin ayuda externa, pero tampoco pasa por un firewall IP ni por la mayoría de las herramientas de seguridad pensadas para redes IT.

En la práctica, esto implica que cualquier dispositivo físicamente conectado al segmento EtherCAT se trata, por diseño, como legítimo.

Esto no es solo una lectura del diseño: un estudio académico sobre la seguridad de EtherCAT confirma que el protocolo carece de los parámetros de seguridad más comunes (autenticación, cifrado, autorización) y queda expuesto a suplantación de MAC, inyección de datos y otros ataques que requieren habilidad técnica avanzada.¹

El EtherCAT Technology Group, responsable del estándar, no lo plantea como un descuido sino como una decisión de diseño: la seguridad se delega en la arquitectura que rodea al protocolo —segmentación, control de acceso físico— en vez de en el protocolo mismo. Según su propia documentación, los sistemas EtherCAT ya cumplen el Security Level 2 de IEC 62443-3-3 sin modificaciones, y para necesidades mayores existen extensiones opcionales y retrocompatibles con autenticación y cifrado.

## Superficie de ataque

- **Acceso físico al segmento**: un puerto libre en un armario, un cable accesible, o un dispositivo intercalado entre maestro y esclavo.
- **El maestro y la estación de ingeniería**: suelen ser un PC o un PLC con software de configuración, a menudo con acceso remoto para mantenimiento.
- **El acceso remoto a ambos**: integradores y proveedores que entran a diagnosticar o actualizar, a veces con credenciales compartidas o conexiones permanentes.

## El error de sincronización como señal

Un fallo de sincronización en un nodo EtherCAT, en operación normal, suele tener causa física: un cable, un conector, un problema de alimentación, interferencia electromagnética. Es el diagnóstico habitual, y la mayoría de las veces es el correcto.

Pero alguien con acceso al segmento puede provocar los mismos síntomas: interrumpir la trama, introducir ruido, desconectar y reconectar un nodo. El síntoma es indistinguible del fallo físico si no se guarda evidencia adicional.

Esto es relevante porque es el mismo problema que ya vimos con la trazabilidad: un sistema "en marcha" no es un sistema "sano", y una avería crónica puede ser indistinguible de un ataque si no hay forma de diferenciarlos.

**Qué evidencia ayuda a distinguirlos:**

- Un registro con marcas de tiempo de cada error, no solo el último.
- Contadores de error por puerto, para ver si el problema es local o se mueve.
- Una topología esperada documentada, para detectar si aparece o desaparece un nodo.

Sin estos tres datos, cada error de sincronización se cierra como "fue el cable" sin forma de comprobarlo.

## Controles, de menor a mayor esfuerzo

1. **Seguridad física de armarios y puertos libres.** Sin coste de ingeniería, solo disciplina: candados, tapas en puertos no usados, inventario de quién tiene llave.
2. **Inventario de nodos esperados con alarma ante cambios de topología.** Requiere documentar una vez la topología correcta y una herramienta que la compare.
3. **Proteger el maestro y la estación de ingeniería.** Igual que cualquier equipo crítico: parches cuando sea posible, cuentas nominales, antivirus si el fabricante lo permite.
4. **Controlar el acceso remoto.** Sin credenciales compartidas, con registro de quién entra y cuándo, idealmente con conexión bajo demanda en vez de permanente.
5. **Monitorizar los contadores de error de forma centralizada.** El esfuerzo más alto: requiere herramientas de monitorización OT y, a menudo, cambios en cómo se reportan las alarmas hoy.

## La idea de fondo

En OT, el diagnóstico de averías y la detección de intrusiones usan las mismas señales. La diferencia no está en qué se mide, sino en si se guarda suficiente evidencia para distinguir una causa de otra. Quien ya documenta bien sus averías tiene la mitad del trabajo de detección hecho sin saberlo.

---

¹ Akpinar, K.O. y Özçelik, I., "Development of the ECAT Preprocessor with the Trust Communication Approach", *Security and Communication Networks*.