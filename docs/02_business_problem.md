# Business Problem

## Background

Large enterprises process thousands of supplier claims, invoices, trade agreements, contracts, promotional documents, freight claims, and other supporting business documents every year.

Before any payment, reimbursement, deduction, or settlement decision can be made, business users must validate the information contained within these documents and determine whether a claim is valid based on supporting evidence and business rules.

Historically, this process was heavily manual and required significant effort from operations teams.

More than 250 business users were involved in reviewing documents across multiple workflows and business functions.

---

# Existing Manual Process

The manual validation process involved several steps.

A business user would typically:

1. Open and review the incoming claim or supporting document.
2. Identify key business information required for validation.
3. Search across multiple enterprise systems and databases.
4. Cross-reference claim details with supporting documents.
5. Verify quantities, dates, amounts, products, and agreements.
6. Check whether business rules were satisfied.
7. Determine whether the claim should be approved, rejected, or escalated.
8. Document the outcome for downstream processing.

This process was repeated for every claim and document package.

Depending on document complexity, validation could take approximately 20 minutes per document.

---

# Business Challenges

## High Manual Effort

The review process required significant human effort and operational resources.

As document volumes increased, scaling the process required adding more reviewers, creating additional operational cost.

The process was dependent on business expertise and manual interpretation of document content.

---

## Risk of Financial Leakage

Business users operated under strict service-level agreements (SLAs).

Because documents needed to be processed quickly, reviewers could not always perform complete validation before payment or settlement decisions were made.

This increased the risk of:

* Invalid claims being approved
* Incorrect payment amounts
* Missed validation checks
* Financial leakage
* Inconsistent review outcomes

Reducing this risk was a major business objective.

---

## Lack of Document Standardization

One of the most significant challenges was the absence of standardized document formats.

Documents originated from multiple external business partners and retailers, each using their own layouts and structures.

Examples included:

* Claims
* Invoices
* Trade agreements
* Promotional documents
* Contracts
* Settlement forms
* Lump-sum agreements

No single template existed across all document categories.

---

## Real-World Document Complexity

The platform needed to support highly variable document conditions including:

* Poorly scanned images converted into PDFs
* Blurry and low-resolution documents
* Rotated pages
* Handwritten notes and annotations
* Checkboxes and tick marks
* Multiple tables within the same document
* Tables spanning multiple pages
* Tables spanning multiple pages without repeated headers
* Documents without visible table borders
* Multiple contracts combined into a single PDF
* Product-specific contract pages grouped within the same document
* Semi-structured and unstructured text

Many of these conditions made traditional extraction approaches unreliable.

---

## Semantic Variability

The challenge extended beyond document layout.

The same business field could appear under different labels depending on the retailer, document type, or source system.

Examples included:

| Business Field     | Possible Variations                                           |
| ------------------ | ------------------------------------------------------------- |
| Purchase Order     | PO Number, PO No, PO #, Purchase Order, Purchase Order Number |
| Amount             | Claim Amount, Invoice Amount, Settlement Amount, Total Amount |
| Product Identifier | UPC, Product Code, Item Number, SKU                           |

This variability made exact label matching difficult and increased maintenance effort for rule-based solutions.

---

## Scalability Challenges

Traditional extraction approaches often required:

* Custom rules
* Template-specific logic
* Retailer-specific mappings
* Ongoing maintenance

Every new retailer or document variation introduced additional engineering effort.

As document diversity increased, maintaining rule-based systems became increasingly expensive and difficult to scale.

---

# Why Traditional Approaches Were Not Sufficient

Several approaches were evaluated.

## Manual Review

Manual review provided flexibility but did not scale efficiently.

Processing times remained high and outcomes could vary depending on reviewer experience.

---

## Template-Based Extraction

Template-based approaches worked well for highly standardized documents but struggled when layouts changed.

Even small formatting changes often required rule updates.

---

## Pure OCR Solutions

OCR technologies successfully converted documents into text but did not solve the business interpretation problem.

OCR could identify text on a page but could not reliably determine:

* Which fields were important
* Which labels mapped to the same business concept
* How information should be interpreted across varying layouts

---

## Rule-Based Extraction

Rule-based approaches relied heavily on:

* Fixed coordinates
* Exact label matching
* Document-specific logic

These approaches became increasingly difficult to maintain as document diversity increased.

---

# Business Requirements

The business required a solution that could:

* Reduce manual review effort
* Improve validation quality
* Reduce financial leakage risk
* Support multiple document types
* Handle highly variable layouts
* Scale across retailers and business units
* Minimize onboarding effort for new document categories
* Provide traceability and auditability
* Integrate with existing enterprise systems

The solution also needed to balance accuracy, scalability, operational efficiency, and long-term maintainability.

---

# Success Criteria

The project was considered successful if it could:

* Significantly reduce manual validation effort
* Reduce claim-review turnaround times
* Improve consistency of validation outcomes
* Handle complex document layouts
* Support onboarding of new document types with minimal engineering effort
* Increase automation while preserving business trust
* Reduce operational costs
* Minimize financial leakage risk

These requirements ultimately shaped the architecture and design decisions that were made throughout the project.
