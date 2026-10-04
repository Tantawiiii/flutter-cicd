# الإعداد مرة واحدة وطريقة تشغيل كل مشروع

## أولًا: إعداد المحرك المركزي

1. أنشئ ريبو `flutter-cicd` على حسابك أو المؤسسة. هذه الحزمة توضع فيه كوثائق أولية.
2. نفّذ ملفات `flutter-ci.yml` و`flutter-release.yml` تحت `.github/workflows` وفق العقد في implementation.md، مع scripts مركزية.
3. ثبّت Flutter وJava وXcode وFirebase CLI بإصدارات محددة ومتوافقة؛ تجنّب الاعتماد على stable المتغير في builds التي تحتاج إعادة إنتاج.
4. اختبر على تطبيق تجريبي Android وiOS، ثم أنشئ release للمحرك. أمثلة `@v1` توضيحية؛ ثبّت commit SHA كاملًا في مشاريع الإنتاج وحدثه بشكل مقصود.
5. فعّل وصول repos التابعة إلى reusable workflows؛ إذا المحرك private، اضبط سياسة access وأيضًا صلاحية checkout للمحرك.

## ثانيًا: إعداد أي مشروع Flutter مرة واحدة

1. جهّز Android productFlavors وiOS schemes/configurations للـdev/prod. iOS schemes يجب أن تكون shared. اسم flavor في config يجب أن يطابق المشروع فعلًا.
2. أضف `lib/main_dev.dart` و`lib/main_prod.dart` أو استخدم entrypoint واحدًا مع defines واضحة.
3. استخدم package/bundle IDs مختلفة، مثل `com.example.app.dev` و`com.example.app`. اسم وأيقونة Dev مختلفان حتى يقدر الفريق يثبت النسختين ويميزهما.
4. سجّل تطبيق Firebase لكل platform/environment وحط google-services.json وGoogleService-Info.plist في المسارات الصحيحة لكل flavor. جهّز tester groups. App Distribution ليست بديلًا عن إعداد runtime Firebase في التطبيق.
5. انسخ examples/project.yaml إلى `cicd/project.yaml` وعدّل الاسم والإصدارات والـentrypoints وFirebase app IDs. domain اختياري ويمكن أن يكون فارغًا.
6. اقرأ API_BASE_URL وAPP_ENV داخل Dart باستخدام `String.fromEnvironment`. لا تحط secrets في dart-define: القيم موجودة داخل التطبيق المبني.
7. أنشئ GitHub Environments بأسماء dev وprod في **ريبو التطبيق**. اجعل release/build/deploy jobs تشير إلى environment المختار. الإعدادات دي جزء من التنفيذ المطلوب، وليست موجودة تلقائيًا في مثال caller.
8. أضف signing secrets وFirebase credential حسب الجدول أدناه. أضف tokens القنوات التي ستشغلها فقط.
9. انسخ examples/release-caller.yml إلى `.github/workflows/release.yml` واستبدل YOUR_ORG وENGINE_SHA الحقيقي.
10. شغل build-only dev Android أولًا، ثم Firebase Android، ثم iOS، ثم prod. وثّق نجاح الحالات قبل توسيع الاستخدام.

## بيانات تحتاجها لكل مشروع

| الاسم المقترح | أين يستخدم؟ |
|---|---|
| ANDROID_KEYSTORE_BASE64 | فك keystore في runner مؤقت |
| ANDROID_KEYSTORE_PASSWORD / ANDROID_KEY_PASSWORD / ANDROID_KEY_ALIAS | Gradle signing config |
| IOS_CERTIFICATE_BASE64 / IOS_CERTIFICATE_PASSWORD | استيراد شهادة distribution في keychain مؤقت |
| IOS_PROVISION_PROFILE_BASE64 | profile متوافق مع bundle ID وexport method |
| IOS_KEYCHAIN_PASSWORD | keychain مؤقت يزال بعد البناء |
| FIREBASE_SERVICE_ACCOUNT_JSON | ADC للـFirebase CLI؛ أو استبدله بـOIDC/WIF بعد إعداد العلاقة |
| TELEGRAM_BOT_TOKEN / TELEGRAM_CHAT_ID | إرسال Telegram |
| WHATSAPP_ACCESS_TOKEN / WHATSAPP_PHONE_NUMBER_ID / WHATSAPP_TO | WhatsApp Business Cloud API |
| ENGINE_READ_TOKEN | فقط إذا checkout المحرك private يحتاج credential مستقلًا |

يمكن organization secrets إتاحتها لمجموعة repos، لكن لا تخلط signing/Firebase identities بين التطبيقات. Secrets الخاصة بالـEnvironment ليست تمريرًا تلقائيًا عبر workflow_call؛ الوظيفة داخل المحرك التي تختار environment تقرأها هناك. بديل آخر: repo/org secrets تمرر صراحة في عقد workflow_call. اختر نموذجًا واحدًا واضحًا؛ المثال يستخدم secrets: inherit داخل نفس organization/user فقط.

## التشغيل المتكرر

Actions → Flutter Release → Run workflow. اختار الفرع والإعدادات واضغط Run. أو بعد ربط gh:

```bash
gh workflow run release.yml --repo YOUR_ORG/YOUR_APP --ref main \
  -f environment=dev -f platform=android -f artifact=apk \
  -f version_type=patch -f distribution=firebase -f notifications=telegram
```

هذا الأمر صالح **بعد تنفيذ المحرك وربط caller**. لا يحتاج إعادة إعداد flavors أو signing في كل مرة.
