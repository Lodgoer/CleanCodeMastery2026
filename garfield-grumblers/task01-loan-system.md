<div dir="rtl">

#### Files to Review

| File | Methods to Analyze | Expected Smells |
|------|-------------------|-----------------|
| `LoanProcessingEngine.cs` | `ProcessSingleRecordWithRetryAsync` | Length > 80 lines |
| `LoanProcessingEngine.cs` | `ReadExcelData` | Length > 60 lines |
| `LoanProcessingEngine.cs` | `LogAuditTrailAsync` | Length > 40 lines |
| `ExternalLoanService.cs` | `ExecuteLoanOperation` | Switch statement |
| `ExternalLoanService.cs` | `ExecuteRequest` | Length > 35 lines |

#### Detection Checklist
```markdown
- [ ] Count lines per method (excluding comments/whitespace)
- [ ] Identify methods with > 30 lines
- [ ] Document nesting depth (> 3 levels)
- [ ] Count parameters (> 5)
- [ ] List all responsibilities per method
```
---

### ‏`ProcessSingleRecordWithRetryAsync` :
- چند کار به طور همزمان انجام میشه
- پارامتر های زیادی استفاده شده
- از object های بدون تایپ مشخص زیاد استفاده شده
- از while استفاده شده
- ‏**Long Method**:
  حدودو 60 خط متد که فقط «پردازش یک رکورد با Retry» نیست؛ جزئیات زیادی از Retry، Logging و Error Handling را هم خودش مدیریت می‌کند.
- ‏**Primitive Obsession**:
  ایا اینکه آبچت هایی مثل object lastRequest = null; داریم این اسمل محسوب بشه؟
- ‏**Long Parameter List**:
  تعداد پارامترها زیاده، اما این الزاماً Long Parameter List نیست چون Smell مربوط به یک متد تعریف‌شده با تعداد زیادی پارامتر است، نه صرفاً هر جایی که یک متد را با آرگومان‌های زیاد صدا بزنیم.
- ‏**Data Clumps**:
  یک سری پارامتر ها مثل record.ChassisNumber، requestType، request، response، result، attempt اگر در بخش‌های مختلف سیستم مرتباً همین گروه داده‌ها کنار هم قرار بگیرند، می‌تواند نشانه‌ی Data Clumps باشد و شاید یک مفهوم مثل ProcessingContext یا AuditContext ارزش استخراج داشته باشد.

---

### ‏`ReadExcelData` :

- چند کار به طور همزمان انجام میشود:
  - خواندن فایل اکسل
  - ساختن columnMap
  - تبدیل ردیف های اکسل به LngRequestRecord
  - مدیریت خطا ها
  - لاگینگ
  - و..
- متد حجم نسبتا زیادی منطق در خودش جمع کرده و بعضی بخش ها میتوانند به متد های جداگانه تقسیم شوند.
- ‏**Long Method**:
  متد حدود 8 خط استو بیش از یک مسئولیت را برعهدا گرفته.
- ‏**Primitive Operation**:
  استفاده از مقادیر 0، 1، 2 برای نمایش مفاهیم مشخص برای مثال record.ProcessingMode = 1 اگر نشانه مقادیر مشخصی هستند بهتر است از enum برای آنها استفاده شود.
- ‏**Data Clumps**:
  مقادیری مثل PlateRegionCode، PlateFirstPart، PlateLetter و PlateSecondPart برای ساختن یک مفهوم واحد یعنی FullPlateNumber همیشه در کنار یکدیگر استفاده می‌شوند. اگر این گروه از داده‌ها در بخش‌های مختلف سیستم نیز مرتباً با هم ظاهر شوند، می‌توان آن‌ها را در abstractionای مثل LicensePlate قرار داد.

---

### ‏`LogAuditTrailAsync` :

- چند کار به طور همزمان انجام میشه:
    - ساختن نام و مسیر فایل
    - بررسی وجود فایل
    - خواندن فایل قبلی
    - سریالایز کردن json ها
    - دیسریالایز کردن لاگ های قبلی
    - مدیریت لاگ خراب
    - اضافه کردن لاگ جدید
    - و..

- ‏**Long Method** :
  متد حدود 60 خط استو مسئولیت های مختلفی را برعهده دارد.
- ‏**Primitive Obssesion**:
  استفاده از object برای پارامترهایی مثل request، response و exceptionInfo باعث از بین رفتن Type Safety می‌شود و بهتر است نوع مشخصی برای آن‌ها تعریف شود. با این حال، این مورد Primitive Obsession کلاسیک محسوب نمی‌شود و بیشتر یک مشکل در طراحی Typeهاست.
- ‏**Long Parameter**:
  متد 8 پارامتر دارد و اینجا Long Parameter List می‌تواند یک Smell واقعی محسوب شود؛ مخصوصاً چون چندین پارامتر مرتبط با یک مفهوم Audit در کنار یکدیگر ارسال می‌شوند. می‌توان این داده‌ها را در یک مدل مثل AuditTrailEntry یا AuditContext قرار داد و متد را ساده‌تر کرد.
- ‏**Data Clumps**:
   پارامترهایی مثل chassisNumber، requestType، request، response، result، exceptionInfo، attempt و isSuccess همگی بخشی از اطلاعات مربوط به یک عملیات Audit هستند و در این متد کنار هم قرار گرفته‌اند. اگر همین گروه از داده‌ها در قسمت‌های مختلف سیستم نیز مرتباً با هم استفاده شوند، می‌تواند نشانه‌ی Data Clumps باشد و استخراج یک abstraction مثل AuditContext می‌تواند مناسب باشد.

---

### ‏`ExecuteLoanOperation` :

- متد کوتاه و نسبتاً ساده است و خودش منطق زیادی ندارد؛ بیشتر نقش هماهنگ‌کننده بین انواع مختلف عملیات را دارد.
- ‏**Primitive Obsession**:
  پارامتر operationType از نوع int است، در حالی که مقدارهای 1، 2، 3 و 4 هرکدام یک مفهوم مشخص در Domain را نشان می‌دهند. استفاده از عددهای جادویی باعث می‌شود معنی کد بدون نگاه کردن به caseها مشخص نباشد. بهتر است به جای آن از یک enum مثل LoanOperationType استفاده شود تا مقدار و معنای آن type-safe و خواناتر باشد.
- ‏**Switch Statements**:
  وجود یک switch برای تعیین رفتار بر اساس operationType می‌تواند نشانه‌ی این Code Smell باشد. با اضافه‌شدن عملیات جدید، این switch نیز باید تغییر کند و اگر چنین شرط‌هایی در بخش‌های مختلف سیستم تکرار شوند، می‌تواند نشانه‌ای برای جایگزین کردن آن با Polymorphism یا Strategy Pattern باشد. در این مثال، به دلیل تعداد کم حالت‌ها و ساده بودن منطق هر case، این Smell شدت بالایی ندارد.

---

### ‏`ExecuteRequest` :

- چند کار به طور همزمان انجام شده:
   - اجرای request
   - بررسی کانکشن ها
   - بررسی خالی بودن response
   - دیسریالایز کردن response
   - تبدیل خطا های مختلف به ServiceResponse
   - مدیریت اکسپشن ها
- ‏**Long Method**:
  متد خیلی طولانی نیست، اما چند مرحله‌ی مختلف از پردازش یک Request را در خودش انجام می‌دهد؛ از اجرای Request و بررسی وضعیت آن گرفته تا Deserialize کردن Response و مدیریت چند نوع خطا. با این حال، چون متد نسبتاً کوتاه و منطق آن مرتبط با یک هدف مشخص یعنی اجرای Request و تبدیل نتیجه به ServiceResponse است، نمی‌توان آن را یک نمونه‌ی شدید از Long Method دانست.
- ‏**Primitive Obsession**:
  پارامتر string operationName می‌تواند در صورت تکرار در بخش‌های مختلف سیستم نشانه‌ای از Primitive Obsession باشد؛ چون یک مفهوم مشخص برای نام سرویس را به شکل string منتقل می‌کند. با این حال، در این مثال به‌تنهایی شواهد کافی برای تشخیص قطعی این Smell وجود ندارد.

</div>
