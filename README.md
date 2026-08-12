# IntelliRAG: Intelligent Enterprise Knowledge Management System

## Overview

**IntelliRAG** is a document-based Retrieval-Augmented Generation (RAG) system designed to answer questions from enterprise annual reports.

The system combines semantic search, keyword matching, phrase matching, and company-aware retrieval to identify relevant information from multiple company documents. Retrieved information is then provided to a language model to generate concise, grounded answers with source attribution.

The project was developed as a college-level AI/ML project to demonstrate the practical implementation of a Retrieval-Augmented Generation pipeline.

---

## Key Features

* 📄 Multi-document knowledge base using enterprise annual reports
* 🔍 Semantic retrieval using FAISS vector search
* 🔑 Keyword-based matching for meaningful query terms
* 📝 Phrase matching for important multi-word expressions
* 🏢 Company-aware retrieval for company-specific questions
* 🤖 Grounded answer generation using Qwen2.5-0.5B-Instruct
* 📚 Source attribution with company, document, page, and chunk information
* 🛡️ Relevance threshold to reduce unsupported answers
* 🚫 Out-of-context protection to reduce hallucinations
* 📊 Concept-based evaluation of generated answers

---

## Dataset

The IntelliRAG knowledge base consists of five enterprise annual reports used as the document dataset.

| Company   | Dataset File    |
| --------- | --------------- |
| Microsoft | `Microsoft.pdf` |
| NVIDIA    | `NVIDIA.pdf`    |
| Amazon    | `Amazon.pdf`    |
| Alphabet  | `Alphabet.pdf`  |
| IBM       | `IBM.pdf`       |

A total of **574 pages** were processed from the five PDF documents.

After text processing and chunking, the knowledge base contains approximately **2,496 text chunks**.

---

## System Architecture

```text
Enterprise Annual Reports
          │
          ▼
   PDF Text Extraction
          │
          ▼
      Text Chunking
          │
          ▼
    Text Embeddings
          │
          ▼
    FAISS Vector Index
          │
          ▼
    Hybrid Retrieval
          │
    ┌─────┴──────────┐
    │                │
    ▼                ▼
Semantic Search   Keyword + Phrase
    │                │
    └───────┬────────┘
            ▼
    Company-Aware Scoring
            │
            ▼
 Relevant Document Chunks
            │
            ▼
   Context Construction
            │
            ▼
 Qwen2.5-0.5B-Instruct
            │
            ▼
    Grounded Answer
            │
            ▼
   Source Attribution
```

---

## Retrieval Approach

IntelliRAG uses a hybrid retrieval strategy that combines semantic and lexical matching.

### 1. Semantic Similarity

Document chunks are converted into embeddings and indexed using **FAISS**.

The system uses an Inner Product index:

```text
FAISS IndexFlatIP
```

Because the embeddings are normalized, inner product corresponds to cosine similarity.

### 2. Keyword Matching

Meaningful words from the query are compared against words present in each retrieved document chunk.

Common stopwords are excluded from keyword matching to focus on meaningful query terms.

### 3. Phrase Matching

The system checks for matching multi-word phrases from the query.

It considers:

* 4-word phrases
* 3-word phrases
* 2-word phrases

This helps identify chunks containing important terminology from the query.

### 4. Company-Aware Scoring

When a query explicitly mentions a company, the retrieval system checks whether the retrieved chunk belongs to that company.

A company-specific relevance boost is applied when the company matches.

### Hybrid Score

The base hybrid score is calculated using:

```text
70% Semantic Similarity
20% Keyword Matching
10% Phrase Matching
```

A company-aware boost is then added when applicable.

---

## Answer Generation

Retrieved chunks are combined into a structured context containing:

* Company
* Source document
* Page number
* Chunk ID
* Extracted text

The context is then passed to:

**Qwen/Qwen2.5-0.5B-Instruct**

The model is instructed to:

* Use only the retrieved context
* Prefer terminology from the source documents
* Avoid outside knowledge
* Avoid guessing
* Provide concise answers
* Return only the requested information

---

## Relevance Checking

Before generating an answer, IntelliRAG checks the strongest retrieved chunk against a relevance threshold.

The current threshold is:

```text
0.50
```

If the best retrieved result does not reach this threshold, the system does not generate an answer from potentially irrelevant information.

Instead, it returns:

```text
I could not find sufficiently relevant information in the provided documents.
```

This provides an additional layer of protection against unsupported answers.

---

## Source Attribution

For every successful query, IntelliRAG can return source information including:

* Company
* Source document
* Page
* Chunk ID
* Hybrid relevance score

Duplicate source-page combinations are removed, and up to three sources are returned.

This makes generated answers more transparent and traceable to the original documents.

---

## Evaluation

The system was evaluated using five company-specific questions.

### Evaluation Results

| Company   | Question                                              | Result |
| --------- | ----------------------------------------------------- | ------ |
| Microsoft | What are Microsoft's three core business priorities?  | ✅ PASS |
| NVIDIA    | What is NVIDIA's main business focus?                 | ✅ PASS |
| Amazon    | What is Amazon's approach to artificial intelligence? | ✅ PASS |
| Alphabet  | What are Alphabet's major business areas?             | ✅ PASS |
| IBM       | What is IBM's strategic focus?                        | ✅ PASS |

### Concept-Based Accuracy

```text
Correct Answers: 5/5
Accuracy: 100%
```

The evaluation uses concept matching rather than requiring an exact sentence match.

For example, the evaluation treats:

```text
AI
```

and:

```text
Artificial Intelligence
```

as equivalent concepts.

---

## Out-of-Context Test

An additional test was performed using the question:

```text
Who is the current President of India?
```

This information was intentionally outside the five-company document dataset.

The system produced:

```text
I could not find sufficiently relevant information in the provided documents.
```

The best hybrid retrieval score was:

```text
0.3470
```

which was below the configured relevance threshold of:

```text
0.50
```

Therefore, the system correctly rejected the query rather than answering it using outside knowledge.

### Result

**Out-of-context test: PASSED ✅**

---

## Technologies Used

### Programming Language

* Python

### Document Processing

* PDF text extraction
* Text chunking
* Regular expressions

### Embeddings

* Sentence-transformer based text embeddings
* 384-dimensional embeddings

### Vector Search

* FAISS
* Inner Product similarity

### Language Model

* Qwen2.5-0.5B-Instruct
* Hugging Face Transformers

### Numerical Processing

* NumPy

### Development Environment

* Google Colab

---

## Project Structure

```text
IntelliRAG/
│
├── IntelliRAG.ipynb
│
└── dataset/
    ├── Microsoft.pdf
    ├── NVIDIA.pdf
    ├── Amazon.pdf
    ├── Alphabet.pdf
    └── IBM.pdf
```

### Files

* **`IntelliRAG.ipynb`** — Main notebook containing the complete IntelliRAG implementation.
* **`dataset/`** — Contains the five annual report PDF files used as the document dataset.

---

## How to Use

1. Open `IntelliRAG.ipynb` in Google Colab.
2. Place the five PDF files in the `dataset` folder.
3. Extract and process the document text.
4. Generate embeddings.
5. Build the FAISS vector index.
6. Enter a question related to the available documents.
7. IntelliRAG retrieves relevant document chunks.
8. The retrieved context is passed to the Qwen model.
9. The system generates a grounded answer.
10. Source information is returned with the answer.

### Example

```text
Question:
What are Alphabet's major business areas?

Answer:
Alphabet's major business areas are Google Services,
Google Cloud, and Other Bets.
```

---

## Limitations

* The knowledge base is limited to five enterprise annual reports.
* The system cannot provide reliable answers to information that is absent from the documents.
* PDF extraction quality can affect retrieval quality.
* The answer-generation model is relatively small and may occasionally produce incomplete responses.
* The system is designed primarily for document-based question answering rather than general-purpose conversational AI.

---

## Future Improvements

Possible future improvements include:

* Improved document chunking strategies
* Reranking retrieved passages
* Better handling of tables in annual reports
* Metadata filtering
* Larger instruction-tuned language models
* Improved answer-length control
* Evaluation using larger benchmark question sets
* Support for additional enterprise documents
* User interface for easier document querying

---

## Project Outcome

IntelliRAG demonstrates how a Retrieval-Augmented Generation system can combine **vector search, lexical matching, company-aware retrieval, grounded generation, and source attribution** to answer questions from enterprise documents.

The final system achieved:

```text
Concept-Based Evaluation: 5/5
Accuracy: 100%
Out-of-Context Test: PASSED
```

The project demonstrates the practical application of:

**Natural Language Processing + Vector Search + Retrieval-Augmented Generation + Large Language Models**

---

## Author

Satyam Raj
