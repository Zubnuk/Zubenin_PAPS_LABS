## Лабораторная работа №3
Тема: Использование принципов проектирования на уровне методов и классов
Цель работы: Получить опыт проектирования и реализации модулей с использованием принципов KISS, YAGNI, DRY, SOLID и др.

## Диаграмма контейнеров

![fLLDRoD54BtxLpGvUQKaS-74QNUpaG3jXY4sS5orxTwanNXcrDDE20Wf2Uo2v8YGlR3Yi80Gukfy14y-ClaBFV-8LpNsU8nZzf1BvAwfUltgrQlEmyHAOtlTeFQU3jj1hwF4fhLnye7d6RczlA7jPg_LIvHfHw5LeLrkHrrlNRUbwKHhMiKgy5RUN_836ThFdZtrkZAnoY3qV0NvKe (1)](https://github.com/user-attachments/assets/107d7786-ce86-475c-8696-87d486b4fde1)


## Диаграмма компонентов
![jLJHJXD157tVhnZh2qs4VV799s11OwBO58cFP6X7sBZTsSnE9OqneSKO4qm8V836Fq1iQONI_iATF_BSRNVJsiOtcypUsNDpppbxxqpNKokDRJOYxq4Orw9cNOaZQvFqYUyRURns6DgZva4pLQQcOwjYkrRJ3RyQrgIfwvrG9IhoWyUhbKZ6Tk_b2xyXKwiCeFr0Dn8TSqN2x9uDRQH4WO](https://github.com/user-attachments/assets/45bb4d05-d351-4036-8f8f-c741e1a55e45)
## Диаграмма последовательностей

![ZPFBZjCm58RtVegZRZ8LdJ8C0ujAWQ4L6YeMZQP5hDtwm5YuSV1bHXLYCzto14Yy0n82LHMNLt3U22TEf0I2Q1P9PF_dztSkFjU6AcFNWb1hP6hNIrI4J0J7rC3n8tY9ooTPDWBtoNrstzpEVVVVh_ajkBtxuNwwNswVRw7dFkPR_x_BR_CFOMldlkJR7WdnXLDYrUgAQfp8r2Wf1AeH9B](https://github.com/user-attachments/assets/98348819-15e3-442a-a964-35df2e57e7bb)
Краткие пояснения

Client инициирует покупку в WebApp.
WebApp отправляет запрос на PurchaseController (API).
PurchaseController вызывает PurchaseService, чтобы сформировать заказ.
PurchaseService проверяет доступность книги через BookRepository.
Если книга доступна, создаётся заказ в OrderRepository, далее идёт запрос на оплату в PaymentGateway.
После успешной оплаты заказ обновляется до статуса «Paid».
Возвращается ответ с подтверждением покупки. Если книги нет в наличии, возвращается ошибка.

## Модель БД
![ZLBBJiCm4BpxArOvGL6hzjfJmqCe5nvg-O1jl83LsAxipQ4W_Xt7k1W75P5BFEFPcOoTbMTqtEjE-9qoUtIeeUtG-vhPgMtFrBJMhDKRmLo8k0DFdXoy2mZF59HXQ6G2FioO5xX3JILper5rNzGqlYxWbhYHP-SP3LC1VnnZBgtT_HcpKToDSkgUNwtxCRDcs-uUUpIi-92zxvqyrI60sF](https://github.com/user-attachments/assets/df944a3e-d62d-4169-b33a-c2ce6e2615a8)

Краткие пояснения

User: покупатель или автор, но в контексте покупки нас интересует именно покупатель.
Book: книга с полем StockQuantity (сколько экземпляров доступно).
Order: заказ, связанный с конкретным пользователем (UserId). Имеет статус (например, Pending, Paid, Cancelled).
OrderItem: отдельная позиция в заказе. Связана с книгой и количеством (если покупают несколько экземпляров).
Payment: информация об оплате конкретного заказа (сумма, дата, статус платежа).
## Применение основных принципов разработки

Ниже приведён пример упрощённого кода (C#), демонстрирующего использование KISS, YAGNI, DRY, SOLID.
// KISS, DRY, SOLID: Пример сервиса для покупок
public class PurchaseService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IBookRepository _bookRepository;
    private readonly IPaymentGateway _paymentGateway;

    // SOLID (DIP): зависимости внедряются через конструктор
    public PurchaseService(
        IOrderRepository orderRepository, 
        IBookRepository bookRepository, 
        IPaymentGateway paymentGateway)
    {
        _orderRepository = orderRepository;
        _bookRepository = bookRepository;
        _paymentGateway = paymentGateway;
    }

    // KISS: метод простой, без лишней логики
    // DRY: нет дублирования проверок в других местах
    public Order CreateOrder(int userId, int bookId)
    {
        var book = _bookRepository.GetById(bookId);
        if (book == null || book.StockQuantity <= 0)
        {
            throw new InvalidOperationException("Книга недоступна");
        }

        var order = new Order
        {
            UserId = userId,
            CreatedAt = DateTime.UtcNow,
            Status = "Pending"
        };

        // Создаём заказ в БД
        var createdOrder = _orderRepository.Create(order);

        // Уменьшаем остаток книги
        book.StockQuantity -= 1;
        _bookRepository.Update(book);

        return createdOrder;
    }

    // YAGNI: не добавляем функционал (например, скидки, налоги), пока он не нужен
    // SOLID (SRP): метод отвечает только за оплату
    public Order PayOrder(int orderId, decimal amount)
    {
        var order = _orderRepository.GetById(orderId);
        if (order == null || order.Status != "Pending")
        {
            throw new InvalidOperationException("Неверный заказ");
        }

        var paymentResult = _paymentGateway.RequestPayment(orderId, amount);
        if (paymentResult.IsSuccess)
        {
            order.Status = "Paid";
        }
        else
        {
            order.Status = "Failed";
        }

        _orderRepository.Update(order);
        return order;
    }
}

KISS (Keep It Simple, Stupid): методы максимально упрощены, без избыточной логики.
YAGNI (You Aren't Gonna Need It): не реализуем дополнительный функционал (скидки, бонусы), пока нет реальной потребности.
DRY (Don't Repeat Yourself): проверка доступности книги и логика оформления заказа сосредоточены в одном месте, не дублируются в других сервисах.
SOLID:
S (Single Responsibility): PurchaseService решает задачу создания заказа и оплаты, не смешивает в себе другие аспекты (например, логику аутентификации).
O (Open/Closed): при необходимости можно добавить новую логику (например, возврат средств) без изменения существующих методов.
L (Liskov Substitution): IOrderRepository, IBookRepository, IPaymentGateway могут иметь разные реализации, сервис остаётся работоспособным.
I (Interface Segregation): интерфейсы разбиты по ролям (Order, Book, Payment), не перегружены лишними методами.
D (Dependency Inversion): сервис зависит от абстракций (интерфейсов), а не от конкретных реализаций.



 ## Дополнительные принципы разработки
BDUF (Big Design Up Front)

Идея: Сперва полностью спроектировать систему, затем приступать к разработке.
Применение: Частичное. На старте мы создали высокоуровневую архитектуру (C4-модель), но избегаем слишком детального «монолитного» проектирования, поскольку гибкий и итеративный подход (Agile) лучше подходит для большинства современных проектов.
SoC (Separation of Concerns)

Идея: Разделять различные аспекты (слои) системы, чтобы они не пересекались в одном модуле.
Применение: Явно используем. У нас разделены WebApp (UI), API (бизнес-логика) и Database (данные). Внутри API — контроллеры, сервисы, репозитории, платёжный шлюз. Это улучшает масштабируемость и читаемость кода.
MVP (Minimum Viable Product)

Идея: Создать минимально работоспособный продукт, чтобы проверить гипотезы и получить обратную связь.
Применение: Принцип согласуется с YAGNI. Мы реализовали только основные функции покупки книги (проверка стока, оплата). Дополнительные функции (скидки, купоны, подарочные сертификаты) пока не реализуем, чтобы быстрее проверить базовый сценарий.
PoC (Proof of Concept)

Идея: Создать прототип, чтобы подтвердить (или опровергнуть) техническую реализуемость идеи.
Применение: Может быть актуально, если мы сомневаемся в совместимости с внешней платёжной системой или хотим проверить производительность. В нашем случае мы можем сделать PoC-интеграцию с «Payment System» до полного внедрения, чтобы убедиться в надёжности и корректности.

 
