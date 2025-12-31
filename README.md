# 🚢 Smart OCR & Legacy Data Integration Pipeline

> **An AI-Assisted Form Filling System: Bridging GenAI (Mistral) with Enterprise Workflows.**

![Project Status](https://img.shields.io/badge/Status-Prototype%20Complete-success)
![Tech Stack](https://img.shields.io/badge/Stack-Go_Gin_%7C_Mistral_AI_%7C_Docker-blue)
![Architecture](https://img.shields.io/badge/Architecture-Human--in--the--Loop-green)

## 🎯 Project Objectives & Philosophy
This project is not just about OCR; it is designed to be an **"Intelligent Assistant" for frontend users**. The goal is to reduce manual data entry time while maintaining strict data integrity.

### Core Design Principles:
1.  **Human-in-the-Loop:** The AI pre-fills the form, but the user holds the final authority to verify and submit.
2.  **Zero Hallucination Policy:** We prioritize accuracy over completeness. The AI is strictly prompted **NOT to guess**. If a field is missing in the document, it must remain empty (`null`) rather than hallucinating a value.
3.  **Decoupling AI from Backend Logic:**
    * We do not force the AI to understand our complex legacy database schema.
    * **The AI's Job:** Purely extraction (Read what is visible).
    * **The Go Middleware's Job:** Map the extracted data to the legacy schema, handle weird column swaps, and standardize formats (Dates, Timezones) before saving.

---

## 🧩 The Challenge: "The Schema Trap"
The core problem wasn't just reading text, but **mapping logic**.
* **The AI's Reality:** Extracted data logically (e.g., `Weight: 20,000`, `Unit: KG`).
* **The Database's Reality:** Due to legacy design, it required storing the *Unit text* inside a column named `total_volume`, while the actual volume went elsewhere.

Direct integration would corrupt the database. A **"Smart Middleware"** was needed to handle this translation safely.

---

## 🏗️ Architecture & Solution

### System Workflow
I designed a Go (Gin) middleware acting as an **Adapter Pattern**:

![System Architecture](assets/system_architecture.png)

### Key Implementation Details (Backend)

* **Strict Extraction Prompts:** Tuned the AI system prompt to ensure "If not found, return null" behavior to prevent data pollution.
* **The Great Switch Logic:** A custom Go function (`ToDBModel`) acts as the bridge, programmatically swapping fields (e.g., Weight Unit -> Volume Column) to satisfy legacy requirements without confusing the AI.
* **Standardization Layer:** The middleware handles data normalization (Date/Time formats) to ensure consistency across all records.

---

## 🛠️ Included Tools: QA & Audit System

To validate the reliability of this "Assistant," I developed a standalone **Frontend Audit Tool**.

### 📂 `tools/audit.html`

A lightweight, single-file HTML/JS dashboard used to measure AI accuracy against human ground truth.

**Key Features:**

* **Visual Validation:** Side-by-side view of the original PDF and AI extracted data.
* **Metrics Calculation:** Automatically calculates **Precision** (Correctness) and **Fill Rate** (Coverage).
* **One-Click Audit:** Pass/Fail buttons for rapid manual review.
* **Reporting:** Exports audit results to CSV for team review.

*(Note: To run this tool fully, it requires the local Go backend server. The source code provided demonstrates the UI/UX and evaluation logic implemented during the internship.)*

---

## 🚀 Key Takeaways

1.  **AI as a Tool, Not a Replacement:** The system empowers users by handling the tedious typing, leaving the verification logic to humans.
2.  **Logic belongs in Code, not AI:** Moving complex business logic (schema mapping) out of the AI prompt and into Go code improved reliability and maintainability.
3.  **Data Quality First:** By enforcing a "Don't Guess" policy, we ensure that the database remains clean.

---

## 👨‍💻 Author

**Web Developer Intern (AI & Data Integration Focus)**

* **Stack:** Go, Python, JavaScript, Docker, Git.
* **Interests:** Backend Development, System Architecture, AI Integration.
