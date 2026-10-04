# التشغيل والأخطاء

| المشكلة | التشخيص والخطوة التالية |
|---|---|
| Flavor غير موجود | طابق config مع Gradle وshared Xcode schemes |
| Dev يتصل بـProd | تحقق من Dart defines وFirebase runtime files وapp IDs؛ domain وحده غير كافٍ |
| analyze أو tests فشلت | توقف قبل signing/upload؛ اصلح المصدر وشغل جديدًا |
| Android signing failed | تحقق alias/password وGradle signing config دون طباعة الأسرار |
| iOS profile mismatch | طابق bundle ID وteam/certificate وexport method وUDIDs |
| Firebase permission denied | تحقق ADC/IAM ومشروع app ID |
| Telegram لا يرسل | Start/chat ID/صلاحية bot في المجموعة |
| WhatsApp rejected | template status/language/parameters/token وAPI version |
| Android نجح وiOS فشل | اكتب partial؛ لا تنشئ نجاحًا عامًا، ولا تحذف Android تلقائيًا |
| Notification فشل | احتفظ بحالة deploy الصحيحة وأعد notification وحدها |

الـjob summary يعرض النسخة والـbuild وكل target وfailed stage وروابط artifacts وlogs. لا تنسخ raw stack traces إلى رسائل خارجية إذا فيها بيانات حساسة. artifacts لها retention وتحتاج صلاحية دخول إلى GitHub؛ لا تعتبرها رابط تنزيل عام دائم.

## إعادة التشغيل

حافظ على source SHA وengine SHA. إذا artifact صالح ارفع نفس الملف بعد التحقق من manifest وchecksum؛ إذا ستعيد البناء كإصدار مختلف احجز build جديدًا. لا تعمل major/minor bump مرة أخرى لمجرد أن notification فشل.

## الصلاحيات

CI على pull_request يقرأ الكود ويختبر دون توقيع أو إرسال رسائل. release على trusted ref فقط. لا تستخدم pull_request_target لبناء كود غير موثوق مع secrets. contents:write يحتاجه finalize فقط عند إنشاء tags/releases؛ caller يجب أن يسمح بالحد الأعلى المطلوب، والـjobs تقلل الصلاحيات المتاحة. نظّف keychains وملفات credentials في always cleanup.
