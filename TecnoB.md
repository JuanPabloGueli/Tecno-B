\* = notas

" ... " = nombre de la función

***archivos.js***

# 

# REQUERIMIENTO A



##### ● Base de datos



&#x20; ***init.sql***



&#x09;Se crea el procedimiento SQL "sp\_update\_book\_metadata"

&#x20; 	WHERE id = p\_id AND user\_id = p\_user\_id hace que el UPDATE solo afecte filas del propietario



##### ● Backend



&#x20;***es.json*** / ***en.json***



&#x09;Se agregaron dos mensajes confirmando que la actualización fue exitosa



***validation\_middleware.js***



&#x09;Se crea "validateBookUpdate" el cual valida que lo campos no este vacíos, sean requeridos y/o sean positivos

&#x09;\*req = Objeto de petición HTTP (req.body debe contener username y password).

&#x09;\*res = Objeto de respuesta HTTP.

&#x09;\*const { title, author, category\_id, pages } = req.body; =  Toma esas propiedades de req.body y crea una variable para cada una

&#x09;Si esta todo bien llama a next() para continuar hacia bookController.updateBook

&#x09;Si algún dato no cumple se devuelve una respuesta HTTP 400 con el mensaje del error

&#x20;

***bookController.js***

&#x20;

&#x09;Se creo la función "updateBook" que es el controlador que actualiza los metadatos

&#x09;Cuando llega a "updateBook()" en ***bookController.js*** desde el next de antes ocurre lo siguiente:

&#x09;	Obtiene id del libro , los nuevos datos de la peticion y el id del usuario

&#x09;	Busca el libro, NO esta -> error 404 y un mensaje de error

&#x09;			esta -> actualiza metadatos usando "updateMetadata" de ***bookRepository.js***  y finalmente manda mensaje de exito

&#x09;	Si ocurre un error,  "next(error)"  lo pasa al ***global\_error\_middleware.js***



***global\_error\_middleware.js***



&#x09;Los errores pasan aca cuando se utilizan next(error) en el controlador o en otras partes de la aplicacion

&#x09;Gestiona el error ER\_NO\_REFERENCED\_ROW\_2 al no encontrar un id de categoria.

&#x09;El middleware lo combierte en un respuesta HTTP 400 con el mensaje CT\_INVALID\_CATEGORY

&#x09;Así evita exponer el error técnico de la base de datos al cliente.



***book\_repository.js***



&#x09;Se agrego la función "updateMetadata" la cual recibe el ID del libro, ID del usuario y el objeto metadatos

&#x09;Extrae del objeto metadata los campos

&#x09;Finalmente ejecuta el procedimiento almacenado "CALL sp\_update\_book\_metadata(?, ?, ?, ?, ?, ?)" que devuelve true si el procedimiento termina correctamente.



***book\_routes.js***



&#x09;Se registra una ruta PUT en el router de libros, si el router esta sobre "/api/books" este atiende solicitudes como PUT /api/books/123 siendo 123 un id de libro

&#x09;Primero toma el id del libro -> valida la peticion exigiendo los cuatro campos requeridos -> actualiza el libro





#### ● Frontend



***books\_front\_controller.js***



&#x09;Crea Caché local de categorías y luego le asigna "const categories = await apiService.request('/api/categories', 'GET');"  en donde availableCategories = categories;

&#x09;Añade un botón Editar junto al botón eliminar de cada libro con un listener que abre openEditBook(b);

&#x09;Se creo la funcion "openEditBook(book)" la cual ejecuta PUT contra /books/:id y recarga la tabla después de guardar.





***ui\_handlers.js:***

&#x09;Crea un modal reutilizable llamado editBookModal.

&#x09;Caga los metadatos

&#x09;Permite modificarlos

&#x09;Convierte categoría y páginas a números.

&#x09;Envía los datos mediante el callback onSave.

&#x09;Muestra los errores dentro del modal y no lo cierra si falla la actualización.



## 

## REQUERIMIENTO B

##### ● Base de datos



***init.sql***



&#x09;Cree la tabla "audit\_logs" y el procedimiento "sp\_create\_audit\_log" el cual inserta en dicha tabla.

&#x09;La tabla "audit\_logs" guarda:

&#x09;	ID de auditoria (autoincremental)

&#x09;	ID del usuario

&#x09;	Descripción de la acción realizada

&#x09;	Descripción del recurso afectado

&#x09;	Dirección IP

&#x09;	Fecha y hora

##### ● Backend



***audit\_repository***



&#x09;Desarrolle la capa de auditoria la cual está compuesta por:

&#x09;	Constante "db" que es la conexión con la base de datos

&#x09;	Clase con un solo método asincrónico el cual llama al procedimiento almacenado con db.execute y espera a que termine mediante async/await para finalmente devolver una promesa

&#x09;	\*async tambien permite declarar que create() devuelve una promesa.

&#x09;	\*Los parámetros " ? " son para evitar interpolar valores directamente en SQL



***admin\_controller / book\_controller***



&#x09;En ambos cree una constante const auditRepository = require('../repositories/audit\_repository'); que se encarga de importar la capa de auditoría en los controladores.

&#x09;La llamada está dentro de un try/catch ya que si hay un error de auditoría se captura e imprime por consola.



&#x09;Se registra:



&#x09;Eliminación de un libro con DELETE\_BOOK en ***book\_controller.js***

&#x09;Eliminación de un usuario con DELETE\_USER en ***admin\_controller***



&#x09;Al eliminar un usuario o un libro, la operación primero modifica recursos y después intenta guardar el evento de auditoría. Aunque falle el registro, su error solo se muestra en consola.









## REQUERIMIENTO C

##### ● Base de datos



&#x09;Se creo la tabla "categories" que contiene un id autoincremental y un texto para las categorías

&#x09;Se creo una "Seed data" para hacer los insert de 5 categorías iniciales

&#x09;Se modifico la tabla "book" para que reciba un id en lugar del nombre de la categoria, además se agrego una clave foránea para la id de las categorías

&#x09;El procedimiento "sp\_create\_book" ahora recibe un id en lugar del nombre de la categoría e inserta en la tabla "book" los datos del libro

&#x09;Se modifico "sp\_find\_books\_by\_user" para que ahora liste los libros de un usuario en especifico usando JOIN con la tabla "categories"

&#x09;Se creo "sp\_get\_all\_categories" para listar las categorías con sus id en orden alfabético



##### ● Backend



Se crearon tres .js



***category\_controller.js***

&#x09;Maneja las peticiones HTTP de categorías.

&#x09;getAll() solicita todas las categorías al repositorio.

&#x09;Devuelve el resultado como JSON.

&#x09;Envía los posibles errores al middleware global.



***category\_repository.js***

&#x09;Es el intermediario entre el controlador y MySQL

&#x09;Consulta la base de datos a traves de "CALL sp\_get\_all\_categories()" y devuelve únicamente rows\[0] que contiene las categorías.

&#x09;\* rows\[0] toma el primer conjunto de resultados que devuelve MySQL



***category\_routes.js***

&#x09;Define GET / , el endpoint HTTP.

&#x09;Requiere un token JWT mediante verifyToken.

&#x09;Conecta la ruta con categoryController.getAll.





***server.js***

&#x09;Se modifico para importar el router de categorías y conectarlo usando app.use('/api/categories', categoryRoutes);



#### ● Frontend



***client\_dashboard.html***

&#x09;Se vacio el <select id="category">



***books\_front\_controller.js***



&#x09;\*() => { ... } es una función flecha que sirve como callback del evento.

&#x09;Se creo la función async "loadCategories"

&#x09;Ahora al cargar la página se ejecuta "loadCategories()" y "loadBooks()"

&#x09;"loadCategories()" obtiene las categorías a partir de GET /api/categories y llena el`<select>`{=html}

&#x09;loadBooks() obtiene los documentos desde /api/books/my-books y renderiza la tabla



&#x09;En "deleteBook" y "loadBooks" se agrego '/api' a la ruta

&#x09;Ahora al subir un libro el formulario manda category\_id en lugar de category como texto









### 3\. Criterios de Evaluación y Calidad de Código



1. Prevención de Inyección SQL: Los repositorios utilizan procedimientos almacenados y parámetros: db.execute('CALL sp\_create\_book(?, ?, ?, ?, ?, ?, ?)', \[...]), " ? " es para evitar interpolar valores directamente en SQL
2. Seguridad XSS : El frontend usa createElement, textContent, replaceChildren y append, sin .innerHTML ni .insertAdjacentHTML.





\* .innerHTML es una propiedad de JavaScript que reemplaza el contenido HTML de un elemento: tbody.innerHTML = '<tr><td>Libro peligroso</td></tr>';

&#x20; El problema aparece cuando el texto proviene de usuarios: tbody.innerHTML = `<tr><td>${unUsuario}</td></tr>`;

&#x20; Si unUsuario contiene código HTML o JavaScript malicioso, el navegador podría ejecutarlo. Esto se conoce como XSS (Cross-Site Scripting).



Bibliotech evita el ingreso de codigo malicioso creando elementos reales, ejemplo:

&#x09;const fila = document.createElement('tr');

&#x09;const celda = document.createElement('td');



&#x09;celda.textContent = datosDelUsuario;

&#x09;fila.appendChild(celda);



textContent trata el dato únicamente como texto. Si contiene etiquetas o scripts, no los ejecuta.





3\. Manejo de Errores Centralizado: Los controladores capturan errores y ejecutan next(error)

&#x09;catch (error)

&#x09;{

&#x20;	   next(error);

&#x09;}

&#x09;global\_error\_middleware.js traduce errores de base de datos



4\. Estándares de Documentación: os métodos existentes siguen el formato Doxygen en controladores, repositorios y SQL. Sin embargo, nuevas implementaciones de auditoría o actualización deben incluir, para cada método nuevo:



@brief.

@param.

@returns.

Explicación técnica con @details o @note.



5.Estilo de Código:

&#x09;Archivos: snake\_case.

&#x09;Variables SQL: snake\_case.

&#x09;JavaScript: camelCase.

&#x09;Clases: PascalCase.

&#x09;Llaves de funciones y bloques en nueva línea, según el estilo Alman del proyecto.

