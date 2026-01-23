## 1. What is a Queue?

A **queue** lets your app **delay or offload slow tasks** to run **in the background**, instead of making users wait.

👉 In simple terms:
**“Do this later, not right now.”**

Example without queue:

* User clicks “Register”
* App sends email (slow)
* User waits… 😐

Example with queue:

* User clicks “Register”
* App responds instantly ✅
* Email is sent in background 📨

---

## 2. Why use a Queue?

Main reasons:

1. 🚀 **Faster response time**
2. 🧠 **Better user experience**
3. 🔥 **Handle heavy tasks safely**
4. 📈 **Scales better**
5. 🛠 **Retry failed jobs automatically**

---

## 3. 10 Real-World Use Cases (Very Common)

Here are **real production cases** 👇

1. **Send emails** (welcome, invoice, reset password)
2. **Generate PDF invoices**
3. **Process uploaded images** (resize, compress)
4. **Send SMS / WhatsApp notifications**
5. **Export Excel / CSV reports**
6. **Sync data with third-party APIs**
7. **Webhook handling**
8. **Background payment verification**
9. **Push notifications**
10. **Bulk database operations**

💡 Rule of thumb:
If it takes **> 1 second**, queue it.

---

## 4. How Queue Works in Laravel (Concept)

```
Request → Dispatch Job → Queue → Worker → Execute Job
```

* **Job** = what to do
* **Queue** = waiting line
* **Worker** = who does the work

---

## 5. How to Write Queue in Laravel (Step by Step)

### Step 1: Set Queue Driver

In `.env`:

```env
QUEUE_CONNECTION=database
```

---

### Step 2: Create Job

```bash
php artisan make:job SendWelcomeEmail
```

---

### Step 3: Job Code Example

```php
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        public string $email
    ) {}

    public function handle()
    {
        Mail::to($this->email)->send(new WelcomeMail());
    }
}
```

⚠️ `ShouldQueue` = this makes it queued

---

### Step 4: Dispatch Job

```php
SendWelcomeEmail::dispatch($user->email);
```

OR with delay:

```php
SendWelcomeEmail::dispatch($user->email)
    ->delay(now()->addMinutes(5));
```

---

### Step 5: Create Queue Table (Database Driver)

```bash
php artisan queue:table
php artisan migrate
```

---

### Step 6: Run Worker

```bash
php artisan queue:work
```

✅ Now your job runs in background.

---

## 6. Available Queue Methods (Important Ones)

### Dispatching

```php
Job::dispatch();
Job::dispatchSync();   // run immediately (no queue)
Job::dispatchAfterResponse(); // after HTTP response
```

---

### Delay

```php
Job::dispatch()->delay(now()->addSeconds(10));
```

---

### Retry & Timeout

```php
public $tries = 3;
public $timeout = 120;
```

---

### Failed Job Handling

```php
public function failed(Throwable $e)
{
    Log::error($e->getMessage());
}
```

---

### Queue Name

```php
public $queue = 'emails';
```

Run specific queue:

```bash
php artisan queue:work --queue=emails
```

---

## 7. Common Queue Commands

```bash
php artisan queue:work
php artisan queue:listen
php artisan queue:restart
php artisan queue:failed
php artisan queue:retry all
php artisan queue:flush
```

---

## 8. Example: Queue Email on User Register

```php
public function register(Request $request)
{
    $user = User::create([...]);

    SendWelcomeEmail::dispatch($user->email);

    return response()->json([
        'message' => 'User registered'
    ]);
}
```

⚡ Fast API, email later.

---

## 9. Best Practices (From Real Projects)

✔ Don’t put heavy logic in controllers
✔ Always use queue for emails
✔ Use **Supervisor** in production
✔ Log failed jobs
✔ Separate queues (`emails`, `exports`, `sync`)

---

## 10. When NOT to Use Queue?

❌ Very small logic
❌ Must return result immediately
❌ Real-time validation

---

# Queue for ERP Use Cases (Real World)

Think of ERP as **many slow + critical operations** happening together.
Queue = **system stability + speed**.

---

## 1. Sales Module

### Use Cases

* Send **invoice email**
* Generate **PDF invoice**
* Update **daily/monthly sales report**
* Sync sales to accounting system

### Flow

```
Create Sale → Save DB → Queue jobs
```

### Jobs

* `GenerateInvoicePdf`
* `SendInvoiceEmail`
* `UpdateSalesSummary`

### Example

```php
GenerateInvoicePdf::dispatch($sale->id);
SendInvoiceEmail::dispatch($sale->id);
```

---

## 2. Accounting Module

### Use Cases

* Posting journal entries
* Calculating tax
* End-of-day closing
* Month-end reconciliation

### Why Queue?

Accounting logic is **heavy** and **sensitive**.

### Example

```php
PostJournalEntry::dispatch($transactionId);
```

---

## 3. Inventory / Stock Management

### Use Cases

* Stock deduction after sale
* Bulk stock import
* Stock recalculation
* Low-stock alerts

### Example

```php
UpdateStockJob::dispatch($productId, $qty);
```

### Advanced

Queue stock updates **per warehouse** to avoid DB locks.

---

## 4. HR & Payroll

### Use Cases

* Salary calculation
* Payslip PDF generation
* Bulk email payslips
* Attendance processing

### Example

```php
GeneratePayrollJob::dispatch($month);
SendPayslipEmail::dispatch($employeeId);
```

---

## 5. Reporting Module

### Use Cases

* Sales reports
* Profit & Loss
* Excel / CSV exports
* Scheduled reports

### Why Queue?

Reports = **heavy SQL + memory**

### Example

```php
ExportSalesReport::dispatch($filters);
```

---

## 6. Notification System (Core ERP Feature)

### Use Cases

* Email
* SMS
* In-app notifications
* Slack / Line / WhatsApp

### Pattern

One event → multiple queued jobs

```php
NotifyUserJob::dispatch($userId, 'invoice_created');
```

---

## 7. Integration with Third-Party Systems

### Use Cases

* Accounting software
* Payment gateways
* Shipping providers
* CRM sync

### Example

```php
SyncInvoiceToXero::dispatch($invoiceId);
```

💡 Always queue external API calls.

---

## 8. Data Import / Migration

### Use Cases

* Import products
* Import customers
* Import historical invoices

### Batch Queue (Very Important)

```php
Bus::batch([
    new ImportProductJob($row1),
    new ImportProductJob($row2),
])->dispatch();
```

---

## 9. Approval Workflow

### Use Cases

* Purchase approval
* Expense approval
* Leave approval

### Queue Usage

* Send approval email
* Log approval history
* Notify next approver

```php
SendApprovalRequest::dispatch($requestId);
```

---

## 10. System Maintenance Tasks

### Use Cases

* Cleanup logs
* Archive old records
* Recalculate summaries
* Backup triggers

```php
ArchiveOldInvoices::dispatch();
```

Usually run via **Scheduler + Queue**.

---

# ERP Queue Architecture (Recommended)

```
HTTP Request
   ↓
Controller (FAST)
   ↓
Dispatch Jobs
   ↓
Queue (Redis)
   ↓
Workers (Supervisor)
```

---

## Recommended Queue Separation (ERP)

```php
public $queue = 'emails';     // emails
public $queue = 'reports';   // heavy SQL
public $queue = 'sync';      // APIs
public $queue = 'default';   // small tasks
```

Worker setup:

```bash
php artisan queue:work --queue=emails,reports,sync
```

---

## Example: ERP Sale Complete Flow

```php
DB::transaction(function () use ($sale) {

    DeductStockJob::dispatch($sale->id);
    GenerateInvoicePdf::dispatch($sale->id);
    SendInvoiceEmail::dispatch($sale->id);
    SyncSaleToAccounting::dispatch($sale->id);

});
```

🔥 Controller stays fast
🔥 ERP stays stable
🔥 Failures don’t break users

---

## Common ERP Queue Problems (And Fix)

### ❌ DB Locked

→ Use Redis queue
→ Smaller jobs

### ❌ Duplicate Jobs

→ Use `ShouldBeUnique`

```php
class SyncSale implements ShouldQueue, ShouldBeUnique
{
    public $uniqueFor = 3600;
}
```

---

## When ERP MUST Use Queue (Golden Rules)

✅ Email
✅ PDF / Excel
✅ External APIs
✅ Bulk operations
✅ Scheduled jobs

If you skip queue → **ERP will feel slow and unstable**

---

# 1. Job Chaining (Sequential Jobs)

## What is Job Chaining?

**Run jobs one by one, in order**
👉 Next job runs **only if previous job succeeds**

```
Job A → Job B → Job C
```

If **Job B fails**, **Job C will NOT run**.

---

## When to Use Chaining (ERP Use Cases)

Use chaining when **order matters**:

* Sale → Deduct stock → Generate invoice → Send email
* Payroll → Calculate salary → Generate payslip → Send payslip
* Import → Validate → Save → Notify admin

---

## ERP Example: Sale Completion Flow

### Jobs

1. `DeductStockJob`
2. `GenerateInvoicePdfJob`
3. `SendInvoiceEmailJob`

### Code

```php
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new DeductStockJob($saleId),
    new GenerateInvoicePdfJob($saleId),
    new SendInvoiceEmailJob($saleId),
])->dispatch();
```

🔥 Clean
🔥 Safe
🔥 Order guaranteed

---

## Adding Delay in Chain

```php
Bus::chain([
    (new GenerateInvoicePdfJob($saleId))->delay(5),
    new SendInvoiceEmailJob($saleId),
])->dispatch();
```

---

## Handle Chain Failure

In job class:

```php
public function failed(Throwable $e)
{
    Log::error('Chain failed: '.$e->getMessage());
}
```

💡 Once failed → chain stops.

---

## Best Practice for Chaining

✔ Small jobs
✔ No API calls that may timeout (or isolate them)
✔ Use Redis queue

---

# 2. Job Batching (Parallel Jobs)

## What is Job Batching?

**Run many jobs at the same time**
Used for **bulk operations**

```
Job 1
Job 2   → All running in parallel
Job 3
```

---

## When to Use Batching (ERP Use Cases)

* Import 10,000 products
* Send payslips to 1,000 employees
* Sync invoices to external system
* Recalculate stock for all products

---

## ERP Example: Bulk Product Import

```php
use Illuminate\Support\Facades\Bus;

$jobs = collect($rows)->map(function ($row) {
    return new ImportProductJob($row);
});

Bus::batch($jobs)
    ->name('Product Import')
    ->dispatch();
```

---

## Batch Callbacks (VERY IMPORTANT)

### Then (all success)

```php
->then(function (Batch $batch) {
    Log::info('Import completed');
})
```

### Catch (first failure)

```php
->catch(function (Batch $batch, Throwable $e) {
    Log::error('Import failed: '.$e->getMessage());
})
```

### Finally (always runs)

```php
->finally(function (Batch $batch) {
    Log::info('Batch finished');
});
```

---

## Full Batch Example

```php
Bus::batch($jobs)
    ->name('Payroll Processing')
    ->then(function () {
        SendPayrollSummary::dispatch();
    })
    ->catch(function ($batch, Throwable $e) {
        NotifyAdmin::dispatch('Payroll failed');
    })
    ->dispatch();
```

---

# 3. Chain vs Batch (ERP Decision Table)

| Scenario                  | Use        |
| ------------------------- | ---------- |
| Stock deduction → invoice | Chain      |
| Payroll per employee      | Batch      |
| Send 1 email              | Single job |
| Import CSV rows           | Batch      |
| Sale workflow             | Chain      |
| Recalculate all stock     | Batch      |

---

# 4. Chain + Batch Together (Advanced ERP Pattern)

### Example: Monthly Payroll

```
Chain:
  1. Prepare payroll
  2. Batch generate payslips
  3. Send summary email
```

### Code

```php
Bus::chain([
    new PreparePayrollJob($month),
    new ProcessPayslipBatchJob($month),
    new SendPayrollSummaryJob($month),
])->dispatch();
```

Inside `ProcessPayslipBatchJob`:

```php
Bus::batch($jobs)->dispatch();
```

🔥 Enterprise-level pattern

---

# 5. Retry & Timeout (ERP Safe Settings)

```php
class ImportProductJob implements ShouldQueue
{
    public $tries = 3;
    public $timeout = 120;
}
```

---

# 6. Batch Table Migration (Required)

```bash
php artisan queue:batches-table
php artisan migrate
```

---

# 7. Monitoring Batches

```php
Bus::findBatch($batchId);
```

Useful for admin dashboard.

---

# 8. Common ERP Mistakes

❌ Huge job logic
❌ Mixing chain & batch wrongly
❌ No retry strategy
❌ Running batch on database queue

---

# 9. Golden Rules for ERP

✅ Chain = workflow
✅ Batch = volume
✅ Keep jobs small
✅ Separate queues
✅ Redis + Supervisor

---

This is **mission-critical** for ERP.
A bad queue failure strategy = duplicated invoices, wrong stock, angry accountants 😅
Let’s do this **properly and production-ready**.

---

# Queue Failure & Retry Strategy (ERP-Grade)

## Core Goal

> **Fail safely, retry smartly, never corrupt data**

---

## 1. Why Jobs Fail (Real ERP Reasons)

Common failures you’ll see:

1. External API timeout (payment, accounting)
2. Database deadlock / lock
3. Network issue
4. Invalid data edge case
5. Worker crash / deploy restart

👉 **Failure is normal**. Strategy matters.

---

## 2. Basic Retry Configuration (Job Level)

```php
class SyncInvoiceJob implements ShouldQueue
{
    public $tries = 5;        // max attempts
    public $timeout = 120;   // seconds

    public function handle()
    {
        // logic
    }
}
```

### ERP Recommendation

| Job Type            | Tries |
| ------------------- | ----- |
| Email               | 3     |
| PDF                 | 2     |
| External API        | 5–10  |
| Critical accounting | 1–3   |

---

## 3. Backoff Strategy (VERY IMPORTANT)

Avoid retrying **immediately**.

```php
public function backoff()
{
    return [10, 60, 300]; // seconds
}
```

Meaning:

* 1st retry after 10s
* 2nd after 1 min
* 3rd after 5 min

Perfect for ERP API calls.

---

## 4. Prevent Duplicate Execution (CRITICAL)

### Use `ShouldBeUnique`

```php
use Illuminate\Contracts\Queue\ShouldBeUnique;

class SyncInvoiceJob implements ShouldQueue, ShouldBeUnique
{
    public $uniqueFor = 3600; // 1 hour

    public function uniqueId()
    {
        return 'invoice-'.$this->invoiceId;
    }
}
```

✅ No double sync
✅ No duplicate invoices

---

## 5. Database Safety (Transactions)

Always wrap **critical writes**:

```php
DB::transaction(function () {
    // update stock
    // write ledger
});
```

If job fails → rollback automatically.

---

## 6. Handle Failed Jobs Properly

### `failed()` Method

```php
use Throwable;

public function failed(Throwable $e)
{
    Log::error('Job failed', [
        'job' => static::class,
        'error' => $e->getMessage(),
    ]);

    NotifyAdminJob::dispatch($e->getMessage());
}
```

📢 Alert admin
📄 Log context

---

## 7. Failed Jobs Table (Must Have)

```bash
php artisan queue:failed-table
php artisan migrate
```

Check failures:

```bash
php artisan queue:failed
```

Retry:

```bash
php artisan queue:retry 5
php artisan queue:retry all
```

---

## 8. Fail Fast vs Retry (ERP Rules)

### Retry these:

✔ Network
✔ Timeout
✔ Temporary API errors

### Fail immediately:

❌ Validation error
❌ Missing invoice
❌ Business rule violation

Example:

```php
if (!$invoice) {
    $this->fail(new Exception('Invoice not found'));
}
```

---

## 9. Queue Retry Until (Time-Based Stop)

```php
public function retryUntil()
{
    return now()->addMinutes(30);
}
```

Useful for:

* Bank callbacks
* Payment confirmation
* Accounting sync

---

## 10. Job Timeout Protection

```php
public $timeout = 90;
public $failOnTimeout = true;
```

Prevents **infinite hanging jobs**.

---

## 11. Chain Failure Strategy (ERP Safe)

```php
Bus::chain([
    new DeductStockJob($saleId),
    new GenerateInvoiceJob($saleId),
    new SyncAccountingJob($saleId),
])->catch(function (Throwable $e) {
    RollbackSaleJob::dispatch($saleId);
    NotifyAdminJob::dispatch('Sale chain failed');
})->dispatch();
```

🔥 One failure → controlled recovery

---

## 12. Batch Failure Strategy

```php
Bus::batch($jobs)
    ->allowFailures()
    ->then(function (Batch $batch) {
        Log::info('Batch done with partial failures');
    })
    ->catch(function (Batch $batch, Throwable $e) {
        NotifyAdminJob::dispatch('Batch failed');
    })
    ->dispatch();
```

Good for imports.

---

## 13. Queue Separation = Failure Isolation

```php
emails   → low risk
reports  → heavy
sync     → external APIs
critical → accounting
```

If `reports` fail → **sales still work**.

---

## 14. Supervisor Restart Strategy (Production)

```bash
php artisan queue:restart
```

Always run on:

* Deploy
* Config change

---

## 15. ERP Golden Rules (Save This)

✅ Idempotent jobs
✅ Use `ShouldBeUnique`
✅ Backoff > retries
✅ Transaction for writes
✅ Log + notify
✅ Redis queue
✅ Supervisor

---

## Example: Perfect ERP Job Template

```php
class SyncPaymentJob implements ShouldQueue, ShouldBeUnique
{
    public $tries = 5;
    public $timeout = 120;
    public $uniqueFor = 1800;

    public function backoff()
    {
        return [30, 120, 300];
    }

    public function handle()
    {
        DB::transaction(function () {
            // sync logic
        });
    }

    public function failed(Throwable $e)
    {
        NotifyAdminJob::dispatch($e->getMessage());
    }
}
```




