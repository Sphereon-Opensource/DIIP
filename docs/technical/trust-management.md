## European Commission List of Trusted Lists

Trust in the EU with eIDAS is organized using so called Trusted Lists (TLs). For eIDAS version 1, [implementing decisions](https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:32015D1505&qid=1716498922739) and [TS 119 612](https://www.etsi.org/deliver/etsi_ts/119600_119699/119612/02.02.01_60/ts_119612v020201p.pdf) are laying down the technical specifications and formats for Member state trusted lists.
Each member state has to provide their Trusted List containing Qualified Trust Service Providers (QTSPs) in XML format.
Trusted lists (TLs) provide information about the status and status history of the trust services from trust service providers (TSPs) regarding compliance with the relevant provisions of the applicable legislation on signatures and trust services for electronic transactions. ETSI TR 119 600 V1.2.1 (2016-03) EU Member States' trusted lists were established in EU by Commission Decision 2009/767/EC and aimed primarily at supporting the validation of advanced electronic signatures supported by a qualified certificate and advanced electronic signature under e-signature Directive supported by both a qualified certificate and by a secure signature creation device, as far as they included as a minimum trust service providers supervised/accredited for issuing qualified certificates.

In order to validate that an advanced electronic signature is supported by a qualified certificate in the context of the applicable EU legislation, a receiving party needs to check the trustworthiness of the qualified status of the certificate and that it has been issued by a trust service provider supervised to issue qualified certificates.

The combined collection of these Trusted Lists, provided by the European Commission, is called the [List of Trusted Lists (LOTL)](https://eidas.ec.europa.eu/efda/tl-browser/#/screen/home) and essentially forms the root of trust within the eIDAS framework. The machine readable form in XML format can be found [here](https://ec.europa.eu/tools/lotl/eu-lotl.xml).
Given organizations in Member States are to accept signatures and PID/QEAA data from other Member States the LOTL is the absolute root of trust in eIDAS.
Work is [underway within ETSI](https://portal.etsi.org/webapp/workProgram/Report_WorkItem.asp?wki_id=69597) to revise the Trust Lists for eIDAS v2, with an expected firs stable draft mid 2024. Up until that point we cannot really say much about how future trusted lists will look like.

## 1) X.509 Certificates for Legal Entities

X.509 Certificates used for Legal Entities are defined in [ETSI EN 319 412-3](https://www.etsi.org/deliver/etsi_en/319400_319499/31941203/01.03.01_60/en_31941203v010301p.pdf), called `Electronic Signatures and Infrastructures (ESI); Certificate Profiles; Part 3: Certificate profile for certificates issued to legal persons` which in turn relies on [rfc5280](https://www.rfc-editor.org/rfc/rfc5280.html)

### Combination of identifiers and attributes

An eIDAS X.509 Certificate issued to a Legal Entity must contain the following attributes for its `subject` field:

- countryName;
- organizationName;
- organizationIdentifier;
- commonName.

There is a benefit for Legal Entities, as it means that an X.509 Certificate next to the public key to validate any signatures created using the accompanying private key, also always includes the certificate identifying the legal entity. This is true for any certificate part of the certificate chain (see next section). The consequence is that it is very straightforward to identify which organization created a signature, and which QTSP issued the Qualified Certificate for the respective Legal Entity.

### Validation

Determining whether a Legal Entity is using an eIDAS certificate issued by a Qualified Trust Service Provider is straightforward. The X.509 certificate is part of a certificate chain, which contains the Root Certificate Authority, which will be the Qualified Trust Service Provider in case of an eIDAS Certificate. All Certificates in the chain need to be checked against the Certificate Revocation Lists (CRL) and the CA certificate of the QTSP needs to be listed in the List of Trusted lists by the EC. As soon as all these conditions are met, it means we have a certificate that is valid, active and has been issued by a QTSP in an EU Member State.

### Certificate Revocation Lists (CRL) and Online Certificate Status Protocol (OCSP)

TODO

## 2) OpenID Federation

OpenID Federation enables trust establishment and maintenance of multi-party federations. It is applying lessons learned from large-scale SAML federations and can be used for OpenID Connect, OAuth 2.0 deployments, Wallet ecosystems
It defines a hierarchical JSON-based trust establishment data structures for participants. It anables trust to be established not relying on traditional X.509 Certificates, although it does support them, as it is builing on top of OAuth2 and JSON Web Keys (JWKs), which in turn support many different key formats, including X.509 Certificates and chains.
Flexibility
It has support for protocol-specific metadata, policies, Trust Marks, and ecosystem roles. Tailored configurations meet specific needs, enhancing adaptability in diverse environments.
Legal Entities can participate in multiple federations with different Trust Anchors and even multiple roles, meaning that a Trust Anchor in one Federation can be an intermediate in another Federation

The Federation API makes Trust relationships transparent and accessible online by providing a standardized way to share and access trust information, enabling real-time navigation of the federation
Adding Legal Entities to and removing entities from organizations and federations is a local operation and only requires a superior to issue an updated Subordinate Statement about the entity below it
Constraints ensure compliance with policies and can prevent malicious parties and misconfigurations from performing actions they’re ineligible for.
Offline Usage is possible through cached trust chains. The trust_chain JWS header parameter can occur within requests, responses, attestations, and digital credentials, making it highly flexible

### Terminology

Below we will explain some of the terminology specific to OpenID Federation.

**3 Categories of Entities:**

A Legal Entity can be categorized in one of 3 roles:

- A Trust Anchor is an Entity whose main purpose is to issue statements about other Entities.
- An Intermediate Entity is an Entity that issues an Entity Statement appearing somewhere in between those issued by the Trust Anchor and the Leaf Entity in a Trust Chain.
- A Leaf Entity is an Entity with no Subordinate Entities. Leaf Entities typically play a protocol role, such as an OpenID Connect Relying Party or (Q)EAA Provider, Wallet Provider or OpenID Connect Identity Provider.

**Trust Anchor, Entity Statements and trust chains:**

- A Trust Anchor is an Entity that represents a trusted third party.
- An Entity Statement is a signed JWT that contains the information needed for an Entity to participate in federation(s), including metadata about itself and policies that apply to other Entities that it is authoritative for.
- A Trust Chain A sequence of Entity Statements that represents a chain starting at a Leaf Entity and ending in a Trust Anchor.
- A Federation Operator is a Legal Entity that is authoritative for a federation. A federation operator administers the Trust Anchor(s) for Entities in its federation
- A Trust Mark is a statement of conformance to a well-scoped set of trust and/or interoperability requirements as determined by an accreditation authority

Below is a graphical example of how 2 Trust Anchors and their intermediates and leaves could look like

```text
.-----------------.            .-----------------.
|  Trust Anchor A |            |  Trust Anchor B |
'------.--.-------'            '----.--.--.------'
       |  |                         |  |  |
    .--'  '---. .-------------------'  |  |
    |         | |                      |  |
.---v.  .-----v-v------.   .-----------'  |
| OP |  | Intermediate |   |              |
'----'  '--.--.--.-----'   |    .---------v----.
           |  |  |         |    | Intermediate |
   .-------'  |  '------.  |    '---.--.--.----'
   |          |         |  |        |  |  |
.--v-.      .-v--.     .v--v.   .---'  |  '----.
| RP |      | OP |     | OP |   |      |       |
'----'      '----'     '----'   |   .--v-.   .-v--.
                                |   | RP |   | RP |
                                |   '----'   '----'
                                |
                        .-------v------.
                        | Intermediate |
                        '----.--.--.---'
                             |  |  |
                       .-----'  |  '----.
                       |        |       |
                    .--v-.   .--v-.   .-v--.
                    | OP |   | RP |   | AS |
                    '----'   '----'   '----'
Legend for leaves:
RP: Relying Party, Credential Verifier
OP: OpenID Provider or PID, (Q)EAA Provider/Credential Issuer
AS: OAuth2 Authorization Server

```

**Metadata and Policies:**

- Metadata declarations are metadata for entity types, like for example, PID/(Q)EEA metadata similar to what is registered for Issuers in the OID4VCI specification, but also Relying Party metadata, allowing to provide metadata about the RP and the credentials it would like to receive, similar to OID4VP.
- Metadata Policies enable superiors to provide and constrain metadata values further down the trustchain. For example, constraining signing algorithms within a Federation. Subordinate entities can never broaden policies created by superiors, they however can further restrict them.

### Trust Chain validation

As described above a Trust Chain is a list of signed JWTs containing Entity Statements and optional Trust Marks starting with a Leaf Node, optional Intermediate Entities and a Trust Anchor. These signed JWTs are thus fully verifiable and already contain all the information.
Verification means to start with the top most message, verify the signature against the Trust Anchor keys If that is okay, use the JWKs (JSON Web Key set) in the statement to verify the next message in the chain
Repeat until end of chain (Trust Anchor Entity Configuration reached). See the [specification](https://openid.net/specs/openid-federation-1_0.html#name-validating-a-trust-chain) for more details on this process.

## 3) European Blockchain Services Infrastructure (EBSI)

TODO

## Comparison of X.509 CRLs, OpenID Federation and EBSI

|                       | X.509                                                | OpenID Fed                                                                                         | EBSI                                                                           |
| --------------------- | ---------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Format                | ASN.1/DER                                            | JWT                                                                                                | JWT (non VC)                                                                   |
| REST APIs             | No                                                   | Yes                                                                                                | Yes                                                                            |
| Attestation           | Public Key Certificate                               | Entity Statement                                                                                   | Verifiable Credentials                                                         |
| amount of Public Keys | 1                                                    | 1 or more                                                                                          | 1 or more                                                                      |
| Chain namme           | Certificate Chain                                    | Trust Chain                                                                                        | Trust Chain                                                                    |
| Chain contents        | Identity, Public Key, constraints, custom extensions | Identity, public KeyS, constraints, specific metadata, Trust Marks, policies, optional X.509 certs | TODO: Identity Verifiable Credentials, identifiers; links to public KeyS (DID) |

|               | X.509                                                                                                                 | OpenID Fed                                                                                                                                                                                           | EBSI                                                                                                                                                                                              |
| ------------- | --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Root entities | Certificate Authority (CA) stores, issues and signs the certificates eIDAS -> (Q)TSP                                  | Federation Operator (FO) An organization that is authoritative for a federation. A FO administers the Trust Anchor(s) for Entities in its federation.                                                | (Root) Trusted Accreditation Organisation (RTAO) verifies, accredits and manages the entities, i.e. Trusted Issuers, that issue electronic documents.                                             |
|               | Registration Authority verifies identity, requesting certificates to be issued by the CA eIDAS -> (Q)TSP (1 time only | Intermediate Entity An Entity that issues an Entity Statement appearing somewhere in between those issued by the Trust Anchor and the Leaf Entity in a Trust Chain. Could also be an Issuer, RP etc. | Trusted Issuer (TI) is responsible to issue certain types of electronic documents and to manage their signing keys with the support of blockchain. In technical terms, it manages a DID document. |
|               | Subject Legal Entities, Organizations (or natural persons) requesting a Certificate                                   | Leaf Entity An Entity with no Subordinate Entities. Legal entities, like issuers, RPs, organizations                                                                                                 | Legal Entities Organizations registered with an identity on EBSI (subjects of credentials)                                                                                                        |

Needs more work and is subject to a certain extent

|      | X.509                                                                                    | OpenID Fed                                                 | EBSI                                                  |
| ---- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------- |
| PROs | Proven technology                                                                        | Independent of PKI formats. Can be used with X.509 as well | History: Keys and schemas                             |
|      | Certificate chains transport identity data as well                                       | Multiple Keys                                              | No Phone home (decentralized)                         |
|      | Used in federated models (eIDAS 1 for example)                                           | Metadata publishing (EAA Providers, wallet Providers, RPs) | Potential resilience (decentralized)                  |
|      | Policies (requirements typically technical like which key types are supported)           |                                                            |                                                       |
|      | Hierarchy                                                                                |                                                            |                                                       |
|      | Allows for multiple roles/locations within Hierarchy                                     |                                                            |                                                       |
| CONs | Requires usage and purchase of “expensive” X.509 certificates                            | New technology                                             | New technology                                        |
|      | “Scalability at large for revocations”                                                   | Not many implementations                                   | Not many implementations                              |
|      | Support for 1 key only                                                                   | Adoption?                                                  | Lots of parties using the same CEF provided libraries |
|      | Officially hierarchical, in reality flat (CAs, intermediate CAs => end user Certificate) | Depends on availability of APIs for keys                   | “Blockchain”, red-flag for some                       |
|      |                                                                                          |                                                            | Adoption?                                             |
|      |                                                                                          |                                                            | Scalability yet to be proven                          |
