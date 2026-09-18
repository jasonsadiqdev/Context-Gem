# ContextGem: Effortless LLM extraction from documents

ContextGem is a free, open-source LLM framework that makes it radically easier to extract structured data and insights from documents — with minimal code.

## Why ContextGem?

Reliable structured extraction from documents typically involves writing extraction prompts, designing validation models, mapping outputs back to source references, orchestrating multi-step pipelines, and tracking usage across LLMs. ContextGem handles all of this through powerful abstractions — describe *what* to extract in natural language, and the framework handles *how*.

The result: structured data with precise paragraph- and sentence-level references, automatic justifications, hierarchical multi-aspect extraction, and a unified, serializable document storage model — all from minimal code.

## Key Features

- Automated dynamic prompts
- Automated data modelling
- Granular reference mapping
- Built-in justifications
- Nested context extraction
- Unified declarative pipeline

## What You Can Build

With minimal code, you can:

- Extract structured data from documents (text, images)
- Identify and analyze key aspects (topics, themes, categories) within documents
- Extract specific concepts (entities, facts, conclusions, assessments) from documents
- Build complex extraction workflows through a simple, intuitive API
- Create multi-level extraction pipelines (aspects containing concepts, hierarchical aspects)

## Installation

```bash
uv add contextgem
```

Or:

```bash
pip install -U contextgem
```

## Quick Start

This example extracts **anomalies** from a legal document — a complex concept requiring contextual understanding. Unlike traditional RAG approaches that might miss subtle inconsistencies, ContextGem analyzes the entire document context to identify content that doesn't belong, complete with source references and justifications.

```python
import os

from contextgem import Document, DocumentLLM, StringConcept


# Sample document text (shortened for brevity)
doc = Document(
    raw_text=(
        "Consultancy Agreement\n"
        "This agreement between Company A (Supplier) and Company B (Customer)...\n"
        "The term of the agreement is 1 year from the Effective Date...\n"
        "The Supplier shall provide consultancy services as described in Annex 2...\n"
        "The Customer shall pay the Supplier within 30 calendar days of receiving an invoice...\n"
        "The purple elephant danced gracefully on the moon while eating ice cream.\n"  # anomaly
        "Time-traveling dinosaurs will review all deliverables before acceptance.\n"  # another anomaly
        "This agreement is governed by the laws of Norway...\n"
    ),
)

# Attach a document-level concept
doc.concepts = [
    StringConcept(
        name="Anomalies",
        description="Anomalies in the document",
        add_references=True,
        reference_depth="sentences",
        add_justifications=True,
        justification_depth="brief",
    )
]

# Define an LLM for extracting information from the document
llm = DocumentLLM(
    model="openai/gpt-4o-mini",  # or another provider/LLM
    api_key=os.environ.get("CONTEXTGEM_OPENAI_API_KEY"),
)

# Extract information from the document
doc = llm.extract_all(doc)  # or `await llm.extract_all_async(doc)`

# Access extracted information
anomalies_concept = doc.concepts[0]
for item in anomalies_concept.extracted_items:
    print("Anomaly:")
    print(f"  {item.value}")
    print("Justification:")
    print(f"  {item.justification}")
    print("Reference paragraphs:")
    for p in item.reference_paragraphs:
        print(f"  - {p.raw_text}")
    print("Reference sentences:")
    for s in item.reference_sentences:
        print(f"  - {s.raw_text}")
    print()
```

## How It Works

### Step 1: Define extraction context

Create a `Document` containing text and/or visual content (contract, invoice, report, CV, etc.) from which an LLM extracts information.

```python
document = Document(raw_text="Non-Disclosure Agreement...")
```

### Step 2: Define what to extract

**Aspects** — extract text segments from the document (sections, topics, themes). Content can be organized hierarchically and combined with concepts for comprehensive analysis.

**Concepts** — extract specific data points with intelligent inference: entities, insights, structured objects, classifications, numerical calculations, dates, ratings, and assessments.

```python
# Extract document sections
aspect = Aspect(
    name="Term and termination",
    description="Clauses on contract term and termination",
)
# Extract specific data points
concept = BooleanConcept(
    name="NDA check",
    description="Is the contract an NDA?",
)
document.add_aspects([aspect])
document.add_concepts([concept])
```

**Alternative**: Configure an **Extraction Pipeline** — a reusable collection of predefined aspects and concepts for consistent extraction across multiple documents.

### Step 3: Run LLM extraction

Configure a **LLM** (cloud or local) to extract aspects/concepts, with support for fallback models and role-based task routing. Alternatively, configure an **LLM Group** to route different aspects/concepts to specialized LLMs (e.g., simple extraction vs. reasoning tasks).

```python
llm = DocumentLLM(
    model="openai/gpt-5-mini",
    api_key="...",
)
document = llm.extract_all(document)
```

## Focused Document Analysis

ContextGem leverages LLMs' long context windows to deliver strong extraction accuracy from individual documents. Unlike RAG approaches, which can struggle with complex concepts and nuanced insights, ContextGem capitalizes on continuously expanding context capacity, evolving LLM capabilities, and decreasing costs. This enables direct extraction from complete documents, eliminating retrieval inconsistencies while optimizing for in-depth single-document analysis. It does not currently support cross-document querying or corpus-wide retrieval — for those use cases, RAG frameworks remain more appropriate.

## Supported LLMs

ContextGem supports both cloud-based and local LLMs via LiteLLM integration:

- **Cloud LLMs**: OpenAI, Anthropic, Google, Azure OpenAI, xAI, and more
- **Local LLMs**: run via providers like Ollama, LM Studio, etc.
- **Model architectures**: works with reasoning/CoT-capable models (e.g., gpt-5) and non-reasoning models (e.g., gpt-4.1)
- **Simple API**: unified interface across LLMs with easy provider switching

For reliable structured extraction, models with performance equivalent to or exceeding `gpt-4o-mini` are recommended; smaller models (e.g., 8B parameter models) may struggle with detailed extraction instructions.

## Optimizations

Guidance is available for maximizing performance, minimizing costs, and improving extraction accuracy: optimizing for accuracy, speed, or cost; handling long documents; choosing the right LLM(s); and troubleshooting issues with small models.

## Serialization

Document objects, pipelines, and LLM configurations can be saved and loaded via built-in serialization methods:

- Save processed documents to avoid repeating expensive LLM calls
- Transfer extraction results between systems
- Persist pipeline and LLM configurations for later reuse

## Security

This project is automatically scanned for security vulnerabilities using multiple tools, including semantic code analysis for vulnerability detection, a Python security linter, and dependency vulnerability monitoring.

## License

Apache 2.0 License.