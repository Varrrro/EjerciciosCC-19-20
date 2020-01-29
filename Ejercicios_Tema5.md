# Provisionamiento en infraestructuras virtuales

## Ejercicio 1: Provisionar una máquina virtual en algún entorno con los que trabajemos habitualmente (incluyendo el programa que se haya realizado para los hitos anteriores) usando Salt.

## Ejercicio 2: Desplegar la aplicación de cualquier otra asignatura donde se tenga ya el código fuente con todos los módulos necesarios usando un _playbook_ de Ansible.

Vamos a ver como ejemplo el despliegue de uno de los microservicios de mi proyecto de la asignatura: `gmicro`.

```yml
---
- hosts: gmicro
  become: yes
  vars_files:
    - env/gmicro.yml
  roles:
    - common
```

Para no confundirnos, hemos apodado cada máquina con el nombre del servicio que acogen. Por eso, este _playbook_ actúa sobre los _hosts_ `gmicro`. Hacemos uso de `become` para ejecutar las tareas como superusuario, ya que nos será necesario. En el apartado `vars_files` indicamos la ruta al fichero que contiene las variables de entorno necesarias para esta máquina (valores de conexión a la cola AMQP, usuario y contraseña de acceso a la base de datos, ...) y, en el apartado `roles`, añadimos un rol `common` que hemos creado para el proyecto.

Vamos a ver ahora las tareas.

```yml
tasks:
    - name: Create docker network
      docker_network:
        name: main

    - name: Run RabbitMQ container
      docker_container:
        name: rabbit
        image: rabbitmq:3
        detach: yes
        networks:
          - name: main
        purge_networks: yes
        ports:
          - "5672:5672"

    - name: Run PostgreSQL container
      docker_container:
        name: db-gmicro
        image: postgres:12
        detach: yes
        networks:
          - name: main
        purge_networks: yes
        env:
          POSTGRES_USER: "{{ db_user }}"
          POSTGRES_PASSWORD: "{{ db_pass }}"
          POSTGRES_DB: "{{ db_name }}"

    - name: Run gmicro container
      docker_container:
        name: gmicro
        image: varrrro/pay-up:gmicro
        detach: yes
        networks:
          - name: main
        purge_networks: yes
        ports:
          - "8080:8080"
        env:
          RABBIT_CONN: "{{ rabbit_conn }}"
          DB_TYPE: "{{ db_type }}"
          DB_CONN: "{{ db_conn }}"
          EXCHANGE: "{{ exchange }}"
          QUEUE: "{{ queue }}"
          CTAG: "{{ ctag }}"
```

En este caso, el despliegue del microservicio en la máquina virtual se realiza con Docker. No vamos a entrar en los detalles específicos de este despliegue, sino que vamos a revisar los aspectos interesantes de un _playbook_ de Ansible.

Básicamente, usamos una serie de `tasks` para crear una red de Docker y ejecutar una serie de contenedores conectados a dicha red. Para lanzar los contenedores, usamos las variables definidas en el fichero de indicado en `vars_files` con la forma `"{{ nombre }}"`.

## Ejercicio 3: Crear un rol `common` que haga ciertas tareas comunes que vayamos a usar en todas las máquinas virtuales de los microservicios de la asignatura (o, para el caso, cualquier otra asignatura).

De nuevo, vamos a ver el rol `common` creado para el proyecto de la asginatura y usado en el _playbook_ visto en el ejercicio anterior. Un rol de Ansible se compone de varias carpetas como `vars` (variables que usa el rol), `defaults` (valores por defecto para estas variables), `tests` (pruebas del correcto funcionamiento del rol) y `tasks` (las tareas que ejecuta el rol). En nuestro caso, al tratarse de un rol sencillo que usamos solo en nuestro proyecto, solo usamos la carpeta `tasks` con las tareas definidas en un fichero `main.yml`.

```yml
---
- name: Install docker
  include_role:
    name: geerlingguy.docker
  vars:
    docker_install_compose: false

- name: Install pip3
  apt:
    pkg: python3-pip
    update_cache: yes

- name: Install docker module
  pip:
    name: docker
```

Como aspecto a destacar, estos ficheros de tareas se diferencian de los _playbooks_ en que solo contienen una lista de _tasks_, no especifican ni _hosts_ (usan los del _playbook_ que lo llama) ni variables (usan las definidas en la carpeta `vars` del rol).
