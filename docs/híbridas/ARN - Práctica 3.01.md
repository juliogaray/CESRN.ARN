![Generalitat Valenciana - CEUO / IES Poeta Paco Mollá (Alicante)](https://raw.githubusercontent.com/juliogaray/recursos/main/img/Cabecera_CEUO_IESPPM_NOFSE_Transparente.svg)

# Conexión remota a una VPC de AWS usando *AWS Client VPN* y *OpenVPN*

## Objetivos
1. Familiarizarse con los servicios de AWS, específicamente con AWS Client VPN.
2. Aprender a configurar una conexión VPN utilizando OpenVPN.
3. Conectar un ordenador con Windows 10/11 a una VPC de AWS de forma segura.
4. Verificar la conectividad y acceder a recursos dentro de la VPC.

![Esquema de la pŕactica 3.01](../img/ARN-Práctica3.01.svg)

### Entrega
Deberás:
- Documentar el proceso seguido, **incluyendo capturas de pantalla.**
- **Explica en cada paso lo que estás haciendo y por qué es necesario o para qué sirve.**
- Responder a estas preguntas:
  1. ¿Qué métodos de autenticación ofrece AWS Client VPN y cuál consideras más seguro? ¿Por qué?
  2. ¿Qué importancia tiene utilizar certificados digitales en lugar de contraseñas para la autenticación en un enlace VPN?
  3. ¿Qué factores podrían influir en la velocidad de la conexión VPN y cómo podrías optimizarla?
  4. Reflexiona sobre los costos asociados con el uso de AWS Client VPN. ¿Qué estrategias podrías implementar para controlar o reducir estos costos?
  5. Como habrás comprobado al acabar la práctica, la gestión de la seguridad de las conexiones es relativamente compleja. Puedes imaginarte que trabajar así con certificados no es sencillo para gente con conocimientos informáticos limitados (o medios). ¿Qué opciones ofrece AWS para simplificar la conexión desde el punto de vista del usuario final, que se conecta a AWS a través de un VPN?

#### Entregas optativas
- Investiga y compara las diferentes opciones de cifrado que ofrece OpenVPN y AWS Client VPN. ¿Qué algoritmos de cifrado son los más utilizados y por qué?
- A continuación responde a esta pregunta: ¿cómo afecta la elección del algoritmo de cifrado al rendimiento y la seguridad de la conexión VPN? Proporciona ejemplos de algoritmos específicos y sus características.
- Configura y prueba alguna alternativa que hayas encontrado en la pregunta n.º 5 de la lista anterior.

&nbsp;


Elabora y entrega un único archivo PDF con todo lo especificado, incluidas las preguntas y sus respuestas correspondientes.  
No olvides indicar en tu documento todos los datos de cada elemento: nombre (dado por ti), ID de AWS del elemento, y características (rango de direcciones IP, conexiones, etcétera).


## Recursos necesarios
- Una cuenta de AWS con permisos para crear y gestionar recursos (obviamente).
- Un ordenador o máquina virtual con Windows 10 u 11. Si usas una máquina virtual, conéctala a la red con *adaptador puente.*
- Software cliente OpenVPN instalado en dicho ordenador.

---

### Paso 1: preparación del entorno en AWS

#### Paso 1.1: crear una VPC

   - Crea una nueva VPC con un rango de IP privado (por ejemplo, `10.0.0.0/16`).
   - Crea una subred privada dentro de la VPC, y asígnale el rango `10.0.3.0/24`.

#### Paso 1.2: crear los certificados digitales para la conexión

AWS Client VPN utiliza **certificados digitales** para autenticar tanto al servidor como a los clientes. Necesitarás crear estos certificados utilizando herramientas como `OpenSSL` o `easy-rsa`. Nosotros vamos a usar `easy-rsa`.

- **Instalar easy-rsa**:
  ```bash
  sudo apt-get install easy-rsa
  ```
  
- **Generar la Autoridad Certificadora (CA)**:
  ```bash
  make-cadir ~/easy-rsa
  cd ~/easy-rsa
  ./easyrsa init-pki
  ./easyrsa build-ca
  ```

- **Generar el certificado del servidor**:
  ```bash
  ./easyrsa build-server-full server nopass
  ```

- **Generar certificados para los clientes**:
  ```bash
  ./easyrsa build-client-full client1.domain.tld nopass
  ```

#### Paso 1.3: subir los certificados a AWS Certificate Manager (ACM)
Una vez generados los certificados, debes subirlos al servicio de gestión de certificados de AWS:

1. Ve a la consola de **AWS Certificate Manager (ACM)**.
2. Haz clic en **Importar certificado**.
3. Copia el certificado del servidor (el contenido del archivo `~/easy-rsa/pki/issued/server.crt`) como **Cuerpo del certificado**.
4. Copia la clave privada del servidor (el contenido del archivo `~/easy-rsa/pki/private/server.key`) como **Clave privada del certificado**.
5. Copia el certificado de la CA (el contenido del archivo `~/easy-rsa/pki/ca.crt`) como **Cadena de certificados**.


#### Paso 1.4: crear el *Endpoint* de *AWS Client VPN*
Ahora, crea el *endpoint* de AWS Client VPN:

1. En la consola de AWS, navega a **VPC > Client VPN Endpoints** (Puntos de conexión de cliente de VPN).
2. Haz clic en **Create Client VPN Endpoint**.
3. Rellena los siguientes campos:
   - **Server certificate ARN**: selecciona el certificado del servidor que has subido previamente.
   - **Client CIDR**: define un rango IP para los clientes que se conectarán (por ejemplo, `10.0.252.0/22`).
   - **Authentication Options**: selecciona **Mutual authentication** y selecciona el ARN del certificado de la CA que subiste.
   - **VPC**: selecciona la VPC y las subredes donde quieres que el tráfico de los clientes llegue.
   - **IP Address**: asegúrate de usar sólo **IPv4**.
   - **Security Groups**: asocia un grupo de seguridad que permitirá el acceso a las instancias dentro de tu VPC.

4. Haz clic en el botón **Create Client VPN Endpoint**.

5. **Configurar grupos de seguridad:**
   - Asegúrate de que los grupos de seguridad asociados a la VPC permitan el tráfico desde la IP del cliente VPN.

6. **Configurar el acceso y la autenticación**

   - Asocia el Client VPN Endpoint con una subred en la VPC
        Ve a la pestaña "Associations" y asocia una subred (esto permite que los clientes accedan a la red interna)  
        El proceso de asociación puede durar varios minutos. Dale tiempo hasta que aparezca el mensaje "Associated".

   - Configura reglas de autorización
        Ve a "Authorization Rules" y añade una regla para 0.0.0.0/0, permitiendo acceso.

### Paso 2: Configuración del cliente

En la página de **Puntos de conexión de Client VPN**, haz clic en **Descargar la configuración del cliente**.

#### 2.1: GNU/Linux (basado en Debian)

1. **Instala OpenVPN:**
```bash
sudo apt install openvpn
```

2. **Modifica el archivo de configuración descargado de AWS:**

Incluye en el archivo las siguientes líneas por encima de la línea que dice `<CA>`:
```ini
ca /home/jgg/easy-rsa/pki/ca.crt
cert /home/jgg/easy-rsa/pki/issued/client1.domain.tld.crt
key /home/jgg/easy-rsa/pki/private/client1.domain.tld.key
```

**OJO:** cambia la ruta para que refleje la localización de tus archivos *.crt y *.key generados en el último apartado del paso 1.2.    

> NOTA: no sería necesario este paso si no hubiésemos seleccionado la opción **Mutual authentication** en las opciones de autenticación del punto de acceso VPN.

#### 2.2: Windows 10/11

1. **Instalar OpenVPN:**
   - Descarga e instala OpenVPN desde su [página oficial](https://openvpn.net/community-downloads/).
   - Importa el archivo de configuración descargado desde AWS Client VPN en OpenVPN, y modifícalo como hemos hecho para GNU/Linux.

2. **Conectar a la VPN:**
   - Abre OpenVPN y selecciona el perfil de VPN que has importado.
   - Conecta a la VPN utilizando las credenciales proporcionadas por AWS.

3. **Verificar la Conexión:**
   - Una vez conectado, verifica que tienes acceso a la VPC.
   - Puedes hacer ping a una instancia EC2 dentro de la VPC o intentar acceder a un recurso interno.

### Paso 3. Ejercicios prácticos

1. **Acceso a una instancia EC2:**
   - Crea una instancia EC2 dentro de la VPC.
   - Intenta conectarte a la instancia EC2 utilizando SSH desde tu ordenador a través de la VPN.

2. **Configurar un Servidor Web:**
   - Instala un servidor web (por ejemplo, Apache o Nginx) en la instancia EC2.
   - Accede al servidor web desde tu navegador utilizando la IP privada de la instancia.

3. **Monitoreo y Registros:**
   - Revisa los registros de conexión en AWS Client VPN para verificar que la conexión se ha establecido correctamente.
   - Utiliza CloudWatch para monitorear el tráfico de la VPN.

## Paso 4: responde a las preguntas y realiza las tareas...

... de la sección «Entrega» 😉
&nbsp;

&nbsp;

&nbsp;

## Colofón
[![CC-BY-NC-SA](https://upload.wikimedia.org/wikipedia/commons/5/55/Cc_by-nc-sa_euro_icon.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es) 2025 [J. Garay](mailto:juliogaray.informatica@iespacomolla.es), [IES Poeta Paco Mollà](https://iespacomolla.es/), Alicante (España)


