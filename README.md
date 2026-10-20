# Tibetan ID

Privacy-preserving access and membership architecture for the New Tibet ecosystem.

> [!IMPORTANT]
> This is a concept architecture for privacy, legal, governance, and security review. It is not yet a production specification.

## Contents

1. [User types and access](#1-user-types-and-access)
2. [Green Books, Blue Books, and proof of personhood](#2-green-books-blue-books-and-proof-of-personhood)
3. [Admission paths](#3-admission-paths)
4. [Foundation Verification Portal](#4-foundation-verification-portal)
5. [Encrypted verification dossier](#5-encrypted-verification-dossier)
6. [Vouches and trust levels](#6-vouches-and-trust-levels)
7. [Onchain records and public attributes](#7-onchain-records-and-public-attributes)
8. [Threshold custody and exceptional access](#8-threshold-custody-and-exceptional-access)
9. [Future CTA integration](#9-future-cta-integration)
10. [Deletion, revocation, and re-entry](#10-deletion-revocation-and-re-entry)
11. [Implementation work](#11-implementation-work)
12. [References](#12-references)

## 1. User types and access

Tibetan ID separates holding the `$TIBET` coin from being a verified or sponsored member.

| User type | How they enter | Access |
| --- | --- | --- |
| `$TIBET` coin holder | Connects a wallet that holds `$TIBET` | **Wallet tab only.** No membership areas, profile verification, voting, member directory, invite creation, or vouching. |
| Sponsored member | Redeems an invite from an eligible Green Book holder | Membership access allowed by the current ecosystem policy; profile remains marked `Invite-only`. |
| Green Book verified member | Passes Green Book, ZKPassport, and Foundation verification | Tibetan membership class plus any community vouches. |
| Blue Book verified member | Passes Blue Book, ZKPassport, and Foundation verification | Supporter membership class plus any community vouches. Cannot invite or vouch. |

A wallet-only user is not treated as a member merely because they hold tokens. Membership revocation or deletion also remains separate from asset ownership: it can remove protected ecosystem access but cannot freeze or move the user's funds.

A wallet-only user may later apply through any membership path, but retains only Wallet-tab access until a membership credential is issued.

```mermaid
flowchart TD
    A[User opens application] --> B{Entry route}
    B -->|Connect wallet holding TIBET| C[Wallet-only session]
    C --> D[Wallet tab]
    C -. blocked .-> E[All membership areas]

    B -->|Apply for membership| F{Admission path}
    F -->|Green Book plus ZKPassport| G[Verified Tibetan member]
    F -->|Blue Book plus ZKPassport| H[Verified Supporter member]
    F -->|Green Book holder invite| I[Sponsored invite-only member]
```

## 2. Green Books, Blue Books, and proof of personhood

- **Green Book - Tibetans.** The Central Tibetan Administration (CTA) Department of Finance maintains the Green Book system for Tibetans and records voluntary contributions.
- **Blue Book - Tibetan supporters.** The CTA describes the Blue Book as the Tibetan Solidarity Partnership, introduced in 1996 for friends and supporters of Tibet.

Within Tibetan ID, Green Book holders and Blue Book holders may both be verified as members. Only a currently eligible Green Book holder may issue invite codes or provide vouches. A Blue Book holder cannot invite or vouch for another person.

### 2.1 Why ZKPassport is added

The book process depends partly on uploaded images and human review. Those materials may be forged, altered, borrowed, or generated with AI, and the Green Book and Blue Book systems do not have the same cryptographic document infrastructure as modern NFC passports.

ZKPassport is therefore used as an additional **proof-of-personhood signal**. It verifies the cryptographic signature of a supported NFC passport or identity document and can prove selected facts without exposing the source document. This reduces the chance that a completely synthetic or AI-generated applicant passes the book process.

It does not prove that the Green Book or Blue Book is genuine, and it does not by itself prove that the book belongs to the passport holder. The Foundation verifier must still assess the book evidence and bind the book submission and ZKPassport proof to the same one-time application session. Suspicious cases require step-up review.

It should also not be treated as an absolute guarantee of one-human-one-account. Replay, renewal, and multiple-document risks require separate nullifier and duplicate-account controls.

The retained ZKPassport results are limited to:

- a successful proof-of-personhood result;
- country; and
- whether the applicant is over 18.

No name, exact date of birth, passport number, address, photograph, or raw passport data is received from ZKPassport.

## 3. Admission paths

### 3.1 Green Book or Blue Book path

1. The applicant creates a one-time application session.
2. The applicant securely uploads the Green Book or Blue Book information required for review.
3. The applicant completes a ZKPassport proof bound to the same session.
4. A trusted Foundation verifier reviews the book evidence and the proof-of-personhood result in the Foundation Verification Portal.
5. If approved, a trusted Foundation wallet records a privacy-minimised EAS attestation and issues a pseudonymous membership credential.
6. Audit-grade evidence moves into the encrypted verification dossier. Temporary copies are erased.

### 3.2 Green Book holder invitation path

1. An eligible Green Book holder creates a single-use, expiring invite code.
2. The applicant enters the invite code.
3. The service verifies that the issuer is a currently eligible Green Book holder and that the code is unused and unexpired.
4. The applicant does not need a Green Book, Blue Book, passport, or ZKPassport proof.
5. The code is consumed, the inviter supplies the first vouch, and the profile is marked `Invite-only`.
6. A trusted Foundation wallet records a sponsored membership attestation and issues a pseudonymous credential.

This path is important for Tibetans who need to remain anonymous, may face personal or political risk if they upload book-linked information, do not have a passport, or do not have access to a compatible NFC document or device. It creates a community-supported route into the application without publicly asserting a government or book-backed identity.

An invite-only profile can later submit additional invite codes from other distinct, eligible Green Book holders. Each valid additional code becomes another vouch and strengthens the member's web-of-trust standing.

```mermaid
flowchart TD
    A[Applicant starts one-time session] --> B{Choose path}

    B -->|Green Book| C[Securely upload Green Book evidence]
    B -->|Blue Book| D[Securely upload Blue Book evidence]
    C --> E[Complete session-bound ZKPassport proof]
    D --> E
    E --> F[Foundation verifier reviews evidence and proof of personhood]

    B -->|Invite code| G[Applicant enters Green Book holder invite]
    G --> H[Validate issuer, expiry, and single use]
    H --> I[No book, passport, or ZKPassport required]
    I --> J[Prepare Invite-only membership with first vouch]

    F --> K{Approved?}
    J --> K
    K -->|No| L[Reject, request correction, or step-up review]
    K -->|Yes| M[Trusted Foundation wallet issues EAS attestation]
    M --> N[Issue credential and profile with admission basis]
    M --> O[Create encrypted verification dossier]

    N --> P[Member may add more Green Book holder invite codes]
    P --> Q[Validate each distinct voucher and update standing]
```

## 4. Foundation Verification Portal

The **Foundation Verification Portal** is proposed as a dedicated, self-hosted service under Foundation control. Applicants submit evidence there and authorised Foundation reviewers inspect personally identifiable information (PII). It is separate from the public application and the ordinary member database; a consumer cloud drive or third-party KYC dashboard is not the system of record.

### 4.1 Applicant side

1. The public application requests a short-lived case identifier from the portal.
2. The applicant uploads evidence directly into an encrypted quarantine area using an expiring upload link.
3. The application session binds the upload, ZKPassport challenge, and later decision together without exposing the applicant's ordinary wallet address.
4. Files are scanned for malware inside the controlled environment and are not sent to consumer file-scanning, analytics, or AI services.
5. The applicant can see the case status, provide corrections, and exercise deletion or appeal rights.

### 4.2 Verifier side

An authorised verifier may view only the cases assigned to them. The review screen should show:

- the original Green Book or Blue Book submission;
- the information visible in that evidence that is needed to judge authenticity;
- the ZKPassport proof-of-personhood receipt, country, and over-18 result;
- image and file metadata, upload time, and cryptographic hashes;
- automated tamper warnings, without using the evidence to train AI models;
- prior decisions or duplicate indicators where policy permits; and
- a structured decision form with reason codes and reviewer notes.

The verifier approves, rejects, requests clearer evidence, or sends the case to step-up review. Suspicious or overridden cases require a second independent reviewer. The final approval is signed by a trusted Foundation wallet and linked to the EAS attestation.

### 4.3 Data-protection safeguards

- **Strong access control:** named accounts, phishing-resistant hardware keys or passkeys, least-privilege roles, no shared reviewer accounts, and rapid offboarding.
- **Restricted review environment:** private network or zero-trust access, short sessions, no download or printing by default, visible case watermarks, and controlled export only with a recorded reason.
- **Encryption:** TLS in transit, per-case encryption at rest, separately managed keys, encrypted backups, and regular key rotation.
- **Auditability:** every view, change, export, decision, and exceptional access is timestamped and signed in a tamper-evident private log.
- **Data minimisation:** no advertising analytics, third-party trackers, PII in application logs, or reuse of evidence for model training.
- **Retention:** temporary upload copies are erased after ingestion. The audit dossier is retained only for the documented appeal, fraud-prevention, and legal period, then cryptographically erased unless a lawful exception applies.
- **Operational safeguards:** periodic access reviews, incident response and breach notification procedures, processor agreements, backup-erasure testing, and an independent privacy and security assessment before launch.
- **Human safeguards:** trained reviewers, confidentiality commitments, conflict-of-interest declarations, and two-person approval for unusual cases or raw-data export.

```mermaid
flowchart LR
    A[Applicant] -->|Expiring encrypted upload| B[Portal quarantine]
    B --> C[Malware scan inside controlled environment]
    C --> D[Assigned Foundation reviewer]
    E[Session-bound ZKPassport receipt] --> D
    D -->|Approve| F[Second review when risk rules require]
    D -->|Correct| G[Applicant correction request]
    D -->|Reject| H[Appeal path]
    F --> I[Foundation wallet signs decision]
    I --> J[EAS attestation with no PII]
    I --> K[Encrypted verification dossier]
    B -->|Erase temporary copy after ingestion| L[Deletion log]
```

## 5. Encrypted verification dossier

The earlier minimal identity-capsule concept is insufficient for later fraud review. An auditor cannot judge whether a book was forged from only a membership class and approval timestamp. It is replaced by an **encrypted verification dossier** that preserves enough evidence to reconstruct and audit the decision while keeping it outside the public chain.

### 5.1 Dossier contents

- Original uploaded Green Book or Blue Book files in the submitted format.
- Cryptographic hashes, upload timestamps, file metadata, and chain-of-custody events.
- Book type and the fields necessary for verification, such as the visible holder details, book identifier, issuing office, status markers, and other authenticity indicators actually present on the document.
- ZKPassport verification receipt: successful proof of personhood, country, over-18 result, session challenge, verification time, and verifier or circuit version. No raw passport data.
- Reviewer identity, review times, reason codes, notes, fraud indicators, and any second-review decision.
- The final membership class, policy version, credential identifier, and EAS attestation UID.
- Consumed invite commitments, vouch receipts, and revocation or appeal history.
- Step-up review material only when necessary, separately labelled and given a shorter retention period where possible.

### 5.2 Why it is auditable

With authorised 2-of-3 access, an independent auditor can compare the original evidence with the recorded decision, verify that the files have not changed, check which reviewer saw them, confirm the ZKPassport receipt and policy version, and reconcile the result with the onchain attestation.

The dossier is deliberately more complete than the public profile. Its existence increases privacy risk, so access, retention, key separation, and deletion must be treated as core security controls rather than administrative details.

## 6. Vouches and trust levels

Tibetan ID uses two separate trust dimensions so that book verification and community confidence are not confused.

### 6.1 Verification basis

| Basis | Meaning |
| --- | --- |
| Wallet-only | Holds `$TIBET`; not a member and has only Wallet-tab access. |
| Invite-only | Entered through one Green Book holder invite; no book or passport claim is made. |
| Green Book verified | Foundation verified Green Book evidence plus ZKPassport proof of personhood. |
| Blue Book verified | Foundation verified Blue Book evidence plus ZKPassport proof of personhood. |

### 6.2 Community web of trust

Every sponsored, Green Book verified, or Blue Book verified member may add further invite codes issued by distinct eligible Green Book holders. Each accepted code adds one vouch to the profile. Blue Book holders may receive vouches but cannot issue them.

The public profile should show the verification basis and vouch count separately, for example:

- `Invite-only · 3 Green Book holder vouches`
- `Green Book verified · 2 Green Book holder vouches`
- `Blue Book verified · 4 Green Book holder vouches`

Voucher names, wallet addresses, and the invite codes themselves are not public. Pairwise voucher identifiers prevent one Green Book holder from being counted twice for the same member without exposing a cross-member social graph.

The proposed maximum of five counted vouches is only an initial idea. Governance may adjust or remove the cap after abuse testing and community review. The active cap must be versioned rather than hard-coded into a permanent identity claim.

```mermaid
flowchart TD
    A[Member profile] --> B[Verification basis]
    A --> C[Community standing]

    B --> D[Invite-only]
    B --> E[Green Book verified]
    B --> F[Blue Book verified]

    C --> G[Enter another Green Book holder invite code]
    G --> H[Check issuer is eligible and distinct]
    H --> I[Consume code and add one vouch]
    I --> J[Display updated vouch count under current policy cap]
```

## 7. Onchain records and public attributes

Ethereum Attestation Service (EAS) is the public integrity and revocation layer, not the PII database. The trusted Foundation wallet or another authorised verifier may issue attestations.

| EAS record | Public contents |
| --- | --- |
| Membership status | High-entropy member commitment, active/restricted/revoked state, verification basis, standing, policy version, and salted dossier commitment |
| Vouch | Member commitment, pairwise voucher identifier, and policy version |
| Access approval | Random request commitment and expiry |
| Audit anchor | Merkle root of the private access log |
| Deletion receipt | Fresh, non-identifying erasure commitment |

EAS must never contain names, contact details, wallet-to-person mappings, book or passport numbers, images, invite codes, raw ZK proofs, dossier locations, or low-entropy hashes of those values.

### 7.1 Country and over-18 status

Country and the over-18 flag are the only ZKPassport attributes eligible for public display. Exact country can nevertheless identify a person by inference during the early stages - for example, if the community knows that only one member is based in the United Kingdom.

Safeguards should therefore include:

1. **Separate consent:** public display is not required to obtain membership.
2. **Cohort threshold:** exact country is withheld until a configurable minimum number of members share that country, initially suggested as at least five.
3. **Coarse fallback:** before the threshold is met, show a broad region or `Withheld for privacy`, without publishing the hidden count.
4. **Combination protection:** apply the threshold to combinations such as country, age flag, verification basis, and joining period, not country alone.
5. **Delayed batching:** publish changes in batches so joining time cannot easily be matched to a particular applicant.
6. **No default onchain disclosure:** store a commitment in EAS and reveal the attributes through the application only after the privacy gate passes. Putting exact country directly onchain would make later withdrawal impossible.
7. **User withdrawal:** a member may stop future application display, while being clearly warned that already public third-party copies cannot be recalled.
8. **No wallet linkage:** public attribute profiles use pseudonymous member identifiers, not ordinary wallet addresses.

The over-18 flag may be less identifying than country, but it is still evaluated as part of the combined privacy threshold.

```mermaid
flowchart TD
    A[Verified country and over-18 result in encrypted dossier] --> B{Member separately consents to display?}
    B -->|No| C[Keep private]
    B -->|Yes| D{Privacy cohort threshold met?}
    D -->|No| E[Show broad region or Withheld for privacy]
    D -->|Yes| F[Show exact country and over-18 flag on pseudonymous profile]
    F --> G[Batch publication and monitor for singling-out risk]
    A --> H[EAS stores commitment only by default]
```

## 8. Threshold custody and exceptional access

Each dossier is encrypted with a unique data key. The decryption key is split into three shares; any authorised two can reconstruct it, while one share alone is useless. The plaintext dossier is not copied to all custodians.

Possible custodians, given only as examples, are:

- **Custodian A:** the New Tibet Foundation or its independently controlled security team;
- **Custodian B:** a separately governed Tibetan cultural, community, or administrative institution; and
- **Custodian C:** an independent professional security trustee, such as a regulated Swiss custody provider or specialist HSM operator.

Reconstruction requires a member-specific purpose, narrow scope, expiry, two independent Safe approvals, and enforcement by a policy-aware vault or HSM. After review, reconstructed material is destroyed and a signed audit receipt is added to the private log.

```mermaid
flowchart TD
    A[Encrypted verification dossier] --> B[Private encrypted storage]
    A --> C[Unique dossier decryption key]
    C --> D[Split key into 3 cryptographic shares]
    D --> E[Share A: Foundation example]
    D --> F[Share B: independent Tibetan institution example]
    D --> G[Share C: independent security trustee example]

    H[Scoped access request] --> I[2-of-3 Safe approval with purpose and expiry]
    I --> J[Policy vault validates approval]
    E --> K{At least 2 approved shares present?}
    F --> K
    G --> K
    J --> K
    K -->|No| L[Deny access]
    K -->|Yes| M[Temporarily reconstruct dossier key]
    B --> N[Decrypt one dossier in restricted session]
    M --> N
    N --> O[Authorised auditor or reviewer]
    O --> P[Destroy plaintext and reconstructed key]
    P --> Q[Signed audit receipt and log-root anchor]
```

## 9. Future CTA integration

A future integration could allow a New Tibet adapter to receive a signed status from the CTA's internal Green Book or Blue Book system. The response should be limited to an opaque reference and a status such as valid, revoked, or unknown. CTA records should not be copied into New Tibet.

A verified revoked status could cause the corresponding membership credential and EAS attestation to be restricted or revoked under the agreed review and appeal rules. The adapter must have no role, allowance, or upgrade authority in token, wallet, or treasury contracts, so it cannot freeze, seize, move, or burn funds.

## 10. Deletion, revocation, and re-entry

An authenticated member may request deletion without a custodian vote or Safe approval. The system closes active sessions, destroys dossier keys and backups, instructs custodians to delete shares, removes replicas and indexes, and revokes the membership credential, subject to documented lawful-retention exceptions.

Member-requested deletion and CTA-triggered membership revocation both restrict services that require active membership. Neither action:

- freezes, seizes, moves, burns, or locks funds;
- revokes the member's wallet keys; or
- calls token or treasury contracts.

A CTA-triggered revocation does not automatically erase the dossier; retention and deletion follow the applicable policy and the member's rights.

A deleted or revoked member may re-enter through a new invite from an eligible Green Book holder. A Blue Book holder cannot provide the invite. The returning person does not need a Green Book, Blue Book, passport, or ZKPassport proof. Re-entry creates a fresh member secret and commitment, records the profile as `Invite-only`, and gives the inviter the first vouch.

## 11. Implementation work

Before launch, the project needs:

- a formal data-flow and threat model for the Foundation Verification Portal;
- final evidence fields and retention periods for the verification dossier;
- final EAS schemas, resolver rules, and verifier-wallet governance;
- executable tests for wallet-only gating, admission, vouching, privacy thresholds, revocation, and non-interference with funds;
- custodian agreements and tested key-recovery and key-erasure procedures;
- appeal, correction, lost-wallet, and step-up review policies;
- a data-protection impact assessment and jurisdictional legal review; and
- independent application, smart-contract, infrastructure, and privacy security reviews.

The five-vouch cap, public-attribute cohort threshold, final trust labels, retention periods, and custodian appointments remain open design decisions.

## 12. References

- [CTA: Green Book Online Application Portal](https://tibet.net/official-announcement-department-of-finance-launches-green-book-online-application-portal/)
- [CTA: Greenbook Digital Portal](https://greenbook.tibet.net/)
- [CTA: Friends of Tibet and the Blue Book](https://tibet.net/support-tibet/friends-of-tibet/)
- [GitHub documentation: Mermaid diagrams in Markdown](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams)
- [Ethereum Attestation Service contracts](https://github.com/ethereum-attestation-service/eas-contracts)
- [ZKPassport](https://zkpassport.id/)
- [ZKPassport circuits](https://github.com/zkpassport/circuits)
