# Manual de Procedimientos: Gestión de Incidencias entre Proveedores de Despacho

## 1. Objetivo

Establecer un procedimiento para revisar solicitudes relacionadas con pedidos o despachos registrados bajo un proveedor diferente al informado por el solicitante.

El objetivo es reducir los tiempos de revisión manual, entregar respuestas consistentes y mantener un registro de las decisiones propuestas.

## 2. Alcance

Este procedimiento aplica a incidencias relacionadas con:

- Pedidos registrados bajo otro proveedor.
- Diferencias entre proveedor informado y proveedor registrado.
- Pedidos que no aparecen en los registros internos.
- Información incompleta o incorrecta.
- Solicitudes que requieren revisión humana.

## 3. Datos necesarios

Para analizar una solicitud se deben considerar:

- Número o identificador del pedido.
- Proveedor que realiza la solicitud.
- Proveedor que figura en el registro.
- Fecha del pedido o despacho.
- Descripción de la incidencia.
- Antecedentes adicionales entregados por el solicitante.

El número de pedido es el dato principal para localizar el registro.

## 4. Procedimiento de validación

### 4.1 Recepción

Registrar la solicitud y comprobar que contenga información suficiente para identificar el pedido y al proveedor solicitante.

Si faltan datos, solicitar los antecedentes necesarios.

### 4.2 Búsqueda del pedido

Consultar los registros internos utilizando el identificador del pedido.

Los resultados posibles son:

- Pedido encontrado: continuar con la comparación.
- Pedido no encontrado: solicitar la verificación del identificador.
- Varios registros coincidentes: derivar a revisión humana.

### 4.3 Comparación de proveedores

Comparar el proveedor informado con el proveedor registrado.

Si ambos coinciden, informar que no se detectan diferencias en ese campo.

Si son diferentes, identificar la discrepancia y preparar una respuesta para revisión humana.

No asumir que existe un error de registro solamente porque los nombres son distintos.

Si el proveedor no aparece en los registros de referencia, no asociarlo por similitud de nombres. Derivar el caso a revisión humana.

## 5. Reglas de gestión

1. Nunca inventar números de pedidos, nombres de proveedores ni fechas.
2. Utilizar los registros internos como fuente de verdad para validar pedidos.
3. No modificar pedidos ni registros de despacho.
4. No confirmar cambios de proveedor sin autorización humana.
5. No considerar que dos proveedores son equivalentes únicamente porque sus nombres son parecidos.
6. Si existen contradicciones entre fuentes, informar la inconsistencia.
7. Si falta información, solicitarla explícitamente.
8. Si la situación no está contemplada en este manual, derivar el caso a revisión humana.
9. Toda respuesta generada debe considerarse una propuesta hasta su validación por una persona responsable.

## 6. Clasificación de solicitudes

### SOLICITUD_RESUELTA

Se utiliza cuando los datos disponibles permiten explicar la situación y preparar una respuesta propuesta.

### SOLICITAR_DATOS

Se utiliza cuando falta información necesaria para identificar o validar el pedido.

Ejemplos:
- No se proporciona número de pedido.
- No se identifica el proveedor solicitante.
- La descripción es insuficiente.

### REVISION_HUMANA

Se utiliza cuando:
- Los registros presentan contradicciones.
- Existen varios pedidos posibles.
- El proveedor no está identificado.
- Se requiere autorizar un cambio.
- No existe evidencia suficiente para concluir.
- La situación no está contemplada en el procedimiento.

## 7. Formato de respuesta del agente

El agente debe entregar:

- clasificacion
- pedido
- proveedor_solicitante
- proveedor_registrado
- analisis
- respuesta_propuesta
- estado_revision

El campo estado_revision debe indicar: Pendiente de revisión humana.

## 8. Ejemplos

### Ejemplo 1: Diferencia entre proveedores

Solicitud: El proveedor Rapido Norte informa que el pedido PED-1002 aparece registrado bajo Carga Sur.

Validación: Se localiza el pedido y se comprueba que el proveedor registrado es Carga Sur.

Resultado: Identificar la diferencia y preparar una respuesta para revisión humana.

### Ejemplo 2: Pedido inexistente

Solicitud: El proveedor consulta por el pedido PED-9999.

Validación: No se encuentra un registro asociado al identificador.

Resultado: Solicitar que se verifique el número del pedido y se envíen antecedentes adicionales.

### Ejemplo 3: Información incompleta

Solicitud: El proveedor informa que tiene problemas con un despacho, pero no indica el número de pedido.

Validación: No existe información suficiente para identificar el registro.

Resultado: Solicitar el número de pedido.

### Ejemplo 4: Situación ambigua

Solicitud: El proveedor reclama que un pedido aparece bajo otro proveedor, pero existen dos registros posibles.

Validación: No es posible determinar con seguridad cuál es el pedido correcto.

Resultado: Derivar a revisión humana y evitar emitir una conclusión.

## 9. Límites del agente

El agente de inteligencia artificial:

- No tiene autoridad para aprobar cambios operacionales.
- No puede modificar información en los sistemas internos.
- No debe inventar información faltante.
- No debe interpretar coincidencias parciales como validaciones definitivas.
- Debe informar cuando la información disponible no permita responder.
- Debe consultar la documentación recuperada mediante RAG.

## 10. Seguridad y revisión

Toda respuesta generada debe quedar pendiente de revisión humana antes de ser enviada como comunicación definitiva o utilizada para modificar información operacional.

La decisión final corresponde al personal responsable de Operaciones y Logística.