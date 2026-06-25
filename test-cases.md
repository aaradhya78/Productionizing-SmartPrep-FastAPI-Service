# SmartPrep AI: Test Cases & Results

This document outlines the testing strategy, unit test cases, and integration test results for the **SmartPrep AI** platform. It validates core frontend, backend, database, and AI functionalities.

---

## 1. Testing Summary

Testing was conducted across three primary categories:
1. **Functional Testing**: Verifying core features (authentication, upload, notes/flashcard/quiz generation, scheduler execution).
2. **Security Testing**: Verifying password hashing (bcrypt), route protection (JWT), and CORS policies.
3. **Performance & Resiliency Testing**: Testing API response times and verifying the Cascading LLM Framework fallback mechanism when offline.

---

## 2. Test Cases & Execution Results

### 2.1. Module 1: User Authentication & Authorization

| Test Case ID | Test Case Description | Test Inputs | Expected Output | Actual Output | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-AUTH-01** | Register new user with valid details | Name: `Aaradhya`, Email: `aaradhya@example.com`, Password: `SecurePass123` | User created in database; password encrypted using bcrypt; HTTP 201 Created | User created successfully; password hashed; HTTP 201 | **PASSED** |
| **TC-AUTH-02** | Register duplicate user email | Name: `Aaradhya`, Email: `aaradhya@example.com` | Database throws unique constraint error; backend returns HTTP 400 Bad Request | Backend returns "Email already exists"; HTTP 400 | **PASSED** |
| **TC-AUTH-03** | Login with valid credentials | Email: `aaradhya@example.com`, Password: `SecurePass123` | Authentication success; returns a valid stateless JSON Web Token (JWT) | JWT generated and returned; HTTP 200 | **PASSED** |
| **TC-AUTH-04** | Access secured API endpoint without JWT | Request `GET /api/materials/all` | Header lacks Auth token; returns HTTP 401 Unauthorized | Access blocked; HTTP 401 | **PASSED** |

---

### 2.2. Module 2: Document Processing & NLP Pipeline

| Test Case ID | Test Case Description | Test Inputs | Expected Output | Actual Output | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-DOC-01** | Upload valid PDF study material | PDF file: `operating_systems_ch1.pdf` (Text format) | File metadata saved in MySQL; text extracted successfully; HTTP 200 OK | File registered; text extracted using PyPDF; HTTP 200 | **PASSED** |
| **TC-DOC-02** | Upload invalid file type | File: `syllabus_photo.jpg` | Backend validation blocks upload; returns HTTP 400 Bad Request | Blocked; returns "Invalid file type"; HTTP 400 | **PASSED** |
| **TC-DOC-03** | NLP Keyword & Topic Extraction | Input text: "An operating system acts as an intermediary..." | Tokenization, stop-word removal, and spaCy extraction returns keywords: `['operating system', 'intermediary', 'hardware']` | spaCy extracts expected noun phrases and standardizes to lowercase | **PASSED** |

---

### 2.3. Module 3: AI Study Aid Generation (Notes, Flashcards, Quizzes)

| Test Case ID | Test Case Description | Test Inputs | Expected Output | Actual Output | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-GEN-01** | Generate study summaries | Document text: `operating_systems_ch1.txt` | LLM returns a structured, clear bulleted summary of key topics | Summary notes generated and formatted; HTTP 200 | **PASSED** |
| **TC-GEN-02** | Generate active recall flashcards | Document text: `network_protocols.txt` | LLM returns a JSON array of question-answer pairs | Flashcards generated with distinct front/back fields; HTTP 200 | **PASSED** |
| **TC-GEN-03** | Generate multiple-choice quiz | Document text: `data_structures.txt` | LLM returns a JSON list of questions, each with 4 options and 1 correct index | MCQs generated with clean option indices; HTTP 200 | **PASSED** |

---

### 2.4. Module 4: Adaptive Study Scheduler

| Test Case ID | Test Case Description | Test Inputs | Expected Output | Actual Output | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-SCH-01** | Create default study timeline | Target date: 10 days away; Subjects: `[OS, DBMS]` | Scheduler allocates topics evenly across available days; saves to MySQL | Calendar populated with balanced tasks; HTTP 200 | **PASSED** |
| **TC-SCH-02** | Adaptive reschedule on quiz failure | Quiz score: 30% on `Process Scheduling` | System flags weakness; increments `Process Scheduling` priority; adds revision task | Priority score increased; revision task added to next day's timetable | **PASSED** |

---

### 2.5. Module 5: Performance Prediction Engine

| Test Case ID | Test Case Description | Test Inputs | Expected Output | Actual Output | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-PRED-01** | Calculate Readiness Score | QuizScore: 80%, MockScore: 75%, ConsistencyScore: 90% | Score calculated: $0.4(80) + 0.4(75) + 0.2(90) = 80\%$ | Score computed to exactly 80.0%; HTTP 200 | **PASSED** |
| **TC-PRED-02** | Forecast Final Exam Score | Calculated Readiness Score: 80% | Predicted Score: $80 \times 0.95 + 2 = 78$ | Predicted final score returned as 78; clamped between 30 and 100 | **PASSED** |

---

### 2.6. Module 6: Cascading LLM Resiliency (Offline Fallback)

| Test Case ID | Test Case Description | Test Inputs | Expected Output | Actual Output | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-FALL-01** | Online Generation (Standard) | Connection active; request quiz generation | System routes request to Google Gemini API; returns successfully | Request completed via Gemini; response latency: 1.4s | **PASSED** |
| **TC-FALL-02** | Offline Fallback (No Internet) | Connection disabled; request quiz generation | FastAPI catches HTTP connection exception; routes request to local Ollama Llama3 server | Fallback executed; quiz generated locally; response latency: 4.8s | **PASSED** |

---

## 3. Performance Metrics Summary

* **Average API Latency (Auth & DB requests)**: 120 ms
* **Average Text Extraction Time (50-page PDF)**: 850 ms
* **AI Generation Latency (Online Gemini)**: 1.5 seconds
* **AI Generation Latency (Offline Fallback Llama3)**: 5.2 seconds
* **Database Query Index Execution Time**: < 10 ms
