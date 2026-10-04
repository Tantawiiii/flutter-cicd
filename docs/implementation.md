# عقد المحرك وخطة التنفيذ

هذه الوثيقة تحدد ما ينفذه المطور قبل تفعيل examples. لا تضع caller داخل .github/workflows في ريبو الشرح نفسه.

## ملفات المحرك المستهدفة

| الملف | المهمة |
|---|---|
| .github/workflows/flutter-ci.yml | PR analyze/tests بلا deployment secrets |
| .github/workflows/flutter-release.yml | workflow_call، prepare، Android/iOS، finalize، notify |
| scripts/validate.py | schema validation، اختيار environment، رفض القيم غير الصحيحة |
| scripts/version.py | مصدر النسخة، reservation، SemVer |
| scripts/build_android.sh | signing + apk/aab |
| scripts/build_ios.sh | keychain/profile + ipa |
| scripts/deploy_firebase.sh | ADC + distribute |
| scripts/notify.py | adapters مستقلة للـTelegram/WhatsApp |
| agent/ | broker integration لاحقًا |

## workflow_call inputs

| input | القيم |
|---|---|
| environment | dev, prod |
| platform | android, ios, all |
| artifact | apk, aab, ipa, all |
| version_type | patch, minor, major, custom |
| custom_version | X.Y.Z، مطلوب مع custom |
| channel | dev, alpha, beta, rc, production |
| distribution | build-only, firebase |
| notifications | config, telegram, whatsapp, both, none |
| config_path | الافتراضي cicd/project.yaml |
| engine_ref | SHA المحرك نفسه، يطابق uses |

ارفض ios+apk وandroid+ipa وأي تضارب platform/artifact. all+all يبني APK/AAB/IPA، لكن اختَر ملف Android واحدًا للتوزيع من firebase_android_artifact حتى لا ترفع نسختين بلا داعٍ. AAB على Firebase يحتاج إعدادات الربط مع Google Play internal app sharing المناسبة؛ APK هو الافتراضي الأبسط للاختبار.

## أوامر البناء المرجعية

```bash
flutter pub get
flutter analyze
flutter test
flutter build apk --flavor development -t lib/main_dev.dart --build-name 1.5.0 --build-number 83 --dart-define APP_ENV=dev
flutter build appbundle --flavor production -t lib/main_prod.dart --build-name 1.5.0 --build-number 83
flutter build ipa --flavor production -t lib/main_prod.dart --build-name 1.5.0 --build-number 83 --export-options-plist ios/ExportOptions.prod.plist
firebase appdistribution:distribute path/to/app.apk --app FIREBASE_APP_ID --groups qa --release-notes-file release-notes.txt
```

الأوامر مرجعية؛ production scripts تستخدم args arrays/quoting والتحقق من config. ExportOptions للـIPA يحتاج method متوافقًا مع التوزيع والشهادة والـprofile. Ad Hoc يحتاج UDIDs للأجهزة ضمن profile؛ لا تفترض أن أي tester يثبت أي IPA.

## نتائج كل منصة

manifest يحتوي app_name، environment، channel، version، build_number، source_sha، engine_sha، run_id، وtargets[] بكل منها platform/artifact/build_status/deploy_status/failed_stage/artifact_name. job outputs وحدها غير كافية لتجميع matrix متعددة؛ احفظ manifest فريدًا لكل target واجمعه في finalize.

## خطة تسليم واختبار

1. validate/config + caller contract + فصل dev/prod.
2. version reservation + Android build-only مع signature verification.
3. Firebase APK + اختبار تنزيل وتثبيت فعلي.
4. iOS signing وIPA + اختبار جهاز مسجل.
5. aggregation/tags + Telegram + WhatsApp template.
6. Agent dispatch/watch/result.

اختبر success وvalidation failure وanalyze failure وsigning failure وFirebase failure وpartial success وقناة واحدة/الاثنين/none وnotification failure وparallel releases وrerun. علامة v1 لا توضع قبل اختبارات end-to-end على مشروع تجريبي.
