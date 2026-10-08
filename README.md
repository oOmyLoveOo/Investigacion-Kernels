Participantes: Izan Villalba Corzo, Paul Alejandro Tamayo Mestanza  y Wuke Zhang

Los anticheats son softwares de seguridad que tienen como funcionalidad detectar modificaciones de sus programas que puedan ejecutar cambios en los ficheros. Tienen acceso a recursos ilimitados dentro del equipo y siguen un modelo de 3 componentes.

Surgieron porque los antivirus se ejecutaban en modo de usuario, por lo que los cheaters empezaron a hacer los exploits en modo Kernel, cuando hicieron los anticheats en el kernel, intentaron hacer los exploits en modo de virtualización para no tocar el sistema operativo pero crearon detecciones de las virtualizaciones.

Después de esta implementación, lo que hicieron los cheaters fue implementarlos mediante PCIe, que a día de hoy sigue investigándose.

Los anticheats funcionan en distintos "Círculos" que van de mas privilegios  a menos privilegios, que se representan como anillos, y funcionan como un flujo de componentes:

- Kernel driver: Se ejecuta como el primer componente del flujo dentro del Ring 0 (Privilegios de kernel elevados), encargándose de registrar e interceptar system calls, escanear la memoria y reforzar la protección del kernel.
- Servicio del modo usuario: Se ejecuta como un servicio de Windows, usualmente se ejecuta con permisos de system. Se comunica con los drivers del kernel gracias a los IOCTLs (llamadas al sistema en Linux). También revisa conexiones con los servidores, maneja los baneos además de coleccionar y transmitir telemetría (Se ejecuta en ring 3)
- Game Injected DLL: Inyectado o cargado por los procesos de los juegos. Revisa los checks por parte del modo usuario, se comunica con el servicio y sirve como endpoint para proteger los procesos de los juegos (Se ejecuta en Ring 3)

<img src="Priv_rings.svg" alt="Anillos de Privilegios" width="500">

Ejemplo del funcionamiento atraves de Battle Eye:

Battle Eye tiene 3 procesos, 2 en Ring 3 y 1 en Ring 0.

El proceso en Ring 0 se llama BEDaisy.exe. Este driver de kernel registra las creaciones de procesos, la creación de hilos y los manejos de operaciones de objetos. Para comunicarse con los componentes de ring 3, usa un IOCTL. 
Entrando a los procesos de Ring 3, el primero que funciona es BEService.exe. Este servicio se comunica por la red con los servidores de Battle Eye, recibe los resultados de los drivers y realiza los baneos (expulsar del servidor). Después, para comunicarse con el DLL, usa una Named Pipe (tuberia con codigo). El DLL se carga como un proceso al iniciar el juego. Es el responsable de que el Usermode sea legitimo dentro del contexto de los procesos del juego, y en caso de que no sean íntegros, se encarga de terminar la conexión con el servidor


Ahora, los anticheats en Linux no funcionan por 2 grandes razones. La primera es que estos anticheats están hechos para leer kernel de Windows y no de Linux, a nivel de usermode 
funciona correctamente, pero el kernel de Linux es distinto al de Windows por eso no puede leerlo. Y, en caso de que hicieran un anticheat que funcionase en el kernel de Linux, dicho kernel es totalmente personalizable y por eso los desarrolladores de anticheats no se molestan en hacerlos para Linux, sin embargo, hay algunos anticheats como Battle Eye que tienen soporte para Linux que corren en Wine.


INCIDENTE DE CROWDSTRIKE 

CrowdStrike es una empresa estadounidense para la protección de endpoints, la nube y datos mediante inteligencia artificial y servicios a respuestas a amenazas.

CrowdStrike produce un conjunto de productos de software de seguridad para empresas, diseñados para proteger las computadoras de los ciberataques. Falcon, el agente de detección y respuesta de endpoints de CrowdStrike, trabaja a nivel del núcleo del sistema operativo en equipos individuales para detectar y prevenir amenazas.

El motivo de la actualización fue que los atacantes empezaron a usar estos canales legítimos (Named Pipes) de Windows para esconder virus y moverse por los equipos sin ser detectados, mientras que el objetivo de la actualización CrowdStrike envió esa nueva regla para detectar y bloquear esa táctica específica, pero el error al leer la regla causó el desastre.

El incidente consistió en que se puso un límite codificado rígidamente en el código fuente de la nueva regla de CrowdStrike, pero la función que llamaba esa variable no verificó si la cantidad de datos que estaba leyendo superaba ese máximo antes de repetirlos, por eso en la actualización del 19 de julio de 2024, esa variable iteradora intentó leer un valor en la memoria más allá del límite, que provoco que los equipos entraran en un bucle del modo de arranque, haciendo que se reiniciasen o fallasen continuamente.

El alcance de este error llego al 1% de todos los equipos del mundo, una estimación de 8.5 millones de equipos de varios sectores. Los mas destacados fueron el de transporte aéreo en el cual lo sistemas de los aeropuertos, presentes en aviones y demás, se pusieron o en cuarentena o dejaron de funcionar,  provocando una parada en tierra en la que ningún vuelo de las aerolíneas estadounidenses como United, Delta y American Airlines pudo despegar, mientras que, en el sector de la salud, mas de 900 sistemas de varios países se vieron afectados, provocando interrupciones en los hospitales, haciendo que se tuviera que cambiar el formato de los tramites a papel temporalmente. Además, varias cirugías y emergencias fueron cerradas o pospuestas.

La solución definitiva al fallo global de CrowdStrike del 19 de julio de 2024 consiste en iniciar el ordenador afectado en Modo Seguro (o en el Entorno de Recuperación de Windows), acceder a la ruta C:\Windows\System32\drivers\CrowdStrike y eliminar permanentemente el archivo corrupto C-00000291*.sys.

CONCLUSION

Comprendiendo el funcionamiento de los anticheats, como verifican los archivos y como se comunican entre los distintos componentes, se puede observar como un simple fallo entre la comunicación de unos componentes puede dar lugar a fallos de múltiples sistemas, sumándole el riesgo de que una filtración de los datos de los sistemas pueda suceder.

FUENTES:

https://s4dbrd.github.io/posts/how-kernel-anti-cheats-work/ "https://s4dbrd.github.io/posts/how-kernel-anti-cheats-work/

https://www.crowdstrike.com/wp-content/uploads/2024/08/Channel-File-291-Incident-Root-Cause-Analysis-08.06.2024.pdf

https://www.crowdstrike.com/wp-content/uploads/2024/07/GlossaryOFTerms.pdf "https://www.crowdstrike.com/wp-content/uploads/2024/07/glossaryofterms.pdf

https://www.premiercontinuum.com/es/resources/interrupcion-microsoft-julio-2024

