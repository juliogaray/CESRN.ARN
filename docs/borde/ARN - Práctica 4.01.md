![Generalitat Valenciana - CEUO / IES Poeta Paco Mollá (Alicante)](https://raw.githubusercontent.com/juliogaray/recursos/main/img/Cabecera_CEUO_IESPPM_NOFSE_Transparente.svg)

# Implementación de Amazon CloudFront con ubicaciones de borde *(Edge Locations)*. Lambda@Edge.

En esta práctica probaremos el servicio Amazon CloudFront de distribución global de contenidos. En principio será necesario realizar la práctica desde la *landing zone* proporcionada por la Conselleria de Educación.

## Objetivos
- Distribuir contenido a través de Amazon CloudFront y analizar cómo el servicio selecciona automáticamente la ubicación de borde más cercana para mejorar la latencia. Configuraremos una distribución de CloudFront para servir contenido estático desde un *bucket* de S3 y contenido dinámico desde un servidor EC2.
- Verificar cómo CloudFront redirige el tráfico mediante pruebas con una VPN.
- Usar Lambda@Edge para modificar el contenido dinámicamente en función de la geolocalización.

### Entrega
Deberás:
- Documentar el proceso seguido, **incluyendo capturas de pantalla.**
- Responder a estas preguntas:
  - ¿Cómo afecta la geolocalización al acceso a CloudFront?  
  - ¿Cómo se elige automáticamente la ubicación de borde más cercana?  
  - ¿Cómo podríamos mejorar aún más el rendimiento usando cacheado y TTL en CloudFront?  

Elabora y entrega un único archivo PDF con todo lo especificado, incluidas las preguntas y sus respuestas correspondientes.  
No olvides indicar en tu documento todos los datos de cada elemento: nombre (dado por ti), ID de AWS del elemento, y características (rango de direcciones IP, conexiones, etcétera).

---

## Parte 1 – configuración del contenido de origen (S3 y EC2)

### Paso 1: creación de un *bucket* en Amazon S3
1. Accede a la consola de AWS y ve a **S3**.  
2. Crea un nuevo bucket con un nombre único (por ejemplo, `mi-bucket-cloudfront-lab`).  
3. Sube un archivo HTML (`index.html`) y una imagen (`logo.png`).  
4. Activa la opción **Hacer público el contenido** en la configuración del bucket.  
5. Copia la URL del archivo `index.html` para comprobar que se puede acceder directamente.  

### Paso 2: configuración de un servidor EC2
1. Lanza una nueva instancia EC2 con Amazon Linux 2.  
2. Conéctate a la instancia vía SSH y ejecuta:  
   ```bash
   sudo dnf update -y
   sudo dnf install -y httpd
   sudo systemctl start httpd
   sudo systemctl enable httpd
   echo "Bienvenido a mi servidor web en EC2" | sudo tee /var/www/html/index.html
   ```
3. Abre el puerto 80 en el **Security Group** para permitir tráfico HTTP.  
4. Copia la IP pública de la instancia y accede desde un navegador para verificar que el servidor web funciona.  

---

## Parte 2 – creación y configuración de Amazon CloudFront

### Paso 3: configuración de una distribución de CloudFront
1. Accede a **CloudFront** en la consola de AWS.  
2. Crea una nueva distribución.  
3. En el **origen**, selecciona el *bucket* de S3.  
4. Activa la opción de **habilitar la compresión automática**.  
5. En **Restricciones geográficas**, bloquea el acceso desde Suiza (ese peligroso país 😉).
6. Crea la distribución y espera a que se propague.  

### Paso 4: comprobación del acceso al contenido desde ubicaciones de borde
1. Copia la URL de la distribución de CloudFront.  
2. Accede al archivo `index.html` a través de CloudFront y verifica la carga.  
3. Usa la herramienta **traceroute** para identificar la ubicación de borde más cercana:
   ```bash
   traceroute <URL_de_CloudFront>
   ```
4. Comparar tiempos de respuesta con el acceso directo a S3.  

---

## Parte 3 – test con VPN y análisis de ubicaciones de borde

### Paso 5: comprobación del cambio de ubicación de borde con VPN
1. Instalar un cliente de VPN gratuito
   - Opciones recomendadas:  
     - **ProtonVPN** (requiere cuenta gratuita).  
     - **Windscribe** (ofrece ubicaciones gratuitas).  
     - **Opera Browser VPN** (si no se quiere instalar software adicional).  

2. Probar CloudFront desde diferentes ubicaciones
   - Conéctate a la VPN en **un país extranjero (que no sea Suiza)**.
   - Usa **traceroute** para verificar desde dónde se está cargando la página:  
     ```bash
     traceroute <URL_de_CloudFront>
     ```
   - Anota la diferencia en los resultados y tiempos de respuesta.

3. Comparar ubicaciones de borde
   - Cambia la VPN a otro país y repite la prueba.
   - Usa herramientas como **WhatsMyIP** para verificar la IP y ubicación.  
   - Comprueba si el contenido sigue cargando rápidamente y si cambia la latencia.  

4. Comprueba el funcionamiento de las restricciones geográficas  
   - Cambia el país de destino de la VPN a Suiza y repite la prueba.
   - Comprueba que no puedes acceder a los contenidos.

---

## Parte 4 – Configuración de un segundo origen (EC2) en CloudFront

### Paso 6: Agregar un segundo origen a CloudFront
1. En **CloudFront**, edita la distribución y agrega un nuevo origen apuntando al nombnre de dominio del EC2.  
2. Configura reglas en **Behaviors** para que las rutas `/static/*` se sirvan desde S3 y `/api/*` desde EC2.  
3. Prueba la distribución con:  
   - `https://<cloudfront-url>/static/index.html` (debería cargar desde S3).  
   - `https://<cloudfront-url>/api/` (debería cargar desde EC2).  

> **NOTA:** cuando definas el origen en el punto 1 de este paso, selecciona HTTP si tu máquina EC2 sólo sirve HTTP. Puedes seleccionar HTTPS si configuras la instancia para servir HTTPS

---

## Parte 5 – Uso de Lambda@Edge para personalizar respuestas

### Paso 7: creación de una función Lambda@Edge

Lambda@Edge se ejecuta en los edge locations de CloudFront, lo que permite modificar las peticiones antes de que lleguen al origen o modificar las respuestas antes de que se entreguen al usuario.  

#### Ejemplo de caso de uso:
Personalizar un mensaje en la web en función del país desde el que accede el usuario.  

#### 1. Crea una función Lambda en AWS
1. Ve a la consola de **AWS Lambda** y crea una nueva función.  
2. Selecciona **Crear desde cero** y configura:  
   - Nombre: `CloudFrontGeoLambda`  
   - Tiempo de ejecución: **Node.js 18.x**  
   - Rol: **Crear un nuevo rol con permisos básicos de ejecución de Lambda**  
3. Reemplaza el código por el siguiente:  

```javascript
exports.handler = async (event) => {
    const request = event.Records[0].cf.request;
    const headers = request.headers;

    // Obtener el país del usuario desde el encabezado de CloudFront
    let country = "desconocido";
    if (headers['cloudfront-viewer-country']) {
        country = headers['cloudfront-viewer-country'][0].value;
    }

    // Modificar la respuesta HTML si es una solicitud a index.html
    if (request.uri === "/index.html") {
        const response = {
            status: '200',
            statusDescription: 'OK',
            headers: {
                'content-type': [{ key: 'Content-Type', value: 'text/html' }],
            },
            body: `<html><body><h1>¡Hola, visitante de ${country}!</h1></body></html>`,
        };
        return response;
    }

    return request;
};
```

4. Guarda y publica la función.  

#### 2. Asocia Lambda@Edge a CloudFront
1. En la consola de **CloudFront**, selecciona tu distribución.  
2. Ve a la pestaña **Behaviors** y selecciona el comportamiento que maneja `index.html`.  
3. En la opción **Lambda Function Associations**, haz clic en **Editar**.  
4. En **Origin Request**, selecciona la función Lambda que creaste.  
5. Asegúrate de que está configurado para ejecutarse en **CloudFront Edge**.  

---

### Paso 8: prueba tu función Lambda@Edge con VPN

1. Accede a tu sitio a través de CloudFront (`https://<cloudfront-url>/index.html`).  
2. La página debería mostrar `¡Hola, visitante de [país_de_origen]!`.  
3. **Activa una VPN y cambia de ubicación (ej. Austria o Alemania).**  
4. Recarga la página y verifica que el mensaje cambia según el país detectado.  
5. Usa la herramienta `curl` para comprobar la cabecera `cloudfront-viewer-country`:  
   ```bash
   curl -I https://<cloudfront-url>/index.html
   ```
   Deberías ver un encabezado similar a:  
   ```
   cloudfront-viewer-country: AT
   ```

---


&nbsp;

&nbsp;

&nbsp;

## Colofón
[![CC-BY-NC-SA](https://upload.wikimedia.org/wikipedia/commons/5/55/Cc_by-nc-sa_euro_icon.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es) 2025 [J. Garay](mailto:juliogaray.informatica@iespacomolla.es), [IES Poeta Paco Mollà](https://iespacomolla.es/), Alicante (España)

