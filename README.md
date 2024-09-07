-- Cria o banco de dados
CREATE DATABASE library_system;
GO

-- Usa o banco de dados criado
USE library_system;
GO

-- Cria a tabela de livros (Books)
CREATE TABLE Books (
    book_id INT PRIMARY KEY IDENTITY(1,1),  -- Gera automaticamente IDs únicos
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NOT NULL,
    genre VARCHAR(100),
    published_year INT
);
GO

-- Cria a tabela de usuários (Users)
CREATE TABLE Users (
    user_id INT PRIMARY KEY IDENTITY(1,1),  -- Gera automaticamente IDs únicos
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL
);
GO

-- Cria a tabela de empréstimos (Loans)
CREATE TABLE Loans (
    loan_id INT PRIMARY KEY IDENTITY(1,1),  -- Gera automaticamente IDs únicos
    book_id INT NOT NULL,
    user_id INT NOT NULL,
    loan_date DATE NOT NULL,
    return_date DATE,
    FOREIGN KEY (book_id) REFERENCES Books(book_id),
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
);
GO

-- Adiciona um novo livro ao catálogo
INSERT INTO Books (title, author, genre, published_year)
VALUES ('O Senhor dos Anéis', 'J.R.R. Tolkien', 'Fantasia', 1954);
GO

-- Atualiza o autor de um livro
UPDATE Books
SET author = 'J. R. R. Tolkien'
WHERE title = 'O Senhor dos Anéis';
GO

-- Exclui um livro do catálogo pelo ID
DELETE FROM Books
WHERE book_id = 1;
GO

-- Busca todos os livros do gênero 'Fantasia'
SELECT * FROM Books WHERE genre = 'Fantasia';
GO

-- Adiciona um novo usuário
INSERT INTO Users (name, email)
VALUES ('Maria Silva', 'maria.silva@email.com');
GO

-- Registra um empréstimo de um livro
INSERT INTO Loans (book_id, user_id, loan_date)
VALUES (1, 1, GETDATE());  -- GETDATE() insere a data atual
GO

-- Registra a devolução de um livro
UPDATE Loans
SET return_date = GETDATE()
WHERE loan_id = 1;
GO
-- Consulta todos os livros emprestados e os respectivos usuários
SELECT Books.title, Users.name, Loans.loan_date
FROM Loans
JOIN Books ON Loans.book_id = Books.book_id
JOIN Users ON Loans.user_id = Users.user_id
WHERE Loans.return_date IS NULL;
GO

-- Subconsulta para encontrar usuários que fizeram mais de 3 empréstimos
SELECT Users.name
FROM Users
WHERE (SELECT COUNT(*) FROM Loans WHERE Loans.user_id = Users.user_id) > 3;
GO
-- Cria uma função para calcular o número total de empréstimos de um usuário específico
CREATE FUNCTION TotalLoans (@user_id INT)
RETURNS INT
AS
BEGIN
    RETURN (SELECT COUNT(*) FROM Loans WHERE user_id = @user_id);
END;
GO

-- Usa a função para ver o total de empréstimos de um usuário
SELECT dbo.TotalLoans(1) AS TotalEmpréstimos;
GO
-- Relatório dos livros que estão atualmente emprestados (ou seja, não devolvidos)
SELECT Books.title, Users.name, Loans.loan_date
FROM Loans
JOIN Books ON Loans.book_id = Books.book_id
JOIN Users ON Loans.user_id = Users.user_id
WHERE Loans.return_date IS NULL;
GO
-- Relatório dos usuários com mais empréstimos
SELECT Users.name, COUNT(Loans.loan_id) AS TotalEmpréstimos
FROM Users
JOIN Loans ON Users.user_id = Loans.user_id
GROUP BY Users.name
ORDER BY TotalEmpréstimos DESC;
GO
-- Trigger que impede o empréstimo de um livro já emprestado
CREATE TRIGGER CheckBookAvailability
ON Loans
FOR INSERT
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Loans WHERE book_id = (SELECT book_id FROM inserted) AND return_date IS NULL)
    BEGIN
        RAISERROR('Este livro já está emprestado e não pode ser emprestado novamente até ser devolvido.', 16, 1);
        ROLLBACK TRANSACTION;
    END
END;
GO
