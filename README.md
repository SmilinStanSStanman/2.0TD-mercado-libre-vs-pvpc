# 2.0TD-mercado-libre-vs-pvpc
Estos sensores sirven para controlar la facturación de un contrato de electricidad 2.0TD de mercado libre y compararla con el coste que supone el PVPC.

Para poder hacer esta comparativa es necesario instalar primero la integración homeassistant-edata de uvejota que se puede encontrar aquí: https://github.com/uvejota/homeassistant-edata#homeassistant-edata

Los sensores creados aquí extraen datos de los atributos de la entidad edata_XXXX (los últimos caracteres dependen del CUPS del contrato a gestionar) de la integración de uvejota.

![Sin título-1](https://user-images.githubusercontent.com/76565446/146772252-075f3e85-6f27-4239-b88b-9b1b8d15f2ce.jpg)

En las zonas censuradas en rojo de la imagen puedes poner el nombre de tu comercializadora.

PASO 1:
Se integra el sensor binario workday ( https://www.home-assistant.io/integrations/workday/ ) para que Home Assistant sepa cuándo es festivo nacional y cuándo no lo es.
Para ello pegamos en el fichero binary_sensor.yaml del Home assistant el contenido de este fichero. Si no tenemos un fichero binary_sensor.yaml lo tendremos que crear:

https://github.com/TripitonPT/2.0TD-mercado-libre-vs-pvpc/blob/main/binary_sensor.yaml
  
PASO 2:
Se crean los sensores de cálculo de la factura de nuestra comercializadora de electricidad pegando en el fichero sensor.yaml de nuestro Home Assistant el contenido de este fichero sensor.yaml. Si no tenemos un fichero sensor.yaml lo tendremos que crear:

https://github.com/TripitonPT/2.0TD-mercado-libre-vs-pvpc/blob/main/sensor.yaml

En las líneas 8, 13, 18, 23 y 28 de este fichero sensor.yaml aparecen unos valores. Ahí deberás poner los precios a los que tu comercializadora te cobra la energía consumida en P1, P2 y P3 y el coste de la potencia en P1 y P2 (puedes localizar esos precios en tu última factura).

En el código pegado en el fichero sensor.yaml verás que aparece repetidamente "sensor.edata_XXXX". Los 4 últimos caracteres debes reemplazarlos por los que tenga el sensor edata de tu integración de uvajota. Estos caracteres dependen del CUPS de tu punto de suministro.

En las líneas 84 y 129 de este fichero sensor.yaml aparece un valor "0.001". Este valor representa el valor aplicable al impuesto eléctrico. Cuando se modifique el tipo impositivo de este impuesto, deberás actualizar este valor (1 euro por MWh consumido en el momento de escribir esto).

En las líneas 94 y 139 de este fichero sensor.yaml aparece un valor "0.10". Este valor representa el porcentaje aplicable al IVA. Cuando se modifique el tipo impositivo de este impuesto, deberás actualizar este valor (10% en el momento de escribir esto).

PASO 3:
En las tarjetas que uvejota ha preparado ( https://github.com/uvejota/homeassistant-edata/wiki/uso ) hacemos un pequeño cambio para que aparezcan los valores de nuestra comercializadora junto con los del PVPC. Modificamos las apexcharts-card que presentan los detalles del mes pasado y los detalles del mes en curso con el siguiente código:

https://github.com/TripitonPT/2.0TD-mercado-libre-vs-pvpc/blob/main/apexcharts-card_mes_pasado.yaml

https://github.com/TripitonPT/2.0TD-mercado-libre-vs-pvpc/blob/main/apexcharts-card_mes_actual.yaml

El estos apexcharts-card se deben cambiar las XXXX de "sensor.edata_XXXX" por los caracteres que se correspondan con el nombre de nuestra entidad. Tambien podéis personalizar el nombre de vuestra comercializadora de electricidad.
