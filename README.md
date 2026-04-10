# Chat Channel Container Location in SHACL

## Overview

The [Solid Chat specification](https://solid.github.io/chat/) defines both the data model and the recommended storage locations for chat data. In the **longchat** variant, it is recommended that each day a new container be created, following a *yyyy/mm/dd* path structure.

This document describes how the storage location of a Solid chat channel can be represented in **SHACL**, using a **dynamic, template-driven model**. 

Key points:

* A ChatChannel can persist over multiple days and so can have **one or more container locations**.
* The location is defined using a **URI template** with variables.
* Variables are bound to **source properties** (e.g., dc:created) to generate final URIs.
* SHACL validates the **structure** of templates and bindings, ensuring metadata consistency.


* [hydra template example](./hydra-template/readme.md)
