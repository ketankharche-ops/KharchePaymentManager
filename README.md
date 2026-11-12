# Kharche Payment Manager — Android source project

A Kotlin Android starter implementation with local Room storage, customer records, payment history, WhatsApp message hand-off, daily due reminders, and scheduled Firebase Firestore backup.

## Implemented in this project
- Customer records: name, phone, address, contract/total amount.
- Payment entries: amount, timestamp and note; customer-wise history.
- Balance summary and basic customer-wise report.
- WhatsApp: opens `wa.me` with a pre-filled reminder. The user reviews and sends manually.
- Local persistence: Room database on the phone.
- Notifications: WorkManager checks unpaid installments daily and posts Android notifications.
- Cloud backup: WorkManager uploads daily snapshots to Firestore after anonymous Firebase sign-in.

## Important behaviour / limitations in this first build
1. When a customer is created with a positive total amount, the app currently creates one initial installment due in 7 days. More granular installment scheduling and editing are the next enhancement.
2. The backup is one-way snapshot backup, not a restore/synchronization workflow. Anonymous Firebase identity is tied to the installation; configure account linking or another recovery method before depending on cross-device restore.
3. Customer and payment edits/deletion, CSV/PDF export, multi-user access, encryption policy, and backup status UI are not included yet.
4. The app stores phone numbers locally and in your Firebase project. Secure your Firebase project and Firestore rules; do not use open/test rules in production.
5. Android 13+ requires the user to grant notification permission. WorkManager reminders are periodic and may be delayed by Android battery optimization; they are not exact-alarm notifications.

## Setup
1. Install Android Studio (current stable) and a JDK 17.
2. Open this folder (`KharchePaymentManager`) in Android Studio and allow Gradle sync.
3. In Firebase Console, create a project and add an Android app with package name `com.kharche.paymentmanager`.
4. Download `google-services.json` and place it in the `app/` directory.
5. In Firebase Authentication, enable **Anonymous** sign-in.
6. Create a Cloud Firestore database. Start in production mode and apply restrictive rules. A basic owner-only rule for this anonymous-auth prototype is:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /paymentManagerBackups/{uid}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

7. Build with **Build > Build Bundle(s) / APK(s) > Build APK(s)**. Install the generated debug APK on your Android phone.
8. Grant notification permission when prompted. Connect to the internet for the scheduled Firebase backup.

## Build configuration
- Package: `com.kharche.paymentmanager`
- Minimum Android: API 26 (Android 8.0)
- Compile/target: API 35
- Kotlin + AndroidX + Room + WorkManager + Firebase Firestore

## Before business use
Test with dummy customers first. Verify backup documents in Firestore, notification permission, WhatsApp number formatting (country code, digits only), and all totals. Configure a durable Firebase account/recovery strategy and production security rules before storing real customer data.

## Build APK using only an Android phone (GitHub Actions)

You do not need Android Studio on your phone. Use Chrome and a GitHub account:

1. Create a private GitHub repository named `KharchePaymentManager`.
2. Upload the contents of this project ZIP to the repository root (the `app`, `build.gradle.kts`, `settings.gradle.kts`, and `.github` entries should be at the root).
3. In Firebase Console, create/register the Android app with package `com.kharche.paymentmanager`; download `google-services.json`.
4. In GitHub repository: Settings → Secrets and variables → Actions → New repository secret. Name it `FIREBASE_CONFIG_JSON`.
5. Open `google-services.json` in a text editor, copy its complete contents, and paste them as the secret value. Do not post this file or secret in chats, issues, or public repositories.
6. Open the repository Actions tab, select **Build Kharche Payment Manager APK**, and choose **Run workflow**.
7. When the workflow succeeds, open its run, scroll to Artifacts, and download `Kharche-Payment-Manager-debug`. Extract the ZIP to obtain `app-debug.apk`.
8. Install the APK on your Android phone. Android may ask you to allow installation from the browser or file manager you used.

The workflow requires the Firebase configuration secret. It does not create a Firebase project or configure Firestore for you. Enable Anonymous Authentication and create Firestore in your own Firebase project before using cloud backup. Apply the restrictive rules shown above. Keep the GitHub repository private because this project handles customer contact and payment information.
