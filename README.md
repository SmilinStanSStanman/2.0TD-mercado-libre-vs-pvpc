# 2.0TD-mercado-libre-vs-pvpc
Estos sensores sirven para controlar la facturacion de un contrato de electricidad 2.0TD de mercado libre y compararla con el coste que supone el PVPC.

Para poder hacer esta comparativa es necesario instalar la integración homeassistant-edata de uvejota que se puede encontrar aqui: https://github.com/uvejota/homeassistant-edata#homeassistant-edata

Los sensores creados aqui extraen datos de los atributos de la entidad edata_XXXX (los ultimos caracteres dependen del CUPS del contrato a gestionar) de la integracion de uvejota.
