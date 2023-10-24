# مستندات پلاگین مایکت برای گودو انجین
زاهنمای پیاده سازی پلاگین مایکت برای گودو انجین

## دریافت پلاگین
جهت دریافت پلاگین به [این](https://gdpars.sellfile.ir/prod-2174792-%D9%BE%D9%84%D8%A7%DA%AF%DB%8C%D9%86+%D9%BE%D8%B1%D8%AF%D8%A7%D8%AE%D8%AA+%D8%AF%D8%B1%D9%88%D9%86+%D8%A8%D8%B1%D9%86%D8%A7%D9%85%D9%87+%D8%A7%DB%8C+%D9%85%D8%A7%DB%8C%DA%A9%D8%AA.html) صفحه مراجه کنید.

## نثب و راه اندزای

ابتدا فایل را از زیپ خارج کرده و بر اساس نسخه گودوی خود فایل ها را در محل پروژه ی خود کپی کنید.

سپس به مسیر `android` و `build` رفته و فایل `gradle.build` را ویاریش کنید و به دنبال `manifestPlaceholders` بگردید و قطعه زیر بعد از آن اضافه کنید.

```gradle
  def marketApplicationId = "ir.mservices.market"
  def marketBindAddress = "ir.mservices.market.InAppBillingService.BIND"
  manifestPlaceholders += [marketApplicationId: "${marketApplicationId}",
                marketBindAddress  : "${marketBindAddress}",
                marketPermission   : "${marketApplicationId}.BILLING"]
```
در آخر فایل را ذخیره کنید.
## مراحل پیاده سازی: ساخت IABHelper
برای شروع کار باید یک `Node` از `IABHelper` ایجاد کنیم. با توجه به اینکه در متد‌ها و `Callbackه`ای مختلف نیاز است به این `Node` دسترسی داشته باشید، بهتر است این `Node` را  در `Scene` خود تعریف کنید. برای ساختن `IABHelper` نیاز است تا کلید رمز عمومی را (که از پنل توسعه‌دهندگان دریافت کرده‌اید) به نود `IABHelper` بدهید:

گودو 3:

```gdscript
const PUBLIC_KEY = ""

onready var iab_helper = IABHelper.new(PUBLIC_KEY)

func _ready():
	...
	add_child(iab_helper)
	...
```
گودو 4:
```gdscript
const PUBLIC_KEY = ""

@onready var iab_helper = IABHelper.new(PUBLIC_KEY)

func _ready():
	...
	add_child(iab_helper)
	...
```

## اتصال به سرویس مایکت (start_setup)
برای اتصال به سرویس پرداخت درون‌برنامه‌ای مایکت کافی است از متد `start_setup` استفاده کنید. این متد را باید قبل از هر متد دیگری صدا بزنید و در صورت متصل شدن به سرویس می‌توانید از سرویس پرداخت درون‌برنامه‌ای مایکت استفاده کنید. حال به سیگنال `on_iab_setup_finished` متصل شوید:

گودو 3:
```gdscript
	iab_helper.start_setup()
	iab_helper.connect("on_iab_setup_finished",self,"_on_iab_setup_finished",[],CONNECT_ONESHOT)

func _on_iab_setup_finished(is_succeed : bool, result : String):
	...
```
گودو 4:
```gdscript
	iab_helper.start_setup()
	iab_helper.on_iab_setup_finished.connect(_on_iab_setup_finished,CONNECT_ONE_SHOT)

func _on_iab_setup_finished(is_succeed : bool, result : String):
	...
```

پس از تلاش برای اتصال به سرویس، متد `_on_iab_setup_finished` همراه با پاسخ فراخوانی می‌شود و نتیجه را اعلام می‌کند. در صورتی که خروجی متد `is_succeed` در پاسخ برابر `true` بود، اتصال با موفقیت انجام شده است. در غیر این صورت برنامهٔ شما نتوانسته به سرویس مایکت متصل شود که در این صورت می‌توانید از متد `result` برای علت خطا استفاده کنید.

## به‌روز کردن اطلاعات خرید (Inventory)
به محض متصل شدن به سرویس مایکت (یعنی زمانی که متد `is_succeed` در پاسخ برابر `true` بود) باید خرید‌های کاربر را به‌روز کنید. به‌روز کردن خرید کاربر‌ها در مدل‌های محصولات مختلف مفهوم‌های مختلفی دارد:

برای محصولات مصرف‌شدنی این معنی را می‌دهد که به هر علت یک خرید مصرف نشده وجود دارد که باید مصرف شود و به کاربر تحویل داده شود. مثلا فرض کنید که در فرایند خرید قبلی، کاربر با مشکل مواجه می‌شود یا به هر دلیلی قبل از مصرف کردن محصول، برنامهٔ شما بسته می‌شود و یا ارتباط کاربر قطع می‌گردد. در این صورت باید در اجرای بعدی این محصول را مصرف کنید و به کاربر تحول دهید.

علت به‌روز کردن محصولات برای محصولات مصرف‌نشدنی این است که از این طریق متوجه می‌شوید که کاربر محصول شما را خریداری کرده است یا نه. در صورتی که خریداری کرده بود، محصول یا همان امکان مورد نظر (مثلا حذف تبلیغات یا مراحل کامل بازی یا …) را برای او فعال می‌کنید.

در واقع می‌توان گفت تمام محصولاتی که مصرف نکرده‌اید،همیشه در `Inventory` برمی‌گردد. در صورتی که مصرف‌شدنی بود که باید همان‌جا مصرف شود و بعد تحویل کاربر شود و در صورتی که مصرف‌نشدنی یا اشتراکی بود، باید به کاربر تحویل داده شود. 

همچنین با استفاده از `Inventory` می‌توانید اطلاعات مربوط به محصولات خود (نام محصول، توضیحات، قیمت و …) که در پنل توسعه‌دهندگان وارد نموده‌اید را دریافت کنید. مثلا فرض کنید که بعد از انتشار، قیمت یا نام یکی از محصولات خود را تغییر می‌دهید. با استفاده از این قابلیت می‌توانید در UI برنامهٔ خود همیشه مشخصات محصولی را نشان دهید که در پنل توسعه‌دهندگان مایکت قرار دارد. به عبارتی با تغییر آن، برنامه شما نیز تغییر می‌کند.

برای به‌روز کردن خرید‌ها باید نام محصولات را در متد `query_inventory_async` قرار داده و به سیگنال `on_query_inventory_finished` متصل شوید تا فرایند به‌روز کردن آغاز شود:

گودو 3:
```gdscript
	iab_helper.query_inventory_async(query_sku_details, more_sku)
	iab_helper.connect("on_query_inventory_finished",self,"_on_query_inventory_finished",[],CONNECT_ONESHOT)

func _on_query_inventory_finished(is_succeed : bool, result : String, inventory : Inventory):
	...
```
گودو 4:
```gdscript
	iab_helper.query_inventory_async(query_sku_details, more_sku)
	iab_helper.on_query_inventory_finished.connect(_on_query_inventory_finished,CONNECT_ONE_SHOT)

func _on_query_inventory_finished(is_succeed : bool, result : String, inventory : Inventory):
	...
```
متد `query_inventory_async` دارای دو پارامتر است که به ترتیب:

متغیر `query_sku_details`: در صورتی که می‌خواهید اطلاعات محصولات به‌روز شود (نام، قیمت و …) این `boolean` را برابر `true` قرار دهید. توجه کنید در صورتی که نمی‌خواهید از این قابلیت استفاده کنید این مقدار را برابر `false` قرار دهید تا سرویس اضافه‌ای صدا نشود.


متغیر `more_sku`: آرایه ای از ID محصولات فروشی (مصرف‌شدنی و مصرف‌نشدنی).

نتیجه این درخواست در `Inventory` در سیگنال `on_query_inventory_finished` بازمی‌گردد. از `result` موفق بودن/نبودن درخواست مشخص می‌شود و خرید‌های به‌روز شده و اطلاعات محصولات در `Inventory` قرار می‌گیرد. `Inventory` شامل دو آرایه با نام‌های `sku_details` و `purchases` است. خرید‌های به‌روز‌ شده در `purchases` قرار می‌گیرد که با استفاده از متد `get_purchase` در کلاس `Inventory` می‌توانید متوجه شوید برای محصول مورد نظرتان خرید به‌روز شده‌ای وجود دارد یا خیر.

همچنین می‌توانید اطلاعات خرید که در پنل توسعه‌دهندگان تنظیم کرده‌اید را در `sku_details` بیابید. توجه کنید در صورتی که مقدار `query_sku_details` را برابر `false` قرار دهید این آرایه برابر با `null` خواهد بود. با استفاده از متد `get_sku_details` در کلاس `Inventory` می‌توانید به اطلاعات خرید محصول مورد نظر دسترسی پیدا کنید و `UI` برنامهٔ خود را به‌روز نمایید.

## فرستادن درخواست‌های خرید درون‌برنامه‌ای

هنگامی که برنامه شما به مایکت متصل شد، شما می‌توانید برای محصولات درون برنامه درخواست خرید بفرستید. مایکت یک رابط کاربری برای روند پراخت کاربران فراهم می‌کند به طوری که برنامه شما درگیر مدیریت ترانکش‌ها نخواهد شد و این قبیل عملیات‌ بر عهده مایکت خواهد بود. وقتی یک محصول خریداری شد، مایکت کاربر را مالک آن می‌داند و تا زمانی که مصرف نشده، از خرید دوباره آن محصول توسط کاربر جلوگیری می‌کند (یعنی شناسه محصول‌‌هایی که هنوز مصرف نشدند با شناسه محصولاتی که کاربر قصد خرید آن‌ها را دارد، یکی باشد). شما باید نحوه مصرف محصولات در برنامه‌ خود را کنترل کنید و مایکت را برای مصرف مطلع سازید تا محصول دوباره برای خرید مهیا شود. شما همچنین می‌توانید به راحتی فهرست محصولاتی که کاربر مالک آن‌ها است را از مایکت بخواهید. این زمانی مفید خواهد بود که مثلا بخواهید هنگام باز شدن برنامه خود بر اساس این محصولات به کاربر سرویس مناسبی ارائه دهید.

فرض کنید که برنامه شما باز شده و با موفقیت به سرویس مایکت متصل گردیده است. کاربر وارد صفحه فروشگاه برنامه می‌شود و لیستی از محصولات شما را به همراه قیمت‌های آن‌ها مشاهده می‌کند (لیستی که با استفاده از `sku_details` در کلاس `Inventory` ساخته‌اید). در کنار هر آیتم لیست، یک دکمه خرید وجود دارد. کاربر با زدن دکمه خرید وارد فرایند خرید آن محصول می‌شود. شروع فرایند خرید تمامی محصولات درون‌برنامه‌ای با استفاده از متد `launch_purchase_flow` در کلاس `IABHelper`، به صورت زیر است:

گودو 3:
```gdscript
	iab_helper.launch_purchase_flow(sku, payload)
	iab_helper.connect("on_iab_purchase_finished",self,"_on_iab_purchase_finished",[],CONNECT_ONESHOT)

func _on_iab_purchase_finished(is_succeed : bool, result : String, purchase : Purchase):
	...
```
گودو 4:
```gdscript
	iab_helper.launch_purchase_flow(sku, payload)
	iab_helper.on_iab_purchase_finished.connect(_on_iab_purchase_finished,CONNECT_ONE_SHOT)

func _on_iab_purchase_finished(is_succeed : bool, result : String, purchase : Purchase):
	...
```
متد `launch_purchase_flow` پارامتر‌های زیر را به ترتیب می‌پذیرد:

متغیر `sku`: شناسه محصول درون‌برنامه‌ای که قصد خرید آن را دارید.

متغیر `payload`: اطلاعات اضافی که مایل هستید مایکت به همراه اطلاعات خرید برای شما برگرداند، استفاده می‌شود.

سپس `IABHelper` پس از بررسی `Signature` خرید با کلید رمز عمومی شما، `on_iab_purchase_finished` را فراخوانی می‌کند. در متد `on_iab_purchase_finished` دو پاسخ `result` و `purchase` باز می‌گردد. `result` نشان می‌دهد که خرید موفق انجام شده است یا خیر و در صورت موفقیت، مشخصات محصول خریداری شده در `purchase` قرار داده می‌شود.

پس از اینکه خرید با موفقیت انجام شد، با توجه به مدل محصول، آن را به کاربر تحویل دهید. اگر محصول برنامه شما مصرف‌نشدنی یا اشتراکی است باید آن را پس از on_iab_purchase_finished موفقیت آمیز به کاربر تحویل دهید. در صورتی که محصول برنامه شما مصرف‌شدنی است ابتدا باید آن را `consume_async` کنید و سپس به کاربر تحویل دهید. `consume_async` کردن یک محصول باعث می‌شود که آن محصول، در لیست `Inventory` (محصولات به‌روز شده) بازنگردد، 
بنابراین محصولات مصرف‌نشدنی و اشتراکی را `consume_async` نکنید.

## مصرف نمودن یک خرید
هنگامی که یک کاربر محصولی را خریداری می‌کند، مالک آن تلقی می‌شود و امکان خرید دوباره آن را نخواهد داشت. شما باید یک درخواست مصرف به مایکت ارسال کنید تا مایکت اجازه خرید دوباره محصول را به کاربر بدهد.

برای مصرف کردن یک محصول، از متد `consume_async` به همراه یک شناسه محصول و اتصال به سیگنال `on_consume_finished` اقدام کنید:

گودو 3:
```gdscript
	iab_helper.consume_async(sku)
	iab_helper.connect("on_consume_finished",self,"_on_consume_finished",[],CONNECT_ONESHOT)

func _on_consume_finished(is_succeed : bool, result : String, purchase : Purchase):
	...
```
گودو 4:
```gdscript
	iab_helper.consume_async(sku)
	iab_helper.on_consume_finished.connect(_on_consume_finished)

func _on_consume_finished(is_succeed : bool, result : String, purchase : Purchase):
	...
```
در سینگال `on_consume_finished` دقیقا مانند `on_iab_purchase_finished`، یک `purchase` برای مشخصات خرید و یک `result` برای نتیجهٔ `consume_async` برمی‌گردد. در صورتی که `is_success` برابر `true` بود، می‌توانید محصول را به کاربر تحویل دهید (مثلا سکهٔ او را افزایش دهید).

این مسئولیت شماست که مزایای هر محصول را به کاربر ارائه دهید. به عنوان مثال اگر یک کاربر در بازی شما پول یا سکه خریداری کند، شما باید مقدار پول یا تعداد سکه او را بروز رسانی نمایید.

