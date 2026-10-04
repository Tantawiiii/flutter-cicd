# AI Agent layer

الـAgent واجهة تشغيل؛ المحرك هو الذي يقرر ويطبق القواعد ويحتفظ بالـsecrets داخل CI.

## مثال الطلب

«طلع نسخة dev Android patch وارفعها Firebase وابعت Telegram». يتحول إلى environment=dev، platform=android، artifact=apk، version_type=patch، distribution=firebase، notifications=telegram.

## عقد أداة مقترح

```json
{
  "repository": "YOUR_ORG/YOUR_APP",
  "ref": "main",
  "environment": "dev",
  "platform": "android",
  "artifact": "apk",
  "version_type": "patch",
  "custom_version": "",
  "channel": "dev",
  "distribution": "firebase",
  "notifications": "telegram"
}
```

هذا عقد تصميم وليس MCP server منفذًا في هذه الحزمة.

## خطوات الربط

1. GitHub App أو broker بصلاحيات محددة: Actions write للتشغيل وActions read للمتابعة وContents read لقراءة config. لا تعطي Agent مفاتيح Firebase أو signing.
2. اسمح بقائمة repos/workflows/environments فقط. تحقق من branch/ref والقيم في enums، ولا تحول نص المستخدم إلى shell commands.
3. استخدم workflow_dispatch عبر API أو gh. workflow يجب أن يكون موجودًا على default branch ليقبل dispatch.
4. اربط الطلب بالـrun باستخدام request ID ومدخل موثق وخزن الربط؛ لا تختار أول run ظهر وتعتبره طلبك عند التشغيل المتزامن. idempotency في broker يمنع dispatch مكررًا عند إعادة الطلب.
5. تابع completion واقرأ summary/manifest وأعد للمستخدم version والنتائج والروابط. انتظار موافقة environment يعرض waiting ولا يعد فشلًا.
6. اترك قواعد حماية prod في GitHub؛ Agent لا يتجاوزها. نشر المتاجر يحتاج adapter مستقلًا وسياسة منفصلة.

## تعليمات جاهزة للـAgent

اقرأ config الحالي. عند غياب مدخل مؤثر اسأل عنه، ولا تفترض prod. شغّل workflow المعتمد فقط على ref محدد. لا تطبع secrets أو logs حساسة. لا تقل تم الرفع حتى يثبت deployment result. في partial success اذكر كل منصة منفصلة. قبل retry تحقق من run السابق والـmanifest كي لا تكرر إصدارًا.
