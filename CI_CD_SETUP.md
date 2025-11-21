# CI/CD Setup - دليل إعداد البناء التلقائي

## نظرة عامة | Overview

تم إعداد نظام CI/CD باستخدام GitHub Actions لبناء تطبيق Android تلقائياً وإنشاء ملفات APK.

A CI/CD system has been set up using GitHub Actions to automatically build the Android app and generate APK files.

## كيفية الاستخدام | How to Use

### 1. تشغيل البناء التلقائي | Automatic Build Trigger

يتم تشغيل البناء تلقائياً عند:
- Push إلى فرع main أو master أو develop
- إنشاء Pull Request إلى هذه الفروع

The build is triggered automatically when:
- Pushing to main, master, or develop branches
- Creating a Pull Request to these branches

### 2. تشغيل يدوي | Manual Trigger

يمكنك تشغيل البناء يدوياً:
1. اذهب إلى تبويب **Actions** في المستودع
2. اختر **Android CI/CD - Build APK**
3. اضغط على **Run workflow**
4. اختر الفرع واضغط **Run workflow**

You can manually trigger the build:
1. Go to the **Actions** tab in the repository
2. Select **Android CI/CD - Build APK**
3. Click **Run workflow**
4. Choose the branch and click **Run workflow**

### 3. تحميل APK | Download APK

بعد اكتمال البناء:
1. اذهب إلى تبويب **Actions**
2. اضغط على آخر workflow run
3. انزل إلى قسم **Artifacts**
4. حمّل ملف APK المطلوب:
   - **app-debug**: نسخة Debug (موقعة بتوقيع Debug)
   - **app-release**: نسخة Release (إذا كان متوفر)

After the build completes:
1. Go to the **Actions** tab
2. Click on the latest workflow run
3. Scroll down to the **Artifacts** section
4. Download the desired APK:
   - **app-debug**: Debug version (signed with debug key)
   - **app-release**: Release version (if available)

## ملفات الإخراج | Output Files

### Debug APK
- **الاسم | Name**: `app-debug.apk`
- **المسار | Path**: `app/build/outputs/apk/debug/`
- **التوقيع | Signing**: Debug signature (للتطوير | for development)
- **الحجم التقريبي | Approximate size**: ~50-100 MB

### Release APK
- **الاسم | Name**: `app-release.apk`
- **المسار | Path**: `app/build/outputs/apk/release/`
- **التوقيع | Signing**: 
  - إذا كان keystore موجود: Release signature
  - إذا لم يكن موجود: Debug signature (fallback)

## إعداد التوقيع للإصدار | Release Signing Setup

⚠️ **تحذير أمني | Security Warning**: لا تضع كلمات المرور في ملفات المشروع أو تدفعها إلى Git!
Never commit passwords in project files or push them to Git!

### الطريقة 1: الإعداد المحلي | Local Setup (For Development)

```bash
# نسخ ملف المثال
cp gradle.properties.example gradle.properties

# تحرير الملف وإضافة بياناتك الحقيقية
# Edit the file and add your real credentials
nano gradle.properties

# إضافة الـ Keystore (لن يتم دفعه للـ Git)
# Add the keystore (won't be pushed to Git)
cp your-keystore.keystore haramblur-release-key.keystore
```

**ملاحظة**: ملفات `gradle.properties` و `haramblur-release-key.keystore` محلية فقط ولن تُدفع إلى Git.
**Note**: Files `gradle.properties` and `haramblur-release-key.keystore` are local only and won't be pushed to Git.

### الطريقة 2: استخدام GitHub Secrets (موصى به للـ CI/CD)

#### الخطوة 1: تشفير الـ Keystore
```bash
base64 haramblur-release-key.keystore > keystore.b64
```

#### الخطوة 2: إضافة Secrets في GitHub
1. اذهب إلى **Settings** > **Secrets and variables** > **Actions**
2. اضغط **New repository secret**
3. أضف الـ Secrets التالية:
   - `KEYSTORE_FILE`: محتوى ملف keystore.b64
   - `KEYSTORE_PASSWORD`: كلمة مرور الـ keystore
   - `KEY_ALIAS`: اسم المفتاح
   - `KEY_PASSWORD`: كلمة مرور المفتاح

#### الخطوة 3: تحديث Workflow
أضف هذه الخطوات قبل البناء في ملف `.github/workflows/android-build.yml`:

```yaml
- name: Decode Keystore
  run: |
    echo "${{ secrets.KEYSTORE_FILE }}" | base64 -d > haramblur-release-key.keystore
  
- name: Build Release APK
  run: ./gradlew assembleRelease --no-daemon
  env:
    KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
    KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
    KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
```

## التقنيات المستخدمة | Technologies Used

- **GitHub Actions**: للبناء التلقائي
- **JDK 17**: Temurin distribution
- **Gradle 8.13**: أداة البناء
- **Android Gradle Plugin 8.5.2**: لبناء تطبيق Android
- **Gradle Caching**: لتسريع البناء

## استكشاف الأخطاء | Troubleshooting

### مشكلة: فشل البناء
- تحقق من سجل الأخطاء في Actions
- تأكد من أن جميع Dependencies متوفرة
- تحقق من إعدادات Java و Gradle

### مشكلة: لا يمكن تحميل APK
- تأكد من اكتمال البناء بنجاح
- تحقق من وجود Artifacts في الـ workflow run
- APK متاح لمدة 30 يوم فقط

### مشكلة: Release APK غير موقع بشكل صحيح
- تأكد من وجود keystore
- تحقق من إعدادات التوقيع في `app/build.gradle.kts`
- استخدم GitHub Secrets للتوقيع الآمن

## الملفات المعدلة | Modified Files

1. **`.github/workflows/android-build.yml`**: Workflow للبناء التلقائي
2. **`app/build.gradle.kts`**: إضافة دعم للبناء بدون keystore
3. **`gradle.properties`**: إزالة مسار Java الثابت
4. **`gradle/libs.versions.toml`**: تحديث AGP إلى 8.5.2
5. **`settings.gradle.kts`**: تبسيط إعدادات المستودعات

## الدعم | Support

للمساعدة أو الإبلاغ عن مشكلة، افتح Issue في المستودع.

For help or to report an issue, open an Issue in the repository.

---

تم الإعداد بواسطة GitHub Copilot | Setup by GitHub Copilot
