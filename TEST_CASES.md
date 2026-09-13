# DigiMunshi (دیجی منشی) — Complete Test Cases & User Interaction Specification Document 📖🎙️

This comprehensive specification details all test cases, user intent utterances (what the user can say in Urdu and Roman Urdu), system features, UI interactions, API endpoints, and safety guardrails for the **DigiMunshi** AI-First Voice Ledger Engine and Mobile Application.

---

## 📋 Table of Contents
1. [System Overview & Architecture](#1-system-overview--architecture)
2. [User Personas & Conversational Capabilities](#2-user-personas--conversational-capabilities)
3. [Voice Intent & Spoken Utterances Test Cases](#3-voice-intent--spoken-utterances-test-cases)
   - [3.1 Record Udhaar (Credit Entry)](#31-record-udhaar-credit-entry)
   - [3.2 Record Payment (Debt Reduction)](#32-record-payment-debt-reduction)
   - [3.3 List Debtors (Outstanding Customer Accounts)](#33-list-debtors-outstanding-customer-accounts)
   - [3.4 Single Customer Balance Query](#34-single-customer-balance-query)
   - [3.5 Total Shop Ledger Balance Query](#35-total-shop-ledger-balance-query)
   - [3.6 Create Customer Account](#36-create-customer-account)
   - [3.7 Delete / Clear Khata](#37-delete--clear-khata)
   - [3.8 Confirmation & Cancellation Commands](#38-confirmation--cancellation-commands)
   - [3.9 Edge Cases, Guardrails & Exception Handling](#39-edge-cases-guardrails--exception-handling)
4. [Mobile App UI & Interaction Test Cases](#4-mobile-app-ui--interaction-test-cases)
   - [4.1 Authentication & Security](#41-authentication--security)
   - [4.2 Home Dashboard & Voice Recording](#42-home-dashboard--voice-recording)
   - [4.3 Khata Book (Customer List)](#43-khata-book-customer-list)
   - [4.4 Customer Ledger Detail View](#44-customer-ledger-detail-view)
5. [Backend REST & Voice API Test Cases](#5-backend-rest--voice-api-test-cases)
6. [Safety Guardrails & System Security Test Cases](#6-safety-guardrails--system-security-test-cases)

---

## 1. System Overview & Architecture

**DigiMunshi** is a voice-controlled digital khata (ledger) application designed specifically for Pakistani micro-merchants and kiryana store owners. 

### Key Technical Capabilities:
- **Speech-To-Text (STT)**: Transcribes spoken Urdu / Roman Urdu audio (WAV, M4A, MP4) via **Uplift AI Scribe**.
- **LLM Tool Processing**: Powered by **Groq LPU (`llama-3.3-70b-versatile`)** for structured tool execution and intent extraction with fallback rule-based parsing.
- **Text-To-Speech (TTS)**: Synthesizes response audio strictly in authentic Urdu script (نستعلیق / اردو) via **Uplift AI Orator** (`prime-time-anchor` voice).
- **Ledger Engine**: Multi-tenant PostgreSQL double-entry ledger with automatic balance tracking and audit trails on Neon Serverless Postgres.
- **Mobile Frontend**: React Native with Expo SDK 52, `expo-av` for audio recording/playback, and Zustand/React Context with event-driven `ledgerVersion` reactivity.

---

## 2. User Personas & Conversational Capabilities

### User Persona: Micro-Merchant / Kiryana Store Owner (دکاندار / منشی)
- Speaks colloquial Urdu, Punjabi-influenced Urdu, or Roman Urdu.
- Uses natural numbers (*"پانچ سو"*, *"ڈھائی ہزار"*, *"ek hazar"*, *"dedh sau"*, *"do hazar paanch sau"*).
- Uses everyday grocery, retail, and construction item names (*"بیڈ"*, *"چینی"*, *"راشن"*, *"سیمنٹ"*, *"دودھ"*, *"گھی"*, *"چاول"*, *"آٹا"*).

---

## 3. Voice Intent & Spoken Utterances Test Cases

### 3.1 Record Udhaar (Credit Entry)
**Tool**: `record_udhaar` (`add_entry`)  
**Purpose**: Record new credit/udhaar when a customer takes goods or money on credit.

| Test Case ID | Spoken Utterance (Urdu / Roman Urdu) | Expected Extracted Name | Expected Extracted Amount | Expected Extracted Item | Expected System Behavior & Response |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **UDH-001** | *"Ali ko 500 rupay udhaar likho"* | علی | Rs 500 | None | New Customer: Creates customer & records Rs 500.<br>Existing Customer: Triggers 2-step confirmation with previous balance readback. |
| **UDH-002** | *"Hammad ne 20,000 ka bed udhaar liya hai"* | حماد | Rs 20,000 | بیڈ | Extracts item 'بیڈ'. Checks unusual amount (> Rs 5,000). Asks confirmation. |
| **UDH-003** | *"Kamran ke khate mein cheeni ke 400 likh do"* | کامران | Rs 400 | چینی | Records Rs 400 with item 'چینی'. Readback includes item description. |
| **UDH-004** | *"Aslam ko dedh hazar ka rashan do"* | اسلم | Rs 1,500 | راشن | Converts 'dedh hazar' (۱,۵۰۰) correctly. |
| **UDH-005** | *"Bilal ke khate mein 2500 add kar do"* | بلال | Rs 2,500 | None | Numerical parsing of '2500'. |
| **UDH-006** | *"Usman ne dhai hazar ka tel liya"* | عثمان | Rs 2,500 | تیل | Converts 'dhai hazar' (۲,۵۰۰) to 2500 and item 'تیل'. |
| **UDH-007** | *"Tariq ko ek lakh rupay udhaar diye"* | طارق | Rs 100,000 | None | Handles large amounts ('ek lakh' -> 100,000) and triggers high-value warning. |
| **UDH-008** | *"Imran ko teen sau ki chai likh do"* | عمران | Rs 300 | چائے | Parses 'teen sau' (300) and item 'چائے'. |

---

### 3.2 Record Payment (Debt Reduction)
**Tool**: `record_payment`  
**Purpose**: Deduct debt when a customer pays money back.

| Test Case ID | Spoken Utterance (Urdu / Roman Urdu) | Expected Extracted Name | Expected Extracted Amount | Expected System Response (TTS Audio & Text) |
| :--- | :--- | :--- | :--- | :--- |
| **PAY-001** | *"Ali ke khate me se 500 kam kar do"* | علی | Rs 500 | *"ٹھیک ہے، علی کے کھاتے میں سے 500 روپے کم کر دیے ہیں۔ اب باقی ادھار [X] روپے ہے۔"* |
| **PAY-002** | *"Hammad se 2000 vasool ho gaye"* | حماد | Rs 2,000 | Deducts Rs 2,000 from Hammad's ledger and reports updated balance. |
| **PAY-003** | *"Kamran ne 1000 jama kar diye"* | کامران | Rs 1,000 | Records payment entry of Rs 1,000. |
| **PAY-004** | *"Aslam ne 500 wapas diye"* | اسلم | Rs 500 | Records payment entry of Rs 500. |
| **PAY-005** | *"Bilal ke bill se 300 minus kar do"* | بلال | Rs 300 | Deducts Rs 300. |
| **PAY-006** | *"Usman ke khate se 1500 kat lo"* | عثمان | Rs 1,500 | Deducts Rs 1,500. |

---

### 3.3 List Debtors (Outstanding Customer Accounts)
**Tool**: `list_debtors`  
**Purpose**: List all customers with non-zero outstanding udhaar balances.

| Test Case ID | Spoken Utterance (Urdu / Roman Urdu) | System Execution | Expected Response |
| :--- | :--- | :--- | :--- |
| **DEB-001** | *"Gahkon ke naam batao jin ka udhaar rehta hai"* | Queries active customers where balance > 0 | Spoken list of top debtors with individual balances (e.g. *"علی کے 500 روپے، حماد کے 2000 روپے اور کامران کے 400 روپے ادھار باقی ہیں۔"*). |
| **DEB-002** | *"Kin kin ka udhaar baqi hai?"* | Queries active debtors | Spoken debtor summary. |
| **DEB-003** | *"Kis kis se paise lene hain?"* | Queries active debtors | Spoken debtor list. |
| **DEB-004** | *"Udhaar walon ki list batao"* | Queries active debtors | Spoken debtor list. |
| **DEB-005** | *"Khatay walon ke naam batao"* | Queries active debtors | Spoken debtor list. |
| **DEB-006** | *(When zero customers owe money)* | Queries active debtors (returns 0) | *"اس وقت کسی بھی گاہک کا کوئی ادھار باقی نہیں ہے۔"* |

---

### 3.4 Single Customer Balance Query
**Tool**: `query_balance_single`  
**Purpose**: Check total debt owed by a specific customer.

| Test Case ID | Spoken Utterance (Urdu / Roman Urdu) | Expected Name | Expected Response |
| :--- | :--- | :--- | :--- |
| **BAL-001** | *"Ali ka kitna udhaar hai?"* | علی | *"علی کا 500 روپے ادھار باقی ہے۔"* |
| **BAL-002** | *"Hammad ke kitne paise hain?"* | حماد | *"حماد کا 20000 روپے ادھار باقی ہے۔"* |
| **BAL-003** | *"Kamran ka balance kitna hai?"* | کامران | *"کامران کا 400 روپے ادھار باقی ہے۔"* |
| **BAL-004** | *"Tariq ka khata batao"* | طارق | Spoken balance for Tariq. |
| **BAL-005** | *"NonExistentCustomer ka kitna udhaar hai"* | N/A | *"اس نام کا کوئی گاہک کھاتے میں موجود نہیں ہے۔"* |

---

### 3.5 Total Shop Ledger Balance Query
**Tool**: `query_balance_all`  
**Purpose**: Check total outstanding debt across all shop customers.

| Test Case ID | Spoken Utterance (Urdu / Roman Urdu) | System Execution | Expected Response |
| :--- | :--- | :--- | :--- |
| **TOT-001** | *"Kul udhaar kitna hai?"* | Calculates sum of all active customer balances | *"سب ملا کر 25900 روپے ادھار باقی ہے، 3 گاہکوں کا۔"* |
| **TOT-002** | *"Sab ka mila kar kitna baqi hai?"* | Aggregates shop total | Spoken aggregate total response. |
| **TOT-003** | *"Total balance kitna hai?"* | Aggregates shop total | Spoken aggregate total response. |
| **TOT-004** | *"Sab ka kitna udhaar hai?"* | Aggregates shop total | Spoken aggregate total response. |

---

### 3.6 Create Customer Account
**Tool**: `create_customer` (`add_customer`)  
**Purpose**: Open a new customer account/khata without immediate transaction entries.

| Test Case ID | Spoken Utterance (Urdu / Roman Urdu) | Expected Name | Expected System Response |
| :--- | :--- | :--- | :--- |
| **CUST-001** | *"Zubair ka naya khata kholo"* | زبیر | *"ٹھیک ہے، زبیر کا نیا کھاتہ کھول دیا گیا ہے۔"* |
| **CUST-002** | *"Waqas ka customer account banao"* | وقاص | Opens customer account for Waqas. |
| **CUST-003** | *"Naveed ko khate me add karo"* | نوید | Opens customer account for Naveed. |
| **CUST-004** | *(If customer already exists)* | زبیر | *"زبیر پہلے سے کھاتے میں موجود ہیں۔"* |

---

### 3.7 Delete / Clear Khata
**Tool**: `delete_customer_khata` (`delete_entry`)  
**Purpose**: Clear or wipe out a customer's entire khata.

| Test Case ID | Spoken Utterance (Urdu / Roman Urdu) | Guardrail Triggered | Expected Response & Prompt |
| :--- | :--- | :--- | :--- |
| **DEL-001** | *"Ali ka khata clear kar do"* | **Khata Deletion Confirmation** | *"علی کا پورا کھاتہ صاف کرنا ہے، 500 روپے — پکا؟"* (Requires confirmation) |
| **DEL-002** | *"Hammad ka khata poora mita do"* | **Khata Deletion Confirmation** | Asks confirmation reading back current balance. |
| **DEL-003** | *"Kamran ka khata khatam kar do"* | **Khata Deletion Confirmation** | Asks confirmation reading back current balance. |

---

### 3.8 Confirmation & Cancellation Commands
**Tool**: `confirm_pending_action`  
**Purpose**: Verbal or UI response to a pending 2-step action prompt.

| Test Case ID | Spoken Response | Confirmation Intent | Resulting Action |
| :--- | :--- | :--- | :--- |
| **CFM-001** | *"Haan"* / *"Ji"* / *"Theek hai"* | `confirmed = True` | Applies pending ledger action, updates database, plays success audio response. |
| **CFM-002** | *"Likh do"* / *"Kar do"* / *"Pakka"* | `confirmed = True` | Applies pending ledger action and updates state. |
| **CFM-003** | *"Nahi"* / *"Cancel"* / *"Mat karo"* | `confirmed = False` | Cancels pending action, plays *"ٹھیک ہے، کینسل کر دیا گیا ہے۔"* audio. |
| **CFM-004** | *"Rehne do"* / *"Roko"* / *"Galat"* | `confirmed = False` | Cancels pending action without database mutation. |
| **CFM-005** | UI Button Click: **تصدیق کریں (Confirm)** | `confirmed = True` | Calls `/api/v1/voice/confirm` with `confirmed: true`. |
| **CFM-006** | UI Button Click: **منسوخ کریں (Cancel)** | `confirmed = False` | Calls `/api/v1/voice/confirm` with `confirmed: false`. |

---

### 3.9 Edge Cases, Guardrails & Exception Handling

| Test Case ID | Scenario / Utterance | Triggered Guardrail | System Response / Behavior |
| :--- | :--- | :--- | :--- |
| **EDGE-001** | Low STT Confidence (< 0.70) or muffled noise | **Low Confidence Guardrail** | *"ٹھیک سے سن نہیں سکا، دوبارہ بولیے؟"* |
| **EDGE-002** | Ambiguous Name (e.g. *"Ali ko 500 likho"* when store has "Ali Khan" & "Ali Raza") | **Disambiguation Guardrail** | *"دو علی ہیں — علی خان یا علی رضا میں سے کون سے؟"* |
| **EDGE-003** | Unusual Amount (e.g. *"Ali ko 50,000 udhaar likho"*) | **Unusual Amount Guardrail** | *"50000 روپے؟ کیا میں نے صحیح سنا؟"* (Requires confirmation prompt) |
| **EDGE-004** | Non-existent customer payment (e.g. *"UnknownUser se 500 kam kar do"*) | **Missing Customer Guardrail** | *"UnknownUser نام کا کوئی گاہک کھاتے میں موجود نہیں ہے۔"* |
| **EDGE-005** | Silences or empty audio file sent to `/voice/process` | **Empty Speech Handler** | Returns `"intent": "unknown"` with audio asking user to repeat. |
| **EDGE-006** | Name Mismatch Prevention (e.g. "Hammad" vs "Ahmed") | **Phonetic Script Guardrail** | `_is_name_match` prevents false fuzzy matching on short Urdu names (ح-م-ا-د vs ا-ح-م-د). |

---

## 4. Mobile App UI & Interaction Test Cases

### 4.1 Authentication & Security

| Test Case ID | Component / Screen | Action / Step | Expected Outcome |
| :--- | :--- | :--- | :--- |
| **AUTH-001** | Register (`(auth)/register.tsx`) | Enter Shop Name, Phone Number, 4-digit PIN -> Submit | Creates shopkeeper account, stores JWT in `expo-secure-store`, redirects to Home. |
| **AUTH-002** | Login (`(auth)/login.tsx`) | Enter valid Phone Number & PIN -> Login | Authenticates shop, persists token securely, redirects to Home (`(main)/index.tsx`). |
| **AUTH-003** | Login (`(auth)/login.tsx`) | Enter incorrect PIN 5 times | Backend locks shop account for lockout period and displays error message. |
| **AUTH-004** | App Launch (`_layout.tsx`) | Launch app with active JWT in SecureStore | Auth Guard bypasses login screen and opens Home dashboard immediately. |
| **AUTH-005** | Header (`index.tsx`) | Tap **لاگ آؤٹ (Logout)** button | Clears tokens from `expo-secure-store`, resets state, redirects to Login screen. |

---

### 4.2 Home Dashboard & Voice Recording

| Test Case ID | Component / Screen | Action / Step | Expected Outcome |
| :--- | :--- | :--- | :--- |
| **HOME-001** | Summary Card | Load Home screen | Displays **کل بقایا ادھار** (Total Outstanding in Rs.) and total customer count. |
| **HOME-002** | Mic Button (`MicButton.tsx`) | Single Tap Mic Button | Starts recording with pulsing audio animation (`🔴 سن رہا ہوں...`). Tap again to stop and submit. |
| **HOME-003** | Mic Button (`MicButton.tsx`) | Press & Hold Mic Button | Records audio continuously while held down; releasing auto-submits audio payload. |
| **HOME-004** | Processing Indicator | While `/voice/process` is in-flight | Displays spinner with text *"آواز سمجھ رہا ہوں..."* |
| **HOME-005** | Response Card (`ResponseDisplay.tsx`)| Upon receiving voice response | Displays user speech (`🗣️ آپ نے کہا`) and DigiMunshi response (`🤖 ڈیجی منشی`) in Nastaliq Urdu. Automatically plays Base64 MP3 response audio. |
| **HOME-006** | Confirmation Modal (`ConfirmationPrompt.tsx`) | Upon pending action require confirmation | Displays prompt card with **تصدیق کریں** (Green) and **منسوخ کریں** (Red) action buttons. |
| **HOME-007** | Real-Time Reactivity | Record new voice entry | Automatically updates `ledgerVersion` context, refreshing Summary Card totals instantly without reloading page. |

---

### 4.3 Khata Book (Customer List)

| Test Case ID | Component / Screen | Action / Step | Expected Outcome |
| :--- | :--- | :--- | :--- |
| **LEDG-001** | Navigation | Tap **کھاتہ دیکھیں** from Home | Navigates to `/ledger` listing all registered shop customers. |
| **LEDG-002** | Customer Card | View list items | Displays customer name in Urdu script alongside their current net balance (`Rs. X,XXX`). |
| **LEDG-003** | Empty State | View list with 0 customers | Displays empty illustration and hint: *"کوئی گاہک موجود نہیں ہے - آواز کے ذریعے نیا ادھار درج کریں"*. |
| **LEDG-004** | Item Tap | Tap customer row | Navigates to `/customer` detail screen passing `id` and `name`. |

---

### 4.4 Customer Ledger Detail View

| Test Case ID | Component / Screen | Action / Step | Expected Outcome |
| :--- | :--- | :--- | :--- |
| **CUST-001** | Customer Header | Open customer detail screen | Displays customer name in bold Urdu script and current total balance. |
| **CUST-002** | Transaction List | View history entries | Displays itemized timeline of entries with dates and timestamps. |
| **CUST-003** | Udhaar Entry Row | View credit entry | Badge displays `💳 ادھار`, red amount formatting, and optional item description (e.g. *بیڈ*, *چینی*). |
| **CUST-004** | Payment Entry Row | View payment entry | Badge displays `💰 وصولی`, green amount formatting. |
| **CUST-005** | Real-Time Sync | Perform voice payment while on customer view | Balance header and history list instantly update upon voice command completion. |

---

## 5. Backend REST & Voice API Test Cases

### 5.1 Auth Endpoints
- **`POST /api/v1/auth/register`**: Validates request parameters (phone, PIN), creates shop, returns JWT pair.
- **`POST /api/v1/auth/login`**: Validates credentials against salted bcrypt hash, handles failure counter and lockout.
- **`POST /api/v1/auth/refresh`**: Validates unrevoked refresh token and issues new access token.

### 5.2 Ledger Endpoints
- **`GET /api/v1/customers`**: Returns customer list sorted by name with aggregated net balances.
- **`GET /api/v1/ledger/balance?customer_id={id}`**: Returns net balance calculated via SQL `coalesce(sum(case entry_type...))`.
- **`GET /api/v1/ledger/summary`**: Returns total shop balance and customer count.
- **`GET /api/v1/ledger/entries?customer_id={id}`**: Returns chronologically ordered transaction history for a customer.

### 5.3 Voice Endpoints
- **`POST /api/v1/voice/process`**: Accepts JSON base64 audio or multipart form data. Orchestrates STT, Groq LLM tool classification, guardrails, DB transaction, and TTS synthesis. Returns `VoiceProcessResponse`.
- **`POST /api/v1/voice/confirm`**: Resolves `pending_action_id` (confirmed/cancelled) and executes/reverts double-entry operation.
- **`GET /api/v1/voice/audio/{audio_id}.mp3`**: Serves synthesized MP3 audio generated by Uplift Orator.
- **`POST /api/v1/voice/realtime-session`**: Creates LiveKit WebRTC session token for streaming Uplift AI voice assistant.

---

## 6. Safety Guardrails & System Security Test Cases

1. **Urdu Transliteration & Nastaliq Guardrail (`to_urdu_script`)**:
   - Converts English/Roman inputs (e.g., `"ali"`, `"bed"`, `"cheeni"`) strictly into authentic Urdu script (`"علی"`, `"بیڈ"`, `"چینی"`), preventing phonetic corruption during speech synthesis.
2. **Name Anagram Isolation Guardrail (`_is_name_match`)**:
   - Strictly enforces character sequence rules for short Urdu names (<= 4 characters), ensuring distinct names like *"حماد"* and *"احمد"* never cross-match.
3. **Double-Entry State Machine**:
   - Ledger transactions default to `confirmed` or `pending_confirmation`. Pending actions expire or require explicit user resolution before balance mutations occur.
4. **Stateless Base64 & Audio Isolation**:
   - Audio payloads are processed in memory and served statelessly to ensure zero persistent storage dependencies on ephemeral server environments.
