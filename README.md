CREATE TABLE Genres (
    GenreID SERIAL PRIMARY KEY,
    GenreName VARCHAR(50) NOT NULL
);

CREATE TABLE Movies (
    MovieID SERIAL PRIMARY KEY,
    Title VARCHAR(150) NOT NULL,
    Duration INT,
    GenreID INT,
    FOREIGN KEY (GenreID) REFERENCES Genres(GenreID)
);

CREATE TABLE Shows (
    ShowID SERIAL PRIMARY KEY,
    MovieID INT,
    ShowDateTime TIMESTAMP NOT NULL,
    TicketPrice DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (MovieID) REFERENCES Movies(MovieID)
);

CREATE TABLE Customers (
    CustomerID SERIAL PRIMARY KEY,
    CustomerName VARCHAR(100) NOT NULL,
    Email VARCHAR(100),
    IsVIP BOOLEAN DEFAULT FALSE
);

CREATE TABLE Tickets (
    TicketID SERIAL PRIMARY KEY,
    ShowID INT,
    CustomerID INT,
    SeatNumber INT NOT NULL,
    PurchaseDate DATE NOT NULL,
    FOREIGN KEY (ShowID) REFERENCES Shows(ShowID),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

INSERT INTO Genres (GenreName) VALUES ('Fantasy'), ('ScienceFiction'), ('Drama');

INSERT INTO Movies (Title, Duration, GenreID) VALUES 
('PawPatrol', 169, 1),
('PeppaPig', 166, 1),
('CrazyFrog', 120, 2),
('Fairies', 189, 3);

INSERT INTO Shows (MovieID, ShowDateTime, TicketPrice) VALUES 
(1, '2026-06-01 18:00:00', 150.00),
(1, '2026-06-01 21:30:00', 180.00),
(2, '2026-06-02 19:00:00', 200.00),
(3, '2026-06-02 17:00:00', 130.00);

INSERT INTO Customers (CustomerName, Email, IsVIP) VALUES 
('Zhora', 'zhora@mail.com', FALSE),
('Panas Myrnyy', 'panas@mail.com', TRUE),
('Lesia Ukrainka', 'lesia@mail.com', FALSE);

INSERT INTO Tickets (ShowID, CustomerID, SeatNumber, PurchaseDate) VALUES 
(1, 1, 12, '2026-05-28'),
(1, 2, 13, '2026-05-28'),
(2, 2, 5, '2026-05-29'),
(3, 3, 22, '2026-05-30'),
(3, 1, 23, '2026-05-30');


Для виконання завдання я обрала структуру бази даних кінотеатру, яка складається з п'яти таблиць. Головною є таблиця фактів Tickets, що фіксує кожну купівлю та містить зовнішні ключі ShowID та CustomerID для зв'язку з глядачами та сеансами. Розклад, час та ціни зберігаються в таблиці Shows, яка через MovieID посилається на таблицю Movies. Самі ж фільми класифікуються за допомогою зовнішнього ключа GenreID, що веде до Genres . Для обліку користувачів створено таблицю Customers.


SELECT 
    c.CustomerName,
    m.Title AS MovieTitle,
    g.GenreName,
    COUNT(t.TicketID) AS TicketsBought,
    SUM(s.TicketPrice) AS TotalSpent
FROM Tickets t
JOIN Customers c ON t.CustomerID = c.CustomerID
JOIN Shows s ON t.ShowID = s.ShowID
JOIN Movies m ON s.MovieID = m.MovieID
JOIN Genres g ON m.GenreID = g.GenreID
WHERE t.PurchaseDate >= '2026-05-25'
GROUP BY c.CustomerName, m.Title, g.GenreName
HAVING SUM(s.TicketPrice) > 140.00
ORDER BY TotalSpent DESC;

Головний запит підраховує сумарні витрати кожного глядача на конкретні фільми, починаючи з 25 травня 2026 року, виводячи лише результати понад 140.
Спочатку оператор JOIN здійснює об'єднання 5 таблиць, збираючи докупи імена клієнтів, назви стрічок та їхні жанри. Одразу після цього конструкція WHERE відсікає квитки, придбані раніше вказаної дати, що оптимізує швидкість обробки даних ще до початку їхнього групування.
Далі блок GROUP BY об'єднує записи за ім'ям глядача, назвою фільму та жанром. Паралельно функція COUNT рахує кількість куплених квитків, а SUM обчислює їхню загальну вартість. Оскільки стандартний WHERE не працює з результатами агрегації, для фільтрації груп із сумою понад 140 застосовується конструкція HAVING. Наприкінці оператор ORDER BY сортує отриманий звіт у порядку спадання за сумою витрат.

WITH ExpensiveShows AS (
    SELECT MovieID, TicketPrice FROM Shows WHERE TicketPrice > 140.00
)
SELECT m.Title, es.TicketPrice
FROM ExpensiveShows es
JOIN Movies m ON es.MovieID = m.MovieID;

У кінці скрипту реалізовано CTE за допомогою ключового слова WITH. Запит всередині блоку ExpensiveShows створює тимчасову таблицю, яка одразу відфільтровує лише найдорожчі сеанси з ціною квитка понад 140. Потім основний запит звертається до цього виразу як до звичайної ізольованої таблиці та об'єднує її з Movies. Це дозволяє відділити логіку фінансової фільтрації від логіки виведення назв фільмів, роблячи SQL-код значно чистішим та зручнішим для читання.
