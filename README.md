# 2.0TD-mercado-libre-vs-pvpc
Estos sensores sirven para controlar la facturacion de un contrato de electricidad 2.0TD de mercado libre y compararla con el coste que supone el PVPC.

Para poder hacer esta comparativa es necesario instalar primero la integración homeassistant-edata de uvejota que se puede encontrar aqui: https://github.com/uvejota/homeassistant-edata#homeassistant-edata

Los sensores creados aqui extraen datos de los atributos de la entidad edata_XXXX (los ultimos caracteres dependen del CUPS del contrato a gestionar) de la integracion de uvejota.

![Sin título-1](https://user-images.githubusercontent.com/76565446/146772252-075f3e85-6f27-4239-b88b-9b1b8d15f2ce.jpg)

En las zonas censuradas en rojo de la imagen puedes poner el nombre de tu comercializadora.

PASO 1:
Se integra el sensor binario workday ( https://www.home-assistant.io/integrations/workday/ ) para que Home Assistant sepa cuando es festivo nacional y cuando no lo es
Para ello ponemos en el fichero binary_sensor-yaml del Home assistant lo siguiente:

- platform: workday
  country: ES
  name: "workday electric"
  
  Si no tenemos un fichero binarý_sensor.yaml lo tendremos que crear.
  
  PASO 2:
Se crean los sensores de calculo de la factura de nuestra comercializadora de electricidad pegando lo siguiente en el fichero sensor.yaml. Si no tenemos un fichero sensor.yaml lo tendremos que crear:

#SENSORES PRECIOS DE MI COMERCIALIZADORA
- platform: template
  sensors:
    precio_electricidad_p1:
      friendly_name: "Consumo P1"
      unit_of_measurement: "€/kWh"
      value_template: >-
        {{ (0.128383) }}
    precio_electricidad_p2:
      friendly_name: "Consumo P2"
      unit_of_measurement: "€/kWh"
      value_template: >-
        {{ (0.109669) }}
    precio_electricidad_p3:
      friendly_name: "Consumo P3"
      unit_of_measurement: "€/kWh"
      value_template: >-
        {{ (0.074910) }}
    precio_potencia_p1:
      friendly_name: "Potencia P1"
      unit_of_measurement: "€/kW/dia"
      value_template: >-
        {{ (0.065072) }}
    precio_potencia_p2:
      friendly_name: "Potencia P2"
      unit_of_measurement: "€/kW/dia"
      value_template: >-
        {{ (0.002683) }}

#SENSORES EXTRAIDOS DE DATOS DATADIS
- platform: template
  sensors:
    precio_electricidad_ahora:
      friendly_name: "Precio ahora"
      unit_of_measurement: "€/kWh"
      value_template: >-
        {% if is_state('binary_sensor.workday_electric', 'off') %}
          {{ (states('sensor.precio_electricidad_p3')) | float }}
        {% else %}
          {% set current_hour = now().hour %}
          {% if 0 <= current_hour < 8 %}
            {{ (states('sensor.precio_electricidad_p3')) | float }}
          {% elif 8 <= current_hour < 10 %}
            {{ (states('sensor.precio_electricidad_p2')) | float }}
          {% elif 10 <= current_hour < 14 %}
            {{ (states('sensor.precio_electricidad_p1')) | float }}
          {% elif 14 <= current_hour < 18 %}
            {{ (states('sensor.precio_electricidad_p2')) | float }}
          {% elif 18 <= current_hour < 22 %}
            {{ (states('sensor.precio_electricidad_p1')) | float }}
          {% else %}
            {{ (states('sensor.precio_electricidad_p2')) | float }}
          {% endif %}
        {% endif %}
    consumo_electricidad_mes_pasado_p1:
      friendly_name: "Consumo P1 mes pasado"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'last_month_p1_kWh')) | float * (states('sensor.precio_electricidad_p1')) | float ) | round(2) }}
    consumo_electricidad_mes_pasado_p2:
      friendly_name: "Consumo P2 mes pasado"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'last_month_p2_kWh')) | float * (states('sensor.precio_electricidad_p2')) | float ) | round(2) }}
    consumo_electricidad_mes_pasado_p3:
      friendly_name: "Consumo P3 mes pasado"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'last_month_p3_kWh')) | float * (states('sensor.precio_electricidad_p3')) | float ) | round(2) }}
    potencia_mes_pasado_p1:
      friendly_name: "Potencia P1 mes pasado"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'contract_p1_kW')) | float * (state_attr('sensor.edata_XXXX', 'last_month_days')) | float * (states('sensor.precio_potencia_p1')) | float ) | round(2) }}
    potencia_mes_pasado_p2:
      friendly_name: "Potencia P2 mes pasado"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'contract_p2_kW')) | float * (state_attr('sensor.edata_XXXX', 'last_month_days')) | float * (states('sensor.precio_potencia_p2')) | float ) | round(2) }}
    impuesto_electrico_mes_pasado:
      friendly_name: "Impuesto electrico mes pasado"
      unit_of_measurement: €
      value_template: >-
          {{ ((0.001) * ((state_attr('sensor.edata_XXXX', 'last_month_p1_kWh')) | float + (state_attr('sensor.edata_XXXX', 'last_month_p2_kWh')) | float + (state_attr('sensor.edata_XXXX', 'last_month_p3_kWh')) | float )) | round(2) }}
    alquiler_contador_mes_pasado:
      friendly_name: "Alquiler contador mes pasado"
      unit_of_measurement: €
      value_template: >-
          {{ ((0.02663) * ((state_attr('sensor.edata_XXXX', 'last_month_days')) | float )) | round(2) }}
    iva_electricidad_mes_pasado:
      friendly_name: "IVA mes pasado"
      unit_of_measurement: €
      value_template: >-
          {{ ((0.10) * ((states('sensor.consumo_electricidad_mes_pasado_p1')) | float + (states('sensor.consumo_electricidad_mes_pasado_p2')) | float + (states('sensor.consumo_electricidad_mes_pasado_p3')) | float + (states('sensor.potencia_mes_pasado_p1')) | float + (states('sensor.potencia_mes_pasado_p2')) | float + (states('sensor.impuesto_electrico_mes_pasado')) | float + (states('sensor.alquiler_contador_mes_pasado')) | float )) | round(2) }}
    total_factura_electricidad_mes_pasado:
      friendly_name: "TOTAL mes pasado"
      unit_of_measurement: €
      value_template: >-
          {{ ((states('sensor.consumo_electricidad_mes_pasado_p1')) | float + (states('sensor.consumo_electricidad_mes_pasado_p2')) | float + (states('sensor.consumo_electricidad_mes_pasado_p3')) | float + (states('sensor.potencia_mes_pasado_p1')) | float + (states('sensor.potencia_mes_pasado_p2')) | float + (states('sensor.impuesto_electrico_mes_pasado')) | float + (states('sensor.alquiler_contador_mes_pasado')) | float + (states('sensor.iva_electricidad_mes_pasado')) | float ) | round(2) }}
    consumo_electricidad_mes_actual_p1:
      friendly_name: "Consumo P1 mes actual"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'month_p1_kWh')) | float * (states('sensor.precio_electricidad_p1')) | float ) | round(2) }}
    consumo_electricidad_mes_actual_p2:
      friendly_name: "Consumo P2 mes actual"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'month_p2_kWh')) | float * (states('sensor.precio_electricidad_p2')) | float ) | round(2) }}
    consumo_electricidad_mes_actual_p3:
      friendly_name: "Consumo P3 mes actual"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'month_p3_kWh')) | float * (states('sensor.precio_electricidad_p3')) | float ) | round(2) }}
    potencia_mes_actual_p1:
      friendly_name: "Potencia P1 mes actual"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'contract_p1_kW')) | float * (state_attr('sensor.edata_XXXX', 'month_days')) | float * (states('sensor.precio_potencia_p1')) | float ) | round(2) }}
    potencia_mes_actual_p2:
      friendly_name: "Potencia P2 mes actual"
      unit_of_measurement: €
      value_template: >-
          {{ ((state_attr('sensor.edata_XXXX', 'contract_p2_kW')) | float * (state_attr('sensor.edata_XXXX', 'month_days')) | float * (states('sensor.precio_potencia_p2')) | float ) | round(2) }}
    impuesto_electrico_mes_actual:
      friendly_name: "Impuesto electrico mes actual"
      unit_of_measurement: €
      value_template: >-
          {{ ((0.001) * ((state_attr('sensor.edata_XXXX', 'month_p1_kWh')) | float + (state_attr('sensor.edata_XXXX', 'month_p2_kWh')) | float + (state_attr('sensor.edata_XXXX', 'month_p3_kWh')) | float )) | round(2) }}
    alquiler_contador_mes_actual:
      friendly_name: "Alquiler contador mes actual"
      unit_of_measurement: €
      value_template: >-
          {{ ((0.02663) * ((state_attr('sensor.edata_XXXX', 'month_days')) | float )) | round(2) }}
    iva_electricidad_mes_actual:
      friendly_name: "IVA mes actual"
      unit_of_measurement: €
      value_template: >-
          {{ ((0.10) * ((states('sensor.consumo_electricidad_mes_actual_p1')) | float + (states('sensor.consumo_electricidad_mes_actual_p2')) | float + (states('sensor.consumo_electricidad_mes_actual_p3')) | float + (states('sensor.potencia_mes_actual_p1')) | float + (states('sensor.potencia_mes_actual_p2')) | float + (states('sensor.impuesto_electrico_mes_actual')) | float + (states('sensor.alquiler_contador_mes_actual')) | float )) | round(2) }}
    total_factura_electricidad_mes_actual:
      friendly_name: "TOTAL mes actual"
      unit_of_measurement: €
      value_template: >-
          {{ ((states('sensor.consumo_electricidad_mes_actual_p1')) | float + (states('sensor.consumo_electricidad_mes_actual_p2')) | float + (states('sensor.consumo_electricidad_mes_actual_p3')) | float + (states('sensor.potencia_mes_actual_p1')) | float + (states('sensor.potencia_mes_actual_p2')) | float + (states('sensor.impuesto_electrico_mes_actual')) | float + (states('sensor.alquiler_contador_mes_actual')) | float + (states('sensor.iva_electricidad_mes_actual')) | float ) | round(2) }}
