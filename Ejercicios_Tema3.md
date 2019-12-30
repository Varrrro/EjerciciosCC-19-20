# Microservicios

## Ejercicio 1: Crear una aplicación básica que use _express_ para devolver alguna estructura de datos del modelo que se viene usando en el curso

En lugar de construir un pequeño microservicio con Node.js y _express_, voy a utilizar extractos del código desarrollado en Go para mi proyecto de la asignatura.

Para tener un servicio que escuche peticiones, debemos crear un _router_, definir una ruta con su controlador y escuchar en un puerto concreto.

```Go
router := mux.NewRouter()
router.HandleFunc("/groups/{groupid}", GetGroupHandler).Methods("GET")
http.ListenAndServe(":8080", router)
```

La función `GetGroupHandler` se encarga de procesar la petición y devolver un grupo concreto de nuestro modelo de datos. Para ello, se hace uso de un _manager_, que es el que controla todas las tareas que se realizan sobre dicho modelo.

```Go
func GetGroupHandler(w http.ResponseWriter, r *http.Request) {
    strid := mux.Vars(r)["groupid"]

	id, err := uuid.Parse(strid)
	if err != nil {
		w.WriteHeader(http.StatusBadRequest)
		return
	}

	g, err := gm.FetchGroup(id)
	if err != nil {
		if _, ok := err.(*NotFoundError); ok {
			w.WriteHeader(http.StatusNotFound)
		} else {
			w.WriteHeader(http.StatusInternalServerError)
		}

		return
	}

	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(&g)
}
```

Como se puede apreciar, se usa la `w.WriteHeader` para indicar el código de estado HTTP devuelto, mientras que con `json.NewEncoder(w).Encode` lo que hacemos es codificar el grupo obtenido del modelo de datos como JSON y escribirlo en el cuerpo de la respuesta usando un _encoder_.

```Go
func (gm *GroupsManager) FetchGroup(id uuid.UUID) (group.Group, error) {
	var g group.Group

	gm.DB.Preload("Members").First(&g, "id = ?", id.String())

	if g.ID != id.String() {
		return g, &NotFoundError{"No group found", id.String()}
	}

	return g, nil
}
```

Es la función `FetchGroup` del _manager_ la que se encarga de buscar un grupo concreto dado su ID. Para ello, hace uso de las funciones de consulta del ORM. En caso de no encontrar ningún grupo con el ID dado, devuelve un error `NotFoundError`, lo que permite a `GetGroupHandler` devolver un código HTTP 404.

## Ejercicio 2: Programar un microservicio en _express_ (o el lenguaje y marco elegido) que incluya variables como en el caso anterior

La función `GetGroupHandler` del ejercicio anterior usa variables de ruta. Si nos volvemos a fijar en su código, vemos que accede al valor de `groupid` de la petición con `mux.Vars(r)["groupid"]`.

## Ejercicio 3: Crear pruebas para las diferentes rutas de la aplicación

Vamos a ver dos ejemplos, uno de prueba unitaria y otro de prueba de integración, que van a servir para probar la ruta mostrada en el ejercicio 1.

La prueba unitaria la realizamos sobre las funciones del _manager_ para comprobar que su funcionalidad es correcta. Por ejemplo, para comprobar que `FetchGroup` devuelve correctamente un grupo, escribiríamos:

```Go
func TestFetchGroup(t *testing.T) {
	g, _ := manager.CreateGroup("test")

	_, err := manager.FetchGroup(uuid.MustParse(g.ID))

	if err != nil {
		t.Errorf("Couldn't fetch group. Error: %s", err.Error())
	}
}
```

Go es un poco peculiar a la hora de realizar pruebas. Como se puede ver, no hay un `expect` con una aserción como suele ocurrir en otros lenguajes. Lo que se hace en Go es lanzar errores manualmente cuando no se cumple alguna condición deseada, como se ve en el ejemplo.

A parte de estas pruebas unitarias, necesitamos pruebas de integración que verifiquen el funcionamiento de los distintos componentes al actuar juntos.

```Go
func TestGetGroup(t *testing.T) {
	g, err := manager.CreateGroup("test")
	if err != nil {
		t.Fail()
	}

	rec := httptest.NewRecorder()
	req, err := http.NewRequest("GET", "/groups/"+g.ID, nil)
	if err != nil {
		t.Fail()
	}

	router.ServeHTTP(rec, req)
	r := rec.Result()

	if r.Header.Get("Content-Type") != "application/json" {
		t.Errorf("Wrong mime-type [Expected]: application/json [Actual]: %s", r.Header.Get("Content-Type"))
	}

	if r.StatusCode != http.StatusOK {
		t.Errorf("Wrong status code [Expected]: %d [Actual]: %d", http.StatusOK, r.StatusCode)
	}

	body, err := ioutil.ReadAll(r.Body)
	if err != nil {
		t.Fail()
	}

	var g2 group.Group
	err = json.Unmarshal(body, &g2)
	if err != nil {
		t.Error("Couldn't parse response as group.")
	}

	if g.ID != g2.ID || g.Name != g2.Name {
		t.Error("Returned group isn't correct.")
	}
}
```

En esta función de prueba, realizamos una petición a la ruta deseada y obtenemos una respuesta, de la cual comprobamos su cabecera `Content-Type`, el código de estado HTTP y el contenido del cuerpo. Así, estamos realizando una prueba de "caja negra" en la que comprobamos que los valores devueltos por el servicio son los deseados.

## Ejercicio 4: Experimentar con diferentes gestores de procesos y servidores web _front-end_ para un microservicio que se haya hecho con antelación, por ejemplo en la sección anterior

Para mi proyecto, he decidido usar la herramienta _supervisord_ como gestor de procesos debido a la dificultad de usar _systemd_ dentro de un contenedor.

La configuración de nuestro microservicio `gmicro` se escribe en un fichero `gmicro.conf`.

```
[supervisord]
nodaemon=true

[program:gmicro]
directory=/usr/local
command=/usr/local/bin/gmicro

autostart=true
autorestart=true
```

En esta configuración, estamos indicando varias cosas:

* `nodaemon` hace que el proceso se ejecute en primer plano.
* `directory` establece el directorio de trabajo.
* `command` indica la ruta del ejecutable con el que se inicia el proceso. En nuestro caso, es la aplicación compilada.
* `autostart` hace que el proceso se lance automáticamente al añadirlo a _supervisord_.
* `autorestart` hace que _supervisord_ vuelva a lanzar el proceso automáticamente si este termina su ejecución por algún error.

Para lanzar el proceso, ejecutamos `supervisord -c /etc/supervisor/conf.d/gmicro.conf` después de haber movido el fichero de configuración a esta ruta.

> __NOTA:__ _supervisord_ debe ejecutarse siempre como _root_.

## Ejercicio 5: Usar _rake_, _invoke_ o la herramienta equivalente en tu lenguaje de programación para programar diferentes tareas que se puedan lanzar fácilmente desde la línea de órdenes

En mi proyecto, estoy utilizando _Tusk_ como _task runner_ o herramienta de ejecución de tareas. A continuación, se van a mostrar algunas de estas tareas.

```YAML
install:
	usage: Install project dependencies
    run: go mod download
```

Con `install` instalamos las dependencias del proyecto definidas en el fichero `go.mod`.

```YAML
test:
    usage: Run tests and generate coverage report
    run: go test ./internal/... -coverprofile coverage.txt
```

Con `test` ejecutamos todas las pruebas de la carpeta `/internal`, que es donde se encuentra el código de nuestra aplicación, y generamos un informe de cobertura `coverage.txt`.

```YAML
coverage:
    usage: Send coverage report to codecov.io
    run: bash <(curl -s https://codecov.io/bash)
```

Con `coverage` se envía el informe de cobertura a [codecov.io](https://codecov.io) para obtener informes detallados en la web.

```YAML
build:
    usage: Compile the application
    run: |
      set -e
      go build gmicro.go
      mv gmicro /usr/local/bin/
```

Con `build` compilamos el código de la aplicación y movemos el ejecutable a `/usr/local/bin`, que es el directorio para la ejecución de aplicaciones.

```YAML
run:
    usage: Run the application
    run: supervisord -c /etc/supervisor/conf.d/gmicro.conf
```

Por último, con `run` lanzamos la aplicación dentro del proceso gestionado por _supervisord_ haciendo uso de su fichero de configuración.
