# Trazabilidad industrial: modelo de amenaza de un sistema que nadie considera crítico

Los sistemas de trazabilidad se consideran "administrativos": registran qué pieza pasó por qué máquina y con qué resultado. Por eso rara vez entran en un análisis de riesgos. Pero en una línea de producción deciden qué pieza se da por buena y cuál no, y eso los convierte en un objetivo con impacto real sobre la producción y la calidad.

## Punto de partida: un fallo accidental

Cuando dos procesos dependientes tienen políticas de retención distintas (uno borra resultados antiguos, otro reintenta indefinidamente los pendientes), los registros pendientes quedan huérfanos y se reintentan para siempre. El sistema se degrada despacio, sin alarma clara, hasta que falla aguas abajo.

Este texto no trata de la avería, sino de lo que revela: un sistema cuya integridad y disponibilidad dependen de ficheros en carpetas compartidas, sin verificación y sin monitorización.

## Modelo de amenaza

**Activos**

- Los resultados por pieza (la decisión de calidad).
- La disponibilidad del flujo de trazabilidad.
- La información que permite reconstruir qué se fabricó y cuándo.

**Quién podría actuar**

- Un atacante que ya está en la red IT y busca moverse hacia la planta.
- Un tercero con acceso remoto legítimo (integrador, proveedor) con cuentas compartidas o permanentes.
- Un insider, con o sin mala intención.
- Malware que no apunta a la planta pero llega a ella (ransomware en un equipo Windows de línea).

**Superficie de ataque**

- Carpetas de intercambio con permisos de escritura y borrado amplios.
- Ficheros de resultado que se aceptan por existir, sin comprobar su origen.
- Equipos Windows de máquina difíciles de parchear porque el software del fabricante no lo soporta.
- Acceso remoto y cuentas compartidas.

## Escenarios de ataque (nivel conceptual)

| Escenario | Qué rompe | Impacto | Por qué se detecta tarde |
|---|---|---|---|
| Borrado o saturación de la carpeta de resultados | Disponibilidad | Línea lenta o parada | Parece una avería crónica |
| Creación o modificación de resultados | Integridad | Piezas defectuosas dadas por buenas | El sistema "funciona" y no hay alarma |
| Borrado de registros históricos | Integridad y trazabilidad | Imposible reconstruir qué se fabricó | Se descubre en una reclamación o auditoría |

En OT el orden de prioridades suele ser disponibilidad, integridad y confidencialidad, al revés que en IT. El segundo escenario es el más difícil de detectar: una pieza mala que el sistema registra como buena no tiene por qué generar ninguna alarma.

## Correspondencia con MITRE ATT&CK for ICS

| Escenario | Técnica orientativa |
|---|---|
| Degradar o parar el flujo | Loss of Availability (T0826) |
| Borrado de datos | Data Destruction (T0809) |
| Resultados falsificados | Spoof Reporting Message (T0856) |
| Entrada mediante acceso remoto | External Remote Services (T0822) |
| Uso de credenciales compartidas | Valid Accounts (T0859) |

La correspondencia es orientativa. Comprueba cada identificador en la matriz oficial.

## Por qué no se detecta

- **No hay línea base.** Nadie sabe cuál es el tamaño normal de la cola ni la tasa normal de errores.
- **Se vigila el proceso, no su salud.** "Está arrancado" no significa "está bien".
- **El ruido crónico oculta el ataque.** Un equipo acostumbrado a un fallo recurrente lo trata como ruido, y un atacante lento se confunde con él.
- **Las dependencias no están documentadas**, así que nadie puede auditarlas.

## Controles y requisitos IEC 62443

Ordenados de menor a mayor esfuerzo, con el requisito fundamental de IEC 62443 que cubre cada uno:

| Control | Esfuerzo | Requisito |
|---|---|---|
| Retención coherente y caducidad de pendientes | Configuración | FR7: disponibilidad de recursos |
| Mínimo privilegio en carpetas de intercambio | Revisión de permisos | FR2: control de uso |
| Monitorización de salud real (colas, latencia, errores repetidos) | Herramientas y umbrales | FR6: respuesta oportuna a eventos |
| Verificación de integridad y origen de los resultados | Cambios en software o flujo | FR3: integridad del sistema |
| Cuentas nominales y acceso remoto controlado | Proceso y herramientas | FR1: identificación y autenticación |
| Segmentación por zonas y conductos | Proyecto de red | Modelo de zonas y conductos |

## Tres preguntas que cualquier planta debería poder responder

1. ¿Qué cuentas y procesos pueden escribir y borrar en cada carpeta de intercambio?
2. ¿Los resultados se verifican (hash, firma, origen), o basta con que el fichero exista?
3. ¿Quién accede a estas máquinas, desde dónde y con qué cuentas?

## La idea de fondo

Un sistema "en marcha" no es un sistema "sano". Y una avería crónica es una prueba gratuita de lo que un atacante podría hacer: tratarla como un hallazgo de seguridad es la forma más barata de encontrar superficie de ataque antes que otro.
