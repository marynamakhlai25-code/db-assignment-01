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

WITH ExpensiveShows AS (
    SELECT MovieID, TicketPrice FROM Shows WHERE TicketPrice > 140.00
)
SELECT m.Title, es.TicketPrice
FROM ExpensiveShows es
JOIN Movies m ON es.MovieID = m.MovieID;
