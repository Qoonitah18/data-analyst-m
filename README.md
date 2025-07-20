# data-analyst-m

## Dataset
Proyek ini menggunakan tiga dataset:
- users_data.csv (full)
- cards_data.csv (full)
- transactions_data.csv (hanya 53880 data yang dapat diimport)

Selanjutnya data akan digabungkan untuk memudahkan analisis

## Langkah Pembuatan Gabungan Data
1. Import ketiga file csv ke MySQL. Di sini saya menggunakan table data import wizard.

2. Buat tabel gabungan `full_data`:
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
