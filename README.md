# BrightCart — Tienda en Línea (Spring Boot)

Aplicación web construida con **Java** y **Spring Boot** que implementa una arquitectura en capas (**Modelo → Repository → Servicio → Controlador**) e inyección de dependencias por constructor para gestionar un catálogo de productos en memoria.

---

## Endpoints esperados

| Método y ruta | Descripción | Resultado / Estado esperado |
| :--- | :--- | :--- |
| `GET http://localhost:8080/` | Tienda con tarjetas de productos | `200 OK` |
| `GET /api/products` | Lista JSON con los productos del catálogo | `200 OK` |
| `GET /api/products/available` | Productos con existencias mayores a cero (`stock > 0`) | `200 OK` |
| `GET /api/products/{id}` | Consulta un producto por su ID | `200 OK` / `404 Not Found` |
| `POST /api/products` | Agrega un producto en memoria hasta reiniciar | `201 Created` / `400 Bad Request` |

---

## Arquitectura y qué hace cada capa

Flujo: `Navegador / Postman` → `ProductController` → `ProductService` → `ProductRepository` → `Map en memoria`.

* **Model (`Product.java`):** Representa la estructura de datos de un producto (`id`, `name`, `category`, `price`, `stock`), junto con su constructor vacío (necesario para que Jackson deserialice el JSON del cuerpo HTTP), constructor con parámetros y *getters*/*setters*. No maneja rutas web ni reglas de negocio.
* **Repository (`ProductRepository.java`):** Anotada con `@Repository`, administra el almacenamiento temporal de datos en memoria usando un `LinkedHashMap<Long, Product>`, precarga los artículos iniciales, asigna el siguiente `id` y provee métodos de acceso a datos (`findAll`, `findById`, `save`). Al estar en memoria, los datos agregados se pierden al reiniciar la app.
* **Service (`ProductService.java`):** Anotada con `@Service` y con `ProductRepository` inyectado por constructor, aplica las reglas de negocio y validaciones del catálogo: filtra productos con `stock > 0`, limpia textos con `StringUtils.trimToEmpty`, valida que `name` y `category` no estén vacíos, verifica que `price > 0` y `stock >= 0`, y lanza `IllegalArgumentException` si algo no cumple.
* **Controller (`ProductController.java`):** Anotada con `@RestController` y `@RequestMapping("/api/products")` y con `ProductService` inyectado por constructor, recibe las peticiones HTTP (`@GetMapping`, `@PostMapping`), lee `@PathVariable` o `@RequestBody`, llama al servicio y devuelve `ResponseEntity` con el código HTTP correspondiente (`200`, `201`, `400` o `404`).

---

## Comprueba la comprensión (Modelo `Product`)

* **¿Por qué `Product` es una clase y no un controlador?** Porque `Product` representa los datos y atributos de un artículo del dominio, mientras que un controlador se encarga de recibir peticiones HTTP, mapear rutas y devolver respuestas web.
* **¿Qué atributo indica si un producto está disponible?** El atributo `stock` (existencias): el producto está disponible cuando `stock > 0` y sin existencias cuando es `0`.
* **¿Para qué ayuda el constructor sin argumentos?** Permite que Jackson (el convertidor JSON de Spring Boot) cree una instancia vacía de `Product` al deserializar un JSON entrante (`@RequestBody`) y luego asigne los valores mediante los *setters*.

---

## Retos y reto adicional

* **Reto principal (`POST /api/products`):** Enviar el JSON con `"name": "Canvas Pencil Case"`, `"category": "Stationery"`, `"price": 149.00` y `"stock": 14`, comprobar que se agregue y aparezca al consultar, y repetir cambiando el nombre del producto.
* **Modificar nombre y precio de un artículo precargado en `ProductRepository`:** Se puede editar directamente su inicialización en el constructor de `ProductRepository` (o actualizar el objeto en el `Map` por su `id`) y reiniciar la aplicación para ver el cambio reflejado.
* **Agregar un artículo con `stock: 0` y comparar `/api/products` con `/api/products/available`:** `GET /api/products` sí incluye el producto con `stock: 0` porque devuelve todos los elementos del repositorio, mientras que `GET /api/products/available` lo excluye porque filtra únicamente los que tienen `stock > 0`.
* **¿Qué capa cambiarías para reemplazar el `Map` en memoria por una base de datos?** Cambiaría la capa **Repository** (y anotar el modelo si se usa JPA), reemplazando el `LinkedHashMap` en memoria por una implementación conectada a base de datos (por ejemplo, con Spring Data JPA o JDBC), sin tener que reescribir las reglas de `ProductService` ni las rutas de `ProductController`.

---

## Reflexión y evaluación

| Pregunta | Respuesta esperada |
| :--- | :--- |
| ¿Qué capa recibe una petición web? | Controlador |
| ¿Qué capa aplica las reglas del catálogo? | Servicio |
| ¿Qué capa almacena los datos de ejemplo? | Repository |
| ¿Qué hace `@Service`? | Registra un servicio administrado por Spring |
| ¿Qué hace la inyección por constructor? | Proporciona un objeto requerido |
| ¿Qué administra Gradle? | Tareas de construcción y bibliotecas externas |
