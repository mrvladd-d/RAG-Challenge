# RAG Challenge - Solution V1

A comprehensive system for analyzing company annual reports using NLP and semantic search to answer complex questions.

## Table of Contents

- [Overview](#overview)
- [System Flow Diagram](#system-flow-diagram)
- [Usage](#usage)
- [Key Components](#key-components)
- [Working with Large Language Models](#working-with-large-language-models)
- [Future Improvements](#future-improvements)

## Overview

The Annual Report Analyzer is a specialized system designed to extract, analyze, and query information from company annual reports. It combines the power of OpenSearch for document retrieval with Large Language Models (LLMs) for sophisticated information extraction and question answering.

The system can:

- Process questions about company financial metrics, operations, corporate actions, and leadership
- Extract structured information from relevant document pages
- Compare metrics across multiple companies
- Provide precise answers with references to source pages

## System Flow Diagram

The following diagram illustrates the main workflow of the Annual Report Analyzer:

```
┌─────────────┐     ┌───────────────────┐     ┌──────────────────────┐
│  Questions  │────▶│ Question Analysis │───▶│  Company & Domain    │
│   Input     │     │      (LLM)        │     │  Identification      │
└─────────────┘     └───────────────────┘     └──────────┬───────────┘
                                                         │
                                                         ▼
┌─────────────────────┐     ┌───────────────┐     ┌──────────────────┐
│  Answer Formulation │◀────│ Data Analysis │◀───│ Document Search  │
│       (LLM)         │     │     (LLM)     │     │   (OpenSearch)   │
└─────────┬───────────┘     └───────────────┘     └──────────────────┘
          │
          ▼
┌─────────────────┐
│  Final Answer   │
│  with Sources   │
└─────────────────┘
```

## Usage

### Input Format

The questions file should be a JSON file with the following structure:

```json
[
  {
    "text": "What was Company A's total revenue?",
    "kind": "number"
  },
  {
    "text": "Did Company B announce any new product launches?",
    "kind": "boolean"
  },
  {
    "text": "Which of the companies \"Company A\", \"Company B\", \"Company C\" had the highest revenue?",
    "kind": "name"
  }
]
```

The `kind` field can be one of:

- `number`: For numerical answers
- `boolean`: For yes/no answers
- `name`: For answers that return a name (like a company or product)
- `names`: For answers that return a list of names

### Output Format

The system produces a JSON file with the following structure:

```json
{
  "answers": [
    {
      "question_text": "What was Company A's total revenue?",
      "value": 1500000000,
      "references": [
        {
          "pdf_sha1": "abc123def456...",
          "page_index": 42
        }
      ]
    },
    ...
  ]
}
```

## Key Components

### 1. Question Analysis

The system analyzes each question to:

- Identify the company being asked about
- Determine the domain of the question (financial, operations, corporate, leadership)
- Generate multiple diverse search queries to find relevant content
- Detect comparison questions that involve multiple companies

### 2. Document Search and Data Extraction

For each question:

1. The system searches OpenSearch using multiple diverse queries
2. It extracts data from the most relevant pages using domain-specific LLM extractors
3. For comparison questions, it collects data for all mentioned companies

### 3. Answer Formulation

The system formulates answers by:

1. Analyzing all the extracted data relevant to the question
2. Generating a precise answer with appropriate format
3. Including confidence level and reasoning
4. Providing references to source pages in the annual reports

## Detailed Code Logic

### 1. Question Processing Flow

When processing a question, the system follows these steps:

1. **Company and Domain Identification**: The question is analyzed by the LLM to identify the company, the domain, and three diverse search queries.

   - This happens in `identify_company_and_domain()` in `llm_service.py`
   - The LLM generates variations of search terms to increase the chance of finding relevant information

2. **Question Type Handling**:

   - **Comparison Questions**: If the question involves comparing multiple companies (detected by `is_comparison_question()`), the system extracts all company names and processes each one separately
   - **Standard Questions**: For questions about a single company, the system processes just that company

3. **Document Search**:

   - The system searches for relevant pages using multiple queries in `search_relevant_pages()`
   - It applies domain-specific boosts to improve search relevance
   - Results are deduplicated across queries to ensure diversity

4. **Data Extraction**:

   - Domain-specific extractors in `llm_service.py` parse each page's content
   - Financial data, business operations, corporate actions, and leadership information are extracted into structured formats
   - A caching mechanism avoids redundant processing of the same queries

5. **Answer Formulation**:

   - The extracted data is sent to an LLM for answer formulation
   - The system includes confidence levels and source references
   - The output is formatted according to the question type (number, boolean, name, names)

### 2. Key Algorithms

#### Company Name Matching

The `find_closest_company_match()` function uses several techniques:

1. Exact matching
2. Special handling for specific company types (e.g., "Odyssey" companies)
3. Prefix matching using the first 4 characters
4. Fuzzy matching as a fallback with a configurable threshold

#### Metric Type Detection

The `determine_metric_type()` function maps keywords in the question to specific metrics:

```python
metric_keywords = {
    "total revenue": "total_revenue",
    "revenue": "total_revenue",
    "total assets": "total_assets",
    # ...many more mappings
}
```

#### Domain Determination

Based on the metric type, the system determines the appropriate domain:

```python
domain_mapping = {
    "total_revenue": "financial",
    "employee_count": "operations",
    "mergers_acquisitions": "corporate",
    "leadership_positions_changed": "leadership",
    # ...and so on
}
```

## Working with Large Language Models

The system is designed to work with LLMs that support structured outputs. It uses the `.beta.chat.completions.parse()` method to receive structured responses matching Pydantic models.

### Model Requirements

The LLM must support:

1. The `.beta.chat.completions.parse()` method or equivalent functionality
2. Structured outputs matching Pydantic models
3. System and user prompts
4. Temperature control for deterministic responses

Check your LLM provider's documentation for specific implementation details.

## Future Improvements

The Annual Report Analyzer could be enhanced with:

1. **Asynchronous Processing**: Implement async/await patterns to process multiple questions and companies concurrently, significantly improving throughput.
2. **Semantic Search Integration**: Enhance the OpenSearch capabilities with embeddings-based semantic search to improve document retrieval beyond keyword matching.
3. **Image Analysis**: Add support for processing charts, graphs, and tables in annual reports using visual AI models.
4. **Confidence Scoring Improvements**: Implement more sophisticated confidence estimation using multiple signals (search relevance, extraction confidence, answer formulation confidence).
5. **Incremental Data Processing**: Add support for incrementally processing and updating the knowledge base as new annual reports become available.
6. **Multi-Document Reasoning**: Enhance the LLM's ability to reason across multiple documents and pages to answer complex questions that span different sections of the annual report.
7. **User Feedback Integration**: Add a mechanism to incorporate user feedback to improve answer quality over time.

## License

[Specify your license here]

## Contact

[Your contact information]
