# 12 factor
- software como servicio
## Caracteristicas
- declarativo
- portabilidad maxima
- uso de plataformas en la nube.
- Minimizar la diferencia entre distintos ambientes(production, staginsX, desarrollo), permitiendo entrega continua(continuous deployment)
- facil de escalar, sin cambios significativos en la arquitectura.

## 1. Codigo base
- Un codigo base dirigido por un software de version.
- muchas entregas
- Codigo base -> repositorio unico

![One codebase maps to many deploys](codebase.png)

- correlacion 1 a 1
 - si hay multiples codigos base no es una app, es un sistema distribuidos
 - aplicaciones que comparter codigo es una violacion de 12-factor, solo se comparte codigo mediante librerias, incluidad mediante un manejador de depencias.
- deploy (desplegar)-> instancia de una app en ejecucion
- deployment(despliege) -> crear un deploy
- el codigo base es el mismo en todos los deploy, pero con diferente version del mismo.

## 2. Dependencias
- todas las dependencias deben declararse mediante una manifesto.
- usa herramientas de dependencias aisladas ("isolation dependecies") durante la ejecucion, para asegurar que no hay dependencias implicitas

## 3. Config
Almacenar la configuracion en el ambiente de ejecucion.
- estricta separacion del codigo y de la configuracion.
- razon -> configuracion varia entre deploy, pero el codigo no.
- ventaja -> open source sin arriesgar seguridad.
- configuracion en variables del sistema.
- vars son manejadas independientemente para cada deploy.

## 4. Servicios Horneados (backing services).
Tratar servicios horneados como recursos adjuntos
- Servicios Horneados("backing services"): es cualquier servicios que se consulme o utiliza mediante la red como parte de una operacion.
- Ejemplos: BBDD, messaging autorización, servicios SMTP (email), cache ...
- Servicios Horneados son administrados por las misma personas que administran las aplicaciones.
- no distinciones entre servicios locales o externos.
- para la app ambos son adjuntados, accesibles y configurados localmente.
- debe ser posible intercambiar un recurso externo por otro interno y viceversa con solo un cambio en la configuracion de la app, es decir sin hacer ningun cambio en el codigo.
- cada servicio horneado es un "recurso".
## 5. Compilar| contruccion(build), liberar| lanzamiento(release), ejecutar(run).
Separacion estricta entre compilacion, liberacion y ejecucion.
### Etapa de compilacion | contrucion(build stage).
- convertir el codigo del repositorio(lugar donde se guarda el codigo) a un ejecutable.
- utilizando la version del codigo de un commit(version especifica) por el proceso de deployment(poner el ejecutable en produccion), la contruccion recoge(fetches, buscalo que no se traducirlo) las dependencia y compila el codigo.
## Etapa de liberacion(release stage)
- coje el ejecutable producido por la etapa anterior(build stage) y lo combina con la configuracion.
- ahora el deploy esta listo para su inmediata ejecucion en el ambiente de ejecucion.
## Etapa de ejecucion(aka runtime).
- la app es ejecutada en el ambiente de ejecucion, al lanzar algunos procesos de la app contra la liberacion(mejor traduccion) seleccionada.

![One codebase maps to many deploys](release.png)

- toda release tiene que tener un numero unico de identificion.(e.j fecha de create en milisegundos o numero de version). Este numero jamas podra ser mutado, se tendra que crear uno nuevo.

Build(construcciones) son iniciadas por el desarrollador cada vez que el codigo es deploy(liberado??).
Runtime por el contrario pueden ser producidas automaticamente. Por eso la etapa de ejecucion debe matener el minimo de partes posible.
## 6. Procesos
Ejecutar la aplicacion como uno o mas procesos inmutables(su estado no cambia).
- procesos son inmutables y no comparten nada entre ellos. Todo los dataos que nesiten ser persistentes deben ser almacenados en servicio horneado mutable(como por ejemplo una base de datos.)

## 7. Anclaje del puerto(port binding)
- la app debe ser totalmente contenida en si misma y no confiar en la injecion en tiempo de ejecucion de un servidor web en el ambiente de ejecuion.
- la app web export HTTP como un servicio anclandose a un puerto.
- notese que tambien el anclaje del puerto significa que una app puede convertirse en un servicio horneado de otra app, dando la URL de la app horneada como recurso en la configuracin de la app consumidora.
## 8. Concurrencia.
Scalar via el modelo de proceso (6).
- "Los procesos de la aplicación de doce factores toman fuertes señales del modelo de proceso de unix para ejecutar daemons de servicios"
- esto no significat que las app no puedan utilizar su propio pool de procesos, pero la aplicacion debe ser capaz de crear multiples procesos en diferentes maquinas fisicas.
- formacion de procesos: array de tipo de procesos  y numero de procesos por cada tipo.
- una app jamas debe ser demonizada(auto restart), debe utilizar el manejador de procesos del Sistema Operativo, para responder a crases del sistema, para logging y para manajar inicializaciones del usuario como restarts y para el sistema.

## 9. Disponibilidad.
Maximizar la robustez con rapida inicializacion y apagado "graceful"(agradecido??)
- app tiene que poder apagarge e iniciarse en cualquier momento. Esto facilita una escalabilidad rapida y elastica, rapido deployment de cambios en el codigo o configuracion, y robustez de deploys en produccion.
- Pequeños tiempos de inicializacion dan una mayor agilidad para los procesos de releases(lanzamientos) y para escalar; y esto ayuda a la robustez, por que el manejador de procesos puede mover con mayor facilidad los procesos (app) a una nueva maquina fisica cuando es requerido.
- los procesos son apagado de correcta cuando reciven un SIGTERM del manejador de procesos.
- Procesos deben ser tambien robustos ante una muerte subita, en casa de fallo por Hardware. Recomendacion Beanstalkd(Que es esto????).

## 10. Paridad Desarrollo / Produccion.
Mantener desarrollo, staging y produccion tan similares como sea posible.
- Problemas historicos:
 - Problema de tiempo: Un desarrollador puede trabajar en el codigo durante dias, semanas o incluso meses antes de poner en produccion.
 - Problema personal: Desarrolladores escribe codigo, Ingenieros operaciones lo ponen en produccion.
 - Problema de las herramientas: Desarrolladores usan diferantes herramientas que en produccion.
- Esta arquitectura esta diseñada para el deploymente continuo al manatener los problemas entre desarrollo y produccion tan tequeños como sea posible.
 - Hacer el problema de tiempo pequeño: un desarrolador debe escribir codigo y ser capaz de poner lo en produccion en horas o incluso minutos.
 - Hacer el problema personal pequeño: desarroladores que escriben el codigo deben estas implicados en el proceso de poner el SW en produccion y observar su comportamiento en produccion.
 - Hacer el problem de las herramientas pequeño: mantener desarrollo y produccion tan similar como sea posible.
- No utilizar las mismas herramientas en desarrollo y produccion puede llevar a grandes problemas en produccion y es un gran problema para el deployment continuo.

## 11 Logs
- Logs dan una vision del comportamiento de la aplicacion.
- Son un flujo de agregaciones, eventos ordenados en tiempo recojidos de los flujos de salida de todos los procesos ejecutados y sus backing services(servicios horneados).
- las applicaciones no debe manejar archivos de logs. En vez de esos los processos escriben a la salida estandar (stdout).
- en desarrollo local, el programador vera el comportamiento de la app, como un flujo continuo de eventos en su terminal.
- en staging o produccion, cada flujo de proceso sera capturado por sus ambiente de ejecucion. recojido junto con otros flujo de la app, y dijido a uno mas destinos para su analisis y archivo.
- Estos archivos, no son visibles ni configurables por la aplicacion por el contrario seran manejos por el ambiente de ejecucion.

- Estos flujos pueden ser enviados a sistemas de analisis de datos. Estos sistemas permiten:
 - Encontrar eventos especificos en el pasado.
 - Crear estadisticas y analizar comportemientos de la aplicacion, como request por minuto.
 -  Activar alertas de acuerdo de heuristicas definidas por el usuario.

## 12 Administrar procesos.

- La administracion de procesos debe ser en un ambiente identico al que utilizan las apps.
- Estos se ejecutaran contra un lanzamiento(release), usando el mismo docigo y configuraciones como cualquier otro proceso ejecutado contra esta release.
- Codigo de administracion debe ser embarcado como codigo de aplicacion, para evitar problemas de sincronizacion.
