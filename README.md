# Company Passport Architecture

> **TODO:** Repository Description

## Project description

> **TODO:** Project description

## Use case

1. **Notary Verification & Incorporation Deed**:
   Company representatives present statutes to a notary, who verifies stakeholder identities and statutes legality, then **issues the deed of incorporation**, formalizing company establishment.

2. **Chamber of Commerce Registration**:
   The company submits incorporation documents to the Chamber of Commerce (KVK), which verifies and approves these documents, issuing a company registration number and formally registering the company.

3. **Tax Registration**:
   The company applies for tax identification and VAT registration with the Tax Authority (Belastingdienst), which verifies the company's legal status and tax eligibility to ensure national tax regulation compliance.

4. **Bank Account Opening**:
   The company approaches a bank to open a business account, necessary for financial transactions. The bank conducts KYC and AML checks to ensure financial regulation compliance before account creation.

![Alt text](assets/use-case-seq.png)

## Terms

### Wallet

In the context of this project, the term *wallet* is defined as a software component capable of receiving and storing *verifiable credentials (VCs)*, and generating and presenting *verifiable presentations (VPs)* based on those VCs.

### Company passport

The company passport is the full set of credentials and claims that describe a company.

### Organizational wallet

A wallet that acts on behalf of the company's identity.

### Personal wallet(s)

The wallet(s) that act on behalf of each of the founders.