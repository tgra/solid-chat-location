# Chat Channel Container Location in SHACL

## Overview

The [Solid Chat specification](https://solid.github.io/chat/) defines both the data model and the recommended storage locations for chat data. In the **longchat** variant, it is recommended that each day a new container be created, following a *yyyy/mm/dd* path structure.

This document describes how the storage location of a Solid chat channel can be represented in **SHACL**, using a **dynamic, template-driven model**. 

Key points:

* A ChatChannel can persist over multiple days and so can have **one or more container locations**.
* Each location is defined using a **URI template** with variables.
* Variables are bound to **source properties** (e.g., dc:created) to generate final URIs.
* SHACL validates the **structure** of templates and bindings, ensuring metadata consistency.



## Target Class

* **Target:** mee:LongChat
* **NodeShape:** chat_shape:ChatChannelShape
* **Property shape applied:** prov:hasLocation → chat_shape:ChatChannelLocationShape




## Shape Definition

### ChatChannelLocationShape

| Property                | Description                                                  | Constraints                                                                |
| ----------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------- |
| chat:locationTemplate | URI template string for chat storage paths                   | Must exist (sh:minCount 1), datatype xsd:string                        |
| chat:locationVariable | List of variables in the template (e.g., yyyy, mm, dd) | Must include at least 3 values, max 3 for standard year/month/day template |
| chat:locationBinding  | Binding for each template variable to a source property      | Blank node, one per variable, minCount = number of variables               |
| chat:resolvedURI      |The URI where this channel’s data is stored, fully expanded.||
---

### Location Binding Structure

Each chat:locationBinding node contains:

| Property              | Description                                                            |
| --------------------- | ---------------------------------------------------------------------- |
| chat:variable       | Name of the variable in the template                                   |
| prov:wasDerivedFrom | RDF property providing the value for the variable (e.g., dc:created) |

This allows clients or tools to dynamically construct URIs from template variables.



## Example Data
```

:chatChannel1 a mee:LongChat ;
    prov:hasLocation [
        chat:locationTemplate "/chat/{yyyy}/{mm}/{dd}/" ;
        chat:locationVariable "yyyy", "mm", "dd" ;
        chat:locationBinding [
            chat:variable "yyyy" ;
            prov:wasDerivedFrom dc:created ;
        ] ;
        chat:locationBinding [
            chat:variable "mm" ;
            prov:wasDerivedFrom dc:created ;
        ] ;
        chat:locationBinding [
            chat:variable "dd" ;
            prov:wasDerivedFrom dc:created ;
        ];
        chat:resolvedURI "/chat/2026/04/08/" ;
    ] .
```

* **Template:** /chat/{yyyy}/{mm}/{dd}/
* **Variables:** yyyy, mm, dd
* **Bindings:** Each variable maps to the chat channel's creation date (dc:created).



## SHACL Validation Rules

1. Each ChatChannel must have at least one prov:hasLocation.
2. Each location must include a **template string** (chat:locationTemplate).
3. Template variables (chat:locationVariable) must match **binding nodes** (chat:locationBinding).
4. Each binding must specify both chat:variable and prov:wasDerivedFrom.
5. Multiple location templates can be used for different storage patterns.



## Client Behavior (Read/Write)

**Write-time behavior:**

1. Read source properties (e.g., dc:created) from the chat channel.
2. Substitute variable values into the template (chat:locationTemplate).
3. Generate the final URI for storage or retrieval.

**Read-time behavior:**

1. Read template and variable mappings.
2. Dynamically reconstruct the URI if needed.
3. Validate structure against expected variables and template.

SHACL validates that the template and bindings are correct, but generating the final URI requires client-side substitution of variables. Runtime checks are needed to ensure the URI is valid and accessible.

Clients must understand template semantics




## Source SHACL Files

```
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix chat: <http://example.org/ns/chat#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix mee: <http://www.w3.org/ns/pim/meeting#> .
@prefix chat_shape: <https://solidproject.org/shapes/chat#> .

chat_shape:ChatChannelShape
    a sh:NodeShape ;
    sh:targetClass mee:LongChat ;
    sh:name "Chat Channel Shape" ;
    sh:description "Shape for chat channels with dynamic location templates." ;
    prov:hasLocation chat_shape:ChatChannelLocationShape .

chat_shape:ChatChannelLocationShape
    a sh:PropertyShape ;
    sh:path prov:hasLocation ;
    sh:minCount 1 ;
    sh:codeIdentifier "chatChannelLocation" ;
    sh:name "Chat Channel Location Shape" ;
    sh:description "Defines a URI template and variable mappings for chat storage." ;
    sh:nodeKind sh:BlankNodeOrIRI ;

    sh:property [
        sh:path chat:locationTemplate ;
        sh:datatype xsd:string ;
        sh:minCount 1 ;
    ] ;
    sh:property [
        sh:path chat:locationVariable ;
        sh:minCount 3 ;
        sh:maxCount 3 ;
    ] ;
    sh:property [
        sh:path chat:locationBinding ;
        sh:nodeKind sh:BlankNodeOrIRI ;
        sh:minCount 3 ;
    ] ;

chat:locationTemplate "/{yyyy}/{mm}/{dd}/" .
```
