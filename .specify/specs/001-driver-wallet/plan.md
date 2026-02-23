# Implementation Plan: Driver Wallet

**Branch**: `001-driver-wallet` | **Date**: 2026-02-24 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-driver-wallet/spec.md`

---

## Summary

Aplikasi Android pencatat keuangan offline-first untuk pengemudi ojek daring. Menggunakan Kotlin + Jetpack Compose dengan arsitektur MVI + Clean Architecture 3 layer. Seluruh data disimpan di Room Database (SQLite) tanpa backend/API. Target utama: pencatatan transaksi dalam < 3 detik.

---

## Technical Context

**Language/Version**: Kotlin 2.3.x
**Primary Dependencies**: Jetpack Compose (Material 3), Hilt, Room, Navigation 3, Vico (chart), WorkManager
**Storage**: Room Database (SQLite) — offline-only, no backend
**Testing**: JUnit 5 (unit), Turbine (Flow testing), MockK (mocking), Compose UI Test (instrumented)
**Target Platform**: Android 8.0+ (Min SDK 26), Target SDK 35
**Project Type**: Mobile App (Android native)
**Performance Goals**: Cold start < 2s, transaction save < 500ms, form open < 300ms
**Constraints**: Fully offline, no internet dependency, single user per device
**Scale/Scope**: 6 screens, 4 database entities, ~30-40 Kotlin files

---

## Constitution Check

*GATE: Must pass before implementation.*

- [x] Clean Architecture 3 Layer (Data / Domain / Presentation) — enforced in project structure
- [x] Max 250 lines per file — enforced by code review
- [x] 1 public class/interface per file — enforced by naming convention
- [x] MVI pattern (State, Event, Effect) for every screen
- [x] Offline-first: Room only, no network dependencies
- [x] Anti-Hallucination Protocol applied: AI must reference constitution + spec before decisions
- [x] Testing: UseCase, DAO, ViewModel must have tests
- [x] YAGNI: No unnecessary abstractions

---

## Dependencies (Locked)

| Library | Purpose | Justification |
|:--|:--|:--|
| **Jetpack Compose BOM** | UI framework | Standar Google untuk Android modern UI |
| **Material 3** | Design system | Konsisten, accessible, dynamic color |
| **Hilt** | Dependency Injection | Compile-time safety, integrasi resmi Jetpack |
| **Room** | Database (SQLite) | Offline-first, type-safe, Kotlin coroutines support |
| **Navigation 3** | Navigasi antar screen | Standar Compose navigation terbaru |
| **Vico** | Grafik batang (chart) | Compose-native, ringan, well-maintained |
| **WorkManager** | Scheduled notification | Reliable background scheduling untuk reminder hutang |
| **Kotlin Coroutines + Flow** | Async & reactive | Standar Kotlin async, mendukung Room reactive queries |
| **JUnit 5** | Unit testing | Standar testing modern |
| **Turbine** | Flow testing | Testing library khusus Kotlin Flow |
| **MockK** | Mocking | Kotlin-native mocking library |

> Tidak boleh ada library tambahan tanpa justifikasi tertulis (Constitution VII: YAGNI).

---

## Screen Map (6 Screens)

```
┌─────────────────────────────────────────────────────┐
│                    APP START                         │
│                       │                              │
│                       ▼                              │
│              ┌─────────────────┐                     │
│              │  DashboardScreen │ ← Home (P1)        │
│              │  - Summary card  │                     │
│              │  - Transaction   │                     │
│              │    list today    │                     │
│              └───────┬─────────┘                     │
│                      │                               │
│         ┌────────────┼────────────┐                  │
│         ▼            ▼            ▼                  │
│  ┌─────────────┐ ┌────────┐ ┌──────────┐            │
│  │ Transaction  │ │  Debt  │ │ History  │            │
│  │ FormScreen   │ │ List   │ │ Screen   │            │
│  │ (Add/Edit)   │ │ Screen │ │ (Weekly/ │            │
│  │     (P1)     │ │  (P2)  │ │ Monthly) │            │
│  └──────┬──────┘ └───┬────┘ │   (P3)   │            │
│         │            │      └──────────┘             │
│         ▼            ▼                               │
│  ┌─────────────┐ ┌──────────┐                        │
│  │ Transaction  │ │  Debt    │                        │
│  │ Detail       │ │  Form    │                        │
│  │ Screen (P1)  │ │  Screen  │                        │
│  └─────────────┘ │  (P2)    │                        │
│                   └──────────┘                        │
└─────────────────────────────────────────────────────┘
```

Bottom Navigation: **Dashboard** | **Hutang** | **Riwayat**

---

## Project Structure (Source Code)

```text
app/
├── src/main/java/com/driverwallet/
│
│   ├── di/                              # Hilt modules
│   │   ├── DatabaseModule.kt
│   │   └── RepositoryModule.kt
│   │
│   ├── feature/
│   │   ├── transaction/                 # Feature: Transaksi
│   │   │   ├── data/
│   │   │   │   ├── local/
│   │   │   │   │   ├── TransactionDao.kt
│   │   │   │   │   └── TransactionEntity.kt
│   │   │   │   └── repository/
│   │   │   │       └── TransactionRepositoryImpl.kt
│   │   │   ├── domain/
│   │   │   │   ├── model/
│   │   │   │   │   └── Transaction.kt
│   │   │   │   ├── repository/
│   │   │   │   │   └── TransactionRepository.kt
│   │   │   │   └── usecase/
│   │   │   │       ├── AddTransactionUseCase.kt
│   │   │   │       ├── DeleteTransactionUseCase.kt
│   │   │   │       ├── GetDailySummaryUseCase.kt
│   │   │   │       ├── GetDailyTransactionsUseCase.kt
│   │   │   │       └── UpdateTransactionUseCase.kt
│   │   │   └── presentation/
│   │   │       ├── dashboard/
│   │   │       │   ├── DashboardScreen.kt
│   │   │       │   ├── DashboardState.kt
│   │   │       │   ├── DashboardEvent.kt
│   │   │       │   ├── DashboardEffect.kt
│   │   │       │   └── DashboardViewModel.kt
│   │   │       ├── form/
│   │   │       │   ├── TransactionFormScreen.kt
│   │   │       │   ├── TransactionFormState.kt
│   │   │       │   ├── TransactionFormEvent.kt
│   │   │       │   ├── TransactionFormEffect.kt
│   │   │       │   └── TransactionFormViewModel.kt
│   │   │       ├── detail/
│   │   │       │   ├── TransactionDetailScreen.kt
│   │   │       │   └── TransactionDetailViewModel.kt
│   │   │       └── components/
│   │   │           ├── TransactionCard.kt
│   │   │           ├── SummaryCard.kt
│   │   │           └── CategorySelector.kt
│   │   │
│   │   ├── debt/                        # Feature: Hutang/Piutang
│   │   │   ├── data/
│   │   │   │   ├── local/
│   │   │   │   │   ├── DebtDao.kt
│   │   │   │   │   ├── DebtEntity.kt
│   │   │   │   │   ├── DebtPaymentDao.kt
│   │   │   │   │   └── DebtPaymentEntity.kt
│   │   │   │   └── repository/
│   │   │   │       └── DebtRepositoryImpl.kt
│   │   │   ├── domain/
│   │   │   │   ├── model/
│   │   │   │   │   ├── Debt.kt
│   │   │   │   │   └── DebtPayment.kt
│   │   │   │   ├── repository/
│   │   │   │   │   └── DebtRepository.kt
│   │   │   │   └── usecase/
│   │   │   │       ├── AddDebtUseCase.kt
│   │   │   │       ├── AddDebtPaymentUseCase.kt
│   │   │   │       ├── GetActiveDebtsUseCase.kt
│   │   │   │       └── GetDebtDetailUseCase.kt
│   │   │   └── presentation/
│   │   │       ├── list/
│   │   │       │   ├── DebtListScreen.kt
│   │   │       │   ├── DebtListState.kt
│   │   │       │   ├── DebtListEvent.kt
│   │   │       │   ├── DebtListEffect.kt
│   │   │       │   └── DebtListViewModel.kt
│   │   │       ├── form/
│   │   │       │   ├── DebtFormScreen.kt
│   │   │       │   ├── DebtFormState.kt
│   │   │       │   ├── DebtFormEvent.kt
│   │   │       │   ├── DebtFormEffect.kt
│   │   │       │   └── DebtFormViewModel.kt
│   │   │       └── components/
│   │   │           ├── DebtCard.kt
│   │   │           └── PaymentHistoryItem.kt
│   │   │
│   │   ├── history/                     # Feature: Riwayat
│   │   │   ├── data/
│   │   │   │   └── repository/
│   │   │   │       └── HistoryRepositoryImpl.kt
│   │   │   ├── domain/
│   │   │   │   ├── model/
│   │   │   │   │   └── PeriodSummary.kt
│   │   │   │   ├── repository/
│   │   │   │   │   └── HistoryRepository.kt
│   │   │   │   └── usecase/
│   │   │   │       ├── GetWeeklySummaryUseCase.kt
│   │   │   │       └── GetMonthlySummaryUseCase.kt
│   │   │   └── presentation/
│   │   │       ├── HistoryScreen.kt
│   │   │       ├── HistoryState.kt
│   │   │       ├── HistoryEvent.kt
│   │   │       ├── HistoryEffect.kt
│   │   │       ├── HistoryViewModel.kt
│   │   │       └── components/
│   │   │           └── PeriodChart.kt
│   │   │
│   │   └── category/                    # Feature: Kategori
│   │       ├── data/
│   │       │   ├── local/
│   │       │   │   ├── CategoryDao.kt
│   │       │   │   └── CategoryEntity.kt
│   │       │   └── repository/
│   │       │       └── CategoryRepositoryImpl.kt
│   │       └── domain/
│   │           ├── model/
│   │           │   └── Category.kt
│   │           ├── repository/
│   │           │   └── CategoryRepository.kt
│   │           └── usecase/
│   │               ├── GetCategoriesUseCase.kt
│   │               └── AddCategoryUseCase.kt
│   │
│   ├── core/                            # Shared utilities
│   │   ├── database/
│   │   │   └── AppDatabase.kt
│   │   ├── navigation/
│   │   │   └── AppNavigation.kt
│   │   ├── notification/
│   │   │   └── DebtReminderWorker.kt
│   │   ├── ui/
│   │   │   └── theme/
│   │   │       ├── Color.kt
│   │   │       ├── Theme.kt
│   │   │       └── Type.kt
│   │   └── util/
│   │       ├── CurrencyFormatter.kt
│   │       └── DateUtils.kt
│   │
│   └── DriverWalletApp.kt              # Application class (@HiltAndroidApp)
│
├── src/test/                            # Unit tests
│   └── java/com/driverwallet/
│       ├── feature/transaction/domain/usecase/
│       ├── feature/debt/domain/usecase/
│       ├── feature/history/domain/usecase/
│       └── feature/*/presentation/      # ViewModel tests
│
└── src/androidTest/                     # Instrumented tests
    └── java/com/driverwallet/
        ├── feature/transaction/data/local/
        ├── feature/debt/data/local/
        └── feature/category/data/local/
```

---

## Complexity Tracking

> No constitution violations detected. All decisions follow YAGNI and simplicity principles.

| Decision | Justification |
|:--|:--|
| Vico for charts | Compose-native, no heavy dependency. Rejected MPAndroidChart (View-based, not Compose). |
| WorkManager for notifications | More reliable than AlarmManager for scheduled tasks. Battery-friendly. |
| Feature-based packaging | Aligns with Clean Architecture per feature. Each feature is independently modifiable. |
| No Repository pattern abstraction beyond interface | Constitution VII: Don't wrap framework. Room DAO is already an abstraction. Repository interface exists only for testability. |
