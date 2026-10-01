# Seguimiento de embarques con IA (n8n)

## Problema
En operaciones de comercio exterior, los forwarders avisan los embarques
por correo y alguien copia a mano contenedor, BL, buque y fechas a una
planilla. Es lento y propenso a errores.

## Qué hace
Un flujo en n8n que lee avisos de embarque (en español e inglés), extrae
8 datos con un modelo de IA (Gemini), valida el número de contenedor con
el dígito de control ISO 6346 y detecta retrasos de ETA mayores a 2 días.

## Cómo funciona
Correos de prueba → modelo de IA (extracción en JSON) → limpieza de la
respuesta → validación y comparación → CSV de resultados.

## Resultados
Probado con 10 correos ficticios que escribí para incluir casos difíciles
(contenedor con espacios, números de referencia que no son el BL, fechas
sin año, formatos de fecha distintos, actualizaciones de demora con datos
faltantes):

- Campos correctos: 80/80 en la primera ejecución [completar: resultado de
  las 3 ejecuciones]
- Valores inventados donde correspondía vacío: 0
- Contenedor inválido detectado: 1/1
- Retrasos de ETA detectados: 2/2 (sin falsas alarmas)

## Problemas que resolví durante el desarrollo
- El modelo devolvía el JSON dentro de un bloque de código (```json) y el
  nodo de extracción fallaba al interpretarlo. Lo resolví con un paso de
  limpieza antes de procesar.
- El modelo configurado por defecto ya no existía en la API (error 404);
  lo cambié por uno vigente.
- Los límites del plan gratuito generaban errores de cuota; activé
  reintentos con espera.

## Decisiones de diseño
- El prompt prohíbe inventar datos (si falta, devuelve null): en
  operaciones, un dato inventado es peor que un dato vacío.
- El modelo no corrige contenedores mal escritos, para que la validación
  del dígito de control detecte el error.
- Temperatura 0 para respuestas consistentes.

## Limitaciones
- Solo 10 correos ficticios; no prueba el comportamiento con correos reales.
- Los retrasos se comparan dentro de un mismo lote, no contra un historial.
- No soporta correos con más de un contenedor ni datos en adjuntos (PDF).

## Próximos pasos
Leer correos reales desde Gmail, guardar el historial en una base de
datos, procesar PDF adjuntos y comparar modelos (Gemini vs. un modelo
local).
