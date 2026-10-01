---
name: control-por-foto
description: Control por foto de Nexus Flex — preparar las carpetas del día y cargar y cerrar lo que un nodo dio, recibió y repartió en un día, a partir de carpetas de fotos de etiquetas (DADOS/RECIBIDOS por día). Usar cuando el usuario quiere "hacer el control por foto", "controlar las fotos del día", "marcar lo que salió con los grupos/cadetes" o suelta carpetas de fotos de etiquetas de Mercado Libre. Requiere el MCP de Nexus Flex local (npx nexusflex-mcp) con escritura habilitada.
---

# Control por foto (Nexus Flex)

Sirve para cualquier nodo y cualquiera de sus grupos logísticos. El servidor hace el trabajo foto por foto (lee el QR o la etiqueta con IA, busca o da de alta el envío sin duplicar, lo asigna y lo cierra en el día real, y le cuelga la foto). Vos guiás al usuario, mostrás lo que va a pasar y **no adivinás nada**: lo dudoso se pregunta.

## 0. Empezar: crear las carpetas

Cuando el usuario dice "hagamos el control por foto del día tal", corré:

`control_foto_preparar fechas:"<día o días>"`

Por seguridad, el MCP solo lee y crea adentro de **la carpeta del control por foto**: la que el usuario eligió al instalar el plugin (si no eligió, `Documentos\Control por foto`). `control_foto_estructura` te dice cuál es (`carpetaDelControl`). Las rutas se pasan relativas a ella (`Lunes 14-9-2026`); si el usuario tiene las fotos en otro lado, pedile que las mueva ahí: una ruta de afuera se rechaza.

Las fechas pueden ser un día (`14/09/2026`), un rango (`14/09 a 19/09`) o una lista (`14/09, 16/09`). Se crea una carpeta por día, con las subcarpetas de todos sus grupos y cadetes con el nombre exacto y un `LEEME.txt`. Decile dónde quedaron y que suelte cada foto en la carpeta que corresponde. Las que no use quedan vacías y no pasa nada.

## 1. Las carpetas

Una carpeta por día. El nombre tiene que traer la fecha (`Lunes 14-9-2026`, `14-09-2026`, `2026-09-14`). Adentro:

| Carpeta | Qué significa | Qué se hace |
|---|---|---|
| `DADOS/<grupo>/` | Lo que el nodo pasó a ese grupo | Se asigna al nodo responsable de la zona del destino en ese grupo |
| `DADOS/<cadete>/` | Lo repartió un cadete del nodo | Cadete asignado, Entregado |
| `RECIBIDOS/…/<cadete>/` | Otro nodo se lo dio y lo repartió ese cadete | Origen = nodo del vendedor → entrega = este nodo, por el grupo que comparten |
| `RECIBIDOS/<grupo>/<cadete>/` | Igual, pero llegó por ese grupo | Cuando el vendedor comparte varios grupos con el nodo |
| `RECIBIDOS/<grupo>/` | Se lo dieron y lo derivó por ese grupo | Cadena: vendedor → este nodo → responsable de ese grupo |

- Los nombres de `<grupo>` y `<cadete>` son los del nodo. Llamá a `control_foto_estructura` para tenerlos y mostrárselos al usuario la primera vez.
- Carpetas intermedias que no son grupo ni cadete (ej. `Matanza`, `1ra vuelta`) se ignoran.
- El mismo paquete en `DADOS/…` y en `RECIBIDOS/…` el mismo día NO es un duplicado: te lo dieron y lo pasaste. Vale DADOS (cómo salió); RECIBIDOS es el respaldo por si algo no se registró y esa foto solo se suma al envío. No le preguntes al usuario por eso.
- Un grupo que solo sale ciertos días (ej. un "Sábado …") va como `DADOS/<ese grupo>`.
- Si las fotos vienen en `.zip`, descomprimilas antes. Las carpetas que empiezan con `_` se ignoran.

## 2. Simular primero (nunca escribir sin mostrar)

`control_foto_carpeta ruta:"<carpeta del día o de varios días>"`, sin `aplicar`.

Mostrá un resumen corto por día: fotos, ok, altas nuevas, preguntas y errores de carpeta. Si hay `zipsSinDescomprimir` o `fuera_de_DADOS_RECIBIDOS`, avisá.

## 3. Resolver las preguntas

- **cuentasMLDesconocidas**: preguntá de qué nodo y cliente es cada cuenta. Mostrá el vendedor, la marca a mano y la foto de ejemplo. Con el id del cliente: `control_foto_foto ruta:<foto> carpeta:<…> fecha:<…> cliente:<id>`. Si es un vendedor de otro nodo, puede vincular o aprender la cuenta ese nodo, y después se vuelve a correr la carpeta.
- **responsable** (la zona tiene varios responsables o ninguno): preguntá quién lo llevó y repetí la foto con `nodoEntrega`.
- **sin_codigo** (la etiqueta no tiene número: es un envío particular que imprimió el vendedor): leé la dirección de la foto (y localidad/destinatario si se ven) y repetí `control_foto_foto ruta:<foto> carpeta:<…> fecha:<…> direccion:<…> localidad:<…>` con `cliente:<id>` o `nodo:<…>` (el nodo que te lo dio, queda sin vendedor). Busca por dirección si ya está cargado; si no, lo carga con un tracking propio de Nexus Flex. Si la dirección no se lee, no la inventes: va a revisión manual.
- **duplicado / no_ml / fuera_de_red**: van a revisión manual. La foto quedó copiada en `<día>/_sin_identificar`.
- **erroresDeCarpeta**: una carpeta no coincide con un grupo o cadete del nodo. Que la renombren (con `control_foto_estructura` a la vista).

**Más cómodo:** al aplicar, lo dudoso se sube solo a la app, en **Logística → 📷 Control por foto**. Ahí cada foto se ve grande y se elige:
- el cliente (de cualquier nodo de la red);
- o el nodo que se lo dio;
- quién lo llevó, o la carpeta.

La opción "Recordar" asocia la cuenta de ML para la próxima vez. No la marques si el vendedor despacha con varias logísticas. En esa misma pantalla están los envíos **sin vendedor** del nodo (paquetes de otro nodo que escaneó un cadete), para asignarles el nodo o el cliente.

## 4. Aplicar

Con el OK del usuario: `control_foto_carpeta ruta:"<…>" aplicar:true`.

- Es reanudable: una foto ya procesada no se repite. Se puede correr de nuevo después de contestar preguntas.
- Cada paquete queda Entregado, con los estados fechados en el día real, su grupo, el nodo que entrega, el cadete y la foto en el historial (la que ven los nodos en su cuenta semanal).
- No toca envíos de fuera de la red del nodo, no crea duplicados y no reasigna lo que ya entró en un cierre semanal congelado. Lo nuevo de una semana ya cerrada entra en el cierre siguiente, marcado como tarde.

## 5. Cierre

Resumí por día lo que se cargó y lo que quedó en `_sin_identificar` con su motivo. El detalle completo queda en `<día>/_control-foto-<fecha>.json`.

## Reglas

- No mueve dinero: registra el movimiento y el traspaso entre nodos; los cierres y las liquidaciones los hace el sistema.
- No adivines un nodo, un cliente ni un responsable. Si no está claro, preguntá.
- Los Flex de cuentas vinculadas los cierra Mercado Libre: el control solo los asigna.
