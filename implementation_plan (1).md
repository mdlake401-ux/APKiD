# Lalita AI — Android Application Implementation Plan

Lalita AI is a multilingual, voice-enabled Android AI companion built with Kotlin and Jetpack Compose, featuring a luxury pink-and-black aesthetic, Bengali/English/Arabic support, a female Bengali voice experience, Free and Premium plans, and secure Gemini AI integration designed for safe APK distribution.

---

## User Review & Critical Decisions

> [!IMPORTANT]
> **Platform Alignment & Clarifications**: The active workspace is a native **Android (Kotlin + Jetpack Compose)** project configured for APK generation rather than a React/Web (`App.tsx`) project. Because the interactive question prompt was dismissed, this plan uses the recommended production defaults below—please request revisions if you prefer any alternative before building.

- **Confirmed Core Scope**: Pink-and-black luxury UI, Bengali + English + Arabic multilingual support (with RTL layout for Arabic), AI text chat and voice chat with a female Bengali voice profile, secure Gemini API access without hardcoded client keys, Free & Premium plans, and the `@Apora_Sahid❤️` developer profile.
- **Recommended Cloud & Security Default (Firebase AI Logic + Firestore + Google Sign-In)**: Route Gemini calls through **Firebase AI Logic + App Check** instead of embedding a raw `GEMINI_API_KEY` inside the APK, and store chat history, active plan status, and daily usage quotas per user in **Cloud Firestore** gated by **Google Sign-In (Credential Manager)**.
- **Recommended Voice Strategy**: Pair **Android `SpeechRecognizer`** (for Bengali, English, and Arabic voice input) with **Android `TextToSpeech`** tuned to select a female Bengali (`bn-BD` / `bn-IN`), English, or Arabic voice profile—giving instant, reliable offline/online speech synthesis with an animated voice-chat mode.
- **Recommended Plan Model**: Provide a **Free** tier (20 daily AI messages + standard voice playback) and a **Premium** tier (unlimited messaging, extended memory, custom voice speed/pitch controls, and priority streaming), backed by Firestore user entitlements and a Play Billing-ready upgrade UI.

---

## 1. Overview & Core Concept

- **What It Does**: Lalita AI lets users hold natural text and voice conversations with an intelligent assistant fluent in **Bengali (বাংলা)**, **English**, and **Arabic (العربية)**. Users can switch languages at any time from the top navigation bar, speak naturally via a live voice-recording button, listen to replies in a warm female Bengali voice, manage Free or Premium membership plans, and explore the `@Apora_Sahid❤️` developer profile.
- **Target Audience / Persona**: Bilingual and multilingual mobile users—especially Bengali, English, and Arabic speakers—who want a stylish, expressive, voice-first AI assistant packaged as a standalone Android APK.
- **Key Value**: Combines effortless trilingual text and voice interaction with enterprise-grade API security (no leaked API keys inside distributed APKs) and a distinctive pink-and-black visual identity.

---

## 2. User Experience & Visual Design

### Key User Flows

1. **Authentication & Welcome Flow**:
   - Users land on a sleek pink-and-black welcome gate with a single-tap **"Sign in with Google"** action (using Jetpack Credential Manager) so their chat threads, language preference, and Free/Premium plan remain synced across reinstalls and devices.
2. **Main Chat & Language Switching Flow**:
   - **Top Navigation Bar**: Displays the Lalita AI emblem, a **Plan Badge** (`Free` / `Premium`), a prominent **Language Dropdown Menu** (`English`, `বাংলা`, `العربية`), and quick navigation to **Plans** and **Developer Profile**.
   - **Empty / Starter State**: Shows a glowing rose-pink Lalita avatar orb and localized quick-prompt chips in the selected language.
   - **Scrollable Chat Stream**: Renders user bubbles (deep rose-plum tint) and Lalita AI bubbles (obsidian carbon with rose border accents), real-time streaming text, per-message **"Listen" (Speaker)** and **"Copy"** actions, and automatic RTL alignment when Arabic is chosen.
   - **Input & Voice Bar**: Pinned bottom composer with an auto-expanding text input field, a **Send** button, and a **Voice Recording** button.
3. **Voice Chat & Female Bengali Voice Flow**:
   - Tapping the **Voice Recording** button opens an active listening state with a pulsing rose-pink audio ring and live speech transcript.
   - Once captured (or when the user taps stop), the transcript is sent to Gemini (`gemini-3.8-flash`), and Lalita AI automatically speaks the response using the configured female Bengali/English/Arabic voice profile with play/pause/stop controls.
4. **Free & Premium Plans Flow**:
   - Users track remaining daily messages on the Free plan via a subtle quota pill. Tapping the badge or hitting the daily limit opens the **Plans Screen**, comparing Free vs. Premium benefits with a clean upgrade flow.
5. **Developer Profile (`@Apora_Sahid❤️`) Flow**:
   - Dedicated **Developer & About** screen featuring `@Apora_Sahid❤️`, role/bio card, supported languages overview, security & App Check badge, and app version details.

### Visual Identity & Theme

- **Aesthetic Direction**: **Obsidian Noir & Rose Neon** — a high-contrast, luxury dark-first aesthetic where deep matte black surfaces make vibrant rose-pink accents, subtle radial glows, and crisp multilingual typography stand out.
- **Color Palette & Mood** *(Material 3 Semantic Tokens)*:
  - *Background / Void*: `#09090D` (Deep Obsidian Black)
  - *Surface / Elevated Card*: `#13131B` (Velvet Charcoal) & `#1B1722` (Rose-Tinted Carbon)
  - *Primary Accent*: `#FF4D8D` (Vibrant Rose Pink)
  - *Secondary / Soft Glow*: `#FF85B3` (Blossom Pink) & `#381426` (Deep Berry Container)
  - *Text & Contrast*: `#F8F4F6` (Crisp Pearl White) primary text, `#B8A8B2` muted rose-gray secondary text, `#FF6B8B` soft coral error state.
- **Typography & Multilingual Script Support**:
  - Bundled locally via `font-util` (zero runtime network font failures): **Playfair Display** or **Plus Jakarta Sans** for expressive display headings and UI labels, paired with clean **Noto Sans Bengali** and **Noto Sans Arabic** fallback support so Latin, Bengali, and Arabic glyphs render seamlessly at accessible `sp` scales.
- **Component Styling & Motion**:
  - Edge-to-edge layout (`enableEdgeToEdge()`) with `WindowInsets.safeDrawing` and `navigationBarsPadding()` so notches and gesture bars never clip the top bar or bottom composer.
  - Spring-based message reveal animations, pulsing voice-visualizer halo (`Canvas`), Material 3 ripple feedback, and minimum `48dp` interactive touch targets across all buttons and dropdown items.
  - Custom pink-and-black adaptive launcher icon (`ic_launcher`) featuring a stylized rose-pink geometric lotus/spark motif on an obsidian background.

---

## 3. Key Product Decisions & Trade-Offs

- **Decision 1: Secure Gemini API Integration via Firebase AI Logic + App Check**
  - *Chosen Approach*: Use **Firebase AI Logic (`Firebase.ai`)** with **`gemini-3.8-flash`** and **Firebase App Check** (Debug provider in debug builds; **reCAPTCHA Enterprise** in release APKs).
  - *Why*: Distributing an APK with a hardcoded `GEMINI_API_KEY` (even via `BuildConfig`) allows anyone to decompile the APK and steal the API key. Firebase AI Logic removes the Gemini API key from the client binary completely and verifies genuine app traffic via App Check attestation.
  - *Alternatives Considered*: Direct REST/SDK calls using `BuildConfig.GEMINI_API_KEY` — rejected because it exposes credentials inside a public distributable APK.

- **Decision 2: Cloud Persistence & Entitlements (Firestore + Google Sign-In)**
  - *Chosen Approach*: Store user preferences, chat messages, daily Free-tier usage counters, and Premium status in **Cloud Firestore** (using the provisioned named database ID) behind **Google Sign-In via Jetpack Credential Manager** and strict per-user Firestore security rules.
  - *Why*: Prevents losing conversations or Premium entitlements when reinstalling or switching devices, and prevents trivial local quota tampering.
  - *Alternatives Considered*: Device-only Room/DataStore storage — passed over because user-authored chat history and Free/Premium plan state should persist reliably across devices.

- **Decision 3: Female Bengali Voice & Multilingual Voice Chat Architecture**
  - *Chosen Approach*: Combine Android **`SpeechRecognizer`** (dynamically switching recognizer locales `bn-BD`, `en-US`, `ar-SA` based on the active top-bar language) with Android **`TextToSpeech`** configured to select female Bengali (`bn-BD` / `bn-IN`), English, and Arabic voices (with tunable pitch/speech rate on Premium).
  - *Why*: Delivers immediate, low-latency, zero-extra-billing speech input and natural female Bengali voice output with graceful fallback when offline or when device TTS packs vary.
  - *Alternatives Considered*: Streaming raw generated audio blobs from a separate cloud TTS endpoint — deferred as unnecessary latency/billing overhead when Android's native speech engines handle trilingual voice chat smoothly.

- **Decision 4: Free vs. Premium Plan Enforcement**
  - *Chosen Approach*: Track daily message usage and plan tier (`FREE` / `PREMIUM`) in Firestore, unlocking unlimited conversations, custom voice tuning, and priority context length for Premium users, with an upgrade screen designed to connect cleanly to Google Play Billing upon Play Store release.
  - *Why*: Allows immediate end-to-end testing of quota limits, upgrade prompts, and Premium perks in distributed APKs without requiring an active Google Play Console merchant account first.

---

## 4. Technical Architecture & Data Strategy *(Technical Reference)*

### Architecture & Component Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                        MainActivity (Edge-to-Edge)                   │
│  1. configureAppCheck(this, intent)   2. Compose Auth & Nav Host     │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │
          ┌────────────────────────┴────────────────────────┐
          ▼                                                 ▼
┌──────────────────────┐                          ┌──────────────────────┐
│      Auth Gate       │                          │   Authenticated UI   │
│  Credential Manager  │                          │  (Navigation Compose)│
│  (Google Sign-In)    │                          └──────────┬───────────┘
└──────────────────────┘                                     │
                 ┌───────────────────────┬───────────────────┼───────────────────┐
                 ▼                       ▼                   ▼                   ▼
     ┌───────────────────┐   ┌───────────────────┐ ┌─────────────────┐ ┌─────────────────┐
     │    Chat Screen    │   │   Voice Overlay   │ │  Plans Screen   │ │  Profile Screen │
     │ - Top Language    │   │ - Mic Permission  │ │ - Free Quota    │ │ - @Apora_Sahid❤️│
     │   Dropdown        │   │ - SpeechRecognizer│ │ - Premium Perks │ │ - App Info      │
     │ - Message Stream  │   │ - Female Bengali  │ │ - Plan Toggle / │ │ - Sign Out      │
     │ - Composer + Mic  │   │   TTS Engine      │ │   Upgrade Flow  │ │                 │
     └─────────┬─────────┘   └─────────┬─────────┘ └────────┬────────┘ └────────┬────────┘
               │                       │                    │                   │
               └───────────────────────┴─────────┬──────────┴───────────────────┘
                                                 ▼
                                    ┌────────────────────────┐
                                    │     ChatViewModel      │
                                    │  (StateFlow<UiState>)  │
                                    └──────┬──────────┬──────┘
                                           │          │
                    ┌──────────────────────┘          └──────────────────────┐
                    ▼                                                        ▼
     ┌──────────────────────────────┐                        ┌──────────────────────────────┐
     │      LalitaAiRepository      │                        │     UserDataRepository       │
     │ - Firebase.ai                │                        │ - Named Firestore Instance   │
     │ - gemini-3.8-flash           │                        │ - Real-time snapshot Flows   │
     │ - Language System Prompts    │                        │ - Daily Quota & Plan State   │
     │ - Streaming + 60s Timeout    │                        │ - Zero-Trust Security Rules  │
     └──────────────────────────────┘                        └──────────────────────────────┘
```

### Core Technologies & Required Modules

- **UI & Navigation**: Kotlin, Jetpack Compose, Material Design 3, Navigation Compose (`@Serializable` type-safe routes), custom bundled Google Fonts (`font-util`), custom adaptive launcher icon.
- **AI & Security**: Firebase AI Logic (`gemini-3.8-flash` with `generateContentStream()`), Firebase App Check (`DebugAppCheckProviderFactory` for debug; `RecaptchaAppCheckProviderFactory` for release).
- **Auth & Cloud Persistence**: Firebase Authentication (Google Sign-In via Jetpack `CredentialManager` + `GetSignInWithGoogleOption`), Cloud Firestore (explicit named database ID + zero-trust `firestore.rules` + local emulator rule validation).
- **Voice & Multilingual**: Android `SpeechRecognizer` (`RECORD_AUDIO` runtime permission flow), Android `TextToSpeech` (female `bn-BD` / `bn-IN`, `en-US`, `ar-SA` voice selector), Compose `LayoutDirection.Rtl` switching for Arabic, localized string resources (`values/`, `values-bn/`, `values-ar/`).

### Data Model & State

- **User Profile & Subscription Document** (`/users/{userId}`):
  - `userId`, `displayName`, `email`, `preferredLanguage` (`EN` | `BN` | `AR`), `planTier` (`FREE` | `PREMIUM`), `dailyMessageCount`, `lastUsageDate`, `voiceAutoPlay`, `voicePitch`, `voiceSpeed`, `createdAt`, `updatedAt`.
- **Chat Message Subcollection** (`/users/{userId}/messages/{messageId}`):
  - `userId`, `role` (`USER` | `MODEL`), `text`, `language` (`EN` | `BN` | `AR`), `isVoiceMessage`, `createdAt`.
- **ViewModel UI State (`StateFlow`)**:
  - Tracks `authState` (`Loading`, `Unauthenticated`, `Authenticated`), `messages`, `streamingResponse`, `selectedLanguage`, `planStatus` (remaining Free quota vs. Premium unlimited), `voiceState` (`Idle`, `Listening`, `Speaking`, `PermissionRequired`, `Error`), and `errorMessage`.

### Interactive Component & State Mapping

- **Top Bar Language Dropdown**: Selecting `English`, `বাংলা`, or `العربية` updates `preferredLanguage` in Firestore, switches Compose `LocalLayoutDirection` (`Rtl` for Arabic, `Ltr` for English/Bengali), updates UI string resources, reconfigures `SpeechRecognizer` and `TextToSpeech` locales, and injects the target language persona into the Gemini system prompt.
- **Scrollable Chat & Streaming Composer**: Submitting a prompt checks Free-plan daily quota (`< 20` messages or `PREMIUM`), appends the user message to Firestore, opens a `withTimeout(60_000L)` streaming session on `gemini-3.8-flash`, renders token chunks live in the chat bubble, persists the completed reply, and optionally triggers female voice playback.
- **Voice Recording Button**: Requests `RECORD_AUDIO` permission via Compose `ActivityResultContracts.RequestPermission` if needed; opens the pink-and-black voice capture sheet, streams partial voice recognition text, and auto-submits on silence/completion.
- **Plans & Developer Screens**: Navigating via top-bar/drawer destinations renders the Free vs. Premium comparison screen (with instant plan preview/upgrade action and quota reset summary) and the `@Apora_Sahid❤️` developer profile screen.

### Distributable APK Generation & Security Process

1. **Backend & Rules Verification**: Provision Firebase (`firestore` + `ai_logic`) with the app's unique `applicationId` and `debug.keystore` SHA-1/SHA-256 fingerprints, validate `firestore.rules` locally with JS and Robolectric emulator tests, and deploy verified rules.
2. **Release Security Hygiene**:
   - No `GEMINI_API_KEY` or secret credentials are bundled in source code, `.env`, or `BuildConfig`.
   - `MainActivity.onCreate()` calls `configureAppCheck(this, intent)` prior to Compose `setContent` so debug builds attach the debug token synchronously (`.commit()`) and release APKs enforce **reCAPTCHA Enterprise** (`recaptcha_site_key`).
3. **APK Build & Distribution Workflow**:
   - **Test APK**: Run `gradle :app:assembleDebug` to produce the signed debug APK for device/emulator verification.
   - **Distributable Release APK / App Bundle**: Configure a release upload keystore (`KEYSTORE_PATH`, `STORE_PASSWORD`, `KEY_PASSWORD`) and reCAPTCHA Enterprise site key, then generate the signed release APK (`gradle :app:assembleRelease`) or Play-ready AAB (`gradle :app:bundleRelease`)—or export directly via AI Studio's APK/AAB build menu—ready to share with users.
