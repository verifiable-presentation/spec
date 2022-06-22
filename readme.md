# Verifiable Presentations

This project enables users to create verifiable presentations (VPs) from verifiable credentials (VCs)
or certificates previously issued to them.

The created verifiable presentations could be of two types:

#### Type 1

A VP that enables the user to indirectly prove a claim stated in a verifiable credential without
needing to share the verifiable credential itself (using selective disclosures and zero knowledge
proofs.

#### Type 2

A VP that combines and presents the claims of multiple verifiable credentials by embedding the
credentials into the presentation and making them accessible to the verifier.

## Process Breakdown

#### Programmatic retrieval of VCs from various sources

For example, retrieving certificates and proofs for a user from their DigiLocker account.

#### Parsing of VCs to retrieve the claim and their presentation.

This involves verifying and reading the VC for a list of claims, and how to render them in the VP.

#### UI for creation of a VP from retrieved VCs

A GUI that allows users to create a template, into which the claims from selected VCs or the VCs
themselves are inserted (depending on the type of VP the user chooses to create).

The GUI could be a web application, a native application or an extension for applications like MS
Word or Google Docs.

#### Signing and storage of VPs

This involves cryptographically signing and then storing the VP as a stream on an immutable ledger
(e.g., a CORD chain), and embedding the stream ID in a QR code that a verifier application can scan
to verify the VP.

#### A verification application

This application should be able to scan the QR code that is a part of the VP, and verify the VP (and
the VCs that are embedded if it is a [Type 2 VP](#type-2)).

### Resources

- [W3C VC Data Model](https://www.w3.org/TR/vc-data-model/)
