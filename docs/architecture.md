# ⚙️ Technical Architecture & Backend Logic

![System Architecture](..\assets\system_architecture.png)

This document details the internal logic of the Go Middleware, specifically focusing on how we bridge the gap between Unstructured AI output and the Rigid Legacy Database.

---

## 1. System Design: The Adapter Pattern

The backend service is built with **Go (Gin Framework)**. We chose Go for its strong typing and struct-based architecture, which acts as a strict gatekeeper against AI hallucinations.

### 🔄 Data Transformation Pipeline

The data flows through 4 distinct stages inside the backend:

1.  **Ingestion:** Receives PDF/Image from Frontend.
2.  **Extraction (AI):** Sends payload to Mistral OCR to get raw JSON.
3.  **Sanitization:** Validates constraints (e.g., "If confidence < 80%, mark as review needed").
4.  **Mapping (The Core Logic):** Transforms the `AIModel` into the `LegacyDBModel`.

---

## 2. The Core Problem: Data Structure Mismatch

One of the biggest challenges was the difference between "Logical Data" (what users see) and "Physical Storage" (what the DB needs).

| Field Name | AI Extracted Format (Logical) | Legacy Database Format (Physical) | Issue |
| :--- | :--- | :--- | :--- |
| **Weight** | `20,500.00` (Number) | `gross_weight` (Number) | ✅ Match |
| **Weight Unit** | `"KGS"` (String) | **`total_volume`** (String) | ⚠️ **Mismatch:** Unit stored in Volume col |
| **Volume** | `45.5` (Number) | `measurement_cbm` (Number) | ✅ Match |
| **Date** | `"2023-12-25"` (ISO) | `"25/12/2023"` (Thai Format) | ⚠️ **Format Mismatch** |

---

## 3. Implementation Logic (Pseudo-Code)

Since the database schema cannot be changed, the Go Middleware handles the **"Mapping Logic"** before the data reaches the frontend for verification.

We use strict Go Structs to parse the AI response, ensuring no unknown fields slip through.

### Step 1: Defining the AI Interface (Input)
```go
// Represents what the AI "sees"
type AIResponse struct {
    GrossWeight float64 `json:"gross_weight"`
    WeightUnit  string  `json:"weight_unit"` // e.g., "KGS", "LBS"
    BookingDate string  `json:"date"`        // e.g., "YYYY-MM-DD"
}

```

### Step 2: The "Mapper" Function (Logic Layer)

This is where the business logic resides. We strictly separate the *Extraction* from the *Saving*.

```go
// MapToLegacyModel converts flexible AI data into rigid DB format.
// Note: This is a simplified version of the internal logic.
func MapToLegacyModel(aiData AIResponse) LegacyDBModel {
    dbModel := LegacyDBModel{}

    // 1. Direct Mapping
    dbModel.GrossWeight = aiData.GrossWeight

    // 2. The "Schema Trap" Solution (Field Swapping)
    // Logic: The legacy system reuses 'TotalVolume' column to store Weight Units
    if aiData.WeightUnit != "" {
        dbModel.TotalVolume = aiData.WeightUnit 
    } else {
        dbModel.TotalVolume = "KGS" // Default fallback
    }

    // 3. Format Standardization
    // Logic: Convert ISO Date to Thai/Legacy Format
    parsedDate, _ := time.Parse("2006-01-02", aiData.BookingDate)
    dbModel.BookingDate = parsedDate.Format("02/01/2006")

    return dbModel
}

```

---

## 4. Zero Hallucination Strategy

To adhere to the "Human-in-the-Loop" philosophy, the backend implements a **Strict Null Policy**:

1. **Prompt Level:** The AI is instructed: *"If the value is not explicitly visible, return `null`. Do not calculate or guess."*
2. **Code Level:** Go checks for nil pointers or empty strings.
* If `BookingNo` is empty -> Flag record as `Status: Incomplete`.
* This forces the Frontend User to manually fill the missing data, ensuring data integrity is never compromised by AI guessing.



---

## 5. Technology Decisions

* **Go (Golang):** Chosen for its performance and strict type system, which is crucial when dealing with messy AI outputs.
* **Gin Web Framework:** For rapid API development and easy middleware integration.
* **Mistral AI:** Selected for its high reasoning capability in understanding document layouts compared to traditional OCR.