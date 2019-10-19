# Arquitecturas software para la nube

### Buscar una aplicación de ejemplo, preferiblemente propia, y deducir qué patrón es el que usa. ¿Qué habría que hacer para evolucionar a un patrón tipo microservicios?

Para mi Trabajo de Fin de Grado, desarrollé un sistema para el ámbito médico compuesto por una aplicación móvil y un servidor con el que se comunicaba para realizar su funcionalidad. Claramente, se usa un patrón cliente-servidor simple, donde un único servidor monolítico es encargado de gestionar todas las funcionalidades.

Para convertir esta arquitectura a una basada en microservicios, tendríamos que aplicar *Domain Driven Design* para definir las entidades de nuestro dominio, que serían doctores, pacientes y tratamientos. Todas las funcionalidades que se llevan a cabo en el sistema deben dar una respuesta "inmediata" al usuario, por lo que descartaríamos una arquitectura de paso de mensajes o basada en eventos. Lo más recomendable sería una arquitectura basada en microservicios, con un microservicio para cada entidad. Podemos mantener la API actual y sus puntos de acceso implementando un API *Gateway* para el nuevo backend, de forma que no tendríamos que realizar ningún cambio en el cliente ya implementado. Por último, necesitaríamos también servicios adicionales, como un sistema de configuración remota o un sistema de logs, para gestionar de manera eficiente y unificada esta arquitectura.

### En la aplicación que se ha usado como ejemplo en el ejercicio anterior, ¿podría usar diferentes lenguajes? ¿Qué almacenes de datos serían los más convenientes?

La aplicación móvil que actúa como cliente está escrita en Dart usando Flutter, mientras que los nuevos microservicios podrían escribirse en multitud de lenguajes, incluso diferiendo unos de otros para atender a razones de eficiencia y rendimiento. En nuestro caso, al tratarse de funcionalidad sencilla, no es muy relevante el lenguaje usado y se podría usar el mismo para todos los servicios.

En cuanto al almacenamiento de datos, la aplicación recoge datos que están bien estructurados ya que se trata de gestión de pacientes e introducción de mediciones, además de que el volumen de datos es pequeño. Por tanto, las bases de datos SQL son más que aptas para nuestra tarea (actualmente usa PostgreSQL). Sin embargo, sí que sería recomendable pasar a una base de datos NoSQL si vemos que la cantidad de datos es muy grande, ya que estas bases de datos permiten una mejor escalabilidad y rendimiento en estos casos.
