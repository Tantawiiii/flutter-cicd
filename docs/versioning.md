# Versioning

## فصل ثلاثة مفاهيم

| المفهوم | المثال | الاستخدام |
|---|---|---|
| Marketing version | 1.5.0 | build-name للتطبيق |
| Build number | 83 | build-number، رقم موجب متزايد وفق قيود Android/Apple |
| Release channel | dev / alpha / beta / rc / production | metadata، أسماء artifacts وtags والإشعارات |

نثبت native build-name على صيغة `X.Y.Z`. `1.5.0-beta.3` تسمية release/tag وليست قيمة صالحة تلقائيًا في كل حقول version عند Apple. channel منفصل عن environment؛ يمكن beta مبنيًا بإعدادات prod.

| المدخل | الحالي 1.4.7 | الناتج |
|---|---|---|
| patch | 1.4.7 | 1.4.8 |
| minor | 1.4.7 | 1.5.0 |
| major | 1.4.7 | 2.0.0 |
| custom | 1.4.7 | القيمة الصريحة مثل 2.4.0 |

## مصدر الحقيقة

اقرأ أحدث manifest ناجح للإصدارات وفق سياسة المشروع، واستخدم pubspec.yaml فقط كبداية bootstrap. لا تعتمد على pubspec ثابت وتعمل patch كل تشغيل: النتيجة ستكون نفس الرقم كل مرة.

احجز build number من سجل دائم بشكل atomic مع concurrency وbaseline أعلى من آخر رقم مستخدم تاريخيًا. run_number حل مبسط لو workflow واحد ثابت وكل releases تمر منه؛ ليس ضمانًا عالميًا عند تغيير workflows أو التشغيل اليدوي أو نقل الريبو. rerun لنفس release يعيد نشر نفس artifact عند الإمكان. إعادة build مختلفة تحتاج حجز رقم جديد.

لا يلزم commit تلقائي إلى pubspec: مرر --build-name و--build-number. خزّن source SHA وengine SHA والنسخة والـbuild والـchannel في release manifest. مثال tag: `prod/v1.5.0+83`، وdev: `dev/v1.5.0+84`.

قبل custom release ارفض version أقل من سياسة الإنتاج، أو tag متكرر، أو رقم build غير صالح. تحديث رقم build لا يعني بالضرورة زيادة marketing version؛ يمكن أكثر من build لنفس 1.5.0.
