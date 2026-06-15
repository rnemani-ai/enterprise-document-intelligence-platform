# Architecture

## Architecture Goals

The platform was designed to address a fundamental challenge faced by enterprise document-processing workflows: extracting and validating information from highly variable documents while maintaining scalability, reliability, and cost efficiency.

Several architectural goals guided the design:

* Minimize manual document review effort
* Improve validation quality
* Reduce financial leakage risk
* Support highly variable document formats
* Enable onboarding of new document types with minimal engineering effort
* Balance extraction accuracy with operational cost
* Provide observability and traceability
* Support enterprise-scale document processing

Rather than building a document-specific solution, the architecture was designed as a reusable enterprise platform capable of supporting multiple document categories and business workflows.

---

# High-Level Architecture

The platform follows a metadata-driven architecture where document-processing behavior is determined through configuration rather than hardcoded logic.

```text
Document
     │
     ▼

Document Metadata
(Doc Type / Retailer / Category)

     │
     ▼

Snowflake Prompt Repository

     │
     ▼

Select Configuration

     │
     ▼

OCR Layer

 ┌───────────────┐
 │ Azure DI      │
 │ PDFPlumber    │
 │ Tesseract     │
 └───────────────┘

     │
     ▼

OCR Text Stored
In Snowflake

     │
     ▼

Chunking Layer

 • TikToken
 • Overlap Strategy
 • 90% Safety Buffer

     │
     ▼

Prompt Construction

     │
     ▼

GPT-3.5 Turbo

     │
     ▼

Validation Layer

 • Schema Validation
 • Pydantic Checks
 • Mandatory Fields
 • Business Rules

     │
     ▼

Confidence Assessment

     │
     ▼

 ┌─────────────────┬─────────────────┐
 │                 │
 ▼                 ▼

High           Review
Confidence     Required

 │                 │

 └────────┬────────┘
          ▼

Extracted Fields

          ▼

Snowflake
```

---

# End-to-End Processing Flow

A typical document follows the workflow below:

1. A document enters the platform.
2. Document metadata is identified.
3. Appropriate extraction configuration is retrieved from Snowflake.
4. OCR processing is performed.
5. OCR text is stored for traceability.
6. Large documents are segmented using token-aware chunking.
7. Prompts are dynamically generated.
8. GPT-3.5 Turbo performs contextual extraction.
9. Validation rules are applied.
10. Confidence assessment is performed.
11. Documents requiring additional verification are routed for human review.
12. Final structured outputs are stored in Snowflake.

This workflow enables consistent processing across diverse document types while maintaining traceability and reliability.

---

# Metadata Repository

One of the most important architectural components is the metadata repository.

Instead of embedding extraction logic directly into application code, document-processing behavior is controlled through metadata stored in Snowflake.

Metadata includes:

* Document Type
* Retailer
* Document Category
* Fields To Extract
* Prompt Templates
* Extraction Instructions
* Validation Requirements

Examples:

```text
Document Type: Trade Agreement

Retailer: Retailer A

Fields:
- UPC
- Effective Date
- End Date
- Discount Amount

Prompt:
Trade Agreement Extraction Prompt
```

This architecture allows new document types to be onboarded through configuration updates rather than software deployments.

---

# OCR Layer

The OCR layer is responsible for converting document content into machine-readable text.

Multiple OCR technologies were evaluated and utilized during development, including:

* Azure Document Intelligence
* PDFPlumber
* Tesseract OCR

The appropriate OCR strategy depended on document complexity and quality.

Azure Document Intelligence demonstrated strong performance for:

* Complex layouts
* Blurry documents
* Rotated pages
* Multi-page documents

In some cases, OCR successfully extracted content that was difficult for humans to read visually.

Challenges remained for:

* Handwritten content
* Poor-quality scans
* Ambiguous character recognition

Examples:

* 0 vs O
* 5 vs S
* 1 vs I

These limitations informed downstream validation and review workflows.

---

# OCR Traceability

A key architectural decision was storing OCR outputs separately from final extraction results.

Benefits included:

* Easier debugging
* Root-cause analysis
* Extraction failure investigations
* Continuous improvement efforts

When extraction issues occurred, teams could determine whether the root cause originated from:

* OCR quality
* Prompt behavior
* Validation logic
* Document quality

This significantly improved observability.

---

# Chunking Layer

Some documents exceeded LLM context-window limitations.

Examples included:

* Large contracts
* Multi-product agreements
* Multi-page documents
* Documents containing numerous tables

To address this challenge, token-aware chunking was implemented.

The chunking process included:

* Token counting using TikToken
* Dynamic segmentation
* Overlapping chunk boundaries
* Response consolidation

---

# Overlap Optimization

Chunk overlap was introduced to preserve context across chunk boundaries.

For example:

```text
Chunk A:
UPC:

Chunk B:
123456789
```

Without overlap, important information could be lost.

Multiple overlap configurations were evaluated before selecting a production configuration.

The final implementation balanced:

* Extraction quality
* Context preservation
* Token utilization
* Processing cost

---

# Context Window Safety Buffer

Another important design decision was limiting context-window utilization to approximately 90% of available capacity.

This accounted for:

* Variable response lengths
* Non-deterministic LLM behavior
* Prompt size variability

The safety buffer reduced runtime failures caused by token-limit overruns and improved system stability.

---

# Prompt Construction Layer

Prompts were dynamically generated using metadata retrieved from Snowflake.

Prompts included:

* Field definitions
* Extraction instructions
* Output format requirements
* Document-specific guidance

Prompt management was externalized from application code, enabling easier maintenance and onboarding.

Before a prompt was promoted into production, multiple iterations and evaluations were performed to optimize extraction quality.

---

# LLM Extraction Layer

GPT-3.5 Turbo was used for contextual extraction.

The model provided value beyond OCR by:

* Understanding document context
* Handling layout variability
* Resolving semantic variability
* Identifying fields across differing formats

For example:

```text
PO Number
PO No
PO #
Purchase Order Number
```

could all be interpreted as the same business field.

This significantly reduced dependency on retailer-specific extraction rules.

---

# Validation Layer

The validation layer served as a critical reliability mechanism.

Validation included:

## Schema Validation

Ensuring outputs matched expected structures.

## Mandatory Field Validation

Verifying required fields were present.

## Format Validation

Examples:

* Date formats
* Numeric values
* Currency values

## Business Rule Validation

Examples:

* Value ranges
* Domain constraints
* Field consistency checks

## Pydantic Validation

Pydantic models provided structured validation and enforcement of expected output schemas.

---

# Confidence Assessment

After validation, results were evaluated for confidence and completeness.

Factors considered included:

* Missing mandatory fields
* Validation failures
* OCR quality concerns
* Document quality issues

The objective was to identify documents requiring additional review before downstream consumption.

---

# Human Review Workflow

The platform adopted a risk-based review model.

Manual review was reserved for higher-risk scenarios including:

* Poor-quality scans
* Handwritten-heavy documents
* Missing required information
* Validation failures
* Low-confidence outputs

This allowed business users to focus their attention where it delivered the greatest value while maximizing automation elsewhere.

---

# Storage Layer

Snowflake served as the central repository for:

## Metadata Repository

* Prompts
* Fields
* Configurations

## OCR Repository

* OCR-extracted text

## Extraction Repository

* Final structured outputs

This centralized architecture simplified maintenance, reporting, auditing, and troubleshooting.

---

# Architectural Decisions That Enabled Scale

Several architectural decisions significantly improved scalability:

## Metadata-Driven Configuration

Configuration rather than code.

## Hybrid Extraction Strategy

Use LLMs only where necessary.

## Dynamic Prompt Selection

Prompts selected based on document metadata.

## Externalized Validation Rules

Business logic separated from extraction logic.

## Human-in-the-Loop Review

Focused reviewer effort on high-risk documents.

## OCR Traceability

Enabled continuous improvement.

These decisions transformed the solution from a document-specific implementation into a reusable enterprise platform capable of supporting future growth.

---

# Key Architectural Takeaway

The most important architectural decision was externalizing document-processing behavior from application code.

By combining metadata-driven configuration, hybrid extraction strategies, validation controls, and human oversight, the platform was able to balance scalability, reliability, maintainability, and business value.

The resulting architecture supported diverse document types while minimizing engineering effort required for onboarding and ongoing maintenance.
