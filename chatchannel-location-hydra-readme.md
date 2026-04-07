# Chat Channel Location Shape (Hydra Template Approach)

## Overview

The `chat_shape:ChatChannelLocationShape` defines how the storage location of chat data is represented using a URI template with variables. This approach uses the Hydra vocabulary to describe how container paths are constructed dynamically.

This design separates:

- Logical chat resource (the chat channel)
- URI template for storage
- Variable mappings used to construct the final container path

---

## Conceptual Model

```

ChatChannel (mee:LongChat)
└── prov:hasLocation → LocationNode
├── hydra:template → "/chat/{year}/{month}/{day}/"
└── hydra:mapping → VariableMapping*
├── hydra:variable ("year", "month", "day")
└── prov:wasDerivedFrom (source property)

```

---

## Core Semantics

### prov:hasLocation (Chat → Location Node)

- Indicates that the chat channel has a structured description of where its data is stored
- The location is defined as a template rather than a fixed URI

---

### hydra:template

- Defines a URI template string with placeholders
- Example:

```

/chat/{year}/{month}/{day}/

```

- Placeholders represent variables that will be substituted at runtime

---

### hydra:mapping

- Defines how variables in the template are populated
- Each mapping specifies:
  - The variable name (`hydra:variable`)
  - The source of its value (`prov:wasDerivedFrom`)

---

### Variable Mapping

| Property              | Meaning                                      |
|----------------------|----------------------------------------------|
| hydra:variable       | Name of the variable in the template         |
| prov:wasDerivedFrom  | RDF property providing the value             |

Example mapping:

- `year` derived from `dc:created`
- `month` derived from `dc:created`
- `day` derived from `dc:created`

---

## How This Meets the Requirements

### 1. Semantics of location predicate when path is not fixed

- The path is explicitly defined as a template using `hydra:template`
- Variables are named and mapped using `hydra:mapping`

Result:

- The location is no longer a fixed string
- It becomes a declarative, machine-readable construction rule

---

### 2. How SHACL gets things

- SHACL validates:
  - Presence of a template
  - Presence of variable mappings
  - Structure of each mapping node

- SHACL ensures:
  - Template exists (`hydra:template`)
  - At least one mapping exists (`hydra:mapping`)
  - Each mapping defines a variable and a source

Important:

- SHACL validates structure but does not execute template expansion

---

### 3. Declaring read/write transformations

The transformation is explicitly declared:

```

Template: /chat/{year}/{month}/{day}/
Mappings:
year  ← dc:created
month ← dc:created
day   ← dc:created

```

Write-time behavior:

1. Read source property (e.g. `dc:created`)
2. Extract required components (year, month, day)
3. Substitute variables into template
4. Produce final URI

Read-time behavior:

- Clients:
  - Read template
  - Read mappings
  - Reconstruct or interpret location dynamically

---

## Design Trade-offs

### Advantages

- Flexible and extensible
- Supports arbitrary path structures
- Explicit variable-to-source mapping
- Standard vocabulary (Hydra)

### Limitations

- More complex than fixed-path approach
- Requires template parsing and substitution logic
- Harder to validate full correctness with SHACL alone

---

## Comparison to Fixed yyyy/mm/dd Approach

| Approach                      | Flexibility | Complexity | Use Case                     |
|------------------------------|------------|------------|------------------------------|
| Fixed year/month/day         | Low        | Low        | Standard log-style storage   |
| Hydra template (this)        | High       | High       | Dynamic or variable paths    |

---

## Implementation Guidance

### Write (client behavior)

1. Read `hydra:template`
2. For each `hydra:mapping`:
   - Resolve `hydra:variable`
   - Extract value from `prov:wasDerivedFrom`
3. Substitute variables into template
4. Generate container URI
5. Store or use the resulting URI

---

### Read (client behavior)

1. Follow `prov:hasLocation`
2. Read:
   - Template (`hydra:template`)
   - Variable mappings (`hydra:mapping`)
3. Optionally:
   - Reconstruct URI
   - Validate expected structure

---

## Comparison with Simple yyyy/mm/dd Approach

| Aspect                     | Simple (Fixed Path)              | Hydra Template (This Approach)     |
|---------------------------|----------------------------------|------------------------------------|
| Path definition           | Implicit                         | Explicit in RDF                    |
| Flexibility               | Low                              | High                               |
| Client complexity         | Low                              | Higher                             |
| SHACL validation          | Easier                           | Structural only                    |
| Interoperability          | Convention-based                 | Self-describing                    |
| Runtime processing        | Minimal                          | Template parsing required          |
| Debuggability             | Easy                             | Moderate                           |
| Spec alignment            | Matches current chat spec bias   | Extends beyond spec                |


## Interoperability Considerations

- The current Solid Chat specification assumes a **conventional yyyy/mm/dd layout**
- This approach introduces a **declarative alternative**
- Clients that do not understand Hydra templates may:
  - Ignore the location description
  - Fall back to default assumptions

Recommendation:

- Consider **dual support**:
  - Provide template (Hydra)
  - Optionally provide resolved URI for compatibility

## When to Use This Approach

Use the Hydra template approach when:

- Storage paths are **not fixed** or may vary across deployments
- You need **multiple possible layouts** (e.g. by user, channel, or date)
- You want **clients to discover path logic dynamically**
- You are building **generic tooling or reusable infrastructure**

Do NOT use this approach when:

- You only need a fixed `/yyyy/mm/dd/` structure
- Simplicity and performance are more important than flexibility
- All clients already agree on a single storage convention



## Summary

This shape provides a flexible, declarative, and extensible model for representing chat storage locations where:

- Paths are defined as URI templates
- Variables are explicitly named and mapped
- Transformation logic is described in RDF

It enables:

- Dynamic path construction
- Interoperability using standard vocabularies
- Clear separation between structure and data

This approach is suitable when chat storage paths are not fixed and need to support varying structures.


This approach:

- Moves from **implicit convention → explicit semantics**
- Enables **dynamic, discoverable storage strategies**
- Trades **simplicity for flexibility**

In practice:

- Use the **simple approach by default**
- Use the **Hydra approach when you need abstraction or reuse**



## Open Design Questions

This model introduces several design decisions that should be agreed across implementations:

- Should variables be restricted (e.g. only `year/month/day`)?
- Should zero-padding be enforced (`04` vs `4`)?
- Should timezones be normalized (UTC vs local)?
- Should the resolved URI be required or optional?
- Should multiple templates be allowed?

These decisions affect interoperability and should be documented at the application or ecosystem level.

