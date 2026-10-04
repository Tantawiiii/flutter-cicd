# Flutter CI/CD Blueprint

مرجع عربي لبناء CI/CD مركزي يعاد استخدامه في مشاريع Flutter.

**الحالة:** ريبو شرح وتصميم وأمثلة تكامل، وليس محرك deployment مكتملًا. ملفات YAML داخل examples لا تشغّل pipeline قبل تنفيذ المحرك المركزي واستبدال القيم التجريبية. لا توجد أي credentials أو رسائل أرسلت أو نسخ رفعت في هذه الحزمة.

## ما الذي نبنيه؟

Flutter + GitHub Actions reusable workflows + Dev/Prod + customizable versioning + Firebase App Distribution + Telegram/WhatsApp configurable notifications + AI Agent interface.

كل مشروع يحتفظ بإعداداته وهوياته وتوقيعه. المحرك المركزي يحتفظ بمنطق التحقق والبناء والتوزيع والإشعارات. إعداد المحرك مرة واحدة لا يلغي إعداد Firebase وsigning الخاص بكل تطبيق.

## اقرأ بالترتيب

1. [Architecture والفلو](docs/architecture.md)
2. [إعداد المحرك والمشاريع](docs/setup.md)
3. [أنواع الإصدارات](docs/versioning.md)
4. [الإشعارات](docs/notifications.md)
5. [ربط AI Agent](docs/agent.md)
6. [عقد المحرك وخطة التنفيذ](docs/implementation.md)
7. [التشغيل والأخطاء](docs/operations.md)
8. [المراجع الرسمية](docs/references.md)

## مثال الاستخدام بعد تنفيذ المحرك

من Actions → Flutter Release → Run workflow: اختار dev أو prod، android أو ios أو all، patch/minor/major/custom، build-only أو firebase، والإشعارات config/telegram/whatsapp/both/none.

المشروع الجديد يضيف ملف `cicd/project.yaml` وملف caller في `.github/workflows/release.yml`. أمثلة الاثنين في [examples](examples).

## حدود النسخة الأولى

Android APK/AAB وiOS IPA، وتوزيع اختباري عبر Firebase. اختيار prod يعني إعدادات الإنتاج، وليس النشر تلقائيًا على Google Play أو App Store. المتاجر وFlutter Web إضافات لاحقة. وجود domain اختياري؛ فصل البيئات يعتمد أيضًا على app IDs وFirebase وflavors.
