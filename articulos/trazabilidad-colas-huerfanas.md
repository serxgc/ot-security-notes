# Trazabilidad industrial: el fallo de disponibilidad que también puede provocar un atacante

En una línea de producción, la trazabilidad conecta tres cosas: la máquina que genera un resultado por pieza, un proceso que lo recoge y lo valida, y el sistema que lo registra. Se diseña pensando en fallos accidentales. Casi nunca se diseña pensando en alguien que quiera romperla.

## El fallo base

Cuando dos procesos dependientes tienen políticas de retención distintas (uno borra resultados antiguos, otro reintenta indefinidamente los pendientes), los registros pendientes quedan huérfanos y se reintentan para siempre. El sistema se degrada despacio, sin una alarma clara, hasta que falla aguas abajo.

## Por qué es un problema de seguridad y no solo de mantenimiento

Ese fallo accidental describe un patrón que un atacante podría reproducir a propósito:

- **Disponibilidad:** quien pueda escribir o borrar en las carpetas compartidas puede provocar la misma degradación a voluntad. Lo que hoy es un descuido de configuración es mañana una denegación de servicio silenciosa. En MITRE ATT&CK for ICS encaja con *Loss of Availability* (T0826) y *Data Destruction* (T0809).
- **Integridad:** si el resultado es un fichero que cualquier proceso con permisos puede crear o modificar, ¿quién garantiza que es legítimo? Una pieza defectuosa que "pasa" por un resultado falsificado es peor que una línea parada, porque nadie lo detecta.
- **Ausencia de detección:** el sistema "sigue funcionando", así que no salta ninguna alerta. Un atacante que degrada despacio se confunde con un fallo crónico, y el equipo lo trata como ruido durante semanas.
- **Acoplamiento no documentado:** nadie sabía que dos políticas dependían entre sí. Lo que no está documentado no se audita, y lo que no se audita no se defiende.

## Tres preguntas que cualquier planta debería poder responder

1. ¿Qué cuentas y procesos tienen permiso de escritura y borrado en cada carpeta de intercambio?
2. ¿Los ficheros de resultado tienen alguna verificación de integridad (hash, firma, origen), o basta con que existan?
3. ¿Quién puede acceder a esas máquinas, desde dónde y con qué cuentas (compartidas o nominales)?

## Controles, de menor a mayor esfuerzo

1. **Retención coherente y caducidad de pendientes.** Un cambio de configuración. Es lo más barato y elimina el fallo base.
2. **Mínimo privilegio en carpetas de intercambio.** Cada proceso solo escribe donde debe. Requiere revisar permisos y puede chocar con software de máquina antiguo.
3. **Monitorización de salud real.** Colas, latencia y errores repetidos con umbral de alerta, no solo si el proceso está arrancado.
4. **Integridad de los resultados.** Verificar origen y consistencia antes de darlos por válidos. Suele implicar cambios en el software o en el flujo.
5. **Segmentación y control del acceso remoto.** Zonas y conductos al estilo IEC 62443. Es el más caro y el que más protege.

## La idea de fondo

En OT, un proceso "en marcha" no es un proceso "sano", y un fallo que parece accidental es una prueba gratuita de lo que un atacante podría hacer. Tratar cada avería crónica como un hallazgo de seguridad es la forma más barata de encontrar superficie de ataque antes que otro.
