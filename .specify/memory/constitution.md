# Financial Driver Wallet — Constitution

> Dokumen ini adalah **aturan tertinggi** proyek. Semua keputusan teknis, spesifikasi, dan implementasi **harus** mematuhi prinsip-prinsip di bawah ini. Tidak ada pengecualian tanpa amandemen tertulis.

---

## Core Principles

### I. Clean Architecture (3 Layer — NON-NEGOTIABLE)

Setiap fitur **wajib** dipisahkan ke dalam tiga layer yang tidak boleh saling menembus:

| Layer | Isi | Boleh Bergantung Pada |
|:--|:--|:--|
| **Data** | Room Entity, DAO, Repository Implementation | Domain |
| **Domain** | Use Case, Repository Interface, Model | Tidak ada (independent) |
| **Presentation** | ViewModel, UI (Composable), State, Event | Domain |

- **Presentation tidak boleh mengimpor apapun dari Data.** Komunikasi hanya melalui Domain.
- Setiap layer adalah package terpisah: `data/`, `domain/`, `presentation/`.
- Tidak ada "God Class" — satu class = satu tanggung jawab (Single Responsibility).

### II. Modularitas Ketat

- **Maksimal 250 baris per file Kotlin.** Jika melebihi, wajib dipecah.
- **Maksimal 1 public class/interface per file.** Private helper classes diperbolehkan.
- **Satu Composable function per file** untuk UI components utama (layar/screen).
- Nama file harus **persis sama** dengan nama class/interface utama di dalamnya.
  - Contoh: `TransactionRepository.kt` berisi `interface TransactionRepository`.

### III. Konvensi Penamaan (Strict)

| Elemen | Format | Contoh |
|:--|:--|:--|
| Package | `lowercase.dot.separated` | `com.driverwallet.feature.transaction.data` |
| Class/Interface | `PascalCase` | `TransactionRepository`, `AddTransactionUseCase` |
| Function | `camelCase` | `getTransactionById()`, `calculateDailyTotal()` |
| Composable | `PascalCase` (kata benda) | `TransactionCard`, `DashboardScreen` |
| State class | `[Screen]State` | `DashboardState`, `TransactionListState` |
| Event class | `[Screen]Event` | `DashboardEvent`, `TransactionFormEvent` |
| Effect class | `[Screen]Effect` | `DashboardEffect` |
| Room Entity | `[Name]Entity` | `TransactionEntity`, `DebtEntity` |
| Room DAO | `[Name]Dao` | `TransactionDao`, `DebtDao` |
| Database | `AppDatabase` | — |

### IV. MVI Pattern (Model-View-Intent)

Setiap screen **wajib** mengikuti pola ini:

```
User Action → Event → ViewModel → State → UI (Composable)
                         ↓
                      Effect (navigasi, snackbar, dll.)
```

- **State** = data class immutable, merepresentasikan seluruh kondisi layar.
- **Event** = sealed class/interface, merepresentasikan semua aksi pengguna.
- **Effect** = sealed class/interface, merepresentasikan efek samping satu kali (one-time).
- ViewModel **tidak boleh** meng-expose MutableState/MutableFlow ke UI. Hanya `StateFlow<State>` dan `SharedFlow<Effect>`.

### V. Offline-First (NON-NEGOTIABLE)

- **Room Database (SQLite)** adalah satu-satunya sumber data. Tidak ada backend/API.
- Semua operasi data harus berjalan **tanpa koneksi internet**.
- Tidak boleh ada dependency yang membutuhkan internet untuk berfungsi.
- Data migration strategy harus didefinisikan untuk setiap perubahan skema.

### VI. Testing Standards

- Setiap Use Case di Domain layer **wajib** memiliki unit test.
- Setiap DAO **wajib** memiliki instrumented test.
- Setiap ViewModel **wajib** memiliki unit test untuk semua Event → State transitions.
- Test harus ditulis **sebelum atau bersamaan** dengan implementasi (tidak boleh ditunda ke "nanti").
- Minimum coverage target: **80%** untuk Domain layer.

### VII. Simplicity & YAGNI

- **Jangan abstraksi yang belum dibutuhkan.** Mulai sederhana, refactor ketika memang perlu.
- **Jangan tambah library yang bisa diselesaikan dengan API bawaan.** Setiap dependency baru harus dijustifikasi.
- Jika ada dua solusi yang sama-sama benar, **pilih yang lebih sederhana.**
- **Jangan bungkus framework** (Compose, Room) dengan abstraksi buatan sendiri kecuali benar-benar diperlukan.

---

## Anti-Hallucination Protocol (Khusus AI Agent)

Aturan ini **wajib** diikuti oleh setiap AI agent yang mengerjakan proyek ini:

1. **Jangan menebak.** Jika ada bagian spesifikasi yang ambigu, tandai dengan `[NEEDS CLARIFICATION]` dan berhenti.
2. **Jangan menambahkan fitur yang tidak diminta.** Ikuti spesifikasi secara literal.
3. **Jika gagal memperbaiki bug yang sama 3 kali**, berhenti dan laporkan ke user dengan deskripsi jelas tentang apa yang sudah dicoba.
4. **Setiap file yang diubah harus disebutkan secara eksplisit** dalam commit message atau PR description.
5. **Jangan pindah ke tugas berikutnya** sebelum tugas saat ini melewati semua acceptance criteria.
6. **Referensikan selalu** constitution.md dan spec.md sebelum membuat keputusan arsitektur.

---

## Tech Stack (Locked)

| Komponen | Teknologi | Versi Minimum |
|:--|:--|:--|
| Language | Kotlin | 2.3.x |
| UI Framework | Jetpack Compose + Material 3 | Latest stable |
| Architecture | MVI + Clean Architecture (3 Layer) | — |
| Database | Room (SQLite) | Latest stable |
| Navigation | Navigation 3 (Compose) | Latest stable |
| DI | Hilt / Koin | Latest stable |
| Async | Kotlin Coroutines + Flow | Latest stable |
| Build | Gradle (Kotlin DSL) | Latest stable |
| Min SDK | 26 (Android 8.0) | — |
| Target SDK | 35 (Android 15) | — |

> Tech stack **tidak boleh diubah** tanpa amandemen konstitusi.

---

## Governance

- **Konstitusi ini mengesampingkan** semua praktik lain jika terjadi konflik.
- **Amandemen** memerlukan: dokumentasi perubahan, justifikasi, dan persetujuan tertulis dari user.
- Semua PR/review **harus memverifikasi kepatuhan** terhadap konstitusi sebelum merge.
- Kompleksitas di atas standar **harus dijustifikasi** secara tertulis.

**Version**: 1.0.0 | **Ratified**: 2026-02-24 | **Last Amended**: 2026-02-24
