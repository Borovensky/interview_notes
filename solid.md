# SOLID
Принципы SOLID — это набор из пяти основных принципов объектно-ориентированного дизайна, предложенных Робертом Мартином. Эти принципы направлены на улучшение масштабируемости, управляемости и поддерживаемости кода, а также на упрощение изменений в программном обеспечении. Вот краткое описание каждого принципа:

1. **Принцип единственной ответственности (Single Responsibility Principle, SRP):** Каждый класс должен иметь только одну причину для изменения. Это означает, что класс должен выполнять только одну задачу или иметь одну область ответственности. Разделение функциональности на несколько классов помогает уменьшить сложность системы и упрощает её поддержку.
   
    #### Пример без SRP 
    Допустим, у нас есть класс UserManager, который занимается управлением пользователями: сохраняет пользователя в базу данных и также отвечает за отображение информации о пользователе на веб-странице. Это нарушает SRP, поскольку у класса две причины для изменения: изменения в логике сохранения данных и изменения в логике отображения данных.
    ```javascript
    class UserManager {
        constructor(user) {
            this.user = user;
        }

        saveUser() {
            // Логика сохранения пользователя в базу данных
            console.log('User saved to database');
        }

        displayUser() {
            // Логика отображения информации о пользователе на веб-странице
            console.log(`Displaying user: ${this.user.name}`);
        }
    }
    ```
    #### Пример с SRP
    Чтобы соответствовать принципу единственной ответственности, мы можем разделить UserManager на два класса: один отвечает за управление данными пользователя (UserDataManager), а другой — за отображение информации о пользователе (UserDisplay).
    ```javascript
    class UserDataManager {
        constructor(user) {
            this.user = user;
        }

        saveUser() {
            // Логика сохранения пользователя в базу данных
            console.log('User saved to database');
        }
    }

    class UserDisplay {
        constructor(user) {
            this.user = user;
        }

        displayUser() {
            // Логика отображения информации о пользователе на веб-странице
            console.log(`Displaying user: ${this.user.name}`);
        }
    }
    ```
    Теперь у нас есть два класса, каждый из которых отвечает за свою область ответственности: UserDataManager управляет данными пользователя, а UserDisplay — отображением информации о пользователе. Это упрощает поддержку и модификацию кода, поскольку изменения в одном аспекте приложения (например, в логике отображения) не повлияют на другие аспекты (например, на логику управления данными).
  
2. **Принцип открытости/закрытости (Open/Closed Principle, OCP):** Программные сущности (классы, модули, функции и т.д.) должны быть открыты для расширения, но закрыты для модификации. Это означает, что можно добавлять новую функциональность без изменения существующего кода, что уменьшает риск внесения ошибок в уже тестированный и проверенный код.

    #### Пример без применения OCP
    Рассмотрим простой пример, в котором у нас есть класс ShapePrinter, который отвечает за печать различных типов фигур. Если мы захотим добавить новый тип фигуры, нам придется изменить метод printShape класса ShapePrinter, что нарушает принцип OCP.
    ```javascript
    class Shape {
        constructor(type) {
            this.type = type;
        }
    }

    class ShapePrinter {
        printShape(shape) {
            if (shape.type === 'circle') {
                console.log('Printing circle');
            } else if (shape.type === 'square') {
                console.log('Printing square');
            }
            // При добавлении нового типа фигуры придется изменять этот метод
        }
    }

    const circle = new Shape('circle');
    const square = new Shape('square');

    const printer = new ShapePrinter();
    printer.printShape(circle);
    printer.printShape(square);
    ```

    #### Пример с применением OCP
    Чтобы соответствовать принципу OCP, мы можем определить общий интерфейс или базовый класс для фигур, который включает метод для печати. Затем каждый конкретный тип фигуры реализует этот интерфейс (или наследует от базового класса), обеспечивая свою реализацию метода печати. Таким образом, ShapePrinter может оставаться неизменным, а новые типы фигур могут быть легко добавлены без модификации существующего кода.
    ```javascript
    // Определяем базовый класс для всех фигур
    class Shape {
        print() {
            throw new Error('This method should be implemented by subclasses');
        }
    }
    // Конкретные реализации для разных типов фигур
    class Circle extends Shape {
        print() {
            console.log('Printing circle');
        }
    }
    class Square extends Shape {
        print() {
            console.log('Printing square');
        }
    }
    // Добавление нового типа фигуры не требует изменений в классе ShapePrinter
    class Triangle extends Shape {
        print() {
            console.log('Printing triangle');
        }
    }
    class ShapePrinter {
        printShape(shape) {
            shape.print();
        }
    }
    const circle = new Circle();
    const square = new Square();
    const triangle = new Triangle();

    const printer = new ShapePrinter();
    printer.printShape(circle);
    printer.printShape(square);
    printer.printShape(triangle);
    ```
    В этом примере ShapePrinter не требует изменений при добавлении новых типов фигур, что соответствует принципу открытости/закрытости. Добавление новой фигуры требует лишь создания нового класса, наследующего Shape и реализующего метод print.

3. **Принцип подстановки Барбары Лисков (Liskov Substitution Principle, LSP):** Объекты в программе можно заменить их наследниками без изменения правильности выполнения программы. Проще говоря, объекты класса-потомка должны могуть заменять объекты класса-предка, не нарушая работу программы.

    #### Пример без применения LSP
    Рассмотрим класс Bird с методом fly, и производный класс Penguin, который является птицей, но не умеет летать. Если мы попытаемся использовать метод fly для объекта класса Penguin, это приведёт к логической ошибке в программе, так как пингвины не летают.
    ```javascript
    class Bird {
        fly() {
            console.log("This bird is flying");
        }
    }

    class Penguin extends Bird {
        fly() {
            throw new Error("Cannot fly");
        }
    }

    function makeBirdFly(bird) {
        bird.fly();
    }

    const bird = new Bird();
    const penguin = new Penguin();

    makeBirdFly(bird); // Работает нормально
    makeBirdFly(penguin); // Выбрасывает исключение, что нарушает LSP
    ``` 
    
    #### Пример с применением LSP
    Чтобы соответствовать LSP, мы можем переосмыслить нашу иерархию классов так, чтобы не нарушать ожидаемое поведение. Вместо того чтобы наследовать Penguin от Bird, который умеет летать, мы можем создать более общий базовый класс Bird без метода fly, и использовать интерфейсы или дополнительные базовые классы для птиц, которые умеют летать.
    ```javascript
    class Bird {
    // Базовый класс для всех птиц
    }

    // Интерфейс летающей птицы
    class FlyingBird extends Bird {
        fly() {
            console.log("This bird is flying");
        }
    }

    // Интерфейс не летающей птицы
    class NonFlyingBird extends Bird {
        // Пингвин не нуждается в методе fly
    }

    class Sparrow extends FlyingBird {
        // Воробей умеет летать
    }

    class Penguin extends NonFlyingBird {
        // Пингвин не умеет летать
    }

    function makeBirdFly(bird) {
        if (bird instanceof FlyingBird) {
            bird.fly();
        }
    }

    const sparrow = new Sparrow();
    const penguin = new Penguin();

    makeBirdFly(sparrow); // Работает нормально
    makeBirdFly(penguin); // Ничего не делает, но и не приводит к ошибке
    ```
    В этом примере Penguin больше не наследуется от класса, предполагающего возможность летать, что соответствует принципу LSP. Мы избегаем нарушения контракта базового класса, создавая отдельные базовые классы для летающих и не летающих птиц, что позволяет нам обрабатывать их соответствующим образом без риска возникновения ошибок.

4. **Принцип разделения интерфейса (Interface Segregation Principle, ISP):** Клиенты не должны зависеть от интерфейсов, которые они не используют. Этот принцип направлен на то, чтобы избежать создания "толстых" или "жирных" интерфейсов, наследование которых заставляет реализующие их классы использовать методы, которые им не нужны.

    #### Пример без применения ISP
    Допустим, у нас есть интерфейс Worker с методами для различных видов работы. Если класс должен реализовать этот интерфейс, но использует не все методы, это приведёт к нарушению ISP.
    ```javascript
    class Worker {
        work() {
            // общая работа
        }

        eat() {
            // перерыв на обед
        }
    }

    class HumanWorker {
        work() {
            console.log("Человек работает");
        }

        eat() {
            console.log("Человек ест");
        }
    }

    class RobotWorker {
        work() {
            console.log("Робот работает");
        }

        eat() {
            // Роботам не нужно есть, но мы вынуждены реализовать этот метод
        }
    }
    ```

    #### Пример с применением ISP
    Чтобы соответствовать принципу разделения интерфейса, мы можем разделить Worker на более мелкие интерфейсы, так что реализующие классы будут содержать только те методы, которые им действительно необходимы. В JavaScript нет явного синтаксиса для интерфейсов, как в некоторых других языках программирования, но мы можем демонстрировать этот принцип через разделение функциональности на разные классы.
    ```javascript
    class Workable {
        work() {
            // Работа
        }
    }

    class Eatable {
        eat() {
            // Еда
        }
    }

    class HumanWorker extends Workable {
        work() {
            super.work();
            console.log("Человек работает");
        }
    }

    class HumanEater extends Eatable {
        eat() {
            super.eat();
            console.log("Человек ест");
        }
    }

    class RobotWorker extends Workable {
        work() {
            super.work();
            console.log("Робот работает");
        }
    }

    // Человек может и работать, и есть
    const humanWorker = new HumanWorker();
    humanWorker.work();

    const humanEater = new HumanEater();
    humanEater.eat();

    // Робот может только работать
    const robotWorker = new RobotWorker();
    robotWorker.work();
    ```
    В этом примере мы разделили обязанности на Workable и Eatable. Теперь HumanWorker и HumanEater реализуют методы, соответствующие их функциям, а RobotWorker реализует только те методы, которые ему нужны для работы, не нарушая при этом принцип разделения интерфейса. Это делает наш код более гибким и обеспечивает, что классы не будут зависеть от методов, которые они фактически не используют.

5. **Принцип инверсии зависимостей (Dependency Inversion Principle, DIP):** Модули высокого уровня не должны зависеть от модулей низкого уровня. Обе типы модулей должны зависеть от абстракций. Кроме того, абстракции не должны зависеть от деталей, а детали должны зависеть от абстракций. Это помогает достичь слабой связанности между модулями системы.
    
    #### Пример без применения DIP
    Рассмотрим систему уведомлений, где класс NotificationService напрямую зависит от конкретного класса отправки уведомлений через email, EmailNotification.
    ```javascript
    class EmailNotification {
        sendEmail(message) {
            console.log(`Sending email with message: ${message}`);
        }
    }

    class NotificationService {
        constructor() {
            this.emailService = new EmailNotification();
        }

        sendNotification(message) {
            this.emailService.sendEmail(message);
        }
    }
    ```
    В этом примере NotificationService напрямую зависит от EmailNotification, что делает его трудно адаптируемым к изменениям, например, если мы захотим добавить отправку уведомлений через SMS.

    #### Пример с применением DIP
    Чтобы применить принцип инверсии зависимостей, мы можем определить абстракцию для службы отправки уведомлений и зависеть от этой абстракции в NotificationService, а не от конкретного класса.
    ```javascript
    // Абстракция службы уведомлений
    class NotificationSender {
        send(message) {
            throw new Error("Method 'send' must be implemented");
        }
    }

    // Конкретная реализация для отправки уведомлений по email
    class EmailNotificationSender extends NotificationSender {
        send(message) {
            console.log(`Sending email with message: ${message}`);
        }
    }

    // Конкретная реализация для отправки уведомлений через SMS
    class SMSNotificationSender extends NotificationSender {
        send(message) {
            console.log(`Sending SMS with message: ${message}`);
        }
    }

    class NotificationService {
        constructor(notificationSender) {
            this.notificationSender = notificationSender;
        }

        sendNotification(message) {
            this.notificationSender.send(message);
        }
    }
    ```
    Теперь NotificationService зависит от абстракции NotificationSender, а не от конкретных деталей отправки уведомлений. Это позволяет легко добавлять новые способы отправки уведомлений, реализуя интерфейс NotificationSender, и делает систему более гибкой и расширяемой.
    ```javascript
    const emailSender = new EmailNotificationSender();
    const notificationService = new NotificationService(emailSender);
    notificationService.sendNotification("Привет, это тестовое уведомление по email!");

    const smsSender = new SMSNotificationSender();
    const smsNotificationService = new NotificationService(smsSender);
    smsNotificationService.sendNotification("Привет, это тестовое уведомление по SMS!");
    ```
    В этом примере, добавление новых способов отправки уведомлений не требует изменений в NotificationService, что соответствует принципу инверсии зависимостей.
