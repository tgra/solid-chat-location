# Chat Channel Location Simple Shape

## Overview

The `chat_shape:ChatChannelLocationShape` defines how the storage location of chat data is represented in a Solid Chat system when the storage path follows a date-based container structure (`YYYY/MM/DD`).

This design separates:

- Logical chat resource (the chat channel)
- Physical storage location (container URI)
- Deterministic structure (year, month, day components)

---

## Conceptual Model

```

ChatChannel (mee:LongChat)
└── prov:hasLocation → LocationNode
├── prov:hasLocation → Container URI
├── year  (xsd:gYear)
├── month (xsd:gMonth)
└── day   (xsd:gDay)

```

---

## Core Semantics

### prov:hasLocation (Chat → Location Node)

- Indicates that the chat channel has an associated storage location description
- This is not just a URI, but a structured object describing how the URI is formed

---

### prov:hasLocation (Location Node → URI)

- Points to the actual container URI in the pod
- Expected format:

```

/chat/YYYY/MM/DD/

```

---

### Date Components

| Property           | Type         | Meaning                         |
|-------------------|--------------|---------------------------------|
| chat_shape:year   | xsd:gYear    | Year component of the path      |
| chat_shape:month  | xsd:gMonth   | Month component (`--MM`)        |
| chat_shape:day    | xsd:gDay     | Day component (`---DD`)         |

These components:

- Represent the structure of the path
- Are typically derived from `dc:created`
- Allow machine validation and reasoning

---

## How This Meets the Requirements

### 1. Semantics of location predicate when path is not fixed

- `prov:hasLocation` points to a structured node instead of a raw URI
- The path structure is explicitly modelled using year/month/day

Result:

- The path is no longer opaque
- Its structure is machine-readable and semantically explicit

---

### 2. How SHACL gets things

- SHACL validates the structure using `sh:property` constraints:
  - Ensures a location node exists
  - Ensures year/month/day are present and correctly typed
  - Enforces cardinality (exactly one of each)

Important:

- SHACL validates data but does not compute values

---

### 3. Declaring read/write transformations

The model represents variables explicitly:

```

dc:created → (year, month, day) → /YYYY/MM/DD/

```

Write-time behavior:

1. Extract date from `dc:created`
2. Populate `year`, `month`, `day`
3. Construct container URI

Read-time behavior:

- Clients can reconstruct or validate the path using structured components

Note:

- Transformation logic is handled by the application or rules, not SHACL itself

---

## Design Trade-offs

### Advantages

- Simple and easy to implement
- Explicit representation of path structure
- Fully compatible with SHACL validation
- Predictable and aligned with log-style chat storage

### Limitations

- Only supports fixed `YYYY/MM/DD` structure
- Not suitable for arbitrary or dynamic path templates
- Transformation logic is external to SHACL

---

## Comparison to Template-Based Approaches

| Approach                      | Flexibility | Complexity | Use Case                     |
|------------------------------|------------|------------|------------------------------|
| year/month/day (this shape)  | Low        | Low        | Fixed date-based paths       |
| hydra:template               | High       | High       | Arbitrary URI templates      |

This shape is intentionally opinionated and simpler, aligning with the Solid Chat specification.

---

## Implementation Guidance

### Write (client behavior)

1. Read `dc:created`
2. Extract:
   - Year → `chat_shape:year`
   - Month → `chat_shape:month`
   - Day → `chat_shape:day`
3. Construct path:
```

/chat/YYYY/MM/DD/

```
4. Write:
- Location node
- Container URI

---

### Read (client behavior)

1. Follow `prov:hasLocation`
2. Read:
- Container URI
- Date components
3. Optionally:
- Validate consistency
- Use for navigation or indexing

---

## Summary

This shape provides a minimal, structured, and SHACL-validatable model for representing chat storage locations where:

- Paths follow a date-based hierarchy
- Variables (year/month/day) are explicitly represented
- The system remains simple and deterministic

It balances:

- Avoiding overly complex template systems
- Avoiding opaque, non-semantic URIs

Result:

- A practical, opinionated model suitable for Solid Chat implementations

