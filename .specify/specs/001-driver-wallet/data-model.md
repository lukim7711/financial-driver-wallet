# Data Model: Driver Wallet

**Branch**: `001-driver-wallet` | **Date**: 2026-02-24 | **Spec**: [spec.md](./spec.md)

---

## Database Overview

- **Database**: `AppDatabase` (Room / SQLite)
- **Version**: 1
- **Entities**: 4 (Transaction, Category, Debt, DebtPayment)
- **Total Tables**: 4

---

## Entity Relationship Diagram

```
┌─────────────────┐       ┌─────────────────────┐
│    Category      │       │     Transaction      │
├─────────────────┤       ├─────────────────────┤
│ id (PK)         │◄──────│ categoryId (FK)      │
│ name            │   1:N │ id (PK)              │
│ iconName        │       │ amount               │
│ type            │       │ type (INCOME/EXPENSE) │
│ isDefault       │       │ note                 │
│ createdAt       │       │ createdAt            │
└─────────────────┘       └─────────────────────┘

┌─────────────────┐       ┌─────────────────────┐
│      Debt        │       │    DebtPayment       │
├─────────────────┤       ├─────────────────────┤
│ id (PK)         │◄──────│ debtId (FK)          │
│ personName      │   1:N │ id (PK)              │
│ totalAmount     │       │ amount               │
│ remainingAmount │       │ note                 │
│ type            │       │ createdAt            │
│ dueDate         │       └─────────────────────┘
│ status          │
│ note            │
│ createdAt       │
└─────────────────┘
```

---

## Table Definitions

### 1. `categories`

| Column | Type | Constraints | Description |
|:--|:--|:--|:--|
| `id` | INTEGER | PRIMARY KEY, AUTOINCREMENT | Unique identifier |
| `name` | TEXT | NOT NULL | Nama kategori (e.g., "Grab", "Bensin") |
| `icon_name` | TEXT | NOT NULL | Nama ikon Material (e.g., "directions_car") |
| `type` | TEXT | NOT NULL, CHECK(IN 'INCOME','EXPENSE','BOTH') | Tipe kategori |
| `is_default` | INTEGER | NOT NULL, DEFAULT 0 | 1 = kategori bawaan, 0 = custom user |
| `created_at` | INTEGER | NOT NULL | Timestamp epoch milliseconds |

**Default Data (Seed)**:

| name | icon_name | type |
|:--|:--|:--|
| Grab | directions_car | INCOME |
| Gojek | two_wheeler | INCOME |
| Maxim | local_taxi | INCOME |
| Shopee Food | fastfood | INCOME |
| Bensin | local_gas_station | EXPENSE |
| Makan | restaurant | EXPENSE |
| Servis Kendaraan | build | EXPENSE |
| Parkir | local_parking | EXPENSE |
| Lainnya | more_horiz | BOTH |

---

### 2. `transactions`

| Column | Type | Constraints | Description |
|:--|:--|:--|:--|
| `id` | INTEGER | PRIMARY KEY, AUTOINCREMENT | Unique identifier |
| `amount` | INTEGER | NOT NULL, CHECK(> 0) | Nominal dalam Rupiah (tanpa desimal) |
| `type` | TEXT | NOT NULL, CHECK(IN 'INCOME','EXPENSE') | Tipe transaksi |
| `category_id` | INTEGER | NOT NULL, FOREIGN KEY → categories(id) | Referensi ke kategori |
| `note` | TEXT | NULLABLE | Catatan opsional |
| `created_at` | INTEGER | NOT NULL | Timestamp epoch milliseconds |

**Indexes**:
- `idx_transactions_created_at` ON `created_at` — untuk query harian, mingguan, bulanan
- `idx_transactions_category_id` ON `category_id` — untuk filter per kategori

---

### 3. `debts`

| Column | Type | Constraints | Description |
|:--|:--|:--|:--|
| `id` | INTEGER | PRIMARY KEY, AUTOINCREMENT | Unique identifier |
| `person_name` | TEXT | NOT NULL | Nama orang yang berhutang/berpiutang |
| `total_amount` | INTEGER | NOT NULL, CHECK(> 0) | Nominal total hutang awal |
| `remaining_amount` | INTEGER | NOT NULL, CHECK(>= 0) | Sisa hutang (auto-calculated) |
| `type` | TEXT | NOT NULL, CHECK(IN 'HUTANG','PIUTANG') | Hutang = saya yang bayar, Piutang = orang lain yang bayar |
| `due_date` | INTEGER | NULLABLE | Tanggal jatuh tempo (epoch ms), opsional |
| `status` | TEXT | NOT NULL, DEFAULT 'ACTIVE', CHECK(IN 'ACTIVE','PAID') | Status hutang |
| `note` | TEXT | NULLABLE | Catatan opsional |
| `created_at` | INTEGER | NOT NULL | Timestamp epoch milliseconds |

**Indexes**:
- `idx_debts_status` ON `status` — untuk filter hutang aktif
- `idx_debts_due_date` ON `due_date` — untuk query reminder

---

### 4. `debt_payments`

| Column | Type | Constraints | Description |
|:--|:--|:--|:--|
| `id` | INTEGER | PRIMARY KEY, AUTOINCREMENT | Unique identifier |
| `debt_id` | INTEGER | NOT NULL, FOREIGN KEY → debts(id) ON DELETE CASCADE | Referensi ke hutang |
| `amount` | INTEGER | NOT NULL, CHECK(> 0) | Nominal pembayaran |
| `note` | TEXT | NULLABLE | Catatan opsional |
| `created_at` | INTEGER | NOT NULL | Timestamp epoch milliseconds |

---

## Key Queries (DAO)

### TransactionDao

```sql
-- Dasbor harian: total pemasukan hari ini
SELECT COALESCE(SUM(amount), 0) FROM transactions 
WHERE type = 'INCOME' AND created_at BETWEEN :startOfDay AND :endOfDay;

-- Dasbor harian: total pengeluaran hari ini
SELECT COALESCE(SUM(amount), 0) FROM transactions 
WHERE type = 'EXPENSE' AND created_at BETWEEN :startOfDay AND :endOfDay;

-- Daftar transaksi hari ini (terbaru di atas)
SELECT t.*, c.name AS categoryName, c.icon_name AS categoryIcon 
FROM transactions t INNER JOIN categories c ON t.category_id = c.id 
WHERE t.created_at BETWEEN :startOfDay AND :endOfDay 
ORDER BY t.created_at DESC;

-- Ringkasan per hari dalam rentang (untuk grafik mingguan/bulanan)
SELECT DATE(created_at/1000, 'unixepoch', 'localtime') AS date,
       SUM(CASE WHEN type = 'INCOME' THEN amount ELSE 0 END) AS totalIncome,
       SUM(CASE WHEN type = 'EXPENSE' THEN amount ELSE 0 END) AS totalExpense
FROM transactions
WHERE created_at BETWEEN :startDate AND :endDate
GROUP BY date ORDER BY date;
```

### DebtDao

```sql
-- Daftar hutang aktif
SELECT * FROM debts WHERE status = 'ACTIVE' ORDER BY due_date ASC NULLS LAST;

-- Hutang jatuh tempo hari ini (untuk notifikasi)
SELECT * FROM debts 
WHERE status = 'ACTIVE' AND due_date BETWEEN :startOfDay AND :endOfDay;

-- Update sisa hutang setelah pembayaran
UPDATE debts SET remaining_amount = remaining_amount - :paymentAmount,
               status = CASE WHEN remaining_amount - :paymentAmount <= 0 THEN 'PAID' ELSE 'ACTIVE' END
WHERE id = :debtId;
```

---

## Migration Strategy

- **Version 1**: Initial schema (semua tabel di atas).
- **Future versions**: Gunakan Room auto-migration atau manual Migration class. Setiap perubahan skema HARUS didokumentasikan di file ini sebelum implementasi.
- **Fallback**: `fallbackToDestructiveMigration()` TIDAK BOLEH digunakan di production build. Data pengguna harus selalu dipertahankan.
