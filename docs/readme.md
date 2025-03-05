# Лабораторная работа №4

**Тема:** Проектирование REST API  
**Цель работы:** Получить опыт проектирования программного интерфейса.

---

## Документация по API

Ниже представлено REST API для сервиса «Управление книгами и рецензиями». Будет показано 8 основных методов (GET, POST, PUT, DELETE), а также обоснованы принятые проектные решения.

### Принятые проектные решения (не менее 8)

1. **Разделение ресурсов**  
   - Выделены два основных ресурса: `books` и вложенный ресурс `reviews`.  
   - Позволяет чётко разграничить операции над книгами и операции над их рецензиями.

2. **Использование стандартных HTTP-методов**  
   - `GET` для получения ресурса или списка ресурсов.  
   - `POST` для создания нового ресурса.  
   - `PUT` для полной замены существующего ресурса.  
   - `DELETE` для удаления ресурса.  
   - Это классический подход в REST для CRUD-операций.

3. **Использование понятных URL**  
   - Для всех операций над книгами: `/api/books` или `/api/books/{bookId}`.  
   - Для операций над рецензиями: `/api/books/{bookId}/reviews` или `/api/books/{bookId}/reviews/{reviewId}`.  
   - URL-адреса отражают иерархию сущностей (книга → рецензии).

4. **Передача данных в формате JSON**  
   - Все ответы и запросы (где требуется тело) используют JSON.  
   - Упрощает интеграцию с фронтендом и другими сервисами.

5. **Использование кодов ответов HTTP**  
   - `200 OK` — успешное получение или обновление ресурса.  
   - `201 Created` — успешное создание нового ресурса.  
   - `204 No Content` — успешное удаление без возвращаемого тела.  
   - `400 Bad Request` — ошибка валидации или неверные параметры.  
   - `404 Not Found` — ресурс не найден.  
   - `500 Internal Server Error` — непредвиденная ошибка сервера.

6. **Валидация входящих данных**  
   - При `POST` и `PUT` запросах проверяется, что поля модели корректны (например, `Title` не пустое).  
   - При нарушении возвращается `400 Bad Request` с описанием ошибки.

7. **Использование вложенных ресурсов**  
   - Рецензии (`reviews`) относятся к конкретной книге (`book`).  
   - Эндпоинты `/api/books/{bookId}/reviews` отражают логику «рецензии принадлежат книге».

8. **Упрощённая реализация хранения**  
   - В рамках демонстрации используется статический список (или словарь) в контроллере вместо полноценной БД.  
   - В реальном проекте это будет заменено на базу данных и репозиторий, но подход к REST-методам останется тем же.

---

## Описание REST API


# REST API для работы с книгами

## 1. Получить список книг
- **Метод**: GET
- **URL**: `/api/books`
- **Описание**: Возвращает список всех книг.
- **Пример запроса**:  
- **Пример ответа** (HTTP 200 OK):
  ```json
  [
    {
      "id": 1,
      "title": "Clean Code",
      "author": "Robert C. Martin",
      "price": 25.50
    },
    {
      "id": 2,
      "title": "Design Patterns",
      "author": "Erich Gamma",
      "price": 30.00
    }
  ]
  ```

---

## 2. Получить книгу по ID
- **Метод**: GET
- **URL**: `/api/books/{bookId}`
- **Описание**: Возвращает информацию о конкретной книге.
- **Пример ответа** (HTTP 200 OK):
  ```json
  {
    "id": 1,
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "price": 25.50
  }
  ```
- **Возможные коды ответа**:
  - `404 Not Found` – если книга не найдена.

---

## 3. Создать новую книгу
- **Метод**: POST
- **URL**: `/api/books`
- **Описание**: Создаёт новую книгу на основании данных, переданных в теле запроса (JSON).
- **Пример тела запроса**:
  ```json
  {
    "title": "Refactoring",
    "author": "Martin Fowler",
    "price": 28.99
  }
  ```
- **Пример ответа** (HTTP 201 Created):
  ```json
  {
    "id": 3,
    "title": "Refactoring",
    "author": "Martin Fowler",
    "price": 28.99
  }
  ```
- **Возможные коды ответа**:
  - `400 Bad Request` – если поля модели некорректны.

---

## 4. Обновить книгу (полностью)
- **Метод**: PUT
- **URL**: `/api/books/{bookId}`
- **Описание**: Полностью заменяет данные книги.
- **Пример тела запроса**:
  ```json
  {
    "title": "Clean Code (2nd Edition)",
    "author": "Robert C. Martin",
    "price": 27.00
  }
  ```
- **Пример ответа** (HTTP 200 OK):
  ```json
  {
    "id": 1,
    "title": "Clean Code (2nd Edition)",
    "author": "Robert C. Martin",
    "price": 27.00
  }
  ```
- **Возможные коды ответа**:
  - `400 Bad Request` – если поля некорректны.
  - `404 Not Found` – если книга не найдена.

---

## 5. Удалить книгу
- **Метод**: DELETE
- **URL**: `/api/books/{bookId}`
- **Описание**: Удаляет книгу по идентификатору.
- **Пример запроса**:
  ```http
  DELETE /api/books/1
  ```
- **Пример ответа** (HTTP 204 No Content):
  ```
  (Пустое тело)
  ```
- **Возможные коды ответа**:
  - `404 Not Found` – если книга не найдена.

---

## 6. Получить список рецензий для книги
- **Метод**: GET
- **URL**: `/api/books/{bookId}/reviews`
- **Описание**: Возвращает список рецензий, связанных с конкретной книгой.
- **Пример запроса**:
  ```http
  GET /api/books/2/reviews
  ```
- **Пример ответа** (HTTP 200 OK):
  ```json
  [
    {
      "id": 1,
      "bookId": 2,
      "text": "Excellent book on design patterns!",
      "rating": 5
    }
  ]
  ```
- **Возможные коды ответа**:
  - `404 Not Found` – если книга не найдена.

---

## 7. Добавить рецензию к книге
- **Метод**: POST
- **URL**: `/api/books/{bookId}/reviews`
- **Описание**: Создаёт новую рецензию для указанной книги.
- **Пример тела запроса**:
  ```json
  {
    "text": "Very informative and well-structured.",
    "rating": 4
  }
  ```
- **Пример ответа** (HTTP 201 Created):
  ```json
  {
    "id": 3,
    "bookId": 2,
    "text": "Very informative and well-structured.",
    "rating": 4
  }
  ```
- **Возможные коды ответа**:
  - `404 Not Found` – если книга не найдена.
  - `400 Bad Request` – если поля рецензии некорректны.

---

## 8. Удалить рецензию
- **Метод**: DELETE
- **URL**: `/api/books/{bookId}/reviews/{reviewId}`
- **Описание**: Удаляет рецензию у конкретной книги.
- **Пример запроса**:
  ```http
  DELETE /api/books/2/reviews/1
  ```
- **Пример ответа** (HTTP 204 No Content):
  ```
  (Пустое тело)
  ```
- **Возможные коды ответа**:
  - `404 Not Found` – если книга или рецензия не найдены.


## Реализация методов

 public class BooksController : ControllerBase
 {
     // Упрощённые хранилища в памяти
     private static List<Book> Books = new List<Book>
     {
         new Book { Id = 1, Title = "Clean Code", Author = "Robert C. Martin", Price = 25.50M },
         new Book { Id = 2, Title = "Design Patterns", Author = "Erich Gamma", Price = 30.00M }
     };

     private static List<Review> Reviews = new List<Review>
     {
         new Review { Id = 1, BookId = 2, Text = "Excellent book on design patterns!", Rating = 5 }
     };

     // 1. GET /api/books
     [HttpGet]
     public ActionResult<IEnumerable<Book>> GetAllBooks()
     {
         return Ok(Books);
     }

     // 2. GET /api/books/{bookId}
     [HttpGet("{bookId}")]
     public ActionResult<Book> GetBookById(int bookId)
     {
         var book = Books.FirstOrDefault(b => b.Id == bookId);
         if (book == null)
             return NotFound();
         return Ok(book);
     }

     // 3. POST /api/books
     [HttpPost]
     public ActionResult<Book> CreateBook([FromBody] Book newBook)
     {
         if (string.IsNullOrWhiteSpace(newBook.Title))
             return BadRequest("Title is required.");

         newBook.Id = Books.Any() ? Books.Max(b => b.Id) + 1 : 1;
         Books.Add(newBook);

         return CreatedAtAction(nameof(GetBookById), new { bookId = newBook.Id }, newBook);
     }

     // 4. PUT /api/books/{bookId}
     [HttpPut("{bookId}")]
     public ActionResult<Book> UpdateBook(int bookId, [FromBody] Book updatedBook)
     {
         var existing = Books.FirstOrDefault(b => b.Id == bookId);
         if (existing == null)
             return NotFound();

         if (string.IsNullOrWhiteSpace(updatedBook.Title))
             return BadRequest("Title is required.");

         existing.Title = updatedBook.Title;
         existing.Author = updatedBook.Author;
         existing.Price = updatedBook.Price;

         return Ok(existing);
     }

     // 5. DELETE /api/books/{bookId}
     [HttpDelete("{bookId}")]
     public IActionResult DeleteBook(int bookId)
     {
         var book = Books.FirstOrDefault(b => b.Id == bookId);
         if (book == null)
             return NotFound();

         // Удалим связанные рецензии
         Reviews.RemoveAll(r => r.BookId == bookId);
         Books.Remove(book);

         return NoContent();
     }

     // 6. GET /api/books/{bookId}/reviews
     [HttpGet("{bookId}/reviews")]
     public ActionResult<IEnumerable<Review>> GetReviewsForBook(int bookId)
     {
         var book = Books.FirstOrDefault(b => b.Id == bookId);
         if (book == null)
             return NotFound();

         var bookReviews = Reviews.Where(r => r.BookId == bookId).ToList();
         return Ok(bookReviews);
     }

     // 7. POST /api/books/{bookId}/reviews
     [HttpPost("{bookId}/reviews")]
     public ActionResult<Review> CreateReview(int bookId, [FromBody] Review newReview)
     {
         var book = Books.FirstOrDefault(b => b.Id == bookId);
         if (book == null)
             return NotFound();

         if (string.IsNullOrWhiteSpace(newReview.Text))
             return BadRequest("Review text is required.");

         newReview.Id = Reviews.Any() ? Reviews.Max(r => r.Id) + 1 : 1;
         newReview.BookId = bookId;
         Reviews.Add(newReview);

         return CreatedAtAction(nameof(GetReviewsForBook), new { bookId = bookId }, newReview);
     }

     // 8. DELETE /api/books/{bookId}/reviews/{reviewId}
     [HttpDelete("{bookId}/reviews/{reviewId}")]
     public IActionResult DeleteReview(int bookId, int reviewId)
     {
         var book = Books.FirstOrDefault(b => b.Id == bookId);
         if (book == null)
             return NotFound();

         var review = Reviews.FirstOrDefault(r => r.Id == reviewId && r.BookId == bookId);
         if (review == null)
             return NotFound();

         Reviews.Remove(review);
         return NoContent();
     }
 }
