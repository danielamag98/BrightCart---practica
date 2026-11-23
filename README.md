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

| :--- | :--- |
| **Model (`Product.java`):** | Representa la estructura de datos de un producto, junto con su constructor vacío, constructor con parámetros y *getters*/*setters*. No maneja rutas web ni reglas de negocio. |
| **Repository (`ProductRepository.java`):** | Anotada con `@Repository`, administra el almacenamiento temporal de datos en memoria usando un `LinkedHashMap<Long, Product>`, precarga los artículos iniciales, asigna el siguiente `id` y provee métodos de acceso a datos (`findAll`, `findById`, `save`). Al estar en memoria, los datos agregados se pierden al reiniciar la app. |
| **Service (`ProductService.java`):** | Anotada con `@Service` y con `ProductRepository` inyectado por constructor, aplica las reglas de negocio y validaciones del catálogo: filtra productos con `stock > 0`, limpia textos con `StringUtils.trimToEmpty`, valida que `name` y `category` no estén vacíos, verifica que `price > 0` y `stock >= 0`, y lanza `IllegalArgumentException` si algo no cumple. |
| **Controller (`ProductController.java`):** | Anotada con `@RestController` y `@RequestMapping("/api/products")` y con `ProductService` inyectado por constructor, recibe las peticiones HTTP (`@GetMapping`, `@PostMapping`), lee `@PathVariable` o `@RequestBody`, llama al servicio y devuelve `ResponseEntity` con el código HTTP correspondiente (`200`, `201`, `400` o `404`). |

---

## Comprueba la comprensión (Modelo `Product`)

* **¿Por qué `Product` es una clase y no un controlador?** Porque `Product` representa los datos y atributos de un artículo del dominio, mientras que un controlador se encarga de recibir peticiones HTTP, mapear rutas y devolver respuestas web.
* **¿Qué atributo indica si un producto está disponible?** El atributo `stock` (existencias).
* **¿Para qué ayuda el constructor sin argumentos?** Permite que Jackson (el convertidor JSON de Spring Boot) cree una instancia vacía de `Product` al deserializar un JSON entrante (`@RequestBody`) y luego asigne los valores mediante los *setters*.

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
