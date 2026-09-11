# Refrigerator Repair Esfahan — Clean Architecture V2

نسخه بازطراحی‌شده سیستم مدیریت تعمیرات یخچال با ASP.NET Core 8 + Razor Pages + EF Core + MySQL.

## تصمیم‌های مهم این نسخه

### 1) مدیرها
- رمز عبور مدیر داخل جدول سفارشی ذخیره نمی‌شود.
- ASP.NET Core Identity مسئول `AspNetUsers`، `PasswordHash`، Lockout و Login است.
- Role مدیر در `AspNetRoles` / `AspNetUserRoles` قرار دارد.
- جدول `AdminProfiles` فقط اطلاعات مدیریتی مثل نام نمایشی و فعال/غیرفعال بودن را نگه می‌دارد.
- حداکثر ۳ مدیر فعال از طریق پنل ساخته می‌شود.

### 2) تکنسین و ثبت‌نام
`TechnicianRegistration` حذف شد.

بهترین حالت برای این پروژه این است که از همان ابتدا یک `Technician` ساخته شود و `AccountStatus=PendingApproval` باشد. بنابراین:
- اطلاعات تکنسین فقط در یک پروفایل اصلی است.
- مدیر همان رکورد را اصلاح و تأیید می‌کند.
- بعد از تأیید فقط Role تکنسین به Identity اضافه می‌شود؛ کپی/حذف بین دو جدول نداریم.
- `TechnicianDocument` جدا باقی مانده، چون هر تکنسین می‌تواند چند مدرک داشته باشد.
- مدارک داخل `wwwroot` قرار نمی‌گیرند.

### 3) رمز تکنسین
نام کاربری تکنسین = کد ملی.
رمز عبور توسط خود تکنسین انتخاب می‌شود.
شماره تلفن به عنوان Password استفاده نمی‌شود، چون قابل حدس است.

### 4) کیف پول
`WalletBalance` از `Technician` حذف شده است.
موجودی = مجموع `WalletTransactions.Amount`.
- شارژ: Amount مثبت
- کمیسیون: Amount منفی
- `BalanceAfter` برای Audit نگه داشته می‌شود.
- `PaymentTransaction` تلاش پرداخت درگاه است.
- `WalletTransaction` دفتر واقعی حسابداری است.
- `PaymentTransactionId` ارتباط شارژ موفق با Ledger را مشخص می‌کند.
- شارژ و کسر کمیسیون داخل Transaction دیتابیس انجام می‌شوند.

### 5) سفارش تکنسین
- هر سفارش حداکثر برای ۵ تکنسین واجد شرایط ارسال می‌شود.
- تکنسینی که همان روز پیشنهاد گرفته، دوباره انتخاب نمی‌شود.
- هر تکنسین در پنل فقط یک پیشنهاد فعال را می‌بیند.
- مهلت قبول هر پیشنهاد ۳۰ دقیقه است.
- با قبول یک تکنسین، پیشنهادهای دیگر همان سفارش منقضی می‌شوند.
- Worker پس‌زمینه پیشنهادهای منقضی را مدیریت و در صورت نیاز فهرست ۵ نفره را تکمیل می‌کند.
- تکنسین باید فعال، تأییدشده، آزاد و بدون تعمیر جاری باشد.
- قبل از پذیرش، شماره تلفن و آدرس مشتری نمایش داده نمی‌شود.
- حداقل موجودی لازم برای باز کردن اطلاعات کامل سفارش: ۵۰۰,۰۰۰ تومان.

### 6) تکمیل تعمیر
فرمول:
`سود خالص مبنا = مبلغ دریافتی - هزینه قطعات - ایاب و ذهاب`

`کمیسیون مرکز = ۲۵٪ سود خالص مبنا`

`سود نهایی تکنسین = سود خالص مبنا - کمیسیون`

اگر کیف پول برای کمیسیون کافی نباشد، سفارش تکمیل نمی‌شود و مبلغ لازم به تکنسین اعلام می‌شود.

### 7) مشتری تکراری
شماره تلفن مشتری Unique است.
در ثبت سفارش، سیستم ابتدا Customer را با شماره تلفن پیدا می‌کند؛ اگر وجود داشته باشد Customer جدید ساخته نمی‌شود و سفارش جدید به همان مشتری وصل می‌شود.

## UI
ظاهر اصلی آبی/سفید و کارت‌های Rounded حفظ شده است، ولی موارد زیر مدرن‌تر شده‌اند:
- کارت‌های متحرک و Shadow نرم
- Success State گرافیکی
- تأیید نهایی قبل از تکمیل تعمیر
- داشبورد آماری تکنسین
- Sidebar پنل مدیریت
- منوی کشویی پنل تکنسین
- Responsive برای موبایل

## اجرای پروژه
1. Visual Studio 2026 و workload مربوط به ASP.NET Core را نصب کنید.
2. MySQL را نصب و Connection String را در `src/Web/appsettings.json` تنظیم کنید.
3. `RefrigeratorRepair.sln` را باز کنید.
4. `RefrigeratorRepair.Web` را Startup Project کنید.
5. Restore و Build و سپس F5.

## مدیر Development
- username: `admin`
- password: `Admin@12345`

این رمز فقط Development است.

## هشدار مهم درباره دیتابیس
مدل دیتابیس این نسخه نسبت به نسخه قبلی تغییر کرده است: `TechnicianRegistration` و `WalletBalance` حذف شده‌اند و روابط جدید اضافه شده‌اند.
چون نسخه آموزشی فعلاً `EnsureCreated` دارد، اگر دیتابیس قبلی را ساخته‌اید، برای تست این نسخه بهتر است دیتابیس Development قبلی را حذف و دوباره ایجاد کنید. در نسخه Production باید EF Core Migration استفاده شود.

## درگاه
`FakePaymentGateway` فقط برای تست محلی است. قبل از Production باید Provider رسمی درگاه، Callback و Verify واقعی، Idempotency و Secretهای امن پیاده‌سازی شوند.


## تنظیمات امنیتی V3-Secure

- Connection String دیگر در `appsettings.json` شامل Username/Password نیست.
- در Development مقدار `ConnectionStrings:DefaultConnection` و `SeedAdmin:Password` را با User Secrets تنظیم کنید.
- FakePaymentGateway فقط در Development ثبت می‌شود؛ Production از `DisabledPaymentGateway` استفاده می‌کند تا پرداخت جعلی ممکن نباشد.
- Identity Lockout: حداکثر ۵ تلاش ناموفق و قفل ۱۵ دقیقه‌ای.
- Cookie احراز هویت: HttpOnly، Secure و SameSite=Lax.
- Rate Limiting عمومی و محدودیت سخت‌تر برای Login فعال شده است.
- Security Headers شامل CSP، X-Content-Type-Options، X-Frame-Options، Referrer-Policy و Permissions-Policy فعال هستند.
- Seed مدیر فقط در Development اجرا می‌شود و رمز آن از Secret خوانده می‌شود.
- Roleها در `AppRoles` متمرکز شده‌اند.
- نرخ کمیسیون و حداقل شارژ کیف پول از Configuration خوانده و در Startup اعتبارسنجی می‌شوند.
- صفحات Admin و Technician با `Cache-Control: no-store` ارسال می‌شوند.

### User Secrets در Development

پس از باز کردن پروژه Web در ترمینال، مقدارهای زیر را تنظیم کنید:

```bash
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "server=localhost;port=3306;database=refrigerator_repair_esfahan;user=refrigerator_app;password=YOUR_DEV_PASSWORD;CharSet=utf8mb4;" --project src/Web
dotnet user-secrets set "SeedAdmin:Password" "YOUR_STRONG_DEV_ADMIN_PASSWORD" --project src/Web
```

در Production همین مقادیر باید از Secret Manager یا Environment Variables سرور تأمین شوند و نباید وارد Git شوند.
