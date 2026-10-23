# Taller 1 — API REST de Videos con Flask-RESTful

## Objetivo

Construir una **API REST** sencilla para gestionar videos usando **Flask**, **Flask-RESTful** y **Flask-SQLAlchemy**, guardando los datos en una base de datos **SQLite**. Al terminar el taller sabrás:

- Preparar un entorno de desarrollo Python aislado con `venv`.
- Organizar un proyecto Flask siguiendo el patrón *application factory*.
- Definir un modelo de datos con SQLAlchemy y persistirlo en SQLite.
- Implementar un recurso REST con los métodos `GET`, `PUT`, `PATCH` y `DELETE`.
- Validar los datos de entrada con `reqparse` y serializar las respuestas con `marshal_with`.
- Responder con los códigos de estado HTTP adecuados (200, 201, 204, 400, 404, 409).
- Probar una API con `curl` y documentar las pruebas realizadas.

## Requisitos previos

- Ubuntu (o WSL2 sobre Windows) con acceso a una terminal.
- Python 3.10 o superior instalado. Verifícalo con:

  ```bash
  python3 --version
  ```

- Conocimientos básicos de Python (funciones, clases, diccionarios) y del protocolo HTTP (métodos y códigos de estado).

> Todos los comandos de esta guía se ejecutan dentro de tu distribución de Linux Ubuntu.
> Si usas Windows, abre tu terminal de Ubuntu/WSL2 antes de continuar.

## Lista de tareas del taller

Esta es la hoja de ruta que seguiremos. Haz un `commit` al terminar cada paso:

1. Estructura del Proyecto
2. Configuración de la aplicación.
3. Modelo de datos.
4. Application factory y punto de entrada.
5. Recurso REST `Video`.
6. Congelar dependencias.
7. Ejecutar y probar la API.
8. Documentar las pruebas.
9. Implementar una mejora.
10. Ajustar el README.


## Paso 1 — Estructura del Proyecto

Este proyecto usa una estructura típica de un proyecto Flask pequeño, basada en el patrón *application factory* (una función que construye la app). Esto facilita crecer el proyecto más adelante (agregar pruebas, más recursos, configuración por ambiente, etc.).

```
lp3-taller1/
├── app/
│   ├── __init__.py          # Application factory (create_app)
│   ├── config.py            # Configuración por ambiente
│   ├── models/
│   │   ├── __init__.py      # Instancia de SQLAlchemy (db)
│   │   └── video.py         # Modelo VideoModel
│   └── resources/
│       ├── __init__.py
│       └── video.py         # Recurso REST Video (GET, PUT, PATCH, DELETE)
├── instance/                # Base de datos SQLite (se crea sola, no se versiona)
├── venv/                    # Entorno virtual (no se versiona)
├── requirements.txt
├── run.py                   # Punto de entrada de la aplicación
├── .gitignore
├── README.md
└── docs/
    ├── GUIA.md              # Este documento
    └── PRUEBAS.md           # Pruebas realizadas (lo creas tú en el Paso 8)
```

> El directorio `instance/` **no** la creas tú: Flask-SQLAlchemy la genera automáticamente la primera vez que se crea la base de datos.

Los archivos de partida de los pasos siguientes ya existen en el proyecto con comentarios `# TODO` que debes completar. Para ver rápidamente todo lo que te falta por hacer, ejecuta desde la raíz del proyecto:

```bash
grep -rn "TODO" app run.py
```


## Paso 2 — Configuración de la aplicación

El archivo `app/config.py` define una clase de configuración por cada ambiente:

| Clase               | Ambiente      | Base de datos                       |
|---------------------|---------------|-------------------------------------|
| `DevelopmentConfig` | `development` | SQLite en `instance/database.db`    |
| `ProductionConfig`  | `production`  | SQLite en `instance/database.db`    |
| `TestingConfig`     | `testing`     | SQLite en memoria (no deja archivos)|

El diccionario `config` permite elegir la clase por su nombre. Ese nombre se lee en `run.py` de la variable de entorno `FLASK_CONFIG` (por defecto, `development`):

```bash
# Ejemplo: ejecutar con la base de datos en memoria
FLASK_CONFIG=testing python3 run.py
```

Este archivo **ya está completo** en el proyecto de partida; solo léelo y entiende para qué sirve cada parámetro (`DEBUG`, `SQLALCHEMY_DATABASE_URI`, `SQLALCHEMY_TRACK_MODIFICATIONS`).


## Paso 3 — Modelo de datos

Un **modelo** es una clase de Python que representa una tabla de la base de datos: cada atributo es una columna y cada objeto es una fila.

Cada video tiene estos atributos:

| Campo   | Tipo SQLAlchemy | Restricciones   | Descripción                                    |
|---------|-----------------|-----------------|------------------------------------------------|
| `id`    | `Integer`       | llave primaria  | Identificador único (lo envía el cliente en la URL) |
| `name`  | `String(100)`   | no nulo         | Nombre (título) del video                      |
| `views` | `Integer`       | no nulo         | Número de vistas (reproducciones)              |
| `likes` | `Integer`       | no nulo         | Número de "me gusta"                           |

### `app/models/__init__.py`

Ya está completo: crea la instancia `db = SQLAlchemy()` que usarán todos los modelos. Más adelante se asocia con la aplicación mediante `db.init_app(app)`.

### `app/models/video.py`

Debes:
- Definir la columna `name` (texto de máximo 100 caracteres, no nulo).
- Definir la columna `views` (entero, no nulo).
- Definir la columna `likes` (entero, no nulo).

La columna `id` y el método `__repr__` ya están escritos para que los tomes de ejemplo.


## Paso 4 — Application factory y punto de entrada

### `app/__init__.py`

Debes completar la función `create_app(config_name)` para que:
- Cree la instancia de Flask con `Flask(__name__)`.
- Cargue la configuración del ambiente con `app.config.from_object(config[config_name])`.
- Inicialice SQLAlchemy con `db.init_app(app)`.
- Cree la API con `Api(app)` y registre el recurso `Video` en la ruta `/api/videos/<int:video_id>` con `api.add_resource(...)`.
- Cree las tablas con `db.create_all()` (dentro de `with app.app_context():`).
- Retorne la instancia `app`.

> `<int:video_id>` es un *convertidor* de Flask: solo acepta números enteros y le entrega el valor ya convertido a `int` al método del recurso. Una URL como `/api/videos/abc` responde 404 automáticamente.

### `run.py`

Ya crea la aplicación leyendo `FLASK_CONFIG`. Debes:
- Ejecutar `app.run(debug=True)` dentro del bloque `if __name__ == "__main__":`.

> El modo `debug=True` es solo para desarrollo: recarga el servidor automáticamente al guardar cambios y muestra errores detallados. **Nunca se usa en producción.**

Revisa los archivos `app/__init__.py` y `run.py` del proyecto de partida: cada `# TODO` indica exactamente qué línea o bloque de código debes escribir.


## Paso 5 — Recurso REST `Video`

En Flask-RESTful, un **recurso** es una clase que hereda de `Resource` y define un método de Python por cada método HTTP que acepta la ruta. Nuestra API tiene un único recurso, `Video`, en la ruta `/api/videos/<video_id>`:

| Método   | Qué hace                                  | Cuerpo JSON de la solicitud                         | Respuesta exitosa          | Errores posibles   |
|----------|-------------------------------------------|-----------------------------------------------------|----------------------------|--------------------|
| `GET`    | Obtiene un video                          | —                                                   | `200` + video              | `404`              |
| `PUT`    | Crea un video con el ID de la URL         | `name`, `views` y `likes` (todos obligatorios)      | `201` + video creado       | `400`, `409`       |
| `PATCH`  | Actualiza los campos enviados             | `name`, `views`, `likes` (todos opcionales)         | `200` + video actualizado  | `404`              |
| `DELETE` | Elimina un video                          | —                                                   | `204` (sin contenido)      | `404`              |

Significado de los errores:
- `400` — falta un campo obligatorio o tiene un tipo incorrecto (lo detecta `reqparse` automáticamente).
- `404` — no existe un video con ese ID.
- `409` — ya existe un video con ese ID (conflicto al intentar crearlo).

> En esta API **el cliente elige el ID** del video (va en la URL), por eso la creación se hace con `PUT` y no con `POST`.

### `app/resources/video.py`

El archivo de partida ya trae completo:
- `resource_fields`: los campos que se incluyen al convertir un `VideoModel` a JSON (lo usa el decorador `@marshal_with`).
- `video_put_args` y `video_update_args`: *parsers* de `reqparse` que validan el cuerpo de las solicitudes `PUT` (campos obligatorios) y `PATCH` (campos opcionales).

Debes completar:

**`abort_if_video_doesnt_exist(video_id)`**
- Consultar el video con `VideoModel.query.filter_by(id=video_id).first()`.
- Si no existe (`None`), llamar a `abort(404, message=...)`.
- Si existe, retornarlo.

**`Video.get(video_id)`**
- Obtener el video con `abort_if_video_doesnt_exist` y retornarlo.

**`Video.put(video_id)`**
- Leer los datos con `video_put_args.parse_args()`.
- Si ya existe un video con ese ID, responder `abort(409, message=...)`.
- Crear un `VideoModel` con el ID de la URL y los datos recibidos.
- Guardarlo con `db.session.add(...)` y `db.session.commit()`.
- Retornar el video junto con el código `201`.

**`Video.patch(video_id)`**
- Leer los datos con `video_update_args.parse_args()`.
- Obtener el video con `abort_if_video_doesnt_exist`.
- Actualizar **solo** los campos que llegaron en la solicitud. Compara con `is not None` y no con `if args["views"]:`, porque `0` es un valor válido (un video puede tener 0 vistas).
- Guardar los cambios con `db.session.commit()` y retornar el video.

**`Video.delete(video_id)`**
- Obtener el video con `abort_if_video_doesnt_exist`.
- Eliminarlo con `db.session.delete(...)` y `db.session.commit()`.
- Retornar una respuesta vacía con el código `204`: `return "", 204`.

Cada `# TODO` del archivo de partida indica exactamente qué línea o bloque debes escribir.


## Paso 6 — Congelar dependencias

Una vez tengas todo instalado y tu proyecto funcionando, guarda las dependencias exactas del proyecto en `requirements.txt`:

```bash
pip freeze > requirements.txt
```

Esto permite que cualquier otra persona (o tú mismo en otro computador) recree el mismo entorno con:

```bash
pip install -r requirements.txt
```


## Paso 7 — Ejecutar y probar la API

Con el entorno virtual activado y parado en la raíz del proyecto:

```bash
python3 run.py
```

Deberías ver algo como:

```
 * Running on http://127.0.0.1:5000
```

Deja esa terminal abierta (es el servidor) y abre **una segunda terminal de Ubuntu/WSL2** para enviar las solicitudes. (Si usas WSL2, también puedes usar un cliente desde Windows, como Postman o la extensión Thunder Client de VS Code: WSL2 expone el puerto automáticamente.)

Usaremos `curl` con la opción `-i`, que muestra el código de estado de la respuesta.

**1. Crear un video (`PUT`)** — debe responder `201` con el video creado:

```bash
curl -i -X PUT http://127.0.0.1:5000/api/videos/1 \
     -H "Content-Type: application/json" \
     -d '{"name": "Tutorial de Flask", "views": 0, "likes": 0}'
```

El cuerpo de la respuesta debe ser algo como:

```json
{
    "id": 1,
    "name": "Tutorial de Flask",
    "views": 0,
    "likes": 0
}
```

**2. Consultar el video (`GET`)** — debe responder `200` con el mismo JSON:

```bash
curl -i http://127.0.0.1:5000/api/videos/1
```

**3. Actualizar el video (`PATCH`)** — debe responder `200` con `views` y `likes` modificados y `name` intacto:

```bash
curl -i -X PATCH http://127.0.0.1:5000/api/videos/1 \
     -H "Content-Type: application/json" \
     -d '{"views": 2500, "likes": 200}'
```

Prueba también que el valor `0` se actualiza correctamente (si no lo hace, revisa el consejo del `is not None` del Paso 5):

```bash
curl -i -X PATCH http://127.0.0.1:5000/api/videos/1 \
     -H "Content-Type: application/json" \
     -d '{"likes": 0}'
```

**4. Probar los errores:**

```bash
# 409: crear un video con un ID que ya existe
curl -i -X PUT http://127.0.0.1:5000/api/videos/1 \
     -H "Content-Type: application/json" \
     -d '{"name": "Otro video", "views": 1, "likes": 1}'

# 400: crear un video sin enviar el campo "likes"
curl -i -X PUT http://127.0.0.1:5000/api/videos/2 \
     -H "Content-Type: application/json" \
     -d '{"name": "Video incompleto", "views": 1}'

# 404: consultar un video que no existe
curl -i http://127.0.0.1:5000/api/videos/999
```

Los errores `400`, `404` y `409` responden con un JSON que contiene una clave `message` explicando el problema. (Si ves caracteres como `ó` en el texto, no es un error: así escribe JSON las letras con tilde, como la `ó`.)

**5. Eliminar el video (`DELETE`)** — debe responder `204` y no mostrar contenido:

```bash
curl -i -X DELETE http://127.0.0.1:5000/api/videos/1
```

Vuelve a consultarlo con `GET`: ahora debe responder `404`.

**6. Verificar la persistencia:** crea un video, detén el servidor con `Ctrl+C`, vuelve a iniciarlo con `python3 run.py` y consulta el video con `GET`. Debe seguir existiendo, porque los datos están guardados en `instance/database.db`.


## Paso 8 — Documentar las pruebas

Crea el archivo `docs/PRUEBAS.md` y documenta las pruebas que realizaste. Debes incluir **al menos una prueba exitosa por cada método** (`GET`, `PUT`, `PATCH`, `DELETE`) y **al menos un caso de error** por cada código `400`, `404` y `409`.

Para cada prueba registra la solicitud, el código esperado, el código obtenido y la respuesta. Puedes usar esta plantilla:

````markdown
# Pruebas de la API de Videos

## 1. Crear un video (PUT)

- **Solicitud:** `PUT /api/videos/1`
- **Cuerpo:**
  ```json
  {"name": "Tutorial de Flask", "views": 0, "likes": 0}
  ```
- **Código esperado:** 201
- **Código obtenido:** 201
- **Respuesta:**
  ```json
  {"id": 1, "name": "Tutorial de Flask", "views": 0, "likes": 0}
  ```
- **Evidencia:** (captura de pantalla de la terminal o del cliente HTTP)

## 2. Consultar un video que no existe (GET)

- **Solicitud:** `GET /api/videos/999`
- **Código esperado:** 404
- ...
````


## Paso 9 — Implementar una mejora

Elige y desarrolla **una (1)** de las siguientes sugerencias de mejora:

1. **Autenticación y autorización**
   - Implementar JWT para autenticar usuarios.
   - Definir roles y permisos.
2. **Documentación**
   - Integrar Swagger/OpenAPI para documentar la API.
   - Añadir ejemplos de uso.
3. **Validación de datos**
   - Mejorar la validación de entrada con `marshmallow` o similar.
   - Manejar errores de forma más específica.
4. **Funcionalidades adicionales**
   - Añadir búsqueda y filtrado.
   - Implementar paginación.
   - Añadir un endpoint para obtener todos los videos.
   - Implementar un sistema de categorías o etiquetas.
5. **Migración de base de datos**
   - Implementar Flask-Migrate para gestionar cambios en el esquema.
6. **Pruebas**
   - Añadir pruebas unitarias.
   - Implementar pruebas de integración.
   - Configurar CI/CD.
7. **Registro y monitoreo**
   - Implementar logging.
   - Añadir métricas y monitoreo.
8. **Despliegue**
   - Dockerizar la aplicación.
   - Configurar para entornos de producción.

Haz el trabajo en `commits` separados, actualiza `requirements.txt` si instalaste nuevas librerías y describe la mejora en tu README (ver Paso 10).


## Paso 10 — Ajustar el README

El `README.md` del proyecto de partida contiene texto de relleno (*Lorem ipsum*). Reemplázalo por información real de **tu** proyecto:

- **Autor:** tu nombre y tu usuario de GitHub.
- **Descripción del proyecto:** qué hace la API y con qué tecnologías está construida.
- **Instalación:** cómo clonar **tu** repositorio, crear el entorno virtual e instalar las dependencias (`pip install -r requirements.txt`).
- **Ejecución:** cómo iniciar el servidor y en qué dirección queda disponible.
- **Uso de la API:** los endpoints con un ejemplo de solicitud y respuesta para cada uno.
- **Pruebas:** un enlace a `docs/PRUEBAS.md`.
- **Mejora implementada:** cuál elegiste y cómo se usa.

> No olvides corregir también las direcciones de los *badges* y del `git clone`, para que apunten a tu repositorio y no al de la plantilla.

---

## Checklist final

- [ ] El entorno virtual se activa sin errores y las tres dependencias están instaladas.
- [ ] La estructura de directorios coincide con la propuesta y existe el `.gitignore`.
- [ ] `VideoModel` define las columnas `name`, `views` y `likes`.
- [ ] `create_app()` crea la app, carga la configuración, inicializa la base de datos, registra el recurso y crea las tablas.
- [ ] `GET /api/videos/<id>` retorna el video (`200`) o `404` si no existe.
- [ ] `PUT /api/videos/<id>` crea el video (`201`), responde `409` si el ID ya existe y `400` si falta un campo.
- [ ] `PATCH /api/videos/<id>` actualiza solo los campos enviados (incluido el valor `0`) o responde `404`.
- [ ] `DELETE /api/videos/<id>` elimina el video (`204`) o responde `404`.
- [ ] Los datos persisten al reiniciar el servidor.
- [ ] `requirements.txt` está generado.
- [ ] La aplicación corre con `python3 run.py` sin errores.
- [ ] `docs/PRUEBAS.md` documenta las pruebas de cada método y de los errores `400`, `404` y `409`.
- [ ] Una (1) mejora implementada y descrita en el README.
- [ ] El README está ajustado con tus datos y no queda texto *Lorem ipsum*.
- [ ] Hay un `commit` por cada paso o ajuste realizado.

