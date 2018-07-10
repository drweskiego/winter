---
layout: post
title: Spring - Transaction Management
---
## Transaction Management

**Why use Transactions?** _To Enforce the **ACID** Principles_

- **A**tomic - Each unit of work is an all-or-nothing operation
- **C**onsistent - Database integrity constraints are never violated
- **I**solated - Isolating transactions from each other
- **D**urable - Committed changes are permanent

**Local Transactions** – Single Resource - Transactions managed by underlying resource
**Global** (distributed) **Transactions** – Multiple Resources - Transaction managed by separate, dedicated transaction manager