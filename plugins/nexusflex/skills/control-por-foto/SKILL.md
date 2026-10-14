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
- **Zips de WhatsApp (caso típico):** cada subcarpeta suele venir como un zip adentro del zip del día («WhatsApp Unknown …zip»). Descomprimí cada uno en una carpeta del mismo nombre, adentro de donde estaba (hasta que no quede ninguno) y mové el `.zip` original a `_zips-originales/<día>/…` para que no se lea dos veces. Esas carpetas intermedias («WhatsApp Unknown …», «1ra vuelta», «2da vuelta», nombres de zona) se ignoran: la foto vale por el `DADOS/<grupo o cadete>` o `RECIBIDOS/<…>` que la contiene.
- **Foto suelta en `DADOS` o `RECIBIDOS` sin grupo ni cadete** (o con un nombre que no es de la lista de `control_foto_estructura`) no se procesa (`HTTP 400` en `erroresDeCarpeta`): no adivines. Preguntá a qué grupo o cadete corresponde y movela (o renombrá la carpeta) antes de aplicar. Los apodos («Mati», «Vale», una zona como «Tablada») no valen: tiene que ser un cadete del nodo o un grupo.
- **El grupo de un cadete lo dice la carpeta, no el nombre.** Si un cadete reparte solo de un grupo (ej. todo lo suyo es de Portela común), poné su carpeta en `RECIBIDOS/<grupo>/<cadete>/`; así el control no pregunta "¿qué grupo?" por cada foto. Si un cadete lleva de dos grupos, no asumas: dejá que lo resuelva el usuario foto por foto en la app.
- **Responsable fijo por zona y día (regla del nodo):** si una zona la cubre siempre la misma persona un día (ej. los sábados, toda La Matanza Norte la entrega un cadete del nodo, y lo que no es del nodo se lo dieron en esa juntada), se carga **una vez** como responsable de la zona con sus días (`asignar_mensajero_zona mensajero:<cadete> metazona:<zona> dias:sab`) y el control lo aplica solo. No se resuelve foto por foto ni se escribe en la skill el nombre de nadie.

## 2. Simular primero (nunca escribir sin mostrar)

`control_foto_carpeta ruta:"<carpeta del día o de varios días>"`, sin `aplicar`. Con varios días, simulá **un día por vez**: la respuesta es larga y se lee mejor; el detalle de cada foto queda en `<día>/_control-foto-<fecha>.json`.

Mostrá un resumen corto por día: fotos, ok, altas nuevas, preguntas y errores de carpeta. Si hay `zipsSinDescomprimir` o `fuera_de_DADOS_RECIBIDOS`, avisá.

**Antes de escribir, buscá duplicados en las altas** (el error de las planillas viejas fue dar de alta de nuevo un paquete que ya estaba): sacá del JSON las altas (`alta:true`), compará cada número contra los trackings del nodo de las últimas semanas y mirá uno por uno los que difieren en 1 dígito. Con números de 11 cifras es normal que varios difieran en 2 dígitos por casualidad: lo que decide es el destinatario y la dirección de la foto. Si es el mismo destino, es el mismo paquete (no se da de alta). Después de aplicar, repetí la búsqueda por cliente + destinatario + dirección en la semana.

## 3. Resolver las preguntas

- **cuentasMLDesconocidas**: preguntá de qué nodo y cliente es cada cuenta. Mostrá el vendedor, la marca a mano y la foto de ejemplo. Con el id del cliente: `control_foto_foto ruta:<foto> carpeta:<…> fecha:<…> cliente:<id>`. Si es un vendedor de otro nodo, puede vincular o aprender la cuenta ese nodo, y después se vuelve a correr la carpeta.
- **responsable** (la zona tiene varios responsables o ninguno): preguntá quién lo llevó y repetí la foto con `nodoEntrega`.
- **sin_codigo** (la etiqueta no tiene número: es un envío particular que imprimió el vendedor): leé la dirección de la foto (y localidad/destinatario si se ven) y repetí `control_foto_foto ruta:<foto> carpeta:<…> fecha:<…> direccion:<…> localidad:<…>` con `cliente:<id>` o `nodo:<…>` (el nodo que te lo dio, queda sin vendedor). Busca por dirección si ya está cargado; si no, lo carga con un tracking propio de Nexus Flex. Si la dirección no se lee, no la inventes: va a revisión manual.
- **destino** (la etiqueta se leyó pero faltan la localidad y/o la dirección; viene en `pregunta.datos.faltan` y lo ya leído en `datos.leido`): leélas de la foto y reenviá la MISMA foto con `localidad:<…> direccion:<…>` (repetí `cliente`/`nodoEntrega`/`forzarNuevo` si ya los habías contestado). Nunca las inventes. Un envío sin esos datos no se despacha ni se cobra y queda en «Requiere datos» (`envios_requieren_datos`).
- **duplicado / no_ml / fuera_de_red**: van a revisión manual. La foto quedó copiada en `<día>/_sin_identificar`.
- **Paquete que ya estaba cargado (nuevo)**: antes de dar de alta, el servidor busca en el MISMO cliente un envío igual: por número de ML (aunque la foto lo lea con 1 o 2 dígitos mal), por orden de la tienda o por destinatario y dirección (±7 días). Si lo encuentra, **usa ese envío** y le pasa los estados y la foto en vez de crear otro, y lo avisa («La etiqueta dice X, en el sistema ya está Y…»). Mostráselo al usuario como «unido», no como alta. Solo pregunta (`duplicado`) cuando el parecido es solo por destino y el envío ya figura Entregado otro día: ahí no adivines, preguntá si es el mismo paquete o uno nuevo. Si el usuario dice que **es otro paquete**, se carga igual y queda anotado para que no vuelva a preguntar.
- **erroresDeCarpeta**: una carpeta no coincide con un grupo o cadete del nodo. Que la renombren (con `control_foto_estructura` a la vista).
- **grupo** (el nodo del vendedor comparte varios grupos con vos y la carpeta es `RECIBIDOS/<cadete>`): hace falta la carpeta del grupo (`RECIBIDOS/<grupo>/<cadete>`). No elijas por tu cuenta: va a la app para resolverla mirando la foto, o se ordena la carpeta y se repite.
- **responsable en un grupo con varios nodos por zona** (Quilmes, Vicente López, San Isidro…) o **sin nadie en la zona**: es la pregunta que más aparece en `DADOS/<grupo>`. No hay respuesta automática: si el usuario no sabe, que quede para la app. Cada vez que el usuario te diga quién cubre una zona, ofrecé dejarlo cargado (`asignar_nodo_zona` / `zona_componer_grupo`, con `dias` si solo cubre algunos días) para que la próxima semana no pregunte.

**Cómo cargar los envíos que YA estaban en el sistema (regla del dueño):** si el paquete ya existe, se le cargan los estados que le falten **con el día en que salió** (el de la carpeta), no con el día de hoy. Lo único que no se toca es un Flex que ya figura «Entregado» por la vinculación con Mercado Libre. Un Flex vinculado que todavía no figura entregado **sí** lleva los estados del día en que salió. Hoy el control por foto, en `DADOS/<grupo>`, lo deja como está con el aviso «Flex en "…": lo cierra Mercado Libre, no se tocó»: esos se cargan después con `envio_entregar` (con `fecha` = el día de la carpeta; no pisa a los que ya vengan entregados).

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
- **Cuidá el servidor:** cada foto se comprime en el API y la memoria de un worker crece (con ~1.200 fotos seguidas llegó a 6 GB y el servidor se cayó, 06/10). Aplicá de a un día, mirá la memoria libre entre uno y otro y, si baja de ~3 GB, recargá los workers (`pm2 sendSignal SIGUSR2 nexusflex-api`, con el OK del dueño). Un `aplicar` cancelado a mitad **no se frena**: lo ya enviado se escribe. Los `TimeoutError` / `fetch failed` en `erroresDeCarpeta` son del servidor, no de la carpeta: se repite el día y retoma donde quedó.
- Aplicá **un día por vez** y, al terminar cada uno, revisá que no haya quedado ningún duplicado (mismo cliente, destinatario y dirección en la semana). Un envío que ya estaba en el sistema no se vuelve a dar de alta: se le suman estados y foto.

## 5. Cierre

Resumí por día lo que se cargó y lo que quedó en `_sin_identificar` con su motivo. El detalle completo queda en `<día>/_control-foto-<fecha>.json`.

## Reglas

- No mueve dinero: registra el movimiento y el traspaso entre nodos; los cierres y las liquidaciones los hace el sistema.
- No adivines un nodo, un cliente ni un responsable. Si no está claro, preguntá.
- Los Flex de cuentas vinculadas los cierra Mercado Libre: el control solo los asigna.
- El «Entregado» que deja el control por foto es una **presunción**: se fecha el día de la foto (hora fija 21:00) y no prueba que el comprador lo haya recibido. Se lo podés decir al usuario si pregunta por un envío que el comprador canceló o devolvió: lo que cuenta para cobrar es lo que Mercado Libre informa y la entrega real registrada.
- Un número de ML leído de una foto sin QR puede traer un dígito mal. Si la dirección y el destinatario coinciden con otro envío del mismo cliente casi con el mismo número, es el mismo paquete: no lo des de alta de nuevo.
