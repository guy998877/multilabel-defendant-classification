# Hebrew Legal Defendant Attribution (Multi-Label)

This project predicts **which defendant(s)** a specific sentence in a Hebrew criminal verdict refers to. It addresses the complexity of legal Hebrew, where subjects are often implied or referred to by generic titles.

## Task Overview

The task is a **multi-label classification** performed at the row (sentence) level.

* **Input:** A single sentence (“snippet”), case metadata (`case_file_name`, `num_defendants`), and the full case text for context mapping.
* **Output:** A compact digit string representing the set of defendants referenced (e.g., `12` for Defendants 1 and 2).

---

## The Challenge: Hebrew Legal Reality

Hebrew verdicts present unique NLP challenges regarding coreference resolution:

* **Implicit References:** Using `"הנאשם"` (The Defendant) without a number. The identity depends on initial definitions (e.g., `"נאשם מס' 1 - פלוני"`).
* **Plural Ambiguity:** `"הנאשמים"` (The Defendants) usually refers to all, but can refer to a specific subset based on the paragraph's theme (e.g., a specific count in the indictment).
* **Contextual Inference:** Sentences regarding probation reports or specific personal circumstances often lack a name but clearly refer to a specific individual based on preceding sentences.

### Solution Strategy

1. **Strict Output Formatting:** Forcing the model into a consistent digit-based schema.
2. **Structured Hebrew Prompting:** Instructing the model in the domain language.
3. **Full-Case Mapping:** Providing the model with the entire verdict text to resolve "Defendant 1 = [Name]" mappings.

---

## Dataset Format

### Input CSV (Manual Labels)

| Column | Description |
| --- | --- |
| `case_file_name` | Unique case identifier. |
| `text` | The specific snippet/sentence to classify. |
| `defendants_str` | **Gold Label:** The ground truth defendants. |
| `num_defendants` | Total count of defendants in that case. |
| `pred` | Prediction column (populated during inference). |

**Example Row:**

```csv
case_file_name,text,defendants_str,pred,num_defendants
40188-07-13,"הנאשמים הביעו בבית המשפט צער וחרטה על מעשיהם.",23,,3

```

### Label Encoding

We represent the predicted set of defendants as a sequence of digits:

* `0`: None
* `1`: Defendant 1
* `23`: Defendants 2 and 3
* `123`: All defendants (in a 3-defendant case)


---

## Pipeline Overview

### 1. Build Full-Case Text Map

Concatenate all rows of a `case_file_name` into a single string. This allows the model to see the "Head" of the verdict where defendants are formally introduced.


### 2. Inference (GPT-4o-mini)

The model processes each row by injecting the snippet, context, and full-case map into a YAML-defined prompt template.

### 3. Evaluation Metrics

Since this is multi-label, we use:

* **Micro Precision/Recall/F1:** Aggregated across all individual defendant decisions.
* **Exact Match Accuracy:** The entire set must be correct.
* **Average Jaccard Score:** Measures overlap (Intersection over Union).
* *Example:* If truth is `12` and pred is `1`, Jaccard provides partial credit ().


---

## Setup & Structure

### Requirements

* Python 3.10+
* `pandas`, `pyyaml`, `openai`

```bash
pip install pandas pyyaml openai
export OPENAI_API_KEY="your_key_here"

```

