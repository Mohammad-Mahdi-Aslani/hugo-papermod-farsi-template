---
title: پست تست
date: 2024-6-21
draft: false
fatags:
  - تست
---
#  ساخت API برای Archillect با استفاده از Cloudflare Workers -  اینترنردز 
![](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/ptispulhzpue.png)

توی این پست با استفاده از Cloudflare Workers که یک سرویس Edge Computing بدون‌سرور (serverless) با جاوا اسکریپت و APIای مشابه ServiceWorker‌هاست، با استفاده از اسکریپتی که در زمان پاسخ به ریکوئست اجرا می‌شه، دو API Endpoint-طور می‌سازیم و یک دکمه دانلود به صفحه اضافه می‌کنیم.  

نتیجه نهایی رو می‌تونید در [archillect.mhsattarian.workers.dev](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Farchillect.mhsattarian.workers.dev%2F&si=hn4cbntamacf&st=post&k=Bk3WJPTLtITJHIHVPEQkabtU4kwtKcwBlDzs%2BJxpYPY%3D) و کد‌ها رو در [ریپازیتوری گیت‌هاب](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fgithub.com%2Fmhsattarian%2Farchillect-api&si=hn4cbntamacf&st=post&k=NHbnXSvk7f%2BmTwNYGuR5XT3Rfx1y%2BWvTyKgOd5mFVC8%3D) ببینید.

* * *

قبل از توضیح پروژه و انجام اون، تو این بخش ‌به‌صورت مختصر به معرفی سرویس‌ها و مفاهیمی که دونستن اون‌ها ‌بد نیست و در درک چگونگی کارکرد پروژه کمک می‌کنه می‌پردازم. اگر با مفاهیم/سرویس‌های بدون‌سرور، Edge Computing و Cloudflare و سرویس ورکر‌ها (service workers) آشنایی دارین می‌تونین این بخش رو رد کنین.

![پروژه Archillect](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/ryxiadtaqynx.png)

پروژه Archillect

[پروژه Archillect](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Farchillect.com%2F&si=hn4cbntamacf&st=post&k=leaIYU9HIGYi3reip%2FHbwPuKFJ8Tu82xTROV84j9ZMs%3D) که اسم اون از ترکیب دو کلمه Archive و Intellect گرفته شده، یک ربات \[برنامه\] هوش مصنوعی (AI) است که ساخته شده تا نوعی محتوی تصویری رو در شبکه‌های مختلف به اشتراک بذاره. پست‌هایی که انتخاب می‌کنه معمولا از Tumblr هستن و تا الان توی [توییتر](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Ftwitter.com%2Farchillect&si=hn4cbntamacf&st=post&k=rpwowUkl3iS5Xd35Th%2BN0fJNZx3ouOrdzVXIn3PWtuY%3D)، [تلگرام](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Ft.me%2Farchillect%2F&si=hn4cbntamacf&st=post&k=jEF3u4NoSE8HSWr%2BVj7qcGGzQ%2FLu%2BbBDq63mmy%2BNuAs%3D)، [اینستاگرام](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Finstagram.com%2Farchillect&si=hn4cbntamacf&st=post&k=zZUK0%2B1ZoOS90SmleKKS9c%2Fqv9w7EQvz8TSMxKYHYwg%3D) و چند شبکه دیگه منتشر می‌شن.

> علاوه‌بر ربات توییتر Archillect، ربات دیگری به اسم Archillect Links هم لینک هر عکسی که منتشر می‌شه رو زیر همون عکس رو ریپلای می‌کنه.

![](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/5ttdftkvzrlq.png)

در ساده‌ترین حالت، ‏[Cloudflare](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fwww.cloudflare.com%2F&si=hn4cbntamacf&st=post&k=noylpJo3zE22p5FGutfOOaMUP4wbZ3drhEQsNYVzMBc%3D) یک HTTP Cache هست که در حال حاضر در ۲۰۰ موقعیت در سرتاسر جهان درحال اجراست. اما Cloudflare سرویس‌های بیشتری از محدود ویژگی‌هایی که استاندارد HTTP برای HTTP Cacheها مشخص می‌کند ارائه می‌دهد، مثل DNS و SSL، مقابله با حملات،‌توزیع بار و موارد دیگه. ‏Cloudflare با ارائه سرویس‌هایی مثل **مدیریت DNS رایگان،** افزونه‌های شخص‌ثالث برای پروسس ریکوئست‌ها و سرویس‌هایی مانند **[1.1.1.1 و Warp](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2F1.1.1.1%2F&si=hn4cbntamacf&st=post&k=WYHOKD7Y668u08Gx4eesAR2YaD%2B1XahIk9NU75w2WbI%3D)** امروزه بسیار مورد استفاده و قدرتمنده.

> برای آشنایی با مدل اجرای بدون‌سرور (serveless) پیشنهاد می‌کنم قسمت «پردازش ابری (Cloud Computing)» از مقدمه پست «[تبدیل توییت به عکس با استفاده از توابع بدون سرور Netlify](https://virgool.io/internerds/tweet2img-euitzv4omwr6)» رو مطالعه کنین.

آمازون AWS به عنوان یکی از بزرگترین سرویس دهنده‌های مدل اجرای بدون‌سرور (serverless) در نقاط مختلفی از دنیا مزارع‌سرورهای (server farm) زیادی، به‌صورت دقیق‌تر [۷۷ مرکز](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fwww.infrastructure.aws%2F&si=hn4cbntamacf&st=post&k=ZaplKrDsqMXiZEY6S%2FxDabBca2HQcHyluSjeTdELxVE%3D)، برای پوشش سرویس‌هاش ایجاد کرده. نقشه پوشش این سرور‌ها:

![نقشه پوشش جغرافیایی سرورهای Amazon AWS](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/y2omruq5ayri.png)

نقشه پوشش جغرافیایی سرورهای Amazon AWS

در نظر بگیرید اگر کاربری در غرب آفریقا درخواستی برای یکی از محتوی/سرویس‌های AWS ارسال کند،‌ پکت‌های شبکه (packets) راه زیادی برای فراهم کردن این محتوی/سرویس طی می‌کنند.

سرویس‌دهنده‌های دیگه نیز مثل Cloudflare و Fastly، با وجود اینکه نسبت به آمازون شرکت‌های بسیار کوچک‌تری محسوب می‌شن، به دلیل نیاز به پوشش سرتاسری و بهبود سرویس‌های DNS و CDNاشون زیرساخت‌های سرور بزرگ و زیادی در سرتاسر دنیا دارند. نقشه پوشش سرور‌های این دو کمپانی:

![نقشه پوشش جغرافیایی سرورهای Cloudflare](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/0gbbdrsqqmx8.png)

نقشه پوشش جغرافیایی سرورهای Cloudflare

![نقشه پوشش جغرافیایی سرورهای Fastly](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/g9baw6cvdmqj.png)

نقشه پوشش جغرافیایی سرورهای Fastly

به دلیل نیاز به یک شبکه جهانی برای ارائه CDN و ملاحظات امنیتی، این کمپانی‌ها سرور‌های بسیار بیشتری در گستره جغرافیایی وسیع‌تری در سراسر جهان‌دارند. در نهایت، تمامی این مراکز ساختمون‌های بزرگی با تعداد زیادی سرور هستند که نتیجه ترکیب اون‌ها یک شبکه عظیمه که امکان اجرا در تقریبا همه‌جا (از نظر فیزیکی)  
\-در لبه (Edge) دریافت و پروسس درخواست‌ها- رو میده. چیزی مشابه Write Once, Run Anywhere از نظر فیزیکی.

برخلاف سرویس AWS آمازون که برنامه‌ها در مراکز مشخص با هزینه مشخص اجرا می‌شن؛ این دو کمپانی با همکاری هم امکانی ایجاد کردند که برنامه‌ها یکبار نوشته شده و همزمان در تمامی این مراکز مستقر (deploy) بشن.

> البته آمازون سرویس مشابه به اسم Lambda Edge داره که امکانات کم و بیش مشابهی رو ارائه میده.

همچنین برای اجرای برنامه‌ها، بجای ایجاد زمان اجراهای (runtime) مختلف، اجازه می‌دهند که کدهای [web assembly](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fyoutu.be%2FHktWin_LPf4&si=hn4cbntamacf&st=post&k=JC%2B3xXMBr9ZP0j9Vnx7dJIGi3sT4IGb%2FSI3tHlum9%2BI%3D) رو در Edge اجرا کنیم.

![](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/xyzfzphynrmt.png)

اگر با جاوا اسکریپت آشنا باشین، احتمالا با مفهوم workerها و به‌ویژه service workerها آشنا هستین؛ ورکر‌ها در حالت کلی اسکریپت‌هایی هستند که قراره در یک ترد (thread) مجزا اجرا بشن و اجرای برنامه اصلی رو که تنها در یک ترد قابل اجراست مختل نکنن.

سرویس ورکر‌ها نوع خاصی از ورکرها هستند که وظایف مخصوصی در قبال Web App کردن وب‌سایت دارن، وظایفی مثل کش کردن (برای استفاده آفلاین)، ارسال ناتیفیکیشن (push message)، همگام‌سازی در پس‌زمینه (background sync) و موارد مثل این. به بیان دیگه سرویس ورکر‌ها یک پراکسی قابل برنامه‌ریزی (programmable network proxy) هستند که اجازه می‌دهند چگونگی مدیریت درخواست‌های (requests) صفحه رو کنترل کنیم.

[ورکرهای کلودفلر (‏Cloudflare Workers)](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fworkers.cloudflare.com%2F&si=hn4cbntamacf&st=post&k=PQo8N0m1fZW%2FAPpXrv2HQw%2FocWH4TvOS%2B0qB7%2B4zEGY%3D) یک راهکار بدون سرور (serverless) و مشابه سرویس‌ورکر‌ها برای edge computing هستند که برای خیلی‌کارها منجمله مدیریت درخواست‌ها استفاده میشن. برنامه‌های نوشته شده که به لطف [web assembly](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fyoutu.be%2FHktWin_LPf4&si=hn4cbntamacf&st=post&k=JC%2B3xXMBr9ZP0j9Vnx7dJIGi3sT4IGb%2FSI3tHlum9%2BI%3D) با زبان‌های javaScript، Rust و ++C قابل نوشته‌شدن هستن، در تمامی سرور نود‌های (Fastly + Cloudflare) دپلوی می‌شن. نتیجه تمامی این‌ها قابلیت نوشتن اسکریپت‌هاییست که به‌شدت قدرتمند هستن،‌ واقعا سریع اجرا می‌شن و انعطاف‌پذیری بسیار زیادی به برنامه‌نویس‌ها میدن.

استفاده از ورکر‌های Cloudflare با محدودیت‌های بسیار سخاوت‌مندانه _۱۰۰ هزار ریکوئست در روز_ و محدودیت‌هایی در حجم فایل‌های ذخیره سازی **رایگان** است.

* * *

به دلیل علاقه‌ای که به پروژه Archillect و محتوی‌ای که به اشتراک می‌ذاره داشتم، می‌خواستم یک API داشته باشم که با استفاده از اون بتونم از این محتوی توی پروژه‌های مختلف استفاده کنم (مشابه استفاده _[عکس‌های تصادفی](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fsource.unsplash.com%2F&si=hn4cbntamacf&st=post&k=9c4IwLJbAEE7rhY4%2BIQohtkoFIDCswZ2tLnppnR1FuE%3D)_ از [unsplash](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Funsplash.com%2F&si=hn4cbntamacf&st=post&k=cZ7gNb3xGZOZoRakWNMI0AwJZz4tvdhtjsbcphzEyJk%3D)) یا اون‌ها رو آرشیو کنم.

> این پروژه API داره اما متاسفانه عمومی نبوده و تنها برای حمایت‌کننده‌ها از طریق Patreon قابل استفاده است.

خوشبختانه عکس‌های Archillect با ID ای به‌ترتیب به‌اشتراک‌گذاری (شماره \[index\])، منتشر می‌شن و این امکان رو می‌داد تا با دونستن ID آخرین عکس به اشتراک گذاشته شده، ساخت یک index برای یک تصادفی خیلی راحت بشه.

با دونستن این مورد، اولین روشی که به‌ذهنم رسید استفاده از RSS Feed و ترکیب اون با سرویس‌هایی مثل [Integromat](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fwww.integromat.com%2F&si=hn4cbntamacf&st=post&k=hBkrkgmPVEowPqwLC%2F88FXxgVVFMpDrjpYSq5G6nScQ%3D) برای بدست آوردن ID آخرین محتوی به‌اشتراک‌گذاشته شده بود. اما Archillect هیچ Feedای ارائه نمی‌داد؛ برای همین سرویس‌هایی مثل [rss.app](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Frss.app%2F&si=hn4cbntamacf&st=post&k=uhOqPH2inPeBdVMHaQS1giS7Unyp4PoEun9n4H6%2FPJs%3D) رو تست کردم تا بتونم خودم برای اون یک Feed درست کنم. اما این سرویس‌ هم محدودیت‌های زیادی مثل محدود بودن تعداد feedها،‌ تبلیغات و به‌روز‌رسانی کند فیدها (هر ۲۴ ساعت) داشتن.

مدتی روش‌های مختلف دیگه‌ای رو تست کردم تا یاد این ویدئو از [Wes Bos](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fwesbos.com%2F&si=hn4cbntamacf&st=post&k=awPa%2FyzaeH2y8ftzpWAtTJoujUXNZ48NXwcom3ND7Pw%3D) افتادم:

> مشابه پست «[تبدیل توییت به عکس با استفاده از توابع بدون سرور Netlify](https://virgool.io/internerds/tweet2img-euitzv4omwr6)» در این پست هم از راهنمایی‌های این ویدئو کمک گرفته‌شده.

[https://youtu.be/48NWaLkDcME](https://youtu.be/48NWaLkDcME)

  
توی این ویدئو Wes Bos با استفاده از یک Cloudflare worker و اتصالش به دامنه خودش یک پروکسی برای دسترسی مستقیم به یک محتوی (عکس یا GIF) از یک صفحه می‌کنه. تقریبا مشابه کاری که ما قصد داریم بکنیم.

در این پروژه از nodejs استفاده می‌کنیم. اگر اون رو نصب ندارید می‌تونید از [سایت اصلی Nodejs](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fnodejs.org%2Fen%2F&si=hn4cbntamacf&st=post&k=OC3lSLsR1mT8wJ9%2BO981miWQTru0HjdJcGcCZWxXYLc%3D) و یا [با استفاده از مدیر بسته (package manger) سیستم‌عاملتون](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fnodejs.org%2Fen%2Fdownload%2Fpackage-manager%2F&si=hn4cbntamacf&st=post&k=f1xacxTgnU92TMGToh8ntsh8gh37aJMmteDyQQ701T4%3D) دانلود و نصب کنید.

> اگر اطمینان ندارید، ورژن LTS را نصب کنید، همچنین پیشنهاد می‌کنم از یک مدیر نسخه (version manager) برای nodejs مثل nvm یا n استفاده کنید:  
> اگر از لینوکس استفاده می‌کنید و node رو نصب ندارید، انتخاب خوبیه که از اول با نصب یکی از اونها اقدام به نصب node کنید. برای مثال با این دستور زیر n رو نصب کنید: curl -L [git.io/n-install](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fgit.io%2Fn-install&si=hn4cbntamacf&st=post&k=u%2BIIOkDNl2vtcEYyIDQdpAGYdx8LnKR8xf2C4hMP1IY%3D) | bash

پس از نصب با اجزای دستورات زیر تو ترمینال از درستی نصبشون مطمئن بشید:

```
node -v
```


```
npm -v
```


همچنین برای این پروژه لازم است در Cloudflare ثبت‌نام کرده و یک اکانت بسازید: [سایت Cloudflare](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fcloudflare.com%2F&si=hn4cbntamacf&st=post&k=5jOflYv94ktb52UOUHYWQjvLZDnX5szUbEX0OvIg9gQ%3D)

* * *

برای ساختن یک Worker می‌تونید وارد حساب Cloudflare خودتون بشین و روی گزینه ورکر از منوی سمت راست کلیک کنین:

![ساخت Worker جدید](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/bjwrj7du9pwp.png)

ساخت Worker جدید

  
اما برای اینکه بتونیم به‌صورت لوکال پروژه رو تست کنیم و در نهایت به‌صورت اتوماتیک اون رو دپلوی کنیم، همچنین اینکه برای مشاهده نتیجه تغییرات مجبور به اعمال تغییرات روی سرور (دستی یا اتوماتیک) نباشیم و همچنین از محدودیت ۱۰۰هزار ریکوئست در روز خرج نکنیم از ابزاری که Cloudflare معرفی کرده به اسم [Wrangler](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fgithub.com%2Fcloudflare%2Fwrangler&si=hn4cbntamacf&st=post&k=stP8f8elLKYD51n6%2BFK%2B7nyiL1rjpuJZUaOYNp2d9bw%3D) استفاده می‌کنیم.

‏wrangler ابزاری تحت خط‌فرمان است که به راحتی با npm نصب میشه:

```
npm install -g @cloudflare/wrangler
```


بعد از نصب نیاز است تا با وارد کردن دستور زیر احراز هویت انجام بدیم:

```
wrangler config
```


بعد از اجرای این دستور مراحل احراز هویت توضیح داده میشه که به‌صورت خلاصه نیاز هست تا وارد اکانت خودتون بشین و از قسمت توکن‌ها یک توکن ایجاد کنین و اون توکن رو در ترمینال وارد کنید.

  
حالا برای ایجاد پروژه از دستور زیر استفاده می‌کنیم. ایجاد پروژه جدید محدود به استفاده از این روش و قالب پیش‌فرض خود کلودفلر نیست و می‌تونید خودتون پروژه‌ای مشابه رو ایجاد کنید، اما برای سادگی و سرعت از همین دستور استفاده می‌کنیم:

```
wrangler generate my-app
# or
wrangler generate my-app https://github.com/cloudflare/worker-template-router
```


> در اینجا نامی که به پروژه می‌دهیم my-app است و در دایرکتوری‌ای با همین نام ساخته خواهد شد.

بعد از ساخته شدن قالب پروژه، پیامی مشاهده می‌کنیم که نیاز است فیلد‌هایی در فایل wrangler.toml رو تغییر بدیم. برای اینکار وارد دایرکتوری پروژه شده و این فایل رو باز می‌کنیم.

وارد اکانت کلودفلر خودتون بشید و مانند تصویر «ساخت Worker جدید» از منوی سمت راست Workers رو انتخاب کنید. در ستون سمت راست داخل کادر می‌تونین account\_id خودتون رو پیدا کنید.

![](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/gatlry8msprx.png)

این مقدار رو کپی‌کرده و داخل فایل wrangler.toml در قسمت account\_id قرار بدین. توجه ‌کنید که این مقدار باید به‌صورت string باشد، بنابراین کاراکتر‌های " را حذف نکنید.

> مستندات و نحوه نصب و آماده‌سازی دقیق‌تر رو می‌تونید از [این لینک](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fdevelopers.cloudflare.com%2Fworkers%2Fquickstart&si=hn4cbntamacf&st=post&k=h6NxoviaeRVy2tXl1cEnP9rrA2lCjQsiH7XUFxJ9Rig%3D) بخونید.

فایل index.js رو باز کنید. این فایل فایل اصلی پروژه است که در ابتدا اجرا می‌شود. مقدار این فایل مشابه زیر است:

> فایل اصلی پروژه با تغییر مقدار پراپرتی main داخل فایل package.json قابل تنظیم است.

```
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  return new Response('Hello worker!', {
    headers: { 'content-type': 'text/plain' },
  })
}
```


همونطور که مشاهده می‌کنید، مشابه service workerهای جاوا اسکریپت عمل‌کرده و با دریافت یک ریکوئست (که با استفاده از eventListenerای که به اونت fetch متصل کرده متوجه اون می‌شه) و پروسس اون، ریسپانس (جواب \[response\]) رو ارسال می‌کنه.

برای اجرا پروژه کافیه از دستور زیر استفاده کنیم:

```
wrangler preview
```


با اجرای این دستور یک محیط تست (playground) پروژه در مرورگر باز می‌شود:

![](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/36g3hhoyk9ou.png)

> آدرسی که در آدرس‌بار مشاهده می‌کنید به‌عنوان آدرس این ورکر در نظر گرفته می‌شود.

در ساده‌ترین حالت در این پروژه، می‌خواهیم سایت archillect بدون تغییر نمایش داده بشه. برای اینکار بدین صورت عمل می‌کنیم که ریکوئست دریافت شده رو گرفته و آدرس اون رو به سایت Archillect تغییر میدیم و ریکوئست جدید رو (که با تابع fetch جاوا اسکریپت ایجاد کردیم) به‌عنوان ریسپانس برمی‌گردونیم.

این اسکریپت مشابه یک پروکسی برای Archillect عمل کرده و با مشاهده آدرس این ورکر، انگار سایت Archillect رو باز کردیم:

```
بعد از هربار تغییر کد‌ها از دستور wrangler preview برای مشاهده و اجرای ورکر استفاده کنید.
```


![ایجاد پروکسی به سایت Archillect](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/dawqltctemuz.png)

ایجاد پروکسی به سایت Archillect

پیش از این گفتم که هر تصویر یک id که نشان‌دهنده index اون هست داره و با اضافه‌کردن این id به انتهای آدرس سایت Archillect می‌تونیم صفحه مربوط به اون عکس رو مشاهده کنیم.  
الان می‌تونیم با اضافه کردن id در انتخاب آدرس ورکر خودمون (که اینجا از آدرس example.com برای نمایش اون استفاده می‌شه) صفحه تصویر رو ببینیم:

![](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/0e7q9aatv0ft.png)

حالا می‌خواهیم یک آدرس در اختیار داشته باشیم که با وارد کردن اون و اضافه کردن id‌ تصویر، فایل تصویر رو در اختیار داشته باشیم. برای اینکار نیاز هست تا در هنگام دریافت ریکوئست و قبل از ارسال ریسپانس، یعنی زمانی نتیجه ریکوئست ما (ریکوئست تغییر مسیر داده شده) به Archillect مشخص میشه، اون رو پروسس کنیم.

نتیحه این ریکوئست طبیعتا یک فایل HTML ‌است؛ برای پروسس این فایل و بدست آوردن یا تغییر مقدار‌هایی که می‌خوایم می‌تونیم از توابع دستکاری string خود javascript استفاده کنیم و یا از API ای که Cloudflare داخل Workerهاش در اختیارمون می‌ذاره به‌نام [HTMLRewriter](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fdevelopers.cloudflare.com%2Fworkers%2Freference%2Fapis%2Fhtml-rewriter%2F&si=hn4cbntamacf&st=post&k=0UUeQMXRJn7PE5eoyKD2Z7kjDXUq3cfOnerZGQfxk0s%3D) استفاده کنیم.

در این قسمت از روش اول استفاده می‌کنیم، برای اینکه بتونیم فایل عکس‌ها رو در آدرسی مثل آدرس زیر در اختیار داشته باشیم، نیاز داریم تا در‌خواست‌ها به آدرس img/ رو به صفحه عکس (آدرس بدون img/) منتقل کرده و با دریافت نتیجه به‌صورت یک فایل HTML،‌ اون رو برای پیدا کردن آدرس عکس پروسس کنیم و در نهایت به آدرس اصلی عکس ریکوئست زده و نتیجه رو برگردانیم.

```
# <worker-address>/<image_id>/img
e.g. example.com/288580/img
```


برای گرفتن آدرس اصلی تصویر، در نتیجه ریکوئست‌مون به Archillect (خط ۲۷)، دنبال متا تگ مربوط به open graph با عبارت og:image می‌گردیم که ساختار زیر رو داره (خط ۳۱) و سپس مقدار content اون رو استخراج می‌کنیم:

```
<meta property=&quotog:image&quot content=&quothttps://66.media.tumblr.com/be97eb9f660e30c01b38b1bbbba1d9e6/tumblr_phi9bhxWnj1vyjf2do1_640.jpg&quot>
```


![ساخت endpointای برای دریافت فایل تصویر](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/fnukhvilqgye.png)

ساخت endpointای برای دریافت فایل تصویر

در نتیجه با اضافه کردن img/ در انتهای آدرس هر تصویر،‌می‌تونیم فایل اون تصویر رو در اختیار داشته باشیم.

> به سرعت بی‌نظیر اجرای این workerها توجه کنید.

حال می‌خواهیم به صفحه تصویر (صفحه تصویر در Archillect) یک دکمه دانلود اضافه کنیم. برای اینکار، مشابه قسمت قبلی نیاز است تا فایل HTML صفحه رو پروسس کنیم این‌بار از روش دوم استفاده می‌کنیم.

فرم کلی استفاده از HTMLRewriter به صورت زیر است:

```
new HTMLRewriter()
  .on('*', new ElementHandler())
  .onDocument(new DocumentHandler())
  .transform(res)
```


برای استفاده از HTMLRewriter، یک instance جدید از این کلاس درست می‌کنیم و با استفاده از متد on و دادن یک سلکتور (Selector) به فرمت سلکتور‌های CSS و یک کلاس مثل ElementHandler به فرم زیر که باید یک یا چند متد element، coments و text به ترتیب برای هندل کردن المان‌های صفحه، کامنت‌ها و متن‌های صفحه پیاده‌سازی کنه یک متد خودمون برای اعمال تغییرات روی صفحه رو می‌سازیم:

```
class ElementHandler {
   element(element) {
      // An incoming element, such as `div`
      console.log(`Incoming element: ${element.tagName}`)
   }
    comments(comment) {
      // An incoming comment
    }
    text(text) {
      // An incoming piece of text
    }
 }
```


سپس با صدا کردن متد transform و دادن مقدار متن HTML خودمون تغییرات را اعمال می‌کنیم.

نتیجه در تصویر زیر قابل مشاهده است.

> به دکمه دانلود در پایین سمت راست تصویر دقت کنید.

![اضافه کردن دکمه دانلود](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/faarscoj1qjh.png)

اضافه کردن دکمه دانلود

برای داشتن endpointای که بتونیم با استفاده از اون ID آخرین عکسی که تا الان منتشر شده رو بگیریم، مشابه قسمت‌های قبل کافیست المان آخرین تصویر را پیدا کرده و شماره ID را از آن استخراج کنیم:

![دریافت ID آخرین تصویر به‌اشتراک گذاشته شده](https://files.virgool.io/upload/users/5179/posts/hn4cbntamacf/11p6mxi4ahh2.png)

دریافت ID آخرین تصویر به‌اشتراک گذاشته شده

*   تصویر پس‌زمینه: کاور از [editorX](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fwww.editorx.com%2F&si=hn4cbntamacf&st=post&k=NmeYPYGWkeTGNk1B%2Bpa9lOi2TkY%2Bopqn%2FqJWtNu9p8Y%3D) و [فونت از صابر راستی‌کردار](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Frastikerdar.github.io%2Fvazir-code-font%2F&si=hn4cbntamacf&st=post&k=rUdNR7CQtxXmEzL2yLSFQcJO6B5PF6f2%2Blps4Gn8%2BxA%3D)
*   مطالب مربوط به بخش «Edge Computing» از [ارائه Steve Klabnik](https://l.vrgl.ir/r?ad=1&l=https%3A%2F%2Fyoutu.be%2FCMB6AlE1QuI&si=hn4cbntamacf&st=post&k=xVOnlJu%2BWYUTnkeZ1%2BAQh904bNvhRSagdY5cm9VXJV4%3D)