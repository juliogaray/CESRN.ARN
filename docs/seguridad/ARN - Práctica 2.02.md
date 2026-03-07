![Generalitat Valenciana - CEUO / IES Poeta Paco Mollá (Alicante)](https://raw.githubusercontent.com/juliogaray/recursos/main/img/Cabecera_CEUO_IESPPM_NOFSE_Transparente.svg)

# Configuración y uso de NACL y *Security Groups*

En esta práctica nos centraremos en la implementación y configuración de **NACL (listas de control de acceso de red)** y ***Security Groups*** en una **VPC**. Aprenderemos cómo interactúan estas herramientas para proteger recursos, cómo priorizar reglas y cómo manejar configuraciones específicas para casos de uso avanzados.

## Objetivos
1. Comprender la interacción entre **NACL** y ***Security Groups**.*
2. Configurar reglas avanzadas para controlar tráfico de red en una VPC.
3. Probar escenarios de acceso permitido y denegado.
4. Implementar una estrategia de seguridad en capas con ambas herramientas.

### Entrega
Deberás:
- Documentar el proceso seguido, **incluyendo capturas de pantalla.**
- Responder a estas preguntas:
  - ¿Qué sucede si configuras reglas contradictorias entre una NACL y un *Security Group?* ¿Qué regla prevalece?
  - ¿Por qué es importante usar tanto NACL como *Security Groups* en una arquitectura en AWS?
  - ¿Cómo puedes mejorar la seguridad de las instancias públicas sin afectar la funcionalidad de la aplicación?
  - ¿Qué ventajas tiene el uso de NACL para bloquear tráfico sospechoso a nivel de subred?

Elabora y entrega un único archivo PDF con todo lo especificado, incluidas las preguntas y sus respuestas correspondientes.  
No olvides indicar en tu documento todos los datos de cada elemento: nombre (dado por ti), ID de AWS del elemento, y características (rango de direcciones IP, conexiones, etcétera).

---

## Infraestructura

Esta práctica incluye:
- Una VPC con varias subredes, públicas y privadas.
- Una instancia EC2 en cada subred para probar conectividad.
- Configuración de NACL y *Security Groups* con reglas específicas.
- Un escenario práctico para simular un ataque y aplicar restricciones.

---

## Paso 1: configuración inicial de la VPC
1. **Crea una VPC con las siguientes configuraciones:**
   - CIDR: `10.0.0.0/16`
   - Subred pública en AZ1: `10.0.1.0/24`
   - Subred privada en AZ1: `10.0.2.0/24`

2. **Agrega una Internet Gateway:**
   - Asocia una **Internet Gateway (IGW)** con la VPC.
   - Configura una tabla de enrutamiento para la subred pública con una ruta a `0.0.0.0/0` apuntando a la IGW.

3. **Crea un NAT Gateway:**
   - Configura una NAT Gateway en la subred pública.
   - Actualiza la tabla de enrutamiento de la subred privada para dirigir el tráfico a Internet a través de la NAT Gateway.

## Paso 2: configuración de *Security Groups*
1. **Crea un *Security Group* para la instancia pública:**
   - Permite tráfico SSH (puerto 22) desde la IP pública que estés usandp (tendrás que averiguarla y anotarla; si estás en el IES, anota la IP pública visible desde el exterior, y luego usa el rango /24 en que esté dicha IP pública).
   - Permite tráfico ICMP (ping) desde cualquier IP.
   - Deniega cualquier otro tráfico.

2. **Crea un *Security Group* para la instancia privada:**
   - Permite tráfico SSH (puerto 22) solo desde la instancia pública.
   - Permite tráfico ICMP (ping) desde la instancia pública.
   - Deniega cualquier otro tráfico.

3. **Asigna los *Security Groups* a las instancias EC2:**
   - Instancia pública: asocia el *Security Group* público.
   - Instancia privada: asocia el *Security Group* privado.

## Paso 3: configuración de NACL
1. **Configura una NACL para la subred pública:**
   - Añade una regla para permitir todo el tráfico entrante y saliente por defecto.
   - Añade una regla para denegar tráfico SSH entrante (puerto 22) desde cualquier IP excepto la del estudiante.

2. **Configura una NACL para la subred privada:**
   - Permite todo el tráfico entrante y saliente entre la subred pública y privada.
   - Deniega tráfico saliente hacia direcciones IP externas no autorizadas.

3. **Asocia las NACL con las subredes:**
   - Subred pública: asocia la NACL pública.
   - Subred privada: asocia la NACL privada.

## Paso 4: probar conectividad
1. **Verifica acceso a la instancia pública:**
   - Intenta acceder mediante SSH desde tu máquina local.
   - Prueba un ping desde cualquier otra IP y verifica que está permitido.

2. **Prueba conectividad con la instancia privada:**
   - Desde la instancia pública, intenta conectarte a la privada usando SSH.
   - Prueba ping desde la instancia pública hacia la privada.
   - ¿Funcionan estas pruebas? ¿Es lógico?
   - ¿Funcionaría un acceso HTTP desde la instancia pública a la privada? ¿Por qué?

3. **Simula un ataque:**
   - Desde una IP diferente (desde otra _subnet_ o usando otra conexión), intenta acceder a la instancia pública por SSH y verifica que el tráfico es bloqueado por las reglas de NACL y *Security Group.*

## Paso 5: analizar y solucionar problemas
1. **Modifica las reglas de NACL:**
   - Añadir una regla temporal para bloquear todo el tráfico ICMP hacia la subred pública y comprueba el impacto.
   - Restaura la regla original.

2. **Modifica los *Security Groups:***
   - Elimina la regla que permite tráfico SSH hacia la instancia privada y prueba la conectividad.
   - Restaura la regla original.

3. **Monitoriza con CloudWatch Logs:**
   - Habilita los registros de flujo de VPC para analizar los intentos de conexión bloqueados.

## Paso 6: responde a las preguntas...

... de la sección «Entrega» :wink:

¡Y a las del paso 4.2! :wink: :wink:

&nbsp;

&nbsp;

&nbsp;

## Colofón
[![CC-BY-NC-SA](https://upload.wikimedia.org/wikipedia/commons/5/55/Cc_by-nc-sa_euro_icon.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.es) 2024 [J. Garay](mailto:juliogaray.informatica@iespacomolla.es), [IES Poeta Paco Mollà](https://iespacomolla.es/), Alicante (España)
