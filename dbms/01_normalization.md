# Dwelling Database Normalization

## 1. Introduction and Scope

This document presents a formal relational normalization analysis of the messaging application schema across its evolutionary stages—from First Normal Form (**1NF**) through Second Normal Form (**2NF**) to Third Normal Form (**3NF**)—along with an evaluation against Boyce–Codd Normal Form (**BCNF**).

The analysis covers the eight target relations present in the final normalized schema:
- `USERS`
- `USER_PROFILES`
- `CONTACTS`
- `BLOCKED_CONTACTS`
- `CONVERSATIONS`
- `CONVERSATION_MEMBERS`
- `MESSAGES`
- `MESSAGE_RECEIPTS`

To make the structural progression transparent, this walkthrough distinguishes between:
- **Actual schema stages (1NF → 2NF → 3NF):** The exact relations, attributes, primary keys (PK), candidate keys (CK), and foreign keys (FK) defined across the three schema revisions.
- **Theoretical normalization mechanics:** Explicit functional dependency ($X \rightarrow Y$) proofs, partial-dependency elimination, and transitive-dependency decomposition explaining *why* attributes migrate between tables at each stage, accompanied by concrete tabular illustrations.

---

## 2. Final Schema Relations, Keys, and Functional Dependencies

A functional dependency (FD) $X \rightarrow Y$ holds on a relation $R$ if, for every valid instance of $R$, any two tuples that agree on attribute set $X$ must also agree on attribute set $Y$. Below is the complete formal specification of the target **3NF** relations.

### 2.1 `USERS`

**Relation:** `USERS(user_id, username, email, password_hash, active_status, created_at, updated_at)`

- **Primary Key / Candidate Key:** $\{\text{user\_id}\}$
- **Declared Alternate Candidate Keys:** $\{\text{username}\}$ (`CK`), $\{\text{email}\}$ (`CK`)
- **Functional Dependencies:**
  - $\text{user\_id} \rightarrow \text{username}, \text{email}, \text{password\_hash}, \text{active\_status}, \text{created\_at}, \text{updated\_at}$
  - $\text{username} \rightarrow \text{user\_id}, \text{email}, \text{password\_hash}, \text{active\_status}, \text{created\_at}, \text{updated\_at}$
  - $\text{email} \rightarrow \text{user\_id}, \text{username}, \text{password\_hash}, \text{active\_status}, \text{created\_at}, \text{updated\_at}$
- **Foreign Keys:** None.

### 2.2 `USER_PROFILES`

**Relation:** `USER_PROFILES(user_id, display_name, about, avatar_url, created_at, updated_at)`

- **Primary Key / Candidate Key:** $\{\text{user\_id}\}$
- **Functional Dependencies:**
  - $\text{user\_id} \rightarrow \text{display\_name}, \text{about}, \text{avatar\_url}, \text{created\_at}, \text{updated\_at}$
- **Foreign Keys:**
  - $\text{user\_id} \rightarrow \text{USERS.user\_id}$ (enforces a strict 1:1 identifying relationship).

### 2.3 `CONTACTS`

**Relation:** `CONTACTS(user_id, contact_id, created_at)`

- **Composite Primary Key / Only Candidate Key:** $\{\text{user\_id}, \text{contact\_id}\}$
- **Functional Dependencies:**
  - $(\text{user\_id}, \text{contact\_id}) \rightarrow \text{created\_at}$
- **Foreign Keys:**
  - $\text{user\_id} \rightarrow \text{USERS.user\_id}$ (owner of the contact entry)
  - $\text{contact\_id} \rightarrow \text{USERS.user\_id}$ (target user added as a contact)

### 2.4 `BLOCKED_CONTACTS`

**Relation:** `BLOCKED_CONTACTS(user_id, blocked_contact_id, created_at)`

- **Composite Primary Key / Only Candidate Key:** $\{\text{user\_id}, \text{blocked\_contact\_id}\}$
- **Functional Dependencies:**
  - $(\text{user\_id}, \text{blocked\_contact\_id}) \rightarrow \text{created\_at}$
- **Foreign Keys:**
  - $\text{user\_id} \rightarrow \text{USERS.user\_id}$ (blocker)
  - $\text{blocked\_contact\_id} \rightarrow \text{USERS.user\_id}$ (blocked target user)

### 2.5 `CONVERSATIONS`

**Relation:** `CONVERSATIONS(conversation_id, title, type, created_by, created_at, updated_at)`

- **Primary Key / Candidate Key:** $\{\text{conversation\_id}\}$
- **Functional Dependencies:**
  - $\text{conversation\_id} \rightarrow \text{title}, \text{type}, \text{created\_by}, \text{created\_at}, \text{updated\_at}$
- **Foreign Keys:**
  - $\text{created\_by} \rightarrow \text{USERS.user\_id}$ (referential relationship; does not make $\text{created\_by}$ unique in `CONVERSATIONS`).
- **Domain Qualification:** For direct (1-on-1) conversations (`type = 'direct'`), `title` is nullable or system-derived, whereas group conversations (`type = 'group'`) carry an explicit `title`.

### 2.6 `CONVERSATION_MEMBERS`

**Relation:** `CONVERSATION_MEMBERS(conversation_id, user_id, role, joined_at, updated_at)`

- **Composite Primary Key / Only Candidate Key:** $\{\text{conversation\_id}, \text{user\_id}\}$
- **Functional Dependencies:**
  - $(\text{conversation\_id}, \text{user\_id}) \rightarrow \text{role}, \text{joined\_at}, \text{updated\_at}$
- **Foreign Keys:**
  - $\text{conversation\_id} \rightarrow \text{CONVERSATIONS.conversation\_id}$
  - $\text{user\_id} \rightarrow \text{USERS.user\_id}$

### 2.7 `MESSAGES`

**Relation:** `MESSAGES(message_id, conversation_id, sender_id, content, reply_to_message_id, created_at, edited_at, deleted_at)`

- **Primary Key / Candidate Key:** $\{\text{message\_id}\}$
- **Functional Dependencies:**
  - $\text{message\_id} \rightarrow \text{conversation\_id}, \text{sender\_id}, \text{content}, \text{reply\_to\_message\_id}, \text{created\_at}, \text{edited\_at}, \text{deleted\_at}$
- **Foreign Keys:**
  - $\text{conversation\_id} \rightarrow \text{CONVERSATIONS.conversation\_id}$
  - $\text{sender\_id} \rightarrow \text{USERS.user\_id}$
  - $\text{reply\_to\_message\_id} \rightarrow \text{MESSAGES.message\_id}$ (nullable self-referencing foreign key supporting threaded replies).

### 2.8 `MESSAGE_RECEIPTS`

**Relation:** `MESSAGE_RECEIPTS(message_id, recipient_id, status, delivered_at, read_at)`

- **Composite Primary Key / Only Candidate Key:** $\{\text{message\_id}, \text{recipient\_id}\}$
- **Functional Dependencies:**
  - $(\text{message\_id}, \text{recipient\_id}) \rightarrow \text{status}, \text{delivered\_at}, \text{read\_at}$
- **Foreign Keys:**
  - $\text{message\_id} \rightarrow \text{MESSAGES.message\_id}$
  - $\text{recipient\_id} \rightarrow \text{USERS.user\_id}$

---

## 3. First Normal Form (1NF)

A relation is in **1NF** if and only if every attribute domain contains only atomic (indivisible) values, there are no repeating groups or multi-valued cells within any tuple, and every relation has a well-defined primary key that uniquely identifies each row.

### 3.1 Actual 1NF Schema Analysis

In the **1NF** revision, the schema consists of three foundational relations:

1. `Users(user_id PK, display_name, about, avatar_url, password_hash, active_status, created_at)`
2. `Messages(message_id PK, sender FK, receiver FK, group_id, group_title, content, status, delivered_at, edited_at, deleted_at)`
3. `Contacts(user_id PK/FK, contact_id PK/FK, is_blocked)`

**How 1NF is achieved:**
- **Elimination of Multi-Valued Contact Lists:** Instead of storing a comma-separated list or array of contact IDs inside a `Users` row, each user-to-contact relationship is stored as an individual tuple in `Contacts`, uniquely identified by the composite primary key $(\text{user\_id}, \text{contact\_id})$.
- **Atomic Message Attributes:** Each message is stored as a distinct row in `Messages` identified by `message_id`, containing atomic scalar fields (`content`, `status`, `delivered_at`, `edited_at`, `deleted_at`).
- **Atomic User Profile & Account Fields:** Scalar user attributes (`display_name`, `about`, `avatar_url`, `password_hash`, `active_status`) occupy individual columns with single values per user.

### 3.2 Illustrative Pre-1NF Decomposition Example

Consider a hypothetical unnormalized (`UNF`) messaging table where a user's contacts and blocked flags were embedded directly inside the `Users` record as repeating groups:

`UnnormalizedUsers(user_id, display_name, contact_ids, blocked_flags)`

| user_id | display_name | contact_ids | blocked_flags |
|---:|---|---|---|
| 101 | Aarav | `102, 105, 109` | `false, true, false` |
| 102 | Meera | `101` | `false` |

Updating whether user `101` has blocked user `105` requires parsing positional arrays inside a single cell, violating atomicity. Decomposing this into `Users` and `Contacts` restores 1NF:

`Users`

| user_id (PK) | display_name |
|---:|---|
| 101 | Aarav |
| 102 | Meera |

`Contacts`

| user_id (PK, FK) | contact_id (PK, FK) | is_blocked |
|---:|---:|---|
| 101 | 102 | `false` |
| 101 | 105 | `true` |
| 101 | 109 | `false` |
| 102 | 101 | `false` |

### 3.3 Anomalies Remaining in the 1NF Schema

While the 1NF schema eliminates repeating groups, it suffers from severe update, insertion, and deletion anomalies that motivate 2NF and 3NF:
- **Redundant Group Metadata in `Messages`:** Every message sent to a group repeats `group_id` and `group_title`. Renaming a group requires updating every historical message row in that group (**update anomaly**), and a group cannot exist before its first message is sent (**insertion anomaly**).
- **Single-Recipient Constraint vs. Group Messaging:** `Messages` carries a single `receiver (FK)` alongside `status` and `delivered_at`, which cannot cleanly track independent delivery/read states across multiple group recipients without duplicating message payloads.
- **Mixed Domain Concerns in `Users`:** Authentication state (`password_hash`, `active_status`) and frequently updated profile metadata (`display_name`, `about`, `avatar_url`) are coupled in one table, while unique login identifiers (`username`, `email`) are not yet separated as candidate keys.

---

## 4. Second Normal Form (2NF)

A relation is in **2NF** if it is in 1NF and every non-prime attribute is **fully functionally dependent** on every candidate key—meaning no non-prime attribute depends on a proper subset of any composite candidate key.

### 4.1 Structural Evolution in the 2NF Schema

The **2NF** revision expands the schema to six relations:

1. `Users(user_id PK, username CK, email CK, password_hash, active_status, profile_id FK)`
2. `Profile(profile_id PK, display_name, about, avatar_url)`
3. `Contacts(user_id PK/FK, contact_id PK/FK, chat_id FK, updated_at)`
4. `Blocked_contacts(user_id PK/FK, blocked_contact_id PK/FK, updated_at)`
5. `Groups(group_id PK, user_id FK, joined_at, updated_at)`
6. `Messages(message_id PK, sender_id FK, chat_group_id FK, reply_to_message_id FK, content, status, delivered_at, edited_at, deleted_at, updated_at)`

### 4.2 Full Functional Dependency Verification Across Relations

- **Composite-Key Relations (`Contacts` and `Blocked_contacts`):**
  - In `Contacts`, the candidate key is $(\text{user\_id}, \text{contact\_id})$. Both non-prime attributes (`chat_id` and `updated_at`) depend on the entire pair:
    $$(\text{user\_id}, \text{contact\_id}) \rightarrow \text{chat\_id}, \text{updated\_at}$$
    Neither `user_id` alone (a user has many contacts and chats) nor `contact_id` alone determines `chat_id` or `updated_at`.
  - In `Blocked_contacts`, the candidate key is $(\text{user\_id}, \text{blocked\_contact\_id})$. The non-prime attribute `updated_at` depends on both attributes together:
    $$(\text{user\_id}, \text{blocked\_contact\_id}) \rightarrow \text{updated\_at}$$
    Extracting blocking state out of `Contacts` into `Blocked_contacts` also ensures that block timestamps exist independently of whether a user remains in the active contact list.
- **Single-Attribute Key Relations (`Users`, `Profile`, `Groups`, `Messages`):**
  - Because `user_id`, `profile_id`, `group_id`, and `message_id` are single-attribute primary keys (and `username`, `email` in `Users` are single-attribute candidate keys), no proper non-empty subset of a candidate key exists. Consequently, partial dependencies on candidate keys are structurally impossible in these relations.
  - Additionally, `Profile` is decoupled from `Users`, and `Groups` is introduced to separate group metadata (`group_id`, `user_id`, `joined_at`, `updated_at`) from individual `Messages` rows (which now reference `chat_group_id (FK)` rather than duplicating `group_title`).

### 4.3 Illustrative Partial-Dependency Decomposition

Suppose group membership and group metadata had been combined into a composite-key table during the transition from 1NF:

`GroupMembershipDraft(group_id, user_id, group_title, joined_at)`

Here, the candidate key is $(\text{group\_id}, \text{user\_id})$, and the functional dependencies are:
- $(\text{group\_id}, \text{user\_id}) \rightarrow \text{joined\_at}$
- $\text{group\_id} \rightarrow \text{group\_title}$

Because `group_title` depends solely on `group_id` (a proper subset of the composite primary key), `GroupMembershipDraft` violates **2NF**. Decomposing this into `CONVERSATIONS` (holding `conversation_id` and `title`) and `CONVERSATION_MEMBERS` (holding `conversation_id`, `user_id`, `role`, `joined_at`, `updated_at`) eliminates the partial dependency so every non-prime attribute depends on the full key.

### 4.4 Residual Dependencies Remaining in 2NF

Although the 2NF schema satisfies 2NF requirements, three structural issues remain for **3NF**:
1. **Surrogate Indirection in `Users` $\rightarrow$ `Profile`:** `Users` holds `profile_id (FK)` referencing `Profile(profile_id PK)`. Because a user has exactly one profile, introducing a separate surrogate `profile_id` creates an unnecessary join indirection and leaves timestamp tracking (`created_at`, `updated_at`) incomplete across `Users` and `Profile`.
2. **Transitive & Multi-Recipient Status Dependencies in `Messages`:** `Messages` still embeds `status` and `delivered_at` directly on the message row. In a group conversation (`chat_group_id`), a single message has *different* delivery and read statuses for each recipient. Storing `status` on `Messages` either restricts messages to a single status or creates a dependency on `(message_id, recipient_id)` rather than `message_id` alone.
3. **Mixed Conversation / Contact Coupling:** In 2NF, `Contacts` holds `chat_id (FK)` for 1-on-1 chats while `Groups` tracks group chats (`group_id`), splitting direct and group conversations across two different structural mechanisms and conflating a user's address book (`Contacts`) with conversation routing (`chat_id`).

---

## 5. Third Normal Form (3NF)

A relation $R$ is in **3NF** if it is in 2NF and, for every non-trivial functional dependency $X \rightarrow A$ that holds on $R$, at least one of the following conditions is satisfied:
1. $X$ is a **superkey** of $R$, or
2. $A$ is a **prime attribute** (an attribute that is part of at least one candidate key of $R$).

In practical schema design, 3NF eliminates **transitive dependencies** ($K \rightarrow X \rightarrow A$, where $K$ is a candidate key and $X$ is a non-key attribute) and ensures that every table models a single cohesive entity or relationship.

### 5.1 Entity & Relationship Decompositions in the 3NF Schema

The **3NF** schema resolves all remaining 2NF anomalies across eight clean relations:

#### A. Unified `CONVERSATIONS` and `CONVERSATION_MEMBERS`
- In 2NF, direct chats were referenced via `Contacts.chat_id` while group chats were tracked in `Groups(group_id, user_id, joined_at, updated_at)`—which conflated the group entity itself with its member list (creating a transitive/repeating issue if multiple members joined the same `group_id`).
- In 3NF, this is decomposed into:
  - `CONVERSATIONS(conversation_id PK, title, type, created_by FK, created_at, updated_at)`: Stores conversation-level attributes once per conversation, unified across both `'direct'` and `'group'` types via the `type` discriminator.
  - `CONVERSATION_MEMBERS(conversation_id PK/FK, user_id PK/FK, role, joined_at, updated_at)`: Represents the many-to-many participation between `USERS` and `CONVERSATIONS`, with member-specific attributes (`role`, `joined_at`, `updated_at`) fully dependent on $(\text{conversation\_id}, \text{user\_id})$.

#### B. Decoupling `MESSAGE_RECEIPTS` from `MESSAGES`
- In 1NF and 2NF, `Messages` stored `status` and `delivered_at` alongside `content`. Because delivery and read timestamps depend on *both* the message and the specific recipient, keeping them in `Messages` violated functional dependency integrity for multi-user conversations and mixed message content lifecycle (`edited_at`, `deleted_at`) with recipient state.
- In 3NF:
  - `MESSAGES(message_id PK, conversation_id FK, sender_id FK, content, reply_to_message_id FK, created_at, edited_at, deleted_at)` stores only immutable/author-controlled attributes determined strictly by `message_id`.
  - `MESSAGE_RECEIPTS(message_id PK/FK, recipient_id PK/FK, status, delivered_at, read_at)` isolates per-recipient tracking (`status`, `delivered_at`, `read_at`) keyed on $(\text{message\_id}, \text{recipient\_id})$.

#### C. Direct Primary-Key Alignment in `USER_PROFILES`
- Rather than maintaining an artificial surrogate `profile_id` in `Users` pointing to `Profile.profile_id`, `USER_PROFILES` uses `user_id (PK, FK)` directly referencing `USERS(user_id)`.
- `USERS` retains strictly account and authentication data (`user_id`, `username`, `email`, `password_hash`, `active_status`, `created_at`, `updated_at`), while `USER_PROFILES` stores public profile presentation (`display_name`, `about`, `avatar_url`, `created_at`, `updated_at`). Neither relation contains any transitive dependency.

#### D. Pure Relationship Tables (`CONTACTS` and `BLOCKED_CONTACTS`)
- Removing `chat_id` from `Contacts` leaves `CONTACTS(user_id PK/FK, contact_id PK/FK, created_at)` as a pure address-book association, matching `BLOCKED_CONTACTS(user_id PK/FK, blocked_contact_id PK/FK, created_at)`. Each table stores only the composite key and the audit timestamp `created_at`.

### 5.2 Illustrative 3NF Transitive-Dependency Decomposition

Consider what happens if `MESSAGES` had retained `conversation_title` or `sender_username` alongside `conversation_id` and `sender_id`:

`DenormalizedMessages(message_id, conversation_id, conversation_title, sender_id, sender_username, content)`

The functional dependencies would be:
- $\text{message\_id} \rightarrow \text{conversation\_id}, \text{conversation\_title}, \text{sender\_id}, \text{sender\_username}, \text{content}$
- $\text{conversation\_id} \rightarrow \text{conversation\_title}$
- $\text{sender\_id} \rightarrow \text{sender\_username}$

Here, $\text{message\_id} \rightarrow \text{conversation\_id} \rightarrow \text{conversation\_title}$ and $\text{message\_id} \rightarrow \text{sender\_id} \rightarrow \text{sender\_username}$ are classic **transitive dependencies** through non-key attributes (`conversation_id` and `sender_id`). Separating `CONVERSATIONS` and `USERS` so `MESSAGES` holds only the foreign keys `conversation_id` and `sender_id` eliminates both transitive chains.

### 5.3 Formal 3NF Verification Across All Eight Relations

| Relation | Candidate Key(s) | Non-Prime Attributes | Non-Trivial FDs | 3NF Compliance |
|---|---|---|---|---|
| `USERS` | `{user_id}`, `{username}`, `{email}` | `password_hash`, `active_status`, `created_at`, `updated_at` | All non-prime attributes depend solely on each candidate key | **Yes (3NF)** |
| `USER_PROFILES` | `{user_id}` | `display_name`, `about`, `avatar_url`, `created_at`, `updated_at` | $\text{user\_id} \rightarrow \text{all non-prime}$ | **Yes (3NF)** |
| `CONTACTS` | `{user_id, contact_id}` | `created_at` | $(\text{user\_id}, \text{contact\_id}) \rightarrow \text{created\_at}$ | **Yes (3NF)** |
| `BLOCKED_CONTACTS` | `{user_id, blocked_contact_id}` | `created_at` | $(\text{user\_id}, \text{blocked\_contact\_id}) \rightarrow \text{created\_at}$ | **Yes (3NF)** |
| `CONVERSATIONS` | `{conversation_id}` | `title`, `type`, `created_by`, `created_at`, `updated_at` | $\text{conversation\_id} \rightarrow \text{all non-prime}$ | **Yes (3NF)** |
| `CONVERSATION_MEMBERS` | `{conversation_id, user_id}` | `role`, `joined_at`, `updated_at` | $(\text{conversation\_id}, \text{user\_id}) \rightarrow \text{all non-prime}$ | **Yes (3NF)** |
| `MESSAGES` | `{message_id}` | `conversation_id`, `sender_id`, `content`, `reply_to_message_id`, `created_at`, `edited_at`, `deleted_at` | $\text{message\_id} \rightarrow \text{all non-prime}$ | **Yes (3NF)** |
| `MESSAGE_RECEIPTS` | `{message_id, recipient_id}` | `status`, `delivered_at`, `read_at` | $(\text{message\_id}, \text{recipient\_id}) \rightarrow \text{all non-prime}$ | **Yes (3NF)** |

---

## 6. Boyce–Codd Normal Form (BCNF) & Semantic Qualifications

A relation $R$ is in **BCNF** if, for every non-trivial functional dependency $X \rightarrow A$, $X$ is a **superkey** of $R$. BCNF is slightly stricter than 3NF because it removes the exception allowing $A$ to be a prime attribute when $X$ is not a superkey.

### 6.1 Key-Based BCNF Evaluation

Across all eight tables in the **3NF** schema:
- Every single-column key table (`USERS`, `USER_PROFILES`, `CONVERSATIONS`, `MESSAGES`) has only candidate keys as determinants of its non-trivial functional dependencies. In `USERS`, all three candidate keys (`user_id`, `username`, `email`) are superkeys, so dependencies among candidate keys (e.g., $\text{email} \rightarrow \text{username}$) still have a superkey on the left-hand side.
- Every composite-key table (`CONTACTS`, `BLOCKED_CONTACTS`, `CONVERSATION_MEMBERS`, `MESSAGE_RECEIPTS`) has only its full composite primary key as a determinant, with no overlapping multi-attribute candidate keys.

Therefore, under the declared key constraints, every relation in the final schema satisfies **both 3NF and BCNF**.

### 6.2 Semantic and Application-Level Qualifications

While the structural dependencies satisfy 3NF and BCNF, two cross-attribute semantic rules in a messaging domain warrant explicit documentation:

1. **Status-Timestamp Progression in `MESSAGE_RECEIPTS`:**
   - `MESSAGE_RECEIPTS` stores `status` alongside `delivered_at` and `read_at`. In application logic, `read_at IS NOT NULL` implies `status = 'read'` and `delivered_at IS NOT NULL` (a message cannot be read before it is delivered). While `read_at` does not uniquely determine a single value of `delivered_at` (so it is not a functional dependency $\text{read\_at} \rightarrow \text{delivered\_at}$), a subtle value-state coupling exists between `status` and the nullability of `delivered_at` / `read_at` (e.g., `status = 'sent'` implies both timestamps are `NULL`). Enforcing this via `CHECK` constraints ensures consistency without requiring further table decomposition.
2. **Nullable `title` in `CONVERSATIONS`:**
   - Because `CONVERSATIONS` unifies 1-on-1 (`type = 'direct'`) and multi-user (`type = 'group'`) chats, `title` is populated for group chats and typically `NULL` for direct chats (where the UI displays the other participant's `USER_PROFILES.display_name`). Strictly speaking, $\text{type} = \text{'direct'} \rightarrow \text{title IS NULL}$ is a conditional domain constraint rather than a general FD $\text{type} \rightarrow \text{title}$ (since group chats have distinct titles). Keeping both conversation types in one table avoids duplicating `CONVERSATION_MEMBERS` and `MESSAGES` foreign keys across separate `DIRECT_CHATS` and `GROUP_CHATS` hierarchies.

---

## 7. Summary of Normalization Progression

| Normal Form | Schema Tables | Key Structural Milestones | Resolved Anomalies |
|---|---|---|---|
| **1NF** | `Users`, `Messages`, `Contacts` | Enforced atomic scalar attributes; separated contact lists into rows using composite key `(user_id, contact_id)`. | Eliminated multi-valued cells and repeating groups within user and message records. |
| **2NF** | `Users`, `Profile`, `Contacts`, `Blocked_contacts`, `Groups`, `Messages` | Separated `Profile` from `Users`; extracted `Blocked_contacts`; introduced `Groups` and `reply_to_message_id`; verified full functional dependence on all composite keys. | Eliminated partial dependencies on composite keys and removed repeated `group_title` text from `Messages`. |
| **3NF / BCNF** | `USERS`, `USER_PROFILES`, `CONTACTS`, `BLOCKED_CONTACTS`, `CONVERSATIONS`, `CONVERSATION_MEMBERS`, `MESSAGES`, `MESSAGE_RECEIPTS` | Unified `CONVERSATIONS` & `CONVERSATION_MEMBERS`; extracted `MESSAGE_RECEIPTS` from `MESSAGES`; aligned `USER_PROFILES` directly on `user_id (PK, FK)`; removed `chat_id` from `CONTACTS`. | Eliminated all transitive and multi-recipient status dependencies; achieved full 3NF and BCNF compliance across all eight relations. |

---

## 8. Conclusion

The evolution of the messaging schema from **1NF** to **3NF** systematically removes data redundancy and structural anomalies:
- **1NF** establishes atomic columns and row-based contact storage.
- **2NF** decomposes user profile attributes, isolated block lists, and group entities while ensuring every non-prime attribute depends on the full primary key.
- **3NF** completes the normalization by separating per-recipient delivery receipts (`MESSAGE_RECEIPTS`) from message payloads (`MESSAGES`), unifying chat metadata and membership (`CONVERSATIONS` and `CONVERSATION_MEMBERS`), and aligning `USER_PROFILES` directly with `USERS(user_id)`.

As a result, every non-prime attribute in the final eight-table schema depends strictly on the whole candidate key and nothing but the candidate key, satisfying both **3NF** and **BCNF**.
