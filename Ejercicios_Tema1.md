# Arquitecturas software para la nube

### Buscar una aplicación de ejemplo, preferiblemente propia, y deducir qué patrón es el que usa. ¿Qué habría que hacer para evolucionar a un patrón tipo microservicios?

Para mi Trabajo de Fin de Grado, desarrollé un sistema para el ámbito médico compuesto por una aplicación móvil y un servidor con el que se comunicaba para realizar su funcionalidad. Claramente, se usa un patrón cliente-servidor simple, donde un único servidor es encargado de gestionar todas las funcionalidades.

Para convertir esta arquitectura a una basada en microservicios, tendríamos que separar las funcionalidades básicas que se llevan a cabo en nuestro servidor monolítico. Esto nos da, esencialmente, tres partes: la autenticación de usuarios, el acceso a la base de datos (operaciones CRUD) y el cálculo de la dosis. Cada una de estas funcionalidades se podría implementar como un microservicio independiente, necesitando también de un sistema de mensajería para llevar a cabo la gestión de la arquitectura y de un sistema de descubrimiento para permitir la fácil incorporación de nuevos servicios en el futuro si fuera necesario.

### En la aplicación que se ha usado como ejemplo en el ejercicio anterior, ¿podría usar diferentes lenguajes? ¿Qué almacenes de datos serían los más convenientes?

La aplicación móvil que actúa como cliente está escrita en Dart usando Flutter, mientras que los nuevos microservicios podrían escribirse en multitud de lenguajes, incluso diferiendo unos de otros, para atender a razones de eficiencia y rendimiento. En nuestro caso, al tratarse de funcionalidad sencilla, no es muy relevante el lenguaje usado y se podría usar el mismo para los 3 servicios.

En cuanto al almacenamiento de datos, la aplicación recoge datos que están bien estructurados ya que se trata de gestión de pacientes e introducción de mediciones, además de que el volumen de datos es pequeño. Por tanto, las bases de datos SQL son más que aptas para nuestra tarea (actualmente usa PostgreSQL). Sin embargo, sí que sería recomendable pasar a una base de datos NoSQL si vemos que la cantidad de datos es muy grande, ya que estas bases de datos permiten una mejor escalabilidad y rendimiento en estos casos.
