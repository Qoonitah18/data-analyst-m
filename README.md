# data-analyst-m

## Dataset
Proyek ini menggunakan tiga dataset:
- users_data.csv (full)
- cards_data.csv (full)
- transactions_data.csv (hanya 53880 data yang dapat diimport)

## Pembuatan Database
CREATE DATABASE mandiri_sekuritas;
USE mandiri_sekuritas;

## Pembuatan Tabel Kosong
CREATE TABLE users (
    id INT PRIMARY KEY,
    current_age INT,
    retirement_age INT,
    birth_year INT,
    birth_month INT,
    gender VARCHAR(10),
    address VARCHAR(255),
    latitude FLOAT,
    longitude FLOAT,
    per_capita_income VARCHAR(20),
    yearly_income VARCHAR(20),
    total_debt VARCHAR(20),
    credit_score INT,
    num_credit_cards INT
);

CREATE TABLE cards (
    id INT PRIMARY KEY,
    client_id INT,
    card_brand VARCHAR(50),
    card_type VARCHAR(50),
    card_number BIGINT,
    expires VARCHAR(10),
    cvv INT,
    has_chip VARCHAR(10),
    num_cards_issued INT,
    credit_limit VARCHAR(20),
    acct_open_date VARCHAR(20),
    year_pin_last_changed INT,
    card_on_dark_web VARCHAR(10)
);

CREATE TABLE transactions (
    id BIGINT PRIMARY KEY,
    date DATETIME,
    client_id INT,
    card_id INT,
    amount VARCHAR(20),
    use_chip VARCHAR(50),
    merchant_id INT,
    merchant_city VARCHAR(100),
    merchant_state VARCHAR(10),
    zip VARCHAR(10),
    mcc INT,
    errors VARCHAR(255)
);

Selanjutnya import ketiga file csv ke MySQL dengan menggunakan fitur table data import wizard

## Pembuatan Gabungan Data
Buat tabel gabungan `full_data`:
CREATE TABLE full_data AS
SELECT 
  t.id AS transaction_id,
  t.date,
  t.client_id,
  t.card_id,
  t.amount,
  t.use_chip,
  t.merchant_id,
  t.merchant_city,
  t.merchant_state,
  t.zip,
  t.mcc,
  t.errors,
  u.gender,
  u.current_age,
  u.per_capita_income,
  u.yearly_income,
  u.credit_score,
  u.num_credit_cards,
  c.card_brand,
  c.card_type,
  c.credit_limit,
  c.card_on_dark_web
FROM transactions t
JOIN users u ON t.client_id = u.id
JOIN cards c ON t.card_id = c.id;

## Analisis
1. Top 10 Users with Highest Spending
SELECT client_id,
       SUM(CAST(REPLACE(REPLACE(amount, '$', ''), ',', '') AS DECIMAL(10,2))) AS total_spent
FROM full_data
GROUP BY client_id
ORDER BY total_spent DESC
LIMIT 10;
`` fungsi:
REPLACE : mengganti simbol sesuai yang diinginkan
CAST : mengubah string menjadi angka
SUM : menjumlahkan
GROUP BY : mengelompokkan
ORDER BY : mengurutkan

3. Average Amount by Card Types
SELECT card_type,
       COUNT(*) AS num_transactions,
       AVG(CAST(REPLACE(REPLACE(amount, '$', ''), ',', '') AS DECIMAL(10,2))) AS avg_amount
FROM full_data
GROUP BY card_type;
`` fungsi:
COUNT : menghitung banyaknya sesuatu
AVG : menghitung rata-rata
DECIMAL : mengubah data menjadi desimal dengan 10 digit dan 2 digit di belakang koma

5. Error Transaction
SELECT COUNT(*) AS total_errors,
       COUNT(DISTINCT client_id) AS affected_users
FROM full_data
WHERE errors IS NOT NULL;
`` fungsi:
DISTINCT : membuang duplikat

7. Total Transaction by Gender
SELECT gender, COUNT(*) AS total_transactions
FROM full_data
GROUP BY gender
ORDER BY total_transactions DESC;

8. Top 10 Cities with Most Transaction
SELECT merchant_city, COUNT(*) AS total_transactions
FROM full_data
GROUP BY merchant_city
ORDER BY total_transactions DESC
LIMIT 10;

9. Transaction Distribution by Age
SELECT current_age, COUNT(*) AS total_transactions
FROM full_data
GROUP BY current_age
ORDER BY current_age;

10. Distribution of Card Brands
SELECT card_brand, COUNT(*) AS total_transactions
FROM full_data
GROUP BY card_brand
ORDER BY total_transactions DESC;
