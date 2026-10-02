---
name: alta-inicial-nodo
description: Alta inicial de un nodo de Nexus Flex — migrar a la plataforma el equipo (dueños, operadores, choferes), los vendedores con sus precios y los saldos que el nodo trae de su sistema anterior. Usar cuando el dueño de un nodo dice "armame el alta inicial", "quiero cargar a mis vendedores/choferes", "pasar mis clientes a Nexus Flex", "migrar desde mi sistema" o suelta una planilla/lista de sus vendedores, choferes o saldos. Requiere el MCP de Nexus Flex con el usuario DUEÑO del nodo.
---

# Alta inicial del nodo (Nexus Flex)

Nexus Flex es el software del nodo (marca blanca): el dueño trae su situación real sin pedirle nada a nadie. Vos le ahorrás completar la planilla: armás el alta con lo que ya tiene.

## 1. Juntar lo que tiene

Pedile lo que ya exista, en cualquier formato: la planilla de su sistema anterior, un Excel de clientes, una lista por WhatsApp, una foto de un cuaderno. **No le pidas que complete la planilla de cero.**

## 2. Armar las filas

Corré `alta_nodo_columnas` y armá tres listas con esas claves:
- **equipo**: dueños («Dueño del nodo»), operadores (con sus permisos Sí/No; sin ninguno = acceso completo) y choferes (nombre con el que se le paga, frecuencia, saldo).
- **vendedores**: tienda, titular (nombre, apellido, DNI), email y WhatsApp, si se lo busca (colecta) con la dirección de retiro, los 4 precios por zona, si el precio tiene IVA, frecuencia, factura (CUIT, razón social, IVA, domicilio fiscal) y su saldo.
- **saldosNodos** (opcional): lo que se debe con otros nodos, con el **grupo** en el que trabajan juntos (si comparten uno, podés dejarlo vacío).

Saldos: monto positivo + sentido. Vendedor «Nos debe» = envíos que no pagó; «Le debemos» = plata suya que tiene el nodo (cobros en destino). Chofer «Le debemos» = viajes sin pagar; «Nos debe» = cobros que no rindió.

**No inventes** emails, DNI, teléfonos ni precios: lo obligatorio que falte, preguntalo (de a varios juntos, en una tabla corta).

## 3. Previsualizar hasta que esté limpio

`alta_nodo_previsualizar` con las tres listas. Corregí cada error (hoja + fila = posición en tu lista) y repetí hasta **0 errores**. Mostrale al dueño un resumen: cuántos usuarios, qué listas de precios se arman (mismos 4 precios = una sola lista) y los saldos. Esperá su OK.

## 4. Confirmar y entregar los accesos

`alta_nodo_confirmar` con las mismas filas. Devuelve la **clave provisoria** de cada uno: dáselas en una tabla (nombre, email, clave) para que se las mande; cada uno la cambia al entrar. Los saldos con otros nodos los confirma el otro nodo desde su app (💰 Saldos iniciales); si todavía no tiene cuenta en Nexus Flex, quedan tomados como válidos: avisale.

## 5. Lo que sigue (en la app)

En **🏢 Datos del nodo** está la tarjeta **Primeros pasos**: subir el logo, la dirección del depósito y vincular Mercado Libre como Mensajería Flex. Manual con video: https://nexusflex.com.ar/manual-nodos/

Si prefiere hacerlo a mano: en Datos del nodo → 📥 Alta inicial baja la planilla de su nodo, la completa y la sube él mismo (mismo resultado).
