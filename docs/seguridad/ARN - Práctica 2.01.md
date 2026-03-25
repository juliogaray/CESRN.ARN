![Generalitat Valenciana - CEUO / IES Poeta Paco Mollá (Alicante)](https://raw.githubusercontent.com/juliogaray/recursos/main/img/Cabecera_CEUO_IESPPM_NOFSE_Transparente.svg)

# Configuración y Uso de AWS Network Firewall

En esta práctica profundizaremos en la configuración y administración de **AWS Network Firewall**. Nos centraremos en el diseño de una arquitectura segura para una VPC y en la inspección avanzada del tráfico.

## Objetivos
1. Implementar **AWS Network Firewall** en una VPC existente para inspeccionar tráfico entrante, saliente e interno.
2. Configurar políticas de firewall avanzadas para bloquear tráfico sospechoso y permitir solo comunicaciones legítimas.
3. Integrar Network Firewall con **Amazon CloudWatch** para monitorear y analizar el tráfico inspeccionado.
4. Practicar la configuración de tablas de enrutamiento para redirigir tráfico hacia los **Firewall Endpoints**.

### Entrega
Deberás:
- Documentar el proceso seguido, **incluyendo capturas de pantalla.**
- Responder a estas preguntas:
  - ¿Qué sucede si no actualizas las tablas de encaminamiento para redirigir el tráfico al Firewall Endpoint?
  - ¿A dónde debe dirigirse el tráfico filtrado que sale del cortafuegos, para que las subredes privadas tengan acceso a Internet?
  - ¿Por qué es importante tener subredes dedicadas para el cortafuegos?
  - ¿Cómo puedes optimizar la política del cortafuegos para evitar falsos positivos en los bloqueos?
  - Si implementaras *AWS Network Firewall* en un entorno de producción, ¿qué medidas adicionales tomarías para garantizar alta disponibilidad?

Elabora y entrega un único archivo PDF con todo lo especificado, incluidas las preguntas y sus respuestas correspondientes.  
No olvides indicar en tu documento todos los datos de cada elemento: nombre (dado por ti), ID de AWS del elemento, y características (rango de direcciones IP, conexiones, etcétera).

---

## Diagrama de la arquitectura

Esta práctica incluye:
- Una VPC con varias subredes distribuidas en dos AZ (zonas de disponibilidad).
- Un **Internet Gateway** para el tráfico saliente a Internet.
- Un **NAT Gateway** para permitir acceso a Internet desde subredes privadas.
- **AWS Network Firewall** configurado con subredes específicas para los *endpoints* del cortafuegos.
- Dos instancias EC2 para probar la conectividad y las reglas del cortafuegos.

---

## Paso 1: configuración inicial de la VPC
1. **Crear una VPC con las siguientes configuraciones:**
   - CIDR: `10.0.0.0/16`
   - Subred pública en AZ1: `10.0.1.0/24`
   - Subred privada en AZ1: `10.0.2.0/24`
   - Subred pública en AZ2: `10.0.3.0/24`
   - Subred privada en AZ2: `10.0.4.0/24`

2. **Agregar una *Internet Gateway:***
   - Asociar la **Internet Gateway (IGW)** con la VPC.
   - Configurar una tabla de encaminamiento para las subredes públicas con una ruta a `0.0.0.0/0` apuntando a la IGW.

3. **Configurar una *NAT Gateway:***
   - Crear una NAT Gateway en la subred pública de AZ1.
   - Actualizar la tabla de enrutamiento de las subredes privadas para dirigir el tráfico hacia Internet a través de la NAT Gateway.

## Paso 2: implementar *AWS Network Firewall*
1. **Crear un Firewall:**
   - Acceder a la consola de AWS Network Firewall y crear un nuevo cortafuegos.
   - Seleccionar la VPC configurada anteriormente.
   - Añadir dos subredes de cortafuegos:
     - Subnet de cortafuegos en AZ1: `10.0.5.0/24`.
     - Subnet de cortafuegos en AZ2: `10.0.6.0/24`.

2. **Configurar una *Firewall Policy:***
   - Crear una nueva **Política de Cortafuegos** con las siguientes reglas:
     - Bloquear solicitudes HTTP (puerto 80) hacia un dominio malicioso (`ejemplo-sitio-maligno.mil`).
     - Permitir tráfico HTTPS (puerto 443) hacia dominios legítimos.
     - Habilitar el análisis de paquetes para registrar intentos de acceso denegado.

3. **Asociar la política al cortafuegos:**
   - Aplicar la política creada al cortafuegos configurado.

4. **Actualizar las tablas de encaminamiento:**
   - ¿Qué tienes que hacer para que el tráfico hacia Internet pase (y sea filtrado) por el cortafuegos?

## Paso 3: probar y verificar el tráfico
1. **Lanzar instancias EC2:**
   - Crear una instancia EC2 en la subred pública de AZ1 (simulará el cliente).
   - Crear otra instancia EC2 en la subred privada de AZ1 (simulará el *backend* protegido).

2. **Probar conectividad:**
   - Desde la instancia pública, intentar acceder a `http://ejemplo-sitio-maligno.mil` y verificar que el acceso está bloqueado.
   - Intentar acceder a un dominio legítimo (como `www.example.com`) y verificar que el acceso está permitido.

3. **Monitorear con CloudWatch:**
   - Configurar **CloudWatch Logs** para el firewall y verificar los registros generados.
   - Analizar eventos de bloqueo y detección de tráfico sospechoso.

## Paso 4: responde a las preguntas...

... de la sección «Entrega» 😉

¡Y a la del paso 2.4! 😉 😉

&nbsp;

&nbsp;

&nbsp;

## Colofón
[![CC-BY-NC-SA](https://upload.wikimedia.org/wikipedia/commons/5/55/Cc_by-nc-sa_euro_icon.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es) 2024 [J. Garay](mailto:juliogaray.informatica@iespacomolla.es), [IES Poeta Paco Mollà](https://iespacomolla.es/), Alicante (España)
