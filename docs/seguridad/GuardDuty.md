![Generalitat Valenciana - CEUO / IES Poeta Paco Mollá (Alicante)](https://raw.githubusercontent.com/juliogaray/recursos/main/img/Cabecera_CEUO_IESPPM_NOFSE_Transparente.svg)
# C.E. RECURSOS Y SERVICIOS EN LA NUBE
# ADMINISTRACIÓN DE REDES EN LA NUBE
----
# Amazon GuardDuty: Introducción y Funcionamiento 

(v20250202)

## 1. ¿Qué es Amazon GuardDuty? ![Icono de Amazon GuardDuty](../img/GuardDuty.svg)
Amazon GuardDuty es un servicio de detección de amenazas en AWS que utiliza inteligencia artificial, aprendizaje automático y fuentes de datos de seguridad para identificar actividades sospechosas en una cuenta de AWS. Su objetivo es ayudar a las organizaciones a detectar accesos no autorizados, ataques de fuerza bruta, minería de criptomonedas y otras actividades maliciosas.

GuardDuty se asemeja a un [**Sistema de Detección de Intrusiones (IDS)**](https://es.wikipedia.org/wiki/Sistema_de_detecci%C3%B3n_de_intrusos) en que supervisa eventos y detecta comportamientos anómalos, pero a diferencia de un IDS tradicional, no requiere implementación de sensores en la red ni configuración de reglas manuales. En su lugar, se basa en **fuentes de datos en la nube** y **aprendizaje automático** para detectar amenazas de manera automática y escalable.

## 2. Características principales
- **Análisis continuo**: supervisa eventos en la infraestructura de AWS en tiempo real;
- **Fuentes de datos integradas**: analiza *VPC Flow Logs, CloudTrail Event Logs* y *DNS Logs;*
- **Inteligencia de amenazas**: utiliza fuentes como [*AWS Threat Intelligence*](https://aws.amazon.com/es/blogs/security/how-aws-threat-intelligence-deters-threat-actors/) y bases de datos externas de amenazas;
- **Alertas automáticas**: genera hallazgos *(findings)* cuando detecta patrones de actividad sospechosos;
- **Escalabilidad y facilidad de uso**: no requiere configuraciones complejas y se adapta a cualquier entorno de AWS.

## 3. ¿Cómo funciona?
Amazon GuardDuty opera en tres fases principales:

1. **Recopilación de datos**: obtiene información de logs de AWS como CloudTrail, VPC Flow Logs y DNS Logs;
2. **Análisis y detección**: aplica algoritmos de aprendizaje automático y correlación de datos para identificar actividades anormales;
3. **Generación de hallazgos**: crea alertas con detalles del incidente detectado y su posible impacto.

## 4. Activación de Amazon GuardDuty
1. Accede a la consola de AWS y navega a **GuardDuty**;
2. Haz clic en **Habilitar GuardDuty**;
3. Configura la cuenta para monitoreo centralizado si gestionas varias cuentas;
4. Revisa y ajusta las configuraciones de notificación y respuesta.

## 5. Interpretación de hallazgos
Cada hallazgo de GuardDuty contiene:
- **Tipo de amenaza**: acceso no autorizado, tráfico sospechoso, comportamiento anormal, etcétera;
- **Severidad**: baja, media, alta o crítica;
- **Origen del evento**: IP, cuenta, usuario o servicio implicado;
- **Acciones recomendadas**: medidas sugeridas para mitigar el riesgo.

## 6. Integración con otros servicios de AWS
GuardDuty puede integrarse con:
- **AWS Security Hub**: para centralizar alertas de seguridad;
- **AWS Lambda**: para ofrecer respuestas automáticas a incidentes;
- **Amazon CloudWatch**: para monitoreo y generación de métricas;
- **AWS Organizations**: para gestión de seguridad en múltiples cuentas.

## 7. Beneficios de usar Amazon GuardDuty
- **Detección avanzada de amenazas** sin necesidad de infraestructura adicional;
- **Reducción del tiempo de respuesta** con hallazgos detallados y en tiempo real;
- **Optimización de costos** al eliminar la necesidad de herramientas de monitoreo de terceros;
- **Cumplimiento de normativas** como [RGPD](https://es.wikipedia.org/wiki/Reglamento_General_de_Protecci%C3%B3n_de_Datos), [PCI DSS](https://es.wikipedia.org/wiki/PCI_DSS) y [*CIS Benchmarks*](https://en.wikipedia.org/wiki/Center_for_Internet_Security#CIS_Controls_and_CIS_Benchmarks).

## 8. Casos de uso
- **Protección de credenciales comprometidas**: identificación de accesos sospechosos con claves robadas.
- **Detección de ataques a instancias EC2**: alertas sobre tráfico inusual o conexiones desde ubicaciones no confiables.
- **Monitoreo de actividades en cuentas de AWS**: identificación de cambios inesperados en permisos o configuraciones.

### 9. Buenas prácticas
- Habilitar *GuardDuty* en todas las regiones de AWS.
- Integrar hallazgos con herramientas de respuesta automatizada.
- Revisar periódicamente los hallazgos y tomar medidas correctivas.
- Utilizar *IAM Roles* y políticas de acceso mínimas necesarias para reducir riesgos.

## 10. Resumen
Amazon GuardDuty es una solución eficaz para detectar amenazas en AWS mediante análisis continuo de logs y aprendizaje automático. Su integración con otros servicios de AWS permite una respuesta rápida y eficaz ante incidentes de seguridad, facilitando la protección de la infraestructura en la nube.

&nbsp;

&nbsp;

&nbsp;

## Referencias

- [Amazon GuardDuty - Información general](https://aws.amazon.com/es/guardduty/). AWS.

## Colofón
 
[![CC-BY-NC-SA](https://upload.wikimedia.org/wikipedia/commons/5/55/Cc_by-nc-sa_euro_icon.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es) 2025 [J. Garay](mailto:juliogaray.informatica@iespacomolla.es), [IES Poeta Paco Mollà](https://iespacomolla.es/), Alicante (España)
