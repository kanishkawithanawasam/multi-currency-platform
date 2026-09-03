# Domain Terminology

## Purpose

This document defines the terminology used throughout the project.

Financial systems frequently use similar words to describe different concepts. In particular, terms such as **account**, **balance**, **transaction**, and **entry** can have different meanings depending on whether they refer to the customer-facing product or the internal accounting ledger.

These definitions establish a consistent vocabulary for architecture, implementation, testing, and documentation.

---

# Customer

A **Customer** is a person who uses the platform and owns a customer-facing financial account.

A customer may hold value in multiple currencies through that account.

The exact customer identity and persistence model has not yet been designed.

---

# Personal Account

A **Personal Account** is the customer-facing financial account provided by the platform.

A personal account can hold value in multiple currencies.

Conceptually:

```text
Customer
   │
   └── Personal Account
          ├── GBP
          ├── EUR
          └── USD
```

The personal account represents the financial product visible to the customer.

It is distinct from an internal **Ledger Account**.

The exact persistence representation of the personal account has not yet been decided.

---

# Currency

A **Currency** identifies the monetary denomination of an amount.

Examples include:

```text
GBP
EUR
USD
```

A monetary amount must always be interpreted together with its currency.

For example:

```text
100 GBP
```

and:

```text
100 EUR
```

represent different financial values and must not be treated as equivalent.

The initial system is intended to support multiple currencies.

---

# Currency Position

A **Currency Position** represents the value a customer holds in a particular currency within their personal account.

For example:

```text
Personal Account

GBP → £500
EUR → €120
USD → $80
```

Each currency position is logically independent.

The term **Currency Position** is currently a domain concept only.

Whether it becomes a dedicated Java entity or database table will be decided during domain and persistence modelling.

---

# Monetary Amount

A **Monetary Amount** consists of both:

* a numerical amount; and
* a currency.

For example:

```text
Amount:   100.00
Currency: GBP
```

A numerical value without a currency is not sufficient to represent money within the financial domain.

The exact Java representation of monetary amounts has not yet been selected.

---

# Customer Balance

A **Customer Balance** is the amount of value available or attributed to a customer for a particular currency.

For example:

```text
Customer: Alice
Currency: GBP
Balance:  £250
```

From the customer's perspective, this represents an asset.

From the platform's accounting perspective, the corresponding amount represents a liability owed to the customer.

The implementation strategy for calculating or storing customer-visible balances has not yet been decided.

---

# Ledger

The **Ledger** is the authoritative accounting record of financial movements within the platform.

It records financial events using double-entry accounting.

A ledger transaction must satisfy:

```text
SUM(DEBITS) = SUM(CREDITS)
```

The ledger is maintained from the accounting perspective of the platform.

---

# Ledger Account

A **Ledger Account** is an internal accounting account against which debits and credits are recorded.

Ledger accounts may represent accounting categories such as:

* platform assets;
* customer liabilities;
* revenue;
* expenses; or
* equity.

A ledger account must not be confused with the customer-facing **Personal Account**.

For example:

```text
Personal Account
```

is a product concept, whereas:

```text
Alice GBP Liability Ledger Account
```

would represent an internal accounting concept.

The exact ledger-account structure has not yet been designed.

---

# Account Type

An **Account Type** identifies the accounting classification of a ledger account.

The standard account types used by the system are:

* Asset;
* Liability;
* Revenue;
* Expense; and
* Equity.

Debit and credit behaviour depends on the account type.

| Account Type | Increase | Decrease |
| ------------ | -------- | -------- |
| Asset        | Debit    | Credit   |
| Expense      | Debit    | Credit   |
| Liability    | Credit   | Debit    |
| Equity       | Credit   | Debit    |
| Revenue      | Credit   | Debit    |

---

# Asset

An **Asset** represents an economic resource controlled by the platform.

Examples may eventually include:

```text
Platform Bank Account
Settlement Account
```

Assets increase with a **debit** and decrease with a **credit**.

---

# Liability

A **Liability** represents an obligation owed by the platform to another party.

Customer-held funds are liabilities of the platform because the platform owes that value to the customer.

Liabilities increase with a **credit** and decrease with a **debit**.

For example:

```text
Customer deposits £100

Platform liability to customer increases by £100

→ CREDIT customer liability
```

---

# Revenue

**Revenue** represents income recognised by the platform.

Revenue increases with a **credit** and decreases with a **debit**.

Revenue accounting has not yet been incorporated into the current product design.

---

# Expense

An **Expense** represents a cost recognised by the platform.

Expenses increase with a **debit** and decrease with a **credit**.

Expense accounting has not yet been incorporated into the current product design.

---

# Equity

**Equity** represents the residual accounting interest in the platform after liabilities are deducted from assets.

Equity increases with a **credit** and decreases with a **debit**.

Equity accounting has not yet been incorporated into the current product design.

---

# Debit

A **Debit** is one side of a double-entry accounting posting.

A debit does not inherently mean that money is added or removed.

Its effect depends on the type of ledger account.

A debit:

```text
increases  Assets
increases  Expenses

decreases  Liabilities
decreases  Equity
decreases  Revenue
```

---

# Credit

A **Credit** is the opposite side of a double-entry accounting posting.

Like a debit, a credit does not inherently mean an increase or decrease in money.

Its effect depends on the ledger account type.

A credit:

```text
decreases  Assets
decreases  Expenses

increases  Liabilities
increases  Equity
increases  Revenue
```

---

# Ledger Transaction

A **Ledger Transaction** represents one atomic accounting event.

It contains multiple ledger postings whose debit and credit totals must balance.

For example, if Alice transfers £20 to Bob:

```text
Ledger Transaction

Dr Alice Liability    £20
Cr Bob Liability      £20
```

The collection of postings represents a single accounting event.

The database representation of a ledger transaction has not yet been designed.

---

# Posting

A **Posting** is an individual debit or credit recorded against a ledger account as part of a ledger transaction.

For example:

```text
Dr Alice Liability £20
```

is one posting.

```text
Cr Bob Liability £20
```

is another posting.

Together, these postings may form one balanced ledger transaction.

The term **posting** is preferred over using the generic term **entry** when referring to an individual debit or credit within a ledger transaction.

---

# Transfer

A **Transfer** is a business operation that moves value from one customer position to another.

A transfer and a ledger transaction are related but are not the same concept.

For example:

```text
Business operation:

Alice transfers £20 to Bob
```

may result in the accounting transaction:

```text
Dr Alice Liability    £20
Cr Bob Liability      £20
```

The **Transfer** describes the business intent.

The **Ledger Transaction** describes its accounting effect.

The detailed transfer lifecycle has not yet been designed.

---

# Deposit

A **Deposit** represents value entering the platform for the benefit of a customer.

Conceptually, if £100 enters the platform:

```text
Dr Platform Asset        £100
Cr Customer Liability    £100
```

The external payment or banking mechanism responsible for the deposit has not yet been designed.

---

# Withdrawal

A **Withdrawal** represents value leaving the platform on behalf of a customer.

Conceptually, it reverses the economic effects of a deposit by reducing both:

* the platform asset involved; and
* the liability owed to the customer.

The external withdrawal mechanism has not yet been designed.

---

# Platform Accounting Perspective

The **Platform Accounting Perspective** means that all ledger debits, credits, assets, and liabilities are interpreted from the point of view of the company operating the platform.

This distinction is particularly important for customer balances.

For example:

```text
Customer perspective:
£100 balance = Asset

Platform perspective:
£100 owed to customer = Liability
```

The ledger consistently uses the platform perspective.

---

# Double-Entry Accounting

**Double-Entry Accounting** is the accounting model used by the platform.

Every financial transaction produces corresponding debit and credit postings.

For every valid ledger transaction:

```text
SUM(DEBITS) = SUM(CREDITS)
```

This allows financial movements to remain balanced and auditable.

---

# Terms Not Yet Finalised

The following terminology may be refined during the next domain-design stage:

* whether **Personal Account** remains the final customer-facing name;
* whether **Currency Position** becomes a first-class domain entity;
* how customer accounts map to ledger accounts;
* how balances are represented or derived;
* the exact structure of ledger transactions and postings; and
* terminology for future foreign-exchange operations.

Any terminology changes should be reflected in this document before being adopted throughout the implementation.