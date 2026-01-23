**Laravel mail sending**, step by step 👍
Here’s the **clean, practical flow** from zero to sending email.

---

## 1️⃣ Configure mail in `.env`

First, set your mail driver (example: Gmail SMTP)

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your_email@gmail.com
MAIL_PASSWORD=your_app_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=your_email@gmail.com
MAIL_FROM_NAME="Invoice App"
```

⚠️ For Gmail, use **App Password**, not your real password.

---

## 2️⃣ Check mail config (`config/mail.php`)

Usually no changes needed, but make sure:

```php
'default' => env('MAIL_MAILER', 'smtp'),
```

---

## 3️⃣ Create a Mailable class

```bash
php artisan make:mail WelcomeMail
```

This creates:

```
app/Mail/WelcomeMail.php
```

---

## 4️⃣ Create email view (Blade)

```bash
mkdir resources/views/emails
touch resources/views/emails/welcome.blade.php
```

Example:

```blade
<h1>Welcome {{ $name }} 👋</h1>

<p>Thanks for joining our app.</p>

<ul>
  <li>Product Management</li>
  <li>Invoice Taking</li>
  <li>Voucher Records</li>
  <li>Sale Screen</li>
  <li>Profile Editing</li>
</ul>
```

---

## 5️⃣ Edit the Mailable class

```php
namespace App\Mail;

use Illuminate\Mail\Mailable;

class WelcomeMail extends Mailable
{
    public $name;

    public function __construct($name)
    {
        $this->name = $name;
    }

    public function build()
    {
        return $this->subject('Welcome to Invoice App')
                    ->view('emails.welcome');
    }
}
```

---

## 6️⃣ Send mail from Controller

```php
use App\Mail\WelcomeMail;
use Illuminate\Support\Facades\Mail;

Mail::to($user->email)->send(
    new WelcomeMail($user->name)
);
```

✅ Email sent immediately.

---

## 7️⃣ (Recommended) Send mail using Queue

### Set queue driver

```env
QUEUE_CONNECTION=database
```

### Create queue table

```bash
php artisan queue:table
php artisan migrate
```

### Send mail later

```php
Mail::to($user->email)
    ->queue(new WelcomeMail($user->name));
```

### Run worker

```bash
php artisan queue:work
```

---

## 8️⃣ Test mail quickly (Tinker)

```bash
php artisan tinker
```

```php
Mail::to('test@gmail.com')->send(
    new App\Mail\WelcomeMail('Test User')
);
```

---

## ✅ Mail sending flow (mental model)

```
Controller
  → Mailable
      → Blade view
          → SMTP
              → Inbox 📬
```

---

# 📧 Laravel Mail – All Common Methods

## 1️⃣ Basic sending methods

### `send()`

Send immediately (sync)

```php
Mail::to($user->email)->send(new WelcomeMail());
```

---

### `queue()`

Send using queue

```php
Mail::to($user->email)->queue(new WelcomeMail());
```

➡ requires queue worker running

---

### `later()`

Send with **delay**

```php
Mail::to($user->email)
    ->later(now()->addMinutes(10), new WelcomeMail());
```

---

## 2️⃣ Addressing methods

### `to()`

```php
Mail::to('user@gmail.com')
```

### `cc()`

```php
Mail::to($user->email)
    ->cc('manager@gmail.com')
```

### `bcc()`

```php
Mail::to($user->email)
    ->bcc('admin@gmail.com')
```

### Multiple recipients

```php
Mail::to(['a@gmail.com', 'b@gmail.com'])
```

---

## 3️⃣ Mail sending variations

### `sendNow()`

Force immediate send even if mailable is queued

```php
Mail::to($user->email)->sendNow(new WelcomeMail());
```

---

### `send()` with raw closure

```php
Mail::send('emails.welcome', ['name' => 'John'], function ($mail) {
    $mail->to('john@gmail.com')
         ->subject('Welcome');
});
```

---

## 4️⃣ Queue-related Mail methods (IMPORTANT)

### `queue()`

```php
Mail::to($user->email)->queue(new WelcomeMail());
```

---

### `later()`

```php
Mail::to($user->email)
    ->later(now()->addSeconds(30), new WelcomeMail());
```

---

### `laterOn()`

Send on **specific queue**

```php
Mail::to($user->email)
    ->laterOn('emails', now()->addMinutes(5), new WelcomeMail());
```

---

### `onQueue()`

Set queue name

```php
Mail::to($user->email)
    ->onQueue('emails')
    ->queue(new WelcomeMail());
```

---

### `onConnection()`

Choose queue connection

```php
Mail::to($user->email)
    ->onConnection('redis')
    ->queue(new WelcomeMail());
```

---

## 5️⃣ Queue + Delay (Best Practice Way)

### Using `ShouldQueue` (Recommended)

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class WelcomeMail extends Mailable implements ShouldQueue
{
    public function build()
    {
        return $this->subject('Welcome')
                    ->view('emails.welcome');
    }
}
```

Then:

```php
Mail::to($user->email)
    ->delay(now()->addMinutes(2))
    ->queue(new WelcomeMail());
```

---

## 6️⃣ Delay inside Mailable

```php
class WelcomeMail extends Mailable implements ShouldQueue
{
    public $delay = 60; // seconds
}
```

or

```php
public function build()
{
    $this->delay(now()->addMinutes(1));
}
```

---

## 7️⃣ Retry & Timeout (Queue mail)

```php
class WelcomeMail extends Mailable implements ShouldQueue
{
    public $tries = 3;
    public $timeout = 30;
}
```

---

## 8️⃣ Mail fake (Testing)

```php
use Illuminate\Support\Facades\Mail;

Mail::fake();

Mail::to($user->email)->send(new WelcomeMail());

Mail::assertSent(WelcomeMail::class);
```

---

## 9️⃣ Mail events

```php
Illuminate\Mail\Events\MessageSending
Illuminate\Mail\Events\MessageSent
```

---

## 🧠 Summary Cheat Sheet

| Method           | Purpose           |
| ---------------- | ----------------- |
| `send()`         | Send immediately  |
| `sendNow()`      | Force immediate   |
| `queue()`        | Send via queue    |
| `later()`        | Delayed send      |
| `laterOn()`      | Delayed + queue   |
| `onQueue()`      | Set queue         |
| `onConnection()` | Set connection    |
| `delay()`        | Delay queued mail |
| `Mail::fake()`   | Testing           |

---

# 📦 Mailable Class – More Real Examples

---

## 1️⃣ Simple Welcome Mail (Markdown)

### Create mailable

```bash
php artisan make:mail WelcomeMail --markdown=emails.welcome
```

### `app/Mail/WelcomeMail.php`

```php
use Illuminate\Mail\Mailable;
use Illuminate\Contracts\Queue\ShouldQueue;

class WelcomeMail extends Mailable implements ShouldQueue
{
    public function __construct(
        public string $name
    ) {}

    public function build()
    {
        return $this
            ->subject('Welcome to Invoice App 🎉')
            ->markdown('emails.welcome');
    }
}
```

---

### `resources/views/emails/welcome.blade.php`

```blade
@component('mail::message')
# Welcome {{ $name }} 👋

Thanks for joining **Invoice App**.

You can now:
- Manage products
- Create invoices
- Record vouchers
- Use the sale screen
- Edit your profile

@component('mail::button', ['url' => config('app.url')])
Go to Dashboard
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

---

## 2️⃣ Invoice Created Mail (With Table)

### Mailable

```php
class InvoiceCreatedMail extends Mailable
{
    public function __construct(
        public $invoice
    ) {}

    public function build()
    {
        return $this
            ->subject('Your Invoice #' . $this->invoice->id)
            ->markdown('emails.invoice-created');
    }
}
```

---

### Markdown View

```blade
@component('mail::message')
# Invoice Created 🧾

Hello {{ $invoice->customer_name }},

Your invoice has been successfully created.

@component('mail::table')
| Item | Qty | Price |
|------|-----|-------|
@foreach ($invoice->items as $item)
| {{ $item->name }} | {{ $item->qty }} | {{ $item->price }} |
@endforeach
@endcomponent

**Total:** {{ $invoice->total }} THB

@component('mail::button', ['url' => url('/invoices/'.$invoice->id)])
View Invoice
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

---

## 3️⃣ OTP / Verification Mail (Clean & Short)

### Mailable

```php
class OtpMail extends Mailable implements ShouldQueue
{
    public function __construct(
        public string $code
    ) {}

    public function build()
    {
        return $this
            ->subject('Your Verification Code')
            ->markdown('emails.otp');
    }
}
```

---

### Markdown View

```blade
@component('mail::message')
# Verification Code 🔐

Use the code below to verify your account:

@component('mail::panel')
## {{ $code }}
@endcomponent

This code will expire in **10 minutes**.

If you didn’t request this, you can ignore this email.

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

---

## 4️⃣ Failed Payment / Alert Mail

```php
class PaymentFailedMail extends Mailable
{
    public function __construct(
        public string $reason
    ) {}

    public function build()
    {
        return $this
            ->subject('Payment Failed ❌')
            ->markdown('emails.payment-failed');
    }
}
```

```blade
@component('mail::message')
# Payment Failed

Unfortunately, your payment could not be processed.

**Reason:** {{ $reason }}

@component('mail::button', ['url' => url('/billing')])
Retry Payment
@endcomponent

Support Team<br>
{{ config('app.name') }}
@endcomponent
```

---

## 5️⃣ Mail With Attachment (PDF Invoice)

```php
public function build()
{
    return $this
        ->subject('Invoice PDF')
        ->markdown('emails.invoice')
        ->attach(storage_path('app/invoices/invoice.pdf'));
}
```

---

## 6️⃣ Mail With Dynamic From / Reply-To

```php
public function build()
{
    return $this
        ->from('billing@invoice.app', 'Billing Team')
        ->replyTo('support@invoice.app')
        ->subject('Billing Info')
        ->markdown('emails.billing');
}
```

---

## 7️⃣ Queue + Delay Example (Production Style)

```php
Mail::to($user->email)
    ->onQueue('emails')
    ->delay(now()->addMinutes(5))
    ->queue(new WelcomeMail($user->name));
```

---

## 8️⃣ Markdown Components You Can Use

| Component       | Usage         |
| --------------- | ------------- |
| `mail::message` | Wrapper       |
| `mail::button`  | CTA button    |
| `mail::panel`   | Highlight box |
| `mail::table`   | Tables        |
| `mail::subcopy` | Footer text   |

Example:

```blade
@component('mail::subcopy')
If you're having trouble clicking the button, copy and paste the URL.
@endcomponent
```

---

# 🎨 Laravel Mail Branding (Colors + Logo)

Laravel Markdown mail is **theme-based**, so you customize it once and **all emails inherit it**.

---

## 1️⃣ Publish Mail Assets (IMPORTANT)

Run this first:

```bash
php artisan vendor:publish --tag=laravel-mail
```

This creates:

```
resources/views/vendor/mail
```

Structure:

```
mail/
 ├── html/
 ├── text/
 └── themes/
     └── default.css
```

---

## 2️⃣ Set Your Brand Color

Open:

```
resources/views/vendor/mail/themes/default.css
```

Find this part:

```css
.button-primary {
    background-color: #3869D4;
    border-color: #3869D4;
}
```

Change to your brand color (example: blue-green SaaS style):

```css
.button-primary {
    background-color: #0d9488;   /* teal-600 */
    border-color: #0d9488;
}
```

You can also tweak text color:

```css
h1, h2, h3 {
    color: #0f172a; /* slate-900 */
}

body {
    background-color: #f8fafc;
}
```

---

## 3️⃣ Set Global Mail Color (Config Way)

In `config/mail.php`:

```php
'markdown' => [
    'theme' => 'default',
    'paths' => [
        resource_path('views/vendor/mail'),
    ],
],
```

In `config/app.php`:

```php
'mail' => [
    'color' => '#0d9488',
],
```

Then in Markdown views, buttons automatically use it.

---

## 4️⃣ Add Your Logo (Header Branding)

Open:

```
resources/views/vendor/mail/html/header.blade.php
```

Replace content with:

```blade
<tr>
<td class="header">
    <a href="{{ config('app.url') }}" style="display: inline-block;">
        <img src="{{ asset('logo.png') }}"
             alt="{{ config('app.name') }}"
             style="height: 40px;">
    </a>
</td>
</tr>
```

📌 Put logo here:

```
public/logo.png
```

✅ PNG or SVG recommended
✅ Height: 32–48px

---

## 5️⃣ Add Brand Footer

Edit:

```
resources/views/vendor/mail/html/footer.blade.php
```

Example:

```blade
<tr>
<td class="footer">
    <p>
        © {{ date('Y') }} {{ config('app.name') }}.  
        Unlocking Business Potential Through Modern Software
    </p>
</td>
</tr>
```

---

## 6️⃣ Custom Button Style (Rounded, Modern)

In `default.css`:

```css
.button {
    border-radius: 8px;
    font-weight: 600;
    text-transform: none;
}
```

Now all buttons look modern ✨

---

## 7️⃣ Example Branded Markdown Mail

```blade
@component('mail::message')
# Welcome to Invoice App 👋

We’re excited to have you onboard.

@component('mail::button', ['url' => config('app.url')])
Open Dashboard
@endcomponent

@component('mail::subcopy')
Need help? Contact support@invoice.app
@endcomponent
@endcomponent
```

➡ Uses your **brand color + logo automatically**

---

## 8️⃣ Multiple Themes (Advanced)

You can create:

```
resources/views/vendor/mail/themes/dark.css
```

Then switch per mail:

```php
public function build()
{
    return $this
        ->subject('Welcome')
        ->markdown('emails.welcome')
        ->theme('dark');
}
```

🔥 Great for **system alerts vs marketing emails**

---

## 9️⃣ Production Tips (Very Important)

✔ Logo must be **public URL** (not local path)
✔ Avoid background images (email clients hate them)
✔ Test on Gmail + Outlook
✔ Keep width ≤ 600px

---

## 🧠 Real-World Brand Setup Checklist

✅ Logo
✅ Primary color
✅ Button radius
✅ Footer text
✅ Consistent tone



