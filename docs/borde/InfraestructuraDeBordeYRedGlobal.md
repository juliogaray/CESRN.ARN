![Generalitat Valenciana - CEUO / IES Poeta Paco Mollá (Alicante)](https://raw.githubusercontent.com/juliogaray/recursos/main/img/Cabecera_CEUO_IESPPM_NOFSE_Transparente.svg)
# C.E. RECURSOS Y SERVICIOS EN LA NUBE
# ADMINISTRACIÓN DE REDES EN LA NUBE
----
# Infraestructura de borde. Red global de AWS

(v20250310)
 

## 1. Introducción a la infraestructura de borde

La **infraestructura de borde** en AWS está diseñada para acercar los servicios de computación, almacenamiento y redes a los usuarios finales con el fin de reducir la latencia y mejorar la experiencia del cliente. Esto se logra mediante el uso de ubicaciones de borde *(Edge Locations), AWS Outposts* y *AWS Wavelength.* En las siguientes líneas vamos a ver una introducción a todos estos conceptos.

### 1.1 Ubicaciones de borde *(Edge Locations)* ![\[Icono de Edge Location\]](../img/EdgeLocation.svg)
Las **ubicaciones de borde *(Edge Locations)*** son [centros de procesamiento de datos (CPD)](https://es.wikipedia.org/wiki/Centro_de_procesamiento_de_datos) distribuidos estratégicamente en diferentes regiones del mundo para acelerar la entrega de contenido mediante [**Amazon CloudFront**](https://aws.amazon.com/es/cloudfront/), el servicio de [CDN (Content Delivery Network)](https://es.wikipedia.org/wiki/Red_de_distribuci%C3%B3n_de_contenidos) de AWS. Sus características principales son:
- Caché de contenido estático y dinámico.
- Seguridad mejorada con [AWS Shield](https://aws.amazon.com/es/shield/), [AWS WAF](https://aws.amazon.com/es/waf/) y [AWS Certificate Manager](https://aws.amazon.com/es/certificate-manager/).
- Integración con [Lambda@Edge](https://aws.amazon.com/es/lambda/edge/) para ejecutar código en ubicaciones de borde y personalizar la experiencia del usuario final.


### 1.2 AWS Outposts ![\[Icono de AWS Outpost\]](../img/Outposts.svg)
***AWS Outposts*** es una extensión de la infraestructura de AWS en las instalaciones del cliente. Ofrece los mismos servicios de AWS en hardware físico ubicado en un centro de datos propio, ideal para aplicaciones con exigencias de baja latencia o regulaciones estrictas sobre la localización de datos.

### 1.3 AWS Wavelength ![\[Icono de AWS Wavelength\]](../img/Wavelength.svg)
***AWS Wavelength*** permite el despliegue de servicios de AWS en [redes 5G](https://es.wikipedia.org/wiki/5G) para minimizar la latencia en aplicaciones que así lo demanden, como juegos en la nube, [IoT](https://es.wikipedia.org/wiki/Internet_de_las_cosas) y transmisión en tiempo real.

Como puedes ver, existen similitudes entre estos dos últimos servicios (AWS Outposts y AWS Wavelength). Sin embargo son servicios bastante diferentes:


| Característica | AWS Outposts | AWS Wavelength |
|--------------------|-------------|---------------|
| **Propósito** | Extiende la infraestructura de AWS a centros de datos locales o instalaciones del cliente. | Permite ejecutar cargas de trabajo en redes 5G para reducir la latencia en aplicaciones móviles y en tiempo real. |
| **Ubicación** | Instalaciones del cliente (data centers privados, fábricas, hospitales, etc.). | **Infraestructura de AWS dentro de redes 5G de operadores de telecomunicaciones.** |
| **Casos de uso** | Aplicaciones con requisitos de baja latencia o regulaciones estrictas sobre la localización de datos. | Aplicaciones que requieren latencias ultrabajas, como juegos en la nube, realidad aumentada/virtual, IoT y transmisión de vídeo en vivo. |
| **Conectividad** | Conectado a la región de AWS más cercana mediante [AWS Direct Connect o VPN](../híbridas/ConectividadparaVPCenAWS.html). | Se integra directamente con redes 5G para minimizar la latencia desde el dispositivo al servidor. |
| **Servicios compatibles** | Soporta servicios de AWS como EC2, EBS, RDS y S3 en entornos locales. | Proporciona acceso a instancias EC2, contenedores y otros servicios en redes de telecomunicaciones 5G. |

&nbsp;

&nbsp;

## 2. Red global de AWS

AWS cuenta con una red global de alto rendimiento que conecta sus CPD y servicios a nivel mundial, proporcionando alta disponibilidad y baja latencia.

### 2.1 Características de la red global de AWS
- **Red troncal de fibra óptica privada:** AWS opera su propia red de fibra para conectar sus regiones y ubicaciones de borde.
- **Múltiples regiones y *Zonas de Disponibilidad (AZ):*** AWS divide sus regiones en zonas de disponibilidad para mejorar la redundancia y la recuperación ante desastres.
- **[AWS Global Accelerator](https://aws.amazon.com/es/global-accelerator/):** es un servicio que optimiza el tráfico de usuarios mediante el enrutamiento hacia la región de AWS más cercana y con mejor rendimiento.
- **Conectividad directa con [AWS Direct Connect](../híbridas/ConectividadparaVPCenAWS.html):** permite a empresas conectar sus redes privadas directamente a AWS sin pasar por Internet, reduciendo latencia y mejorando la seguridad.

### 2.2 Beneficios de la Red Global de AWS
- **Baja latencia y alta velocidad:** gracias a su red privada de fibra óptica.
- **Seguridad avanzada:** cifrado de tráfico entre regiones y servicios de seguridad integrados.
- **Alta disponibilidad y tolerancia a fallos:** gracias a la distribución en múltiples regiones y zonas de disponibilidad.

## 3. Caso de uso: Netflix y *Amazon CloudFront*
Como hemos dicho, **Amazon CloudFront** es el servicio de AWS que utiliza la infraestructura de borde para distribuir contenido con baja latencia. 

Por otra parte, [**Netflix**](https://es.wikipedia.org/wiki/Netflix) es una de las plataformas de [*streaming*](https://es.wikipedia.org/wiki/Streaming) más populares del mundo. Y utiliza Amazon CloudFront para mejorar la entrega de contenido de vídeo a sus usuarios.

### 3.1 Exigencias de Netflix  ![\[Icono de Netflix\]](../img/Netflix_2015_N_logo.svg)
- **Alta demanda de ancho de banda:** Netflix transmite grandes volúmenes de datos, fundamentalmente vídeo en alta calidad (Full HD, 4K, HDR).
- **Latencia y [*buffering*](https://www.cloudflare.com/es-es/learning/video/what-is-buffering/)*:*** para garantizar una reproducción fluida, es esencial minimizar la latencia y el tiempo de carga.
- **Escalabilidad global:** con millones de usuarios en diferentes regiones, Netflix necesita una infraestructura de entrega de contenido eficiente y distribuida.

### 3.2 Solución con Amazon CloudFront ![\[Icono de CloudFront\]](../img/CloudFront.svg)
Netflix utiliza Amazon CloudFront como parte de su red de distribución de contenido (CDN) para ofrecer sus servicios de manera óptima a sus clientes. Para ello hace uso de:

1. **Caché de contenido en ubicaciones de borde:**
   - Cuando un usuario reproduce una película, CloudFront verifica si el contenido ya está almacenado en una **ubicación de borde cercana**.
   - Si está disponible, lo entrega desde ese punto en lugar de la región central de AWS, reduciendo la latencia.

2. **Carga eficiente desde el origen:**
   - Si el contenido no está en la ubicación de borde, CloudFront lo solicita desde los servidores de origen de Netflix en AWS y lo almacena para futuras peticiones.

3. **Optimización de rutas de tráfico:**
   - CloudFront selecciona la mejor ruta de entrega a través de la red global de AWS, evitando congestión y reduciendo tiempos de carga.

4. **Seguridad avanzada:**
   - Netflix usa **AWS Shield** y **AWS WAF** para proteger su infraestructura contra ataques DDoS y tráfico malicioso.

### 3.3 Beneficios para Netflix
✔ **Menos latencia:** los usuarios obtienen contenido desde ubicaciones cercanas, reduciendo *buffering.*  
✔ **Escalabilidad global:** CloudFront maneja millones de peticiones simultáneamente sin afectar al rendimiento.  
✔ **Reducción de costos:** al disminuir la carga en los servidores de origen, Netflix optimiza el uso de su infraestructura.  
✔ **Seguridad mejorada:** protección contra ataques y gestión de tráfico malicioso.  

Como puedes ver, Netflix es un cliente ideal para el servicio de Amazon CloudFront, que permite a su cliente ofrezcer una experiencia de *streaming* fluida y sin interrupciones a nivel mundial.


## 4. Conclusión
La infraestructura de borde y la red global de AWS permiten entregar contenido y aplicaciones con baja latencia, alta disponibilidad y seguridad. Servicios como Amazon CloudFront, AWS Outposts y AWS Wavelength facilitan el despliegue de soluciones eficientes para empresas y usuarios finales a nivel global.

&nbsp;

&nbsp;

&nbsp;

# Colofón
[![CC-BY-NC-SA](https://upload.wikimedia.org/wikipedia/commons/5/55/Cc_by-nc-sa_euro_icon.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es) 2025 [J. Garay](mailto:juliogaray.informatica@iespacomolla.es), [IES Poeta Paco Mollà](https://iespacomolla.es/), Alicante (España)



