# Key Engineering Decisions

## Introduction

Building a document intelligence platform required more than selecting OCR and LLM technologies.

Several architectural and engineering decisions were made to balance scalability, maintainability, reliability, cost, and business value.

This document outlines the most important decisions made during the design and implementation of the platform and the reasoning behind them.

---

# Decision 1: Metadata-Driven Architecture

## Challenge

Traditional document extraction systems often rely on hardcoded logic.

When a new document type, retailer, or business workflow is introduced, developers must modify code, perform testing, and redeploy the application.

As document diversity increases, maintenance becomes increasingly difficult.

---

## Decision

Externalize document-processing behavior into metadata stored in Snowflake.

Metadata included:

* Document Type
* Retailer
* Document Category
* Fields To Extract
* Prompt Templates
* Validation Rules

---

## Why

This approach enabled:

* Faster onboarding
* Reduced maintenance effort
* Better scalability
* Greater flexibility

New document types could often be onboarded through configuration updates rather than software development.

---

## Outcome

The platform evolved from a document-specific implementation into a reusable enterprise capability.

---

# Decision 2: Hybrid Extraction Strategy

## Challenge

Not every document requires LLM-based reasoning.

Using LLMs for all documents increases:

* Cost
* Latency
* Complexity

without necessarily improving results.

---

## Decision

Adopt a hybrid extraction strategy.

Depending on document complexity, the platform could leverage:

* OCR extraction
* Traditional parsing techniques
* Open-source libraries
* LLM-based extraction

---

## Why

Use the simplest solution that solves the problem.

Reserve LLMs for situations where contextual understanding delivers measurable value.

---

## Outcome

Improved:

* Cost efficiency
* Processing speed
* Scalability
* Explainability

while maintaining extraction quality.

---

# Decision 3: Separate OCR Evaluation From LLM Evaluation

## Challenge

When extraction failures occur, it is important to understand where the failure originated.

Without clear separation, teams may incorrectly blame the LLM when the underlying issue is OCR quality.

---

## Decision

Store OCR outputs separately from extraction results.

Evaluate OCR quality independently.

---

## Why

Enable root-cause analysis.

Determine whether issues originate from:

* OCR extraction
* Prompt design
* Validation logic
* Document quality

---

## Outcome

Improved troubleshooting and continuous improvement efforts.

---

# Decision 4: Dynamic Prompt Selection

## Challenge

Different document types require different extraction instructions.

Hardcoding prompts within application code creates maintenance challenges.

---

## Decision

Store prompts in Snowflake and dynamically retrieve them based on:

* Document Type
* Retailer
* Document Category

---

## Why

Support multiple document categories without requiring code changes.

Enable prompt evolution independently of application releases.

---

## Outcome

Improved maintainability and simplified onboarding of new document categories.

---

# Decision 5: Iterative Prompt Development

## Challenge

Prompt quality directly impacts extraction performance.

Prompts developed in a single iteration often fail to generalize across document variations.

---

## Decision

Adopt an iterative prompt development process.

For new document categories:

1. Evaluate OCR performance.
2. Create an initial prompt.
3. Test against sample documents.
4. Refine extraction instructions.
5. Validate results.
6. Promote the finalized prompt to Snowflake.

---

## Why

Reduce extraction variability and improve reliability.

---

## Outcome

More consistent extraction quality across document categories.

---

# Decision 6: Token-Aware Chunking

## Challenge

Many documents exceeded LLM context-window limitations.

Examples included:

* Large contracts
* Multi-page agreements
* Product-specific pricing documents
* Documents containing multiple tables

---

## Decision

Implement token-aware chunking using TikToken.

---

## Why

Prevent context-window overflows while enabling processing of large documents.

---

## Outcome

Support for large documents without sacrificing extraction coverage.

---

# Decision 7: Overlapping Chunk Strategy

## Challenge

Important information can be split across chunk boundaries.

Example:

```text
Chunk A:
PO Number:

Chunk B:
123456789
```

Without context preservation, extraction quality can degrade.

---

## Decision

Implement overlapping chunk boundaries.

Multiple overlap values were evaluated before selecting a production configuration.

---

## Why

Preserve contextual continuity while balancing token usage.

---

## Outcome

Improved extraction accuracy across chunk boundaries.

---

# Decision 8: Context-Window Safety Buffer

## Challenge

LLMs are not deterministic.

Response lengths can vary between executions.

Using the full context window increases the risk of runtime token-limit failures.

---

## Decision

Utilize approximately 90% of the available context window.

Reserve remaining capacity as a safety buffer.

---

## Why

Improve system stability and reduce processing failures.

---

## Outcome

More predictable runtime behavior and reduced token-limit errors.

---

# Decision 9: Validation Before Consumption

## Challenge

LLM outputs cannot be consumed blindly in enterprise workflows.

Incorrect outputs can introduce downstream business risk.

---

## Decision

Introduce a dedicated validation layer before storing results.

Validation included:

* Schema validation
* Mandatory field checks
* Data type validation
* Format validation
* Business rule validation
* Pydantic validation

---

## Why

Improve reliability and trustworthiness.

---

## Outcome

Reduced downstream errors and improved confidence in extracted outputs.

---

# Decision 10: Confidence-Based Human Review

## Challenge

Full manual review limits scalability.

Fully automated processing introduces risk.

---

## Decision

Adopt a confidence-based review model.

Human review is reserved for:

* Poor-quality documents
* Handwritten-heavy documents
* Missing mandatory fields
* Validation failures
* Low-confidence extractions

---

## Why

Balance automation with risk management.

---

## Outcome

Reduced manual effort while maintaining business trust.

---

# Decision 11: Observability By Design

## Challenge

AI systems can be difficult to debug.

Without observability, continuous improvement becomes difficult.

---

## Decision

Store and track:

* OCR outputs
* Extraction outputs
* Validation outcomes

as separate artifacts.

---

## Why

Enable detailed analysis of failures and performance trends.

---

## Outcome

Improved monitoring, debugging, and iterative improvement.

---

# Decision 12: Configuration Over Custom Development

## Challenge

Every new document type increases maintenance burden in traditional systems.

---

## Decision

Favor configuration changes over application code changes whenever possible.

---

## Why

Reduce engineering effort required for platform expansion.

---

## Outcome

Improved scalability and reduced long-term maintenance costs.

---

# Key Takeaway

The success of the platform was driven not by a single technology choice, but by a series of engineering decisions focused on scalability, maintainability, reliability, and business value.

The combination of metadata-driven configuration, hybrid extraction, validation controls, human oversight, observability, and experimentation-based optimization transformed the solution from a document extraction tool into a reusable enterprise document intelligence platform.
