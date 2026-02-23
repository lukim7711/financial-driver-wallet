# Tasks: Driver Wallet

**Input**: Design documents from `/specs/001-driver-wallet/`
**Prerequisites**: plan.md ✅, spec.md ✅, data-model.md ✅

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story (US1–US7)
- Paths relative to `app/src/main/java/com/driverwallet/`

---

## Phase 1: Setup (Project Initialization)

**Purpose**: Android project scaffolding, Gradle config, Hilt setup

- [ ] T001 Initialize Android project with Kotlin 2.3.x, set applicationId `com.driverwallet`, minSdk 26, targetSdk 35
- [ ] T002 Configure `build.gradle.kts` (app) with all dependencies: Compose BOM, Material 3, Hilt, Room, Navigation 3, Vico, WorkManager, Coroutines, JUnit 5, Turbine, MockK
- [ ] T003 [P] Configure `build.gradle.kts` (project) with Hilt plugin, KSP plugin, Compose compiler
- [ ] T004 [P] Create `DriverWalletApp.kt` — Application class annotated `@HiltAndroidApp`
- [ ] T005 [P] Create `MainActivity.kt` — Single Activity annotated `@AndroidEntryPoint`, sets Compose content

**Checkpoint**: Project compiles and runs showing blank screen ✅

---

## Phase 2: Foundation (Shared Infrastructure)

**Purpose**: Database, theme, utilities, DI modules — BLOCKS all user stories

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Database & DI

- [ ] T006 Create `CategoryEntity.kt` in `feature/category/data/local/` — Room entity per data-model.md
- [ ] T007 [P] Create `TransactionEntity.kt` in `feature/transaction/data/local/` — Room entity per data-model.md
- [ ] T008 [P] Create `DebtEntity.kt` in `feature/debt/data/local/` — Room entity per data-model.md
- [ ] T009 [P] Create `DebtPaymentEntity.kt` in `feature/debt/data/local/` — Room entity per data-model.md
- [ ] T010 Create `CategoryDao.kt` in `feature/category/data/local/` — DAO with CRUD + getDefaults query
- [ ] T011 [P] Create `TransactionDao.kt` in `feature/transaction/data/local/` — DAO with all queries from data-model.md
- [ ] T012 [P] Create `DebtDao.kt` in `feature/debt/data/local/` — DAO with active debts, due today, update remaining
- [ ] T013 [P] Create `DebtPaymentDao.kt` in `feature/debt/data/local/` — DAO with insert + getByDebtId
- [ ] T014 Create `AppDatabase.kt` in `core/database/` — Room database with all 4 entities, version 1, prepopulate 9 default categories via Callback
- [ ] T015 Create `DatabaseModule.kt` in `di/` — Hilt module providing AppDatabase and all 4 DAOs
- [ ] T016 Create `RepositoryModule.kt` in `di/` — Hilt module binding all Repository interfaces to implementations

### Domain Models

- [ ] T017 [P] Create `Category.kt` in `feature/category/domain/model/` — domain model (data class, no Room annotations)
- [ ] T018 [P] Create `Transaction.kt` in `feature/transaction/domain/model/` — domain model with formatted amount
- [ ] T019 [P] Create `Debt.kt` in `feature/debt/domain/model/` — domain model with computed properties
- [ ] T020 [P] Create `DebtPayment.kt` in `feature/debt/domain/model/` — domain model
- [ ] T021 [P] Create `PeriodSummary.kt` in `feature/history/domain/model/` — domain model for weekly/monthly chart data

### Core Utilities

- [ ] T022 [P] Create `CurrencyFormatter.kt` in `core/util/` — format Long to "Rp 50.000" with dot separator
- [ ] T023 [P] Create `DateUtils.kt` in `core/util/` — startOfDay(), endOfDay(), startOfWeek(), startOfMonth() helpers

### Theme

- [ ] T024 [P] Create `Color.kt` in `core/ui/theme/` — Green for income, Red for expense, Material 3 palette
- [ ] T025 [P] Create `Type.kt` in `core/ui/theme/` — Typography definitions
- [ ] T026 Create `Theme.kt` in `core/ui/theme/` — DriverWalletTheme with dynamic color support

**Checkpoint**: Database seeded with 9 categories, all Hilt modules compile, theme applied ✅

---

## Phase 3: User Story 1+2+3 — Dashboard + Pencatatan + Daftar Transaksi (Priority: P1) 🎯 MVP

**Goal**: User bisa mencatat transaksi < 3 detik, melihat dasbor harian, dan daftar transaksi hari ini

**Independent Test**: Buka app → lihat dasbor Rp 0 → tap "+" → pilih kategori → masukkan nominal → simpan → dasbor ter-update → transaksi muncul di daftar

### Repository Layer (Data)

- [ ] T027 Create `CategoryRepository.kt` (interface) in `feature/category/domain/repository/`
- [ ] T028 [P] Create `CategoryRepositoryImpl.kt` in `feature/category/data/repository/` — implements interface, maps Entity↔Domain
- [ ] T029 Create `TransactionRepository.kt` (interface) in `feature/transaction/domain/repository/`
- [ ] T030 [P] Create `TransactionRepositoryImpl.kt` in `feature/transaction/data/repository/` — implements interface, maps Entity↔Domain

### Use Cases (Domain)

- [ ] T031 [P] [US1] Create `AddTransactionUseCase.kt` in `feature/transaction/domain/usecase/` — validates amount > 0, delegates to repository
- [ ] T032 [P] [US2] Create `GetDailySummaryUseCase.kt` in `feature/transaction/domain/usecase/` — returns Flow<DailySummary> with income, expense, nett
- [ ] T033 [P] [US3] Create `GetDailyTransactionsUseCase.kt` in `feature/transaction/domain/usecase/` — returns Flow<List<Transaction>> ordered by newest
- [ ] T034 [P] [US3] Create `DeleteTransactionUseCase.kt` in `feature/transaction/domain/usecase/`
- [ ] T035 [P] [US4] Create `GetCategoriesUseCase.kt` in `feature/category/domain/usecase/` — returns Flow<List<Category>>

### Presentation — Dashboard (US2 + US3)

- [ ] T036 Create `DashboardState.kt` in `feature/transaction/presentation/dashboard/` — data class with dailyIncome, dailyExpense, nettIncome, transactions list, isLoading
- [ ] T037 [P] Create `DashboardEvent.kt` in `feature/transaction/presentation/dashboard/` — sealed interface: LoadToday, DeleteTransaction(id)
- [ ] T038 [P] Create `DashboardEffect.kt` in `feature/transaction/presentation/dashboard/` — sealed interface: NavigateToForm, NavigateToDetail(id), ShowDeleteConfirmation(id)
- [ ] T039 Create `DashboardViewModel.kt` in `feature/transaction/presentation/dashboard/` — @HiltViewModel, collect daily summary + transactions, handle events
- [ ] T040 Create `SummaryCard.kt` in `feature/transaction/presentation/components/` — Composable showing income (green), expense (red), nett
- [ ] T041 [P] Create `TransactionCard.kt` in `feature/transaction/presentation/components/` — Composable showing category icon, name, amount, time, swipe-to-delete
- [ ] T042 Create `DashboardScreen.kt` in `feature/transaction/presentation/dashboard/` — Composable with SummaryCard + LazyColumn of TransactionCards + FAB "+"

### Presentation — Transaction Form (US1)

- [ ] T043 Create `TransactionFormState.kt` in `feature/transaction/presentation/form/` — data class with amount, selectedCategory, type, note, categories list, error
- [ ] T044 [P] Create `TransactionFormEvent.kt` in `feature/transaction/presentation/form/` — sealed interface: SetAmount, SelectCategory, SetType, SetNote, Save
- [ ] T045 [P] Create `TransactionFormEffect.kt` in `feature/transaction/presentation/form/` — sealed interface: NavigateBack, ShowError(msg)
- [ ] T046 Create `CategorySelector.kt` in `feature/transaction/presentation/components/` — Composable grid of category chips with icons
- [ ] T047 Create `TransactionFormViewModel.kt` in `feature/transaction/presentation/form/` — @HiltViewModel, validate + save via AddTransactionUseCase
- [ ] T048 Create `TransactionFormScreen.kt` in `feature/transaction/presentation/form/` — Composable with numeric keyboard auto-open, category selector, type toggle, save button

### Navigation

- [ ] T049 Create `AppNavigation.kt` in `core/navigation/` — Navigation 3 setup with routes: Dashboard, TransactionForm, TransactionDetail, DebtList, DebtForm, History
- [ ] T050 Update `MainActivity.kt` — wire AppNavigation + DriverWalletTheme

### Tests (P1)

- [ ] T051 [P] [US1] Unit test `AddTransactionUseCaseTest.kt` — test validation (amount 0 fails, valid saves)
- [ ] T052 [P] [US2] Unit test `GetDailySummaryUseCaseTest.kt` — test correct income/expense/nett calculation
- [ ] T053 [P] [US3] Unit test `GetDailyTransactionsUseCaseTest.kt` — test ordering, date filtering
- [ ] T054 [P] [US2] Unit test `DashboardViewModelTest.kt` — test state changes on events using Turbine
- [ ] T055 [P] [US1] Unit test `TransactionFormViewModelTest.kt` — test validation, save success/failure
- [ ] T056 Instrumented test `TransactionDaoTest.kt` — test insert, daily query, delete with real Room DB
- [ ] T057 [P] Instrumented test `CategoryDaoTest.kt` — test default seed, custom insert

**Checkpoint**: MVP complete — user can record transactions < 3 seconds, dashboard shows daily summary, list shows today's transactions ✅

---

## Phase 4: User Story 4 — Kategori Transaksi (Priority: P2)

**Goal**: User bisa melihat kategori dengan ikon visual dan menambah kategori custom

**Independent Test**: Buka form transaksi → lihat 9 kategori default dengan ikon → tap "+ Kategori Baru" → masukkan nama → simpan → kategori baru muncul

- [ ] T058 [US4] Create `AddCategoryUseCase.kt` in `feature/category/domain/usecase/` — validates unique name, delegates to repository
- [ ] T059 [US4] Update `CategorySelector.kt` — add "+ Kategori Baru" button at end of grid
- [ ] T060 [US4] Create `AddCategoryDialog.kt` in `feature/transaction/presentation/components/` — dialog with name field + icon picker
- [ ] T061 [P] [US4] Unit test `AddCategoryUseCaseTest.kt` — test duplicate name rejection, successful add

**Checkpoint**: Categories fully functional with custom add ✅

---

## Phase 5: User Story 5 — Manajemen Hutang/Piutang (Priority: P2)

**Goal**: User bisa mencatat hutang/piutang, bayar cicilan, lihat sisa, dan dapat reminder

**Independent Test**: Buka tab Hutang → tap "+" → isi form → simpan → bayar cicilan Rp 200.000 → sisa berkurang → tandai lunas jika sisa = 0

### Repository & Use Cases

- [ ] T062 Create `DebtRepository.kt` (interface) in `feature/debt/domain/repository/`
- [ ] T063 Create `DebtRepositoryImpl.kt` in `feature/debt/data/repository/` — implements interface, handles payment + remaining calculation
- [ ] T064 [P] [US5] Create `AddDebtUseCase.kt` in `feature/debt/domain/usecase/`
- [ ] T065 [P] [US5] Create `AddDebtPaymentUseCase.kt` in `feature/debt/domain/usecase/` — validates payment ≤ remaining, updates debt status
- [ ] T066 [P] [US5] Create `GetActiveDebtsUseCase.kt` in `feature/debt/domain/usecase/`
- [ ] T067 [P] [US5] Create `GetDebtDetailUseCase.kt` in `feature/debt/domain/usecase/` — returns debt + payment history

### Presentation — Debt List

- [ ] T068 Create `DebtListState.kt` in `feature/debt/presentation/list/`
- [ ] T069 [P] Create `DebtListEvent.kt` in `feature/debt/presentation/list/`
- [ ] T070 [P] Create `DebtListEffect.kt` in `feature/debt/presentation/list/`
- [ ] T071 Create `DebtCard.kt` in `feature/debt/presentation/components/` — shows person, amount, remaining, status badge (LUNAS = green)
- [ ] T072 Create `DebtListViewModel.kt` in `feature/debt/presentation/list/`
- [ ] T073 Create `DebtListScreen.kt` in `feature/debt/presentation/list/` — LazyColumn of DebtCards + FAB

### Presentation — Debt Form

- [ ] T074 Create `DebtFormState.kt` in `feature/debt/presentation/form/`
- [ ] T075 [P] Create `DebtFormEvent.kt` in `feature/debt/presentation/form/`
- [ ] T076 [P] Create `DebtFormEffect.kt` in `feature/debt/presentation/form/`
- [ ] T077 Create `PaymentHistoryItem.kt` in `feature/debt/presentation/components/` — shows payment amount + date
- [ ] T078 Create `DebtFormViewModel.kt` in `feature/debt/presentation/form/` — handles add debt + add payment
- [ ] T079 Create `DebtFormScreen.kt` in `feature/debt/presentation/form/` — form fields + payment history list + "Bayar" button

### Notification

- [ ] T080 Create `DebtReminderWorker.kt` in `core/notification/` — WorkManager worker that queries debts due today, shows local notification
- [ ] T081 Schedule daily WorkManager in `DriverWalletApp.kt` — PeriodicWorkRequest every 24h, check due debts

### Tests

- [ ] T082 [P] [US5] Unit test `AddDebtPaymentUseCaseTest.kt` — test overpayment rejection, auto-PAID status
- [ ] T083 [P] [US5] Unit test `DebtListViewModelTest.kt` — test active/paid filtering
- [ ] T084 Instrumented test `DebtDaoTest.kt` — test insert, payment update, remaining calculation

**Checkpoint**: Hutang/piutang fully functional with cicilan and notification ✅

---

## Phase 6: User Story 6 — Riwayat & Ringkasan (Priority: P3)

**Goal**: User bisa lihat ringkasan keuangan mingguan dan bulanan dengan grafik batang

**Independent Test**: Masukkan transaksi 7 hari → buka tab Riwayat → lihat grafik mingguan → tap batang → lihat detail hari itu

### Repository & Use Cases

- [ ] T085 Create `HistoryRepository.kt` (interface) in `feature/history/domain/repository/`
- [ ] T086 Create `HistoryRepositoryImpl.kt` in `feature/history/data/repository/` — aggregates transactions by period
- [ ] T087 [P] [US6] Create `GetWeeklySummaryUseCase.kt` in `feature/history/domain/usecase/`
- [ ] T088 [P] [US6] Create `GetMonthlySummaryUseCase.kt` in `feature/history/domain/usecase/`

### Presentation

- [ ] T089 Create `HistoryState.kt` in `feature/history/presentation/`
- [ ] T090 [P] Create `HistoryEvent.kt` in `feature/history/presentation/`
- [ ] T091 [P] Create `HistoryEffect.kt` in `feature/history/presentation/`
- [ ] T092 Create `PeriodChart.kt` in `feature/history/presentation/components/` — Vico bar chart (green income, red expense)
- [ ] T093 Create `HistoryViewModel.kt` in `feature/history/presentation/`
- [ ] T094 Create `HistoryScreen.kt` in `feature/history/presentation/` — Tabs (Minggu/Bulan) + PeriodChart + transaction list on tap

### Tests

- [ ] T095 [P] [US6] Unit test `GetWeeklySummaryUseCaseTest.kt`
- [ ] T096 [P] [US6] Unit test `HistoryViewModelTest.kt`

**Checkpoint**: Riwayat mingguan & bulanan with charts functional ✅

---

## Phase 7: User Story 7 — Edit Transaksi + Detail (Priority: P3)

**Goal**: User bisa lihat detail transaksi dan mengedit nominal/kategori

**Independent Test**: Tap transaksi di daftar → lihat detail → tap Edit → ubah nominal → simpan → dasbor ter-update

- [ ] T097 [US7] Create `UpdateTransactionUseCase.kt` in `feature/transaction/domain/usecase/`
- [ ] T098 [US7] Create `TransactionDetailScreen.kt` in `feature/transaction/presentation/detail/` — detail view + Edit button
- [ ] T099 [US7] Create `TransactionDetailViewModel.kt` in `feature/transaction/presentation/detail/`
- [ ] T100 [US7] Update `TransactionFormScreen.kt` — support edit mode (prefill existing data via transaction ID)
- [ ] T101 [US7] Update `TransactionFormViewModel.kt` — handle both add (new) and update (existing) flows
- [ ] T102 [P] [US7] Unit test `UpdateTransactionUseCaseTest.kt`
- [ ] T103 [P] [US7] Unit test `TransactionDetailViewModelTest.kt`

**Checkpoint**: Full CRUD for transactions complete ✅

---

## Phase 8: Polish & Cross-Cutting

**Purpose**: Final integration, navigation wiring, edge cases

- [ ] T104 Wire bottom navigation in `AppNavigation.kt` — 3 tabs: Dashboard, Hutang, Riwayat
- [ ] T105 [P] Add swipe-to-delete with confirmation dialog in `DashboardScreen.kt` (US3 edge case)
- [ ] T106 [P] Add input validation: max amount 999.999.999 in `TransactionFormViewModel.kt` (edge case)
- [ ] T107 [P] Add empty state illustrations for Dashboard, Debt List, History
- [ ] T108 [P] Add loading shimmer/skeleton for all screens
- [ ] T109 Run full app smoke test: complete all acceptance scenarios from spec.md
- [ ] T110 Run quickstart validation: cold start < 2s, transaction save < 500ms

**Checkpoint**: Production-ready app ✅

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1 (Setup)           → no dependencies
Phase 2 (Foundation)      → depends on Phase 1 — BLOCKS all user stories
Phase 3 (US1+US2+US3 P1)  → depends on Phase 2
Phase 4 (US4 P2)          → depends on Phase 3 (uses CategorySelector from Phase 3)
Phase 5 (US5 P2)          → depends on Phase 2 only (independent from transactions)
Phase 6 (US6 P3)          → depends on Phase 3 (needs transaction data)
Phase 7 (US7 P3)          → depends on Phase 3 (extends transaction feature)
Phase 8 (Polish)          → depends on all phases complete
```

### Parallel Opportunities

- **Phase 4 + Phase 5** can run in parallel (different features, no file conflicts)
- **Phase 6 + Phase 7** can run in parallel (different screens, no file conflicts)
- Within each phase, all tasks marked `[P]` can run in parallel

### Implementation Strategy (Recommended for AI Agent)

```
Phase 1 → Phase 2 → Phase 3 (MVP ✅)
                         ↓
               Phase 4 + Phase 5 (parallel)
                         ↓
               Phase 6 + Phase 7 (parallel)
                         ↓
                     Phase 8 (polish)
```

Total: **110 tasks** across **8 phases**
