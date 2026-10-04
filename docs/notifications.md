# Telegram وWhatsApp

## التحكم

| الاختيار في Run workflow | النتيجة |
|---|---|
| config | إعداد القنوات حسب البيئة في project.yaml |
| telegram | Telegram فقط |
| whatsapp | WhatsApp فقط |
| both | الاثنان |
| none | بدون إشعارات |

الـoverride يحدد القنوات، وon_success/on_failure في config يحددان أحداث الإرسال. failures تشمل partial failure، ويمكن إضافة on_cancelled لاحقًا كإعداد مستقل. فشل إرسال قناة لا يمنع تجربة القناة الأخرى.

## الرسالة الموحدة

```text
✅ تم رفع إصدار
التطبيق: Herbandoil
البيئة: dev | القناة: beta
النسخة: 1.5.0+83
Android: uploaded | iOS: uploaded
التوزيع: Firebase App Distribution
المصدر: commit SHA
بواسطة: GitHub actor
المدة: total elapsed
Logs: GitHub Actions run URL
```

عند الفشل: الحالة failure/partial، والمنصة والمرحلة failed_stage، وملخص خطأ منقح. لو version لم يتحدد بسبب فشل validation، اكتب «لم يحدد» بدل رقم وهمي. نجاح build-only رسالته «تم البناء» وليس «تم الرفع». رابط تنزيل Firebase لا يضاف إلا إن رجعه deployment بشكل موثوق.

## Telegram مرة واحدة

أنشئ bot عبر BotFather، واحفظ token في Secret. افتح محادثة معه واضغط Start، أو أضفه إلى مجموعة لها صلاحية الإرسال. احفظ chat ID الصحيح. استخدم Bot API `sendMessage` بنص عادي لتجنب أخطاء Markdown escaping. حد النص 4096 حرفًا؛ اختصر تفاصيل الخطأ وأرسل logs URL. لا تستخدم token في CLI logs ولا تطبع request URL لأنه يحتوي token.

## WhatsApp مرة واحدة

جهّز WhatsApp Business Platform/Cloud API ورقم إرسال وaccess token وصلاحياته. recipients يوافقون على استقبال الإشعارات. إشعارات CI قد تكون خارج نافذة خدمة 24 ساعة: جهّز approved template مناسبًا للنجاح والفشل، باللغة وبترتيب parameters المطابقين، بدل افتراض إمكانية text message دائمًا.

خزّن graph API version وtemplate name/language في config، وtoken/phone ID/recipient في Secrets. تحقق من API version مدعوم وقت التنفيذ؛ لا تستخدم قيمة latest مخفية.

قبول API للرسالة لا يثبت وصولها للمستخدم. لو محتاج delivery status، أضف webhook provider لاحقًا؛ workflow يوثق accepted/failed فقط.

## سياسة الفشل

افصل deployment_status عن notification_status. إذا Firebase نجح وWhatsApp فشل، النسخة مازالت مرفوعة. سجّل warning واضحًا في job summary. جرّب retry محدودًا مع timeout/backoff لـ429 و5xx، واحترم Retry-After. لا تعيد الرفع بسبب فشل إشعار. network timeout قد يعني وصول رسالة بالفعل؛ retries قد تكررها، فلا تدّع exactly-once.
