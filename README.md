# 2.0TD-mercado-libre-vs-pvpc
Estos sensores sirven para controlar la facturacion de un contrato de electricidad 2.0TD de mercado libre y compararla con el coste que supone el PVPC.

Para poder hacer esta comparativa es necesario instalar primero la integración homeassistant-edata de uvejota que se puede encontrar aqui: https://github.com/uvejota/homeassistant-edata#homeassistant-edata

Los sensores creados aqui extraen datos de los atributos de la entidad edata_XXXX (los ultimos caracteres dependen del CUPS del contrato a gestionar) de la integracion de uvejota.

![Sin título-1](https://user-images.githubusercontent.com/76565446/146772252-075f3e85-6f27-4239-b88b-9b1b8d15f2ce.jpg)

En las zonas censuradas en rojo de la imagen puedes poner el nombre de tu comercializadora.

PASO 1:
Se integra el sensor binario workday ( https://www.home-assistant.io/integrations/workday/ ) para que Home Assistant sepa cuando es festivo nacional y cuando no lo es.
Para ello pegamos en el fichero binary_sensor.yaml del Home assistant el contenido de este fichero. Si no tenemos un fichero binarý_sensor.yaml lo tendremos que crear:

https://github.com/TripitonPT/2.0TD-mercado-libre-vs-pvpc/blob/main/binary_sensor.yaml
  
PASO 2:
Se crean los sensores de calculo de la factura de nuestra comercializadora de electricidad pegando en el fichero sensor.yaml de nuestro Home Assistant el contenido de este fichero sensor.yaml. Si no tenemos un fichero sensor.yaml lo tendremos que crear:

https://github.com/TripitonPT/2.0TD-mercado-libre-vs-pvpc/blob/main/sensor.yaml

En las lineas 8, 13, 18, 23 y 28 de este fichero sensor.yaml aparecen unos valores. Ahí deberas poner los precios a los que tu comercializadora te cobra la energia consumida en P1, P2 y P3 y el coste de la potencia en P1 y P2 (puedes localizar esos precios en tu ultima factura).

En el codigo pegado en el fichero sensor.yaml veras que aparece repetidamente "sensor.edata_XXXX". Los 4 ultimos caracteres debes reemplazarlos por los que tenga el sensor edata de tu integracion edata de uvajota. Estos caracteres dependen del CUPS de tu punto de suministro.

En la linea 84 y 129 de este fichero sensor.yaml aparece un valor "0.001". Este valor representa el porcentaje aplicable al impuesto electrico. Cuando se modifique el tipo impositivo de este impuesto, deberás actualizar este valor.

En la linea 94 y 139 de este fichero sensor.yaml aparece un valor "0.10". Este valor representa el porcentaje aplicable al IVA. Cuando se modifique el tipo impositivo de este impuesto, deberás actualizar este valor.
