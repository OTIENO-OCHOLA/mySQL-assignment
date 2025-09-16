# mySQL-assignment
-- Library Management System Database
-- Created by: [Your Name]
-- Date: [Current Date]

-- Create the database
CREATE DATABASE IF NOT EXISTS library_management_system;
USE library_management_system;

-- Table 1: Members (Library Members/Users)
CREATE TABLE IF NOT EXISTS members (
    member_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone_number VARCHAR(15),
    address TEXT,
    date_of_birth DATE NOT NULL,
    membership_date DATE NOT NULL DEFAULT (CURRENT_DATE),
    membership_status ENUM('Active', 'Suspended', 'Expired') DEFAULT 'Active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_email (email),
    INDEX idx_membership_status (membership_status)
);

-- Table 2: Authors
CREATE TABLE IF NOT EXISTS authors (
    author_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    birth_date DATE,
    death_date DATE,
    nationality VARCHAR(50),
    biography TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_author_name (last_name, first_name)
);

-- Table 3: Publishers
CREATE TABLE IF NOT EXISTS publishers (
    publisher_id INT AUTO_INCREMENT PRIMARY KEY,
    publisher_name VARCHAR(100) NOT NULL UNIQUE,
    address TEXT,
    contact_email VARCHAR(100),
    contact_phone VARCHAR(15),
    established_year YEAR,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_publisher_name (publisher_name)
);

-- Table 4: Books
CREATE TABLE IF NOT EXISTS books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    isbn VARCHAR(13) UNIQUE NOT NULL,
    title VARCHAR(255) NOT NULL,
    author_id INT NOT NULL,
    publisher_id INT NOT NULL,
    publication_year YEAR,
    genre VARCHAR(50) NOT NULL,
    language VARCHAR(30) DEFAULT 'English',
    page_count INT,
    description TEXT,
    total_copies INT NOT NULL DEFAULT 1,
    available_copies INT NOT NULL DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (author_id) REFERENCES authors(author_id) ON DELETE CASCADE,
    FOREIGN KEY (publisher_id) REFERENCES publishers(publisher_id) ON DELETE CASCADE,
    
    INDEX idx_title (title),
    INDEX idx_genre (genre),
    INDEX idx_isbn (isbn),
    INDEX idx_author (author_id)
);

-- Table 5: Book_Authors (Many-to-Many relationship for books with multiple authors)
CREATE TABLE IF NOT EXISTS book_authors (
    book_id INT NOT NULL,
    author_id INT NOT NULL,
    contribution_type ENUM('Primary', 'Co-Author', 'Editor', 'Translator') DEFAULT 'Primary',
    
    PRIMARY KEY (book_id, author_id),
    FOREIGN KEY (book_id) REFERENCES books(book_id) ON DELETE CASCADE,
    FOREIGN KEY (author_id) REFERENCES authors(author_id) ON DELETE CASCADE,
    
    INDEX idx_author_book (author_id, book_id)
);

-- Table 6: Loans (Borrowing transactions)
CREATE TABLE IF NOT EXISTS loans (
    loan_id INT AUTO_INCREMENT PRIMARY KEY,
    member_id INT NOT NULL,
    book_id INT NOT NULL,
    loan_date DATE NOT NULL DEFAULT (CURRENT_DATE),
    due_date DATE NOT NULL,
    return_date DATE,
    status ENUM('Active', 'Returned', 'Overdue') DEFAULT 'Active',
    fine_amount DECIMAL(8,2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE,
    FOREIGN KEY (book_id) REFERENCES books(book_id) ON DELETE CASCADE,
    
    INDEX idx_member_loans (member_id),
    INDEX idx_book_loans (book_id),
    INDEX idx_loan_status (status),
    INDEX idx_due_date (due_date)
);

-- Table 7: Reservations
CREATE TABLE IF NOT EXISTS reservations (
    reservation_id INT AUTO_INCREMENT PRIMARY KEY,
    member_id INT NOT NULL,
    book_id INT NOT NULL,
    reservation_date DATE NOT NULL DEFAULT (CURRENT_DATE),
    status ENUM('Pending', 'Fulfilled', 'Cancelled') DEFAULT 'Pending',
    priority INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE,
    FOREIGN KEY (book_id) REFERENCES books(book_id) ON DELETE CASCADE,
    
    UNIQUE KEY unique_active_reservation (member_id, book_id, status),
    INDEX idx_reservation_status (status),
    INDEX idx_reservation_date (reservation_date)
);

-- Table 8: Fines
CREATE TABLE IF NOT EXISTS fines (
    fine_id INT AUTO_INCREMENT PRIMARY KEY,
    member_id INT NOT NULL,
    loan_id INT NOT NULL,
    amount DECIMAL(8,2) NOT NULL,
    reason ENUM('Overdue', 'Damage', 'Lost') NOT NULL,
    status ENUM('Pending', 'Paid', 'Waived') DEFAULT 'Pending',
    issued_date DATE NOT NULL DEFAULT (CURRENT_DATE),
    paid_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE,
    FOREIGN KEY (loan_id) REFERENCES loans(loan_id) ON DELETE CASCADE,
    
    INDEX idx_fine_status (status),
    INDEX idx_member_fines (member_id)
);

-- Table 9: Library Staff
CREATE TABLE IF NOT EXISTS staff (
    staff_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone_number VARCHAR(15),
    position VARCHAR(50) NOT NULL,
    hire_date DATE NOT NULL DEFAULT (CURRENT_DATE),
    salary DECIMAL(10,2),
    status ENUM('Active', 'Inactive', 'On Leave') DEFAULT 'Active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_staff_email (email),
    INDEX idx_staff_position (position)
);

-- Insert sample data
INSERT INTO members (first_name, last_name, email, phone_number, address, date_of_birth, membership_date) VALUES
('John', 'Doe', 'john.doe@email.com', '555-0101', '123 Main St, Cityville', '1990-05-15', '2024-01-15'),
('Jane', 'Smith', 'jane.smith@email.com', '555-0102', '456 Oak Ave, Townsville', '1985-08-22', '2024-02-20'),
('Bob', 'Johnson', 'bob.johnson@email.com', '555-0103', '789 Pine Rd, Villageton', '1992-12-03', '2024-03-10');

INSERT INTO authors (first_name, last_name, birth_date, death_date, nationality) VALUES
('George', 'Orwell', '1903-06-25', '1950-01-21', 'British'),
('J.K.', 'Rowling', '1965-07-31', NULL, 'British'),
('Stephen', 'King', '1947-09-21', NULL, 'American'),
('Jane', 'Austen', '1775-12-16', '1817-07-18', 'British');

INSERT INTO publishers (publisher_name, address, contact_email, established_year) VALUES
('Penguin Books', 'London, UK', 'info@penguin.com', 1935),
('Scholastic', 'New York, USA', 'contact@scholastic.com', 1920),
('Random House', 'New York, USA', 'info@randomhouse.com', 1927);

INSERT INTO books (isbn, title, author_id, publisher_id, publication_year, genre, page_count, total_copies, available_copies) VALUES
('9780451524935', '1984', 1, 1, 1949, 'Dystopian Fiction', 328, 5, 5),
('9780439064866', 'Harry Potter and the Chamber of Secrets', 2, 2, 1998, 'Fantasy', 341, 3, 3),
('9781501142970', 'It', 3, 3, 1986, 'Horror', 1138, 2, 2),
('9780141439518', 'Pride and Prejudice', 4, 1, 1813, 'Romance', 432, 4, 4);

INSERT INTO staff (first_name, last_name, email, phone_number, position, hire_date, salary) VALUES
('Sarah', 'Wilson', 'sarah.wilson@library.com', '555-0201', 'Librarian', '2020-06-15', 55000.00),
('Mike', 'Chen', 'mike.chen@library.com', '555-0202', 'Assistant Librarian', '2022-01-10', 42000.00);

-- Create views for common queries
CREATE VIEW active_loans AS
SELECT l.loan_id, m.first_name, m.last_name, b.title, l.loan_date, l.due_date, l.status
FROM loans l
JOIN members m ON l.member_id = m.member_id
JOIN books b ON l.book_id = b.book_id
WHERE l.status = 'Active';

CREATE VIEW available_books AS
SELECT b.book_id, b.title, a.first_name AS author_first, a.last_name AS author_last, 
       b.genre, b.available_copies
FROM books b
JOIN authors a ON b.author_id = a.author_id
WHERE b.available_copies > 0;

CREATE VIEW member_loan_history AS
SELECT m.member_id, m.first_name, m.last_name, 
       COUNT(l.loan_id) AS total_loans,
       SUM(CASE WHEN l.status = 'Active' THEN 1 ELSE 0 END) AS active_loans,
       SUM(CASE WHEN l.return_date > l.due_date THEN 1 ELSE 0 END) AS overdue_count
FROM members m
LEFT JOIN loans l ON m.member_id = l.member_id
GROUP BY m.member_id, m.first_name, m.last_name;

-- Create stored procedures
DELIMITER //

CREATE PROCEDURE BorrowBook(IN p_member_id INT, IN p_book_id INT)
BEGIN
    DECLARE available_count INT;
    DECLARE active_loans_count INT;
    
    -- Check if book is available
    SELECT available_copies INTO available_count FROM books WHERE book_id = p_book_id;
    
    -- Check if member has too many active loans (max 5)
    SELECT COUNT(*) INTO active_loans_count FROM loans 
    WHERE member_id = p_member_id AND status = 'Active';
    
    IF available_count > 0 AND active_loans_count < 5 THEN
        -- Create loan record
        INSERT INTO loans (member_id, book_id, due_date)
        VALUES (p_member_id, p_book_id, DATE_ADD(CURRENT_DATE, INTERVAL 14 DAY));
        
        -- Update available copies
        UPDATE books SET available_copies = available_copies - 1 WHERE book_id = p_book_id;
        
        SELECT 'SUCCESS' AS result, 'Book borrowed successfully' AS message;
    ELSE
        IF available_count = 0 THEN
            SELECT 'ERROR' AS result, 'Book not available' AS message;
        ELSE
            SELECT 'ERROR' AS result, 'Member has reached maximum loan limit' AS message;
        END IF;
    END IF;
END //

CREATE PROCEDURE ReturnBook(IN p_loan_id INT)
BEGIN
    DECLARE v_book_id INT;
    DECLARE v_due_date DATE;
    DECLARE v_fine_amount DECIMAL(8,2);
    
    -- Get book ID and due date
    SELECT book_id, due_date INTO v_book_id, v_due_date 
    FROM loans WHERE loan_id = p_loan_id;
    
    -- Calculate fine if overdue
    IF CURRENT_DATE > v_due_date THEN
        SET v_fine_amount = DATEDIFF(CURRENT_DATE, v_due_date) * 0.50; -- $0.50 per day
        INSERT INTO fines (member_id, loan_id, amount, reason, status)
        SELECT member_id, p_loan_id, v_fine_amount, 'Overdue', 'Pending'
        FROM loans WHERE loan_id = p_loan_id;
    END IF;
    
    -- Update loan record
    UPDATE loans 
    SET return_date = CURRENT_DATE, 
        status = 'Returned',
        fine_amount = COALESCE(v_fine_amount, 0)
    WHERE loan_id = p_loan_id;
    
    -- Update available copies
    UPDATE books SET available_copies = available_copies + 1 WHERE book_id = v_book_id;
    
    SELECT 'SUCCESS' AS result, 'Book returned successfully' AS message;
END //

DELIMITER ;

-- Create triggers
DELIMITER //

CREATE TRIGGER before_loan_insert
BEFORE INSERT ON loans
FOR EACH ROW
BEGIN
    IF NEW.due_date IS NULL THEN
        SET NEW.due_date = DATE_ADD(CURRENT_DATE, INTERVAL 14 DAY);
    END IF;
END //

CREATE TRIGGER update_book_availability
AFTER UPDATE ON loans
FOR EACH ROW
BEGIN
    IF NEW.status = 'Returned' AND OLD.status != 'Returned' THEN
        UPDATE books SET available_copies = available_copies + 1 WHERE book_id = NEW.book_id;
    END IF;
END //

DELIMITER ;

-- Show database structure information
SELECT 'Database created successfully with:' AS '';
SELECT COUNT(*) AS 'Total Tables' FROM information_schema.tables 
WHERE table_schema = 'library_management_system';

SELECT table_name, table_rows 
FROM information_schema.tables 
WHERE table_schema = 'library_management_system' 
ORDER BY table_name;