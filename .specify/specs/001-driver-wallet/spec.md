# Feature Specification: Driver Wallet — Pencatat Keuangan Ojol

**Feature Branch**: `001-driver-wallet`
**Created**: 2026-02-24
**Updated**: 2026-02-25
**Status**: Draft
**Input**: User description: "Aplikasi pencatat keuangan offline untuk pengemudi ojek daring di Jakarta. Fitur utama: pencatatan transaksi cepat (<3 detik), dasbor harian, dan manajemen hutang/cicilan."

---

## User Scenarios & Testing

### User Story 1 — Pencatatan Transaksi Cepat (Priority: P1)

Sebagai pengemudi ojol, saya ingin **mencatat pemasukan dan pengeluaran dalam kurang dari 3 detik** tanpa harus mengetik banyak, agar saya bisa langsung kembali bekerja setelah menyelesaikan order.

Pengemudi biasanya mencatat sambil berhenti sebentar di pinggir jalan. Mereka butuh cara tercepat: pilih kategori → masukkan nominal → simpan. Selesai.

**Why this priority**: Ini adalah alasan utama aplikasi ini ada. Tanpa pencatatan cepat, aplikasi ini tidak memiliki nilai. Semua fitur lain bergantung pada data transaksi yang dihasilkan fitur ini.

**Independent Test**: Bisa diuji sepenuhnya dengan membuka aplikasi, mengetuk tombol "+", memilih kategori, memasukkan angka, dan mengetuk "Simpan". Transaksi harus muncul di daftar.

**Acceptance Scenarios**:

1. **Given** aplikasi terbuka di halaman utama, **When** user mengetuk tombol tambah (+), **Then** form pencatatan muncul dalam < 300ms dengan keyboard numerik otomatis terbuka.
2. **Given** form pencatatan terbuka, **When** user memilih kategori "Grab/Gojek" dan memasukkan nominal "50000" lalu tekan "Simpan", **Then** transaksi tersimpan ke database dan user kembali ke halaman utama dalam < 1 detik.
3. **Given** form pencatatan terbuka, **When** user memilih tipe "Pengeluaran" dan kategori "Bensin", memasukkan nominal "20000" lalu tekan "Simpan", **Then** transaksi pengeluaran tersimpan dengan tanda negatif dan saldo harian berkurang.
4. **Given** form pencatatan terbuka, **When** user memasukkan nominal 0 atau kosong lalu tekan "Simpan", **Then** sistem menampilkan error "Nominal harus lebih dari 0" dan tidak menyimpan.
5. **Given** perangkat dalam mode Airplane, **When** user menyimpan transaksi, **Then** transaksi tetap tersimpan karena seluruhnya offline.

---

### User Story 2 — Dasbor Harian (Priority: P1)

Sebagai pengemudi ojol, saya ingin **melihat ringkasan keuangan hari ini dalam satu layar** — total pemasukan, total pengeluaran, dan pendapatan bersih — agar saya tahu apakah hari ini sudah cukup menguntungkan.

**Why this priority**: Dasbor adalah hal pertama yang dilihat pengemudi saat membuka aplikasi. Ini memberikan motivasi dan kontrol atas keuangan harian mereka. Sama pentingnya dengan pencatatan.

**Independent Test**: Bisa diuji dengan memasukkan beberapa transaksi hari ini, lalu memverifikasi bahwa total pemasukan, pengeluaran, dan nett income ditampilkan dengan benar di halaman utama.

**Acceptance Scenarios**:

1. **Given** user membuka aplikasi, **When** halaman utama tampil, **Then** dasbor menampilkan: total pemasukan hari ini, total pengeluaran hari ini, dan pendapatan bersih (pemasukan - pengeluaran).
2. **Given** belum ada transaksi hari ini, **When** halaman utama tampil, **Then** semua angka menunjukkan Rp 0 dengan pesan "Belum ada transaksi hari ini".
3. **Given** user menambahkan transaksi baru, **When** kembali ke halaman utama, **Then** angka dasbor langsung ter-update secara real-time tanpa perlu refresh manual.
4. **Given** user menghapus sebuah transaksi, **When** kembali ke dasbor, **Then** angka dasbor berkurang sesuai nominal transaksi yang dihapus.

---

### User Story 3 — Daftar Transaksi Harian (Priority: P1)

Sebagai pengemudi ojol, saya ingin **melihat daftar semua transaksi hari ini** di bawah dasbor, diurutkan dari yang terbaru, agar saya bisa mengecek apakah ada yang terlewat.

**Why this priority**: Melengkapi dasbor. Tanpa daftar, pengemudi tidak bisa memverifikasi detail angka di dasbor.

**Independent Test**: Setelah mencatat 5 transaksi, daftar menampilkan ke-5 transaksi dengan urutan terbaru di atas.

**Acceptance Scenarios**:

1. **Given** ada 5 transaksi hari ini, **When** user scroll ke bawah dasbor, **Then** daftar menampilkan 5 item dengan urutan waktu terbaru di atas.
2. **Given** daftar transaksi tampil, **When** user mengetuk salah satu item, **Then** detail transaksi muncul (kategori, nominal, waktu, catatan opsional).
3. **Given** daftar transaksi tampil, **When** user swipe item ke kiri, **Then** opsi "Hapus" muncul dengan konfirmasi "Yakin hapus transaksi ini?".
4. **Given** user mengkonfirmasi hapus, **When** proses selesai, **Then** item hilang dari daftar dan dasbor ter-update.

---

### User Story 4 — Kategori Transaksi (Priority: P2)

Sebagai pengemudi ojol, saya ingin **transaksi dikelompokkan per kategori** (Grab, Gojek, Maxim, Bensin, Makan, Servis, dll.) agar saya tahu ke mana uang saya pergi.

**Why this priority**: Tanpa kategori, data transaksi hanyalah angka tanpa makna. Kategori memberikan insight kepada pengemudi.

**Independent Test**: Buat transaksi di 3 kategori berbeda, lalu verifikasi masing-masing muncul dengan label kategori yang benar.

**Acceptance Scenarios**:

1. **Given** form pencatatan terbuka, **When** user mengetuk field kategori, **Then** daftar kategori muncul dengan ikon visual untuk setiap kategori.
2. **Given** daftar kategori tampil, **Then** minimal tersedia kategori default: Grab, Gojek, Maxim, Shopee Food, Bensin, Makan, Servis Kendaraan, Parkir, Lainnya.
3. **Given** user memilih kategori "Lainnya", **When** form tersimpan, **Then** user bisa menambahkan catatan manual untuk menjelaskan transaksi.
4. **Given** user ingin menambah kategori baru, **When** mengetuk "+ Kategori Baru" di daftar, **Then** user bisa memasukkan nama dan memilih ikon, dan kategori baru tersimpan permanen.

---

### User Story 5 — Manajemen Hutang/Piutang Personal (Priority: P2)

Sebagai pengemudi ojol, saya ingin **mencatat hutang dan piutang personal** (siapa yang saya hutangi, berapa, dan kapan jatuh tempo), agar saya tidak lupa membayar atau menagih.

**Why this priority**: Banyak pengemudi pinjam-meminjam uang antar sesama pengemudi. Fitur ini mencegah konflik dan lupa bayar.

**Independent Test**: Catat 1 hutang, lakukan pembayaran cicilan sebagian, verifikasi sisa hutang berkurang.

**Acceptance Scenarios**:

1. **Given** user membuka menu "Hutang/Piutang", **When** mengetuk "+", **Then** form muncul dengan field: Nama orang, Nominal, Tipe (Hutang/Piutang), Tanggal jatuh tempo (opsional), Catatan.
2. **Given** ada hutang Rp 500.000, **When** user mencatat pembayaran Rp 200.000, **Then** sisa hutang menjadi Rp 300.000 dan riwayat pembayaran tercatat.
3. **Given** ada hutang yang jatuh tempo hari ini, **When** user membuka aplikasi, **Then** notifikasi lokal muncul mengingatkan hutang tersebut.
4. **Given** sisa hutang = Rp 0, **When** user melihat daftar hutang, **Then** item tersebut ditandai "LUNAS" dengan warna hijau.

---

### User Story 6 — Riwayat & Ringkasan Mingguan/Bulanan (Priority: P3)

Sebagai pengemudi ojol, saya ingin **melihat ringkasan keuangan per minggu dan per bulan**, agar saya bisa mengevaluasi penghasilan jangka panjang.

**Why this priority**: Memberikan perspektif jangka panjang. Tidak kritis untuk operasional harian, tapi sangat berguna untuk perencanaan.

**Independent Test**: Masukkan transaksi selama 7 hari berbeda, buka ringkasan mingguan, verifikasi total benar.

**Acceptance Scenarios**:

1. **Given** user membuka halaman "Riwayat", **When** memilih tab "Minggu Ini", **Then** tampil ringkasan: total pemasukan, pengeluaran, nett, dan grafik batang per hari.
2. **Given** user memilih tab "Bulan Ini", **When** data tampil, **Then** ringkasan bulanan muncul dengan grafik batang per minggu.
3. **Given** user mengetuk salah satu batang di grafik, **When** detail muncul, **Then** menampilkan daftar transaksi hari/minggu tersebut.

---

### User Story 7 — Edit Transaksi (Priority: P3)

Sebagai pengemudi ojol, saya ingin **mengedit transaksi yang sudah tercatat** jika saya salah memasukkan nominal atau kategori.

**Why this priority**: Koreksi data penting, tapi tidak se-urgent pencatatan. Pengemudi biasanya mengedit di akhir hari.

**Independent Test**: Buat transaksi, edit nominalnya, verifikasi perubahan tersimpan dan dasbor ter-update.

**Acceptance Scenarios**:

1. **Given** user mengetuk transaksi di daftar, **When** mengetuk tombol "Edit", **Then** form edit muncul dengan data transaksi ter-prefill.
2. **Given** user mengubah nominal dari 50.000 ke 75.000 lalu simpan, **When** kembali ke dasbor, **Then** angka dasbor ter-update sesuai perubahan.
3. **Given** user mengubah kategori transaksi, **When** simpan, **Then** kategori baru tersimpan dan tampil di daftar.

---

### User Story 8 — Cicilan Tetap Bulanan (Priority: P2)

Sebagai pengemudi ojol, saya ingin **mencatat cicilan tetap bulanan** (pinjol, cicilan motor, HP, elektronik) dengan jadwal dan pengingat otomatis, agar saya tidak telat bayar dan kena denda.

Banyak pengemudi memiliki cicilan tetap bulanan (Kredivo, SPayLater, Akulaku, cicilan motor, cicilan HP). Berbeda dari hutang personal (US5), cicilan tetap memiliki nominal tetap per bulan, jadwal berulang, dan konsekuensi denda jika telat.

**Why this priority**: Cicilan tetap adalah beban finansial terbesar banyak pengemudi ojol. Telat bayar 1 hari saja bisa kena denda yang signifikan. Fitur ini terpisah dari US5 karena memiliki data model dan alur yang berbeda.

**Independent Test**: Catat cicilan Kredivo 12 bulan, bayar 1 cicilan, verifikasi sisa berkurang, jumlah cicilan terbayar bertambah, dan jadwal berikutnya tampil benar.

**Acceptance Scenarios**:

1. **Given** user membuka tab "Hutang" dan mengetuk "+", **When** memilih mode "Cicilan Tetap", **Then** form muncul dengan field: Jenis cicilan (Pinjol/Motor/HP/Elektronik/Lainnya), Nama platform/dealer, Total hutang, Cicilan per bulan, Tanggal jatuh tempo bulanan (1-31), Jumlah total cicilan, Sudah dibayar berapa kali, Pengaturan denda (opsional), Catatan.
2. **Given** ada cicilan Kredivo Rp 3.000.000 (12x Rp 250.000, jatuh tempo tgl 15, sudah bayar 3x), **When** user membayar cicilan ke-4, **Then** paid_installments menjadi 4, remaining menjadi Rp 2.000.000, dan cicilan berikutnya (ke-5) ditampilkan dengan tanggal 15 bulan depan.
3. **Given** cicilan jatuh tempo 3 hari lagi, **When** DebtReminderWorker berjalan pukul 08:00 WIB, **Then** notifikasi muncul: "Cicilan [Nama] Rp [nominal] jatuh tempo 3 hari lagi (tgl [X])".
4. **Given** cicilan sudah lewat jatuh tempo dan belum dibayar bulan ini, **When** user membuka detail cicilan, **Then** denda dihitung otomatis sesuai pengaturan: Flat (nominal tetap), Persentase per bulan (% × cicilan), atau Persentase per hari (% × cicilan × jumlah hari telat).
5. **Given** semua cicilan sudah dibayar (paid_installments = total_installments), **When** user melihat daftar hutang, **Then** item ditandai "LUNAS" dengan badge hijau dan remaining_amount = 0.
6. **Given** user membuka detail cicilan, **When** melihat riwayat pembayaran, **Then** semua pembayaran sebelumnya tampil dengan nomor cicilan, tanggal, dan nominal.

---

### Edge Cases

- **Nominal sangat besar**: Apa yang terjadi jika user memasukkan nominal > 999.999.999? → Sistem harus membatasi dan menampilkan error.
- **Tanggal berubah saat mencatat**: Jika user mulai mencatat pukul 23:59 dan menyimpan pukul 00:01, transaksi masuk ke tanggal mana? → Tanggal saat tombol "Simpan" ditekan.
- **Storage penuh**: Bagaimana jika penyimpanan perangkat penuh? → Tampilkan pesan "Penyimpanan penuh" dan sarankan hapus data lama.
- **Reinstall aplikasi**: Data hilang jika user reinstall? → Ya, karena offline-only. (Backup/export bisa jadi fitur masa depan.)
- **Mata uang**: Selalu Rupiah (Rp), tidak perlu multi-currency.
- **Multiple user**: Tidak ada. Satu device = satu pengemudi.
- **Tanggal jatuh tempo cicilan 31**: Jika bulan hanya punya 28-30 hari, gunakan hari terakhir bulan tersebut (e.g., tgl 31 di Februari → tgl 28/29).
- **Denda persentase harian akumulasi**: Denda dihitung dari jumlah hari kalender sejak jatuh tempo, bukan hari kerja.

---

## Requirements

### Functional Requirements

- **FR-001**: Sistem HARUS mampu menyimpan transaksi (pemasukan/pengeluaran) dengan field: nominal, kategori, tipe, timestamp, catatan opsional.
- **FR-002**: Sistem HARUS menampilkan dasbor harian dengan total pemasukan, total pengeluaran, dan pendapatan bersih yang ter-update secara reaktif.
- **FR-003**: Sistem HARUS menampilkan daftar transaksi hari ini, diurutkan dari yang terbaru.
- **FR-004**: Sistem HARUS menyediakan minimal 9 kategori default (Grab, Gojek, Maxim, Shopee Food, Bensin, Makan, Servis Kendaraan, Parkir, Lainnya).
- **FR-005**: Sistem HARUS mendukung CRUD (Create, Read, Update, Delete) untuk transaksi.
- **FR-006**: Sistem HARUS mendukung pencatatan hutang/piutang personal dengan field: nama orang, nominal, tipe (hutang/piutang), tanggal jatuh tempo (opsional), catatan.
- **FR-007**: Sistem HARUS mendukung pembayaran cicilan hutang personal dan menghitung sisa otomatis.
- **FR-008**: Sistem HARUS menampilkan ringkasan mingguan dan bulanan dengan grafik batang.
- **FR-009**: Sistem HARUS beroperasi sepenuhnya offline tanpa koneksi internet.
- **FR-010**: Sistem HARUS memformat semua angka sebagai mata uang Rupiah (Rp) dengan pemisah ribuan titik.
- **FR-011**: Sistem HARUS menampilkan notifikasi lokal untuk hutang yang jatuh tempo.
- **FR-012**: User HARUS bisa menambahkan kategori baru (custom category).
- **FR-013**: Sistem HARUS mendukung pencatatan cicilan tetap bulanan dengan field: jenis cicilan (Pinjol/Motor/HP/Elektronik/Lainnya), nama platform/dealer, total hutang, cicilan per bulan, tanggal jatuh tempo bulanan (1-31), jumlah total cicilan, sudah dibayar berapa kali, catatan.
- **FR-014**: Sistem HARUS mendukung 3 tipe denda keterlambatan cicilan: flat (nominal tetap), persentase per bulan, dan persentase per hari.
- **FR-015**: Sistem HARUS menghitung denda keterlambatan secara otomatis berdasarkan tipe denda dan jumlah hari/bulan keterlambatan.
- **FR-016**: Sistem HARUS menampilkan notifikasi pengingat cicilan 3 hari sebelum jatuh tempo (early reminder) dan pada hari jatuh tempo.

### Key Entities

- **Transaction**: Representasi satu transaksi keuangan. Atribut: id, nominal, tipe (income/expense), kategori, catatan, timestamp. Relasi: belongs to Category.
- **Category**: Kelompok transaksi. Atribut: id, nama, ikon, tipe (income/expense/both), isDefault. Bisa dibuat custom oleh user.
- **Debt**: Representasi satu hutang, piutang, atau cicilan tetap. Atribut: id, nama orang/platform, nominal total, sisa, tipe (hutang/piutang), jenis (personal/installment), kategori cicilan (pinjol/motor/hp/elektronik/lainnya), cicilan per bulan, tanggal jatuh tempo, jumlah total cicilan, sudah bayar berapa kali, tipe denda, besar denda, status (active/paid), catatan.
- **DebtPayment**: Representasi satu pembayaran cicilan. Atribut: id, debt_id, nominal, timestamp, catatan. Relasi: belongs to Debt.

---

## Success Criteria

### Measurable Outcomes

- **SC-001**: User dapat menyelesaikan pencatatan transaksi (buka form → simpan) dalam **kurang dari 3 detik** (3 tap maksimal).
- **SC-002**: Dasbor harian menampilkan data yang benar **100% akurat** sesuai transaksi yang tercatat.
- **SC-003**: Aplikasi berjalan sempurna **tanpa internet** — tidak ada crash, error, atau fitur yang hilang saat offline.
- **SC-004**: Waktu startup aplikasi (cold start) **kurang dari 2 detik** pada perangkat mid-range.
- **SC-005**: Semua operasi database (insert, update, delete) selesai dalam **kurang dari 500ms**.
- **SC-006**: Hutang/piutang personal yang sudah lunas ditandai otomatis dan riwayat cicilan tersimpan lengkap.
- **SC-007**: Cicilan tetap menampilkan jadwal berikutnya secara akurat, paid/total ter-update setelah pembayaran, dan denda keterlambatan dihitung otomatis sesuai konfigurasi (flat/% bulanan/% harian).
