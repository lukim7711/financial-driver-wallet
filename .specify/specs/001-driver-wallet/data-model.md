# Data Model: Driver Wallet

**Branch**: `001-driver-wallet` | **Date**: 2026-02-25 | **Spec**: [spec.md](./spec.md)

---

## Database Overview

- **Database**: `AppDatabase` (Room / SQLite)
- **Version**: 2 (migrated from v1 — see Migration Strategy)
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

┌──────────────────────┐       ┌─────────────────────┐
│        Debt           │       │    DebtPayment       │
├──────────────────────┤       ├─────────────────────┤
│ id (PK)              │◄──────│ debtId (FK)          │
│ personName           │   1:N │ id (PK)              │
│ totalAmount          │       │ amount               │
│ remainingAmount      │       │ note                 │
│ type (HUTANG/PIUTANG)│       │ createdAt            │
│ dueDate              │       └─────────────────────┘
│ status               │
│ note                 │
│ createdAt            │
│ ── v2 columns ──     │
│ debtKind             │
│ debtCategory         │
│ monthlyAmount        │
│ installmentDay       │
│ totalInstallments    │
│ paidInstallments     │
│ penaltyType          │
│ penaltyAmount        │
└──────────────────────┘
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

#### Kolom Original (v1)

| Column | Type | Constraints | Description |
|:--|:--|:--|:--|
| `id` | INTEGER | PRIMARY KEY, AUTOINCREMENT | Unique identifier |
| `person_name` | TEXT | NOT NULL | Nama orang (personal) atau platform/dealer (installment) |
| `total_amount` | INTEGER | NOT NULL, CHECK(> 0) | Nominal total hutang awal |
| `remaining_amount` | INTEGER | NOT NULL, CHECK(>= 0) | Sisa hutang (auto-calculated) |
| `type` | TEXT | NOT NULL, CHECK(IN 'HUTANG','PIUTANG') | Hutang = saya yang bayar, Piutang = orang lain yang bayar |
| `due_date` | INTEGER | NULLABLE | Tanggal jatuh tempo tunggal (epoch ms), untuk hutang personal |
| `status` | TEXT | NOT NULL, DEFAULT 'ACTIVE', CHECK(IN 'ACTIVE','PAID') | Status hutang |
| `note` | TEXT | NULLABLE | Catatan opsional |
| `created_at` | INTEGER | NOT NULL | Timestamp epoch milliseconds |

#### Kolom Baru (v2 — Cicilan Tetap, US8)

| Column | Type | Constraints | Description |
|:--|:--|:--|:--|
| `debt_kind` | TEXT | NOT NULL, DEFAULT 'PERSONAL', CHECK(IN 'PERSONAL','INSTALLMENT') | Jenis hutang: personal (US5) atau cicilan tetap (US8) |
| `debt_category` | TEXT | NULLABLE, CHECK(IN 'PINJOL','MOTOR','HP','ELEKTRONIK','LAINNYA') | Kategori cicilan (hanya jika debt_kind = INSTALLMENT) |
| `monthly_amount` | INTEGER | NULLABLE, CHECK(> 0) | Nominal cicilan tetap per bulan (hanya jika INSTALLMENT) |
| `installment_day` | INTEGER | NULLABLE, CHECK(BETWEEN 1 AND 31) | Tanggal jatuh tempo bulanan, e.g., 15 = setiap tgl 15 (hanya jika INSTALLMENT) |
| `total_installments` | INTEGER | NULLABLE, CHECK(> 0) | Jumlah total cicilan, e.g., 12 bulan (hanya jika INSTALLMENT) |
| `paid_installments` | INTEGER | NULLABLE, DEFAULT 0, CHECK(>= 0) | Jumlah cicilan yang sudah dibayar |
| `penalty_type` | TEXT | NULLABLE, CHECK(IN 'FLAT','PERCENT_MONTHLY','PERCENT_DAILY') | Jenis denda keterlambatan |
| `penalty_amount` | INTEGER | NULLABLE | Nominal denda: Rupiah jika FLAT, basis poin jika % (e.g., 50 = 0.5%, 100 = 1.0%, 500 = 5.0%) |

**Catatan**: Semua kolom v2 bersifat NULLABLE dengan DEFAULT agar backward-compatible dengan data hutang personal v1 yang sudah ada.

**Indexes**:
- `idx_debts_status` ON `status` — untuk filter hutang aktif
- `idx_debts_due_date` ON `due_date` — untuk query reminder hutang personal
- `idx_debts_debt_kind` ON `debt_kind` — untuk filter personal vs installment (v2)
- `idx_debts_installment_day` ON `installment_day` — untuk query reminder cicilan bulanan (v2)

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

### DebtDao — Hutang Personal (US5)

```sql
-- Daftar hutang personal aktif
SELECT * FROM debts 
WHERE debt_kind = 'PERSONAL' AND status = 'ACTIVE' 
ORDER BY due_date ASC NULLS LAST;

-- Hutang personal jatuh tempo hari ini (untuk notifikasi)
SELECT * FROM debts 
WHERE debt_kind = 'PERSONAL' AND status = 'ACTIVE' 
AND due_date BETWEEN :startOfDay AND :endOfDay;

-- Update sisa hutang setelah pembayaran
UPDATE debts SET remaining_amount = remaining_amount - :paymentAmount,
               status = CASE WHEN remaining_amount - :paymentAmount <= 0 THEN 'PAID' ELSE 'ACTIVE' END
WHERE id = :debtId;
```

### DebtDao — Cicilan Tetap (US8)

```sql
-- Daftar cicilan tetap aktif
SELECT * FROM debts 
WHERE debt_kind = 'INSTALLMENT' AND status = 'ACTIVE' 
ORDER BY installment_day ASC;

-- Cicilan yang jatuh tempo pada tanggal tertentu (untuk notifikasi harian)
-- Cocokkan installment_day dengan hari ini
SELECT * FROM debts 
WHERE debt_kind = 'INSTALLMENT' AND status = 'ACTIVE' 
AND installment_day = :todayDayOfMonth;

-- Cicilan yang jatuh tempo dalam 3 hari ke depan (untuk early reminder)
-- :todayDay, :day2, :day3 = tanggal hari ini, besok, lusa
SELECT * FROM debts 
WHERE debt_kind = 'INSTALLMENT' AND status = 'ACTIVE' 
AND installment_day IN (:todayDay, :day2, :day3);

-- Update setelah bayar cicilan tetap
UPDATE debts SET 
  remaining_amount = remaining_amount - :monthlyAmount,
  paid_installments = paid_installments + 1,
  status = CASE WHEN paid_installments + 1 >= total_installments THEN 'PAID' ELSE 'ACTIVE' END
WHERE id = :debtId;

-- Daftar semua hutang (personal + installment) dengan filter
SELECT * FROM debts 
WHERE (:kindFilter IS NULL OR debt_kind = :kindFilter)
AND (:statusFilter IS NULL OR status = :statusFilter)
ORDER BY 
  CASE WHEN status = 'ACTIVE' THEN 0 ELSE 1 END,
  created_at DESC;
```

### DebtDao — Perhitungan Denda (computed di Domain layer)

```
Denda dihitung di CalculatePenaltyUseCase (Domain layer), BUKAN di SQL.
Alasan: logika denda melibatkan perhitungan tanggal yang kompleks
(hari keterlambatan, bulan keterlambatan) yang lebih tepat dilakukan
di Kotlin menggunakan java.time API.

Rumus:
- FLAT: penalty_amount (langsung)
- PERCENT_MONTHLY: (penalty_amount / 100.0) × monthly_amount × bulan_telat
- PERCENT_DAILY: (penalty_amount / 100.0) × monthly_amount × hari_telat

Catatan: penalty_amount disimpan sebagai basis poin integer.
Contoh: 50 = 0.5%, 100 = 1.0%, 500 = 5.0%
Rumus konversi: persentase_aktual = penalty_amount / 100.0
```

---

## Migration Strategy

### Version 1 → Version 2 (Cicilan Tetap — US8)

**Alasan**: Menambahkan fitur Cicilan Tetap (US8) memerlukan kolom baru di tabel `debts`.

**Strategi**: Manual Migration menggunakan `ALTER TABLE ADD COLUMN`. Semua kolom baru bersifat NULLABLE dengan DEFAULT agar data existing (hutang personal v1) tetap valid tanpa modifikasi.

```sql
-- Migration_1_2.kt
ALTER TABLE debts ADD COLUMN debt_kind TEXT NOT NULL DEFAULT 'PERSONAL';
ALTER TABLE debts ADD COLUMN debt_category TEXT;
ALTER TABLE debts ADD COLUMN monthly_amount INTEGER;
ALTER TABLE debts ADD COLUMN installment_day INTEGER;
ALTER TABLE debts ADD COLUMN total_installments INTEGER;
ALTER TABLE debts ADD COLUMN paid_installments INTEGER DEFAULT 0;
ALTER TABLE debts ADD COLUMN penalty_type TEXT;
ALTER TABLE debts ADD COLUMN penalty_amount INTEGER;

-- New indexes
CREATE INDEX IF NOT EXISTS idx_debts_debt_kind ON debts(debt_kind);
CREATE INDEX IF NOT EXISTS idx_debts_installment_day ON debts(installment_day);
```

**Validasi**: Setelah migration, semua hutang personal existing akan memiliki `debt_kind = 'PERSONAL'` dan semua kolom baru = NULL/0, sehingga flow US5 tidak terpengaruh.

**⚠️ PENTING**: `fallbackToDestructiveMigration()` TIDAK BOLEH digunakan (Constitution Section V). Data pengguna harus selalu dipertahankan.

### Future Versions

Gunakan Room auto-migration atau manual Migration class. Setiap perubahan skema HARUS didokumentasikan di file ini sebelum implementasi.
