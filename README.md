
# Node.js Kick-off Workshop: Proyecto Real

## Introducción
Bienvenidos al proyecto de introducción a Node.js. Este proyecto está diseñado para aplicar los conceptos básicos de Node.js y Express que has aprendido durante la primera semana del curso. Crearás una API RESTful que interactúa con el sistema de archivos en lugar de una base de datos.

## Instrucciones de Entrega
1. Crea un repositorio en GitHub llamado `nodejs-first-project`.
2. Sigue las instrucciones y completa los objetivos establecidos en este documento.
3. Sube tu proyecto a GitHub y comparte el enlace del repositorio.

## Objetivos
- Configurar un entorno de desarrollo Node.js.
- Crear un servidor web utilizando Node.js y Express.
- Implementar rutas y middlewares en Express.
- Leer y escribir datos en el sistema de archivos utilizando el módulo `fs`.
- Manejar errores y asegurar la API con middlewares.

## Descripción del Proyecto
Crearás una API RESTful para gestionar una lista de tareas (To-Do List). Las tareas se almacenarán en un archivo JSON en el sistema de archivos.

## Historias de Usuario

### 1. Como usuario, quiero poder crear una nueva tarea para agregarla a mi lista de tareas.
- **Ruta:** POST `/tasks`
- **Cuerpo de la solicitud:**
  ```json
  {
    "title": "Nombre de la tarea",
    "description": "Descripción de la tarea"
  }
  ```
- **Respuesta:**
  ```json
  {
    "message": "Tarea creada exitosamente",
    "task": {
      "id": 1,
      "title": "Nombre de la tarea",
      "description": "Descripción de la tarea"
    }
  }
  ```

### 2. Como usuario, quiero poder ver todas mis tareas para revisarlas.
- **Ruta:** GET `/tasks`
- **Respuesta:**
  ```json
  [
    {
      "id": 1,
      "title": "Nombre de la tarea",
      "description": "Descripción de la tarea"
    },
    ...
  ]
  ```

### 3. Como usuario, quiero poder ver una tarea específica por su ID para conocer sus detalles.
- **Ruta:** GET `/tasks/:id`
- **Respuesta:**
  ```json
  {
    "id": 1,
    "title": "Nombre de la tarea",
    "description": "Descripción de la tarea"
  }
  ```

### 4. Como usuario, quiero poder actualizar una tarea existente para modificar su información.
- **Ruta:** PUT `/tasks/:id`
- **Cuerpo de la solicitud:**
  ```json
  {
    "title": "Nuevo nombre de la tarea",
    "description": "Nueva descripción de la tarea"
  }
  ```
- **Respuesta:**
  ```json
  {
    "message": "Tarea actualizada exitosamente",
    "task": {
      "id": 1,
      "title": "Nuevo nombre de la tarea",
      "description": "Nueva descripción de la tarea"
    }
  }
  ```

### 5. Como usuario, quiero poder eliminar una tarea para mantener mi lista organizada.
- **Ruta:** DELETE `/tasks/:id`
- **Respuesta:**
  ```json
  {
    "message": "Tarea eliminada exitosamente"
  }
  ```

## Requisitos del Proyecto

### 1. Configuración del Entorno
- Descarga e instala Node.js desde [nodejs.org](https://nodejs.org/). Recomendamos la versión LTS.
- Verifica la instalación con los siguientes comandos:
  ```sh
  node -v
  npm -v
  ```

### 2. Inicialización del Proyecto
- Inicia un nuevo proyecto Node.js:
  ```sh
  mkdir nodejs-first-project
  cd nodejs-first-project
  npm init -y
  ```

### 3. Instalación de Dependencias
- Instala Express:
  ```sh
  npm install express
  ```

### 4. Creación de la API RESTful

#### Estructura del Proyecto
```
nodejs-first-project/
├── data/
│   └── tasks.json
├── src/
│   ├── routes/
│   │   └── tasks.js
│   ├── middlewares/
│   │   └── errorHandler.js
│   └── app.js
├── package.json
└── index.js
```

#### 1. Crear el archivo `tasks.json`
- Crea la carpeta `data` y el archivo `tasks.json`:
  ```sh
  mkdir data
  echo "[]" > data/tasks.json
  ```

#### 2. Crear el Servidor con Express
- Crea la carpeta `src` y el archivo `app.js`:
  ```js
  const express = require("express"); // Importamos Express
  const tasksRoutes = require("./routes/tasks"); // Importamos las rutas de la API
  const errorHandler = require("./middlewares/errorHandler"); // Importamos el middleware para manejo de errores

  const app = express(); // Instanciamos Express
  const PORT = 3000; // Puerto del servidor en donde se ejecutará la API

  app.use(express.json()); // Middleware para parsear el cuerpo de las solicitudes en formato JSON. Tambien conocido como middleware de aplicación.
  app.use("/tasks", tasksRoutes); // Middleware para manejar las rutas de la API. Tambien conocido como middleware de montaje o de enrutamiento.
  app.use(errorHandler); // Middleware para manejar errores.

  app.listen(PORT, () => {
    console.log(`Server running at http://localhost:${PORT}/`);
  });
  ```

#### 3. Crear las Rutas de la API
- Crea la carpeta `routes` y el archivo `tasks.js`:
  ```js
  const express = require("express");
  const fs = require("fs");
  const path = require("path");

  const router = express.Router();
  const tasksFilePath = path.join(__dirname, "../../data/tasks.json");

  // Leer tareas desde el archivo
  const readTasks = () => {
    const tasksData = fs.readFileSync(tasksFilePath); // Leer el archivo. Este poderoso metodo nos permite leer archivos de manera sincrona.
    return JSON.parse(tasksData); // Retornar los datos en formato JSON.
  };

  // Escribir tareas en el archivo
  const writeTasks = (tasks) => {
    fs.writeFileSync(tasksFilePath, JSON.stringify(tasks, null, 2)); // Escribir los datos en el archivo. Este poderoso metodo nos permite escribir archivos de manera sincrona.
  };

  // Crear una nueva tarea
  router.post("/", (req, res) => {
    const tasks = readTasks();
    const newTask = {
      id: tasks.length + 1, // simulamos un id autoincrementable
      title: req.body.title, // obtenemos el titulo de la tarea desde el cuerpo de la solicitud
      description: req.body.description, // obtenemos la descripcion de la tarea desde el cuerpo de la solicitud
    };
    tasks.push(newTask);
    writeTasks(tasks);
    res.status(201).json({ message: "Tarea creada exitosamente", task: newTask });
  });

  // Obtener todas las tareas
  router.get("/", (req, res) => {
    const tasks = readTasks();
    res.json(tasks);
  });

  // Obtener una tarea por ID
  router.get("/:id", (req, res) => {
    const tasks = readTasks();
    const task = tasks.find((t) => t.id === parseInt(req.params.id));
    if (!task) {
      return res.status(404).json({ message: "Tarea no encontrada" });
    }
    res.json(task);
  });

  // Actualizar una tarea por ID
  router.put("/:id", (req, res) => {
    const tasks = readTasks();
    const taskIndex = tasks.findIndex((t) => t.id === parseInt(req.params.id));
    if (taskIndex === -1) {
      return res.status(404).json({ message: "Tarea no encontrada" });
    }
    const updatedTask = {
      ...tasks[taskIndex],
      title: req.body.title,
      description: req.body.description,
    };
    tasks[taskIndex] = updatedTask;
    writeTasks(tasks);
    res.json({ message: "Tarea actualizada exitosamente", task: updatedTask });
  });

  // Eliminar una tarea por ID
  router.delete("/:id", (req, res) => {
    const tasks = readTasks();
    const newTasks = tasks.filter((t) => t.id !== parseInt(req.params.id));
    if (tasks.length === newTasks.length) {
      return res.status(404).json({ message: "Tarea no encontrada" });
    }
    writeTasks(newTasks);
    res.json({ message: "Tarea eliminada exitosamente" });
  });

  module.exports = router;
  ```

#### 4. Crear Middleware para Manejo de Errores
- Crea la carpeta `middlewares` y el archivo `errorHandler.js`:
  ```js
  const errorHandler = (err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ message: "Ocurrió un error en el servidor" });
  };

  module.exports = errorHandler;
  ```

#### 5. Archivo de Entrada del Proyecto
- Crea el archivo `index.js` en la raíz del proyecto:
  ```js
  require("./src/app");
  ```

## 5. Ejecución del Proyecto
- Ejecuta el proyecto con el siguiente comando:
  ```sh
  node index.js
  ```

- Deberás ver un mensaje similar al siguiente:
  ```
  Server running at http://localhost:3000/
  ```

- Abre tu navegador y accede a [http://localhost:3000/tasks](http://localhost:3000/tasks) para probar la API.

## 6. Probando la API (Ejemplo con POST)

Para probar nuestra API, podemos utilizar herramientas como [Postman](https://www.postman.com/) o [Insomnia](https://insomnia.rest/). Descarga e instala una de estas aplicaciones para realizar las siguientes pruebas.

Para este ejemplo, consideraremos que estamos utilizando Postman. Una vez descargado e instalado, sigue los pasos a continuación:

1. Abre Postman y encontrarás una interfaz similar a la siguiente:

<img src="./assets/postman_1.png" alt="Postman" width="500">

2. Le daremos click al boton `new` para crear una colección de peticiones. 

3. Se nos abrirá una venta donde elegiremos colección. En postman, las colecciones son grupos de peticiones que se pueden ejecutar juntas. Es mas que todo para organizar las peticiones que haremos a nuestra API y poder compartirlas con otros desarrolladores.

<img src="./assets/postman_2.png" alt="Postman" width="500">

4. Le damos un nombre a nuestra colección, en este caso `Node.js Kick-off Project`. A este punto puedes indagar más sobre funcionalidades de postman como variables de entorno, autenticación, etc. Te dejaré un link a la documentación oficial de postman [aquí](https://learning.postman.com/docs/getting-started/introduction/). Y un video de youtube que explica como usar postman mas a profundidad [aquí](https://www.youtube.com/watch?v=iFDQ3NFs95M&list=PLDbrnXa6SAzUsLG1gjECgFYLSZDov09fO).

<img src="./assets/postman_3.png" alt="Postman" width="500">

<img src="./assets/postman_4.png" alt="Postman" width="500">

5. Ahora crearemos una petición para crear una tarea. Para ello, le daremos click al boton `...` que se encuentra al lado de la colección que acabamos de crear y seleccionaremos `Add request`.

<img src="./assets/postman_5.png" alt="Postman" width="500">

6. Se nos abrirá una ventana donde podremos configurar nuestra petición. Le daremos un nombre a nuestra petición, en este caso `Create Task`. En el campo `Request URL` colocaremos la dirección de nuestra API, en este caso `http://localhost:3000/tasks`. En el campo `Request type` seleccionaremos `POST`. En el campo `Body` seleccionaremos `raw` y en el tipo de dato seleccionaremos `JSON` y en el campo inferior colocaremos el siguiente JSON:

```json
{
  "title": "Comprar leche",
  "description": "Ir al supermercado y comprar leche"
}
```

<img src="./assets/postman_6.png" alt="Postman" width="500">

<img src="./assets/postman_7.png" alt="Postman" width="500">

7. Ahora le daremos click al boton `Send` y veremos la respuesta de nuestra API.

<img src="./assets/postman_8.png" alt="Postman" width="500">

A este punto, ya debes suponer que el archivo `tasks.json` creado en la carpeta `data` contiene la tarea que acabamos de crear. 

<img src="./assets/tasks.json_1.png" alt="Postman" width="500">

Si no ves una respuesta similar a la que se muestra en la imagen, es posible que haya un error en tu código. Revisa el paso a paso y asegúrate de que todo esté configurado correctamente.

## 7. Probando la API (Resto de Verbos HTTP)

Como trabajo autonomo, prueba el resto de los verbos HTTP que se mencionan en las historias de usuario los cuales son:

- GET `/tasks`
- GET `/tasks/:id`
- PUT `/tasks/:id`
- DELETE `/tasks/:id`

## 8. Preguntas de Reflexión y trabajo investigativo

1. ¿Qué es el filesystem (fs) en Node.js y para qué se utiliza?
En Node.js el módulo fs(filesystem) es un módulo nativo. Es un conjunto de funcionalidades
que permite interactuar con el sistema de archivos del sistema operativo. Este módulo proporciona
métodos para leer y escribir archivos, manipular directorios y realizar operaciones de consulta y más. 

Algunas de las operaciones comunes que se pueden realizar con fs incluyen:
*Lectura y escritura de archivos
*Crear archivos
*Actualizar archivos
*Borrar archivos
*Manipulación de directorios(crear,renombrar,eliminar)
*Cambio de permisos y propietarios de archivos 

2. ¿Qué es un middleware en Express y cuál es su propósito?

Son funciones intermedias que tienen acceso tanto a la solicitud (request) como a la respuesta (response)
en una aplicación Express u otro framework similar. 

Estas funciones pueden realizar tareas como:

*Procesamiento de la solicitud: Modificar la solicitud entrante antes de que llegue a la ruta principal de la aplicación.
*Autenticación y autorización:Verificar credenciales, permisos u otros aspectos de seguridad antes de permitir el acceso a ciertas rutas.
*Manejo de errores: Capturar errores y manejarlos de manera centralizada para evitar que afecten la funcionalidad de la aplicación.
*Logging y seguimiento: Registrar información sobre las solicitudes entrantes y salientes para propósitos de auditoría o depuración.
*Compresión y transformación de datos: Comprimir respuestas o transformar datos antes de enviarlos al cliente.
*Caché de respuestas : Almacenar en caché respuestas para mejorar la eficiencia y la velocidad de la aplicación.

3. ¿Qué es un endpoint en una API RESTful y cuál es su función?

En una API RESTful, un endpoint es una URL específica en el servidor que se utiliza para interactuar con
los recursos de la API. 
Los endpoints definen las rutas a las que pueden enviarse las solicitudes HTTP y generalmente representan una acción o un conjunto de acciones que se pueden realizar en un recurso. Cada endpoint corresponde a una combinación de una URL y un método HTTP (GET, POST, PUT, DELETE, etc.).

4. ¿Qué son los verbos HTTP y cuáles son los más comunes?

Los verbos HTTP, también conocidos como métodos HTTP, son comandos que indican el tipo de acción que se desea realizar en un recurso determinado en un servidor web. Los más comunes son:

*GET:  Solicita la representación de un recurso específico. No debería tener efectos secundarios, es decir, no debería modificar ningún dato en el servidor. 
Uso común: Obtener datos de un servidor (como una página web, una imagen, o información en formato JSON).
Ejemplo: GET /users/123

*POST: Envía datos al servidor para crear un nuevo recurso. Los datos se incluyen en el cuerpo de la solicitud.
Uso común: Crear nuevos registros en una base de datos.
Ejemplo: POST /users con un cuerpo de solicitud conteniendo los datos del nuevo usuario.

*PUT: Envía datos al servidor para actualizar un recurso existente. Si el recurso no existe, PUT puede crear uno nuevo.
Uso común: Actualizar información de un recurso existente.
Ejemplo: PUT /users/123 con un cuerpo de solicitud conteniendo los datos actualizados del usuario.

*DELETE: Elimina un recurso específico del servidor.
Uso común: Eliminar un registro de una base de datos.
Ejemplo: DELETE /users/123

*PATCH: Aplica modificaciones parciales a un recurso existente. A diferencia de PUT, PATCH generalmente solo envía los campos que necesitan ser actualizados.
Uso común: Actualizar parcialmente un recurso. 
Ejemplo: PATCH /users/123 con un cuerpo de solicitud conteniendo solo los campos que necesitan ser modificados.

5. ¿Qué es JSON y por qué es utilizado en las API RESTful?

JSON (JavaScript Object Notation) es un formato de texto ligero para el intercambio de datos. Es fácil de leer y escribir tanto para los humanos como para las máquinas. JSON se utiliza ampliamente en las API RESTful debido a varias razones: 

*SIMPLICIDAD : JSON es fácil de entender y usar, lo que lo hace accesible para desarrolladores de todos los niveles.
*ESTRUCTURA BASADA EN TEXTO: Como es un formato de texto, JSON se puede enviar y recibir fácilmente a través de redes utilizando protocolos como HTTP.
*COMPATIBILIDAD CON JAVASCRIPT:  JSON se deriva de la sintaxis de objetos de JavaScript, por lo que es nativamente compatible con el lenguaje y se puede trabajar con él fácilmente en navegadores web y entornos de servidor basados en JavaScript.
*LIGERO: JSON es compacto y eficiente, lo que lo hace adecuado para la transferencia de datos a través de redes con limitaciones de ancho de banda.
*FORMATO ESTÁNDAR: JSON es un estándar abierto y ampliamente soportado, lo que facilita la interoperabilidad entre diferentes sistemas y tecnologías.

En resumen, JSON es el formato de datos preferido en las APIs RESTful debido a su simplicidad, facilidad de uso, compatibilidad con múltiples lenguajes de programación y su eficiencia para el intercambio de datos entre clientes y servidores.

6. En lo que respecta al envio de datos a lo largo de los verbos http responde:

    - ¿Qué es el body de una petición?

    El body (cuerpo) de una petición HTTP es la parte de la solicitud donde se incluyen los datos que se quieren enviar al servidor. El body es utilizado principalmente en las peticiones que implican el envío de datos, como POST, PUT y PATCH, aunque también puede estar presente en otras solicitudes dependiendo de la necesidad.

    - ¿Qué es el body de una respuesta?

    El body de una respuesta HTTP es la parte de la respuesta que contiene los datos enviados desde el servidor al cliente. Este cuerpo puede incluir diversos tipos de contenido, como texto, JSON, HTML, XML, imágenes, entre otros, dependiendo de lo que el servidor necesite devolver al cliente.

    - ¿Qué es el query de una petición?

    El query (o query string) de una petición HTTP es la parte de la URL que contiene los parámetros de consulta. Se usa para enviar datos al servidor en el contexto de una solicitud GET y se encuentra después del símbolo ? en la URL.

    - ¿Qué es el params de una petición?

    Los params (o route parameters) de una petición HTTP son variables definidas en la ruta (URL) que se utilizan para identificar recursos específicos. Estos parámetros se definen usando dos puntos : en la definición de la ruta en el servidor y se extraen del URL en tiempo de ejecución.

7. En lo que respecta al verbo POST responde:
    - ¿Qué es un verbo POST y cuál es su propósito?
    - ¿Cuándo se utiliza un verbo POST?
    - ¿En qué se diferencia un verbo POST de los otros verbos HTTP como GET, PUT y DELETE?
    - ¿Como se envian datos en un verbo POST?
8. En lo que respecta al verbo GET responde:
    - ¿Qué es un verbo GET y cuál es su propósito?
    - ¿Cuándo se utiliza un verbo GET?
    - ¿En qué se diferencia un verbo GET de los otros verbos HTTP como POST, PUT y DELETE?
9. En lo que respecta al verbo PUT responde:
    - ¿Qué es un verbo PUT y cuál es su propósito?
    - ¿Cuándo se utiliza un verbo PUT?
    - ¿En qué se diferencia un verbo PUT de los otros verbos HTTP como POST, GET y DELETE?
10. En lo que respecta al verbo DELETE responde:
    - ¿Qué es un verbo DELETE y cuál es su propósito?
    - ¿Cuándo se utiliza un verbo DELETE?
    - ¿En qué se diferencia un verbo DELETE de los otros verbos HTTP como POST, GET y PUT?
11. ¿Qué es un status code y cuáles son los más comunes?
12. ¿Cuales son los status code mas comunes para el verbo POST?
13. ¿Cuales son los status code mas comunes para el verbo GET?
14. ¿Cuales son los status code mas comunes para el verbo PUT?
15. ¿Cuales son los status code mas comunes para el verbo DELETE?