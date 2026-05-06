---
title: "Regex Fundamentals for API Designers: Pattern Validation in RAML"
slug: "regex-fundamentals-pattern-validation-raml"
description: "Master regular expression basics and apply them to validate API parameters in RAML specs — from simple patterns to real-world field constraints."
author: "Gonzalo Marcos"
date: 2026-05-05
status: draft
lang: en
category: api-management
tags:
  - raml
  - api-designer
  - anypoint-platform
  - best-practices
  - reference
  - english
type: tutorial
difficulty: beginner
read_time: 14
platform:
  - anypoint-platform
canonical_url: ""
---

![Draft](https://img.shields.io/badge/Status-Draft-7f8c8d) ![English](https://img.shields.io/badge/Lang-English-4a4a4a) ![Anypoint Platform](https://img.shields.io/badge/Platform-Anypoint_Platform-00A0DF?logo=mulesoft&logoColor=white) ![Beginner](https://img.shields.io/badge/Level-Beginner-2ecc71) ![Tutorial](https://img.shields.io/badge/Type-Tutorial-8e44ad) ![14 min](https://img.shields.io/badge/Read_Time-14_min-lightgrey)

# Regex Fundamentals for API Designers: Pattern Validation in RAML

You are designing an API and you need a query parameter that only accepts dates in `YYYY-MM-DD` format. Or a URI parameter that must be a UUID. Or a header that follows a specific token pattern. How do you enforce these constraints directly in the API specification?

The answer is **regular expressions** — compact patterns that describe the shape of valid input. RAML supports regex natively through the `pattern` property, letting you push validation logic into the contract itself rather than burying it in implementation code.

This tutorial teaches you regex from the ground up and shows you how to apply every concept directly in RAML parameter definitions. By the end, you will be able to read, write, and troubleshoot patterns for any field in your API specs.

---

## Prerequisites

- Basic familiarity with YAML syntax
- A text editor or access to Anypoint API Designer
- No prior regex experience required

## Overview

We will cover:

1. **What regular expressions are** and why they matter for API design
2. **Core building blocks**: literals, metacharacters, character classes
3. **Quantifiers**: controlling repetition
4. **Anchors and groups**: precision and structure
5. **RAML integration**: using `pattern` in types, query parameters, URI parameters, and headers
6. **Real-world patterns**: dates, UUIDs, emails, semantic versions, and more

## Table of Contents

- [Step 1 — Understanding Regex Basics](#step-1--understanding-regex-basics)
- [Step 2 — Character Classes](#step-2--character-classes)
- [Step 3 — Quantifiers](#step-3--quantifiers)
- [Step 4 — Anchors and Boundaries](#step-4--anchors-and-boundaries)
- [Step 5 — Groups and Alternation](#step-5--groups-and-alternation)
- [Step 6 — Using Patterns in RAML Types](#step-6--using-patterns-in-raml-types)
- [Step 7 — Validating URI Parameters](#step-7--validating-uri-parameters)
- [Step 8 — Validating Query Parameters and Headers](#step-8--validating-query-parameters-and-headers)
- [Step 9 — Real-World Pattern Library](#step-9--real-world-pattern-library)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)

---

### Step 1 — Understanding Regex Basics

A regular expression (regex) is a sequence of characters that defines a search pattern. Think of it as a template that describes what valid text looks like.

**Literal characters** match themselves exactly:

| Pattern | Matches | Does NOT match |
|---------|---------|----------------|
| `hello` | "hello" | "Hello", "HELLO" |
| `123` | "123" | "1234", "12" |
| `api` | "api" | "API", "Api" |

**Metacharacters** have special meaning:

| Symbol | Meaning | Example | Matches |
|--------|---------|---------|---------|
| `.` | Any single character | `a.c` | "abc", "a1c", "a-c" |
| `\d` | Any digit (0–9) | `\d\d` | "42", "07", "99" |
| `\w` | Word character (letter, digit, underscore) | `\w\w\w` | "abc", "a_1", "X9z" |
| `\s` | Whitespace (space, tab, newline) | `a\sb` | "a b", "a\tb" |
| `\D` | Any NON-digit | `\D\D` | "ab", "--", "!?" |
| `\W` | Any NON-word character | `\W` | " ", "-", "!" |

> [!NOTE]
> The uppercase versions (`\D`, `\W`, `\S`) are the negated counterparts of their lowercase versions. This is a universal regex convention.

Here is how a simple literal pattern looks in a RAML type definition:

```yaml
types:
  CountryCode:
    type: string
    pattern: "^[A-Z][A-Z]$"
    description: ISO 3166-1 alpha-2 country code (e.g., US, GB, IT)
```

This enforces that the field contains exactly two uppercase letters — nothing more, nothing less.

---

### Step 2 — Character Classes

Character classes let you define a **set of allowed characters** at a specific position.

**Syntax:** Square brackets `[ ]` enclose the allowed characters.

| Pattern | Meaning | Matches |
|---------|---------|---------|
| `[abc]` | Exactly one of: a, b, or c | "a", "b", "c" |
| `[0-9]` | Any digit (range) | "0", "5", "9" |
| `[a-z]` | Any lowercase letter | "a", "m", "z" |
| `[A-Z]` | Any uppercase letter | "A", "M", "Z" |
| `[a-zA-Z]` | Any letter (either case) | "a", "Z", "m" |
| `[0-9a-fA-F]` | Any hexadecimal digit | "0", "a", "F", "9" |
| `[^0-9]` | Any character EXCEPT digits | "a", "-", " " |

> [!TIP]
> The caret `^` inside brackets **negates** the class. Outside brackets it means "start of string" (we cover this in Step 4). Context matters!

**Combining ranges:**

```
[a-zA-Z0-9_]   →  same as \w (word characters)
[0-9]          →  same as \d (digits)
[^a-zA-Z]     →  anything that is NOT a letter
```

**RAML example — Hexadecimal color code:**

```yaml
types:
  HexColor:
    type: string
    pattern: "^#[0-9a-fA-F]{6}$"
    description: CSS hex color (e.g., #FF5733, #00a0df)
    examples:
      valid: "#00A0DF"
```

This pattern says: start with `#`, followed by exactly 6 hexadecimal characters.

---

### Step 3 — Quantifiers

Quantifiers control **how many times** the preceding element can repeat.

| Quantifier | Meaning | Example | Matches |
|------------|---------|---------|---------|
| `*` | Zero or more | `a*` | "", "a", "aaa" |
| `+` | One or more | `a+` | "a", "aaa" (NOT "") |
| `?` | Zero or one (optional) | `colou?r` | "color", "colour" |
| `{n}` | Exactly n times | `\d{4}` | "2026", "1234" |
| `{n,}` | At least n times | `\d{2,}` | "12", "123", "9999" |
| `{n,m}` | Between n and m times | `\d{1,3}` | "1", "42", "255" |

> [!WARNING]
> A common mistake is confusing `*` (zero or more) with `+` (one or more). Using `*` when you mean `+` allows empty matches, which might make a required field pass validation with an empty string.

**RAML example — Phone number with optional country code:**

```yaml
types:
  PhoneNumber:
    type: string
    pattern: "^\\+?[1-9]\\d{6,14}$"
    description: |
      International phone number (E.164 format).
      Optional + prefix, 7 to 15 digits total, cannot start with 0.
    examples:
      with_plus: "+14155552671"
      without_plus: "14155552671"
```

Breaking it down:
- `\\+?` — optional literal `+` sign (escaped because `+` is a quantifier)
- `[1-9]` — first digit must be 1–9
- `\\d{6,14}` — followed by 6 to 14 more digits

> [!NOTE]
> In RAML/YAML strings, backslashes must be **double-escaped** (`\\d` instead of `\d`) because YAML itself interprets backslash sequences. This is the most common source of pattern errors in RAML specs.

---

### Step 4 — Anchors and Boundaries

Anchors do not match characters — they match **positions** in the string.

| Anchor | Meaning | Example | Matches | Does NOT match |
|--------|---------|---------|---------|----------------|
| `^` | Start of string | `^hello` | "hello world" | "say hello" |
| `$` | End of string | `world$` | "hello world" | "world cup" |
| `^...$` | Entire string must match | `^\\d{3}$` | "123" | "1234", "12" |

**Why anchors matter in RAML:**

Without anchors, a pattern like `\d{3}` would match "123" inside "abc123xyz" — it only checks that the pattern exists *somewhere* in the string. In RAML, you almost always want to validate the **entire** value:

```yaml
# BAD - matches "abc123xyz" because "123" exists within it
types:
  BadExample:
    type: string
    pattern: "\\d{3}"

# GOOD - requires the entire string to be exactly 3 digits
types:
  GoodExample:
    type: string
    pattern: "^\\d{3}$"
```

> [!IMPORTANT]
> Always use `^` and `$` anchors in RAML patterns unless you have a specific reason not to. Without them, partial matches can slip through validation.

---

### Step 5 — Groups and Alternation

**Parentheses `( )`** group parts of a pattern together, allowing you to apply quantifiers to sequences or define alternatives.

**Alternation `|`** means "OR" — match the left side or the right side.

| Pattern | Meaning | Matches |
|---------|---------|---------|
| `(ab)+` | One or more repetitions of "ab" | "ab", "abab", "ababab" |
| `cat\|dog` | "cat" or "dog" | "cat", "dog" |
| `(http\|https)://` | "http://" or "https://" | "http://", "https://" |
| `(dev\|qa\|prod)` | One of three environments | "dev", "qa", "prod" |

**RAML example — Environment-specific base URI parameter:**

```yaml
baseUri: https://{environment}.api.example.com/v1

baseUriParameters:
  environment:
    type: string
    pattern: "^(dev|staging|prod)$"
    description: Deployment environment
    examples:
      development: "dev"
      production: "prod"
```

**RAML example — API version header:**

```yaml
types:
  ApiVersion:
    type: string
    pattern: "^v[1-9]\\d*(\\.\\d+){0,2}$"
    description: |
      API version string (e.g., v1, v2.1, v3.0.1).
      Must start with 'v' followed by a non-zero major version.
    examples:
      major_only: "v1"
      major_minor: "v2.1"
      full: "v3.0.1"
```

Breaking it down:
- `^v` — starts with literal "v"
- `[1-9]\\d*` — major version: first digit 1–9, followed by optional digits
- `(\\.\\d+){0,2}` — zero to two groups of `.` followed by digits (minor and patch)
- `$` — end of string

---

### Step 6 — Using Patterns in RAML Types

In RAML, the `pattern` property applies to any `string` type. You can use it in reusable type definitions that are then referenced across your entire API specification.

```yaml
#%RAML 1.0

title: Order Management API
version: v1

types:
  # Reusable validated types
  UUID:
    type: string
    pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$"
    description: RFC 4122 UUID (lowercase, hyphenated)

  ISODate:
    type: string
    pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])$"
    description: ISO 8601 date (YYYY-MM-DD)

  Email:
    type: string
    pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    description: Email address (simplified validation)

  CurrencyCode:
    type: string
    pattern: "^[A-Z]{3}$"
    description: ISO 4217 currency code (e.g., USD, EUR, GBP)

  SemanticVersion:
    type: string
    pattern: "^(0|[1-9]\\d*)\\.(0|[1-9]\\d*)\\.(0|[1-9]\\d*)(-[a-zA-Z0-9]+(\\.[a-zA-Z0-9]+)*)?(\\+[a-zA-Z0-9]+(\\.[a-zA-Z0-9]+)*)?$"
    description: Semantic Versioning 2.0.0 (e.g., 1.0.0, 2.1.3-beta.1)

  # Type that uses validated types
  Order:
    type: object
    properties:
      orderId: UUID
      customerEmail: Email
      createdAt: ISODate
      currency: CurrencyCode
      amount:
        type: number
        minimum: 0.01
```

> [!TIP]
> Define validated types in a **RAML Library** (`.raml` file with `#%RAML 1.0 Library` header) and import them across multiple APIs. This gives you a single source of truth for patterns and avoids copy-paste drift.

---

### Step 7 — Validating URI Parameters

URI parameters are embedded in resource paths. Regex patterns ensure that only well-formed values reach your implementation.

```yaml
#%RAML 1.0

title: Customer API
version: v1
baseUri: https://api.example.com/{version}

baseUriParameters:
  version:
    type: string
    pattern: "^v[1-9]$"
    description: API major version (v1, v2, etc.)
    example: "v1"

types:
  CustomerId:
    type: string
    pattern: "^CUS-[A-Z0-9]{8}$"
    description: Customer identifier (e.g., CUS-A1B2C3D4)

  OrderId:
    type: string
    pattern: "^ORD-\\d{4}-[A-Z0-9]{6}$"
    description: Order identifier with year prefix (e.g., ORD-2026-X9F3A1)

/customers/{customerId}:
  uriParameters:
    customerId:
      type: CustomerId
      description: Unique customer identifier
      example: "CUS-A1B2C3D4"

  get:
    description: Retrieve a customer by ID

  /orders/{orderId}:
    uriParameters:
      orderId:
        type: OrderId
        description: Unique order identifier
        example: "ORD-2026-X9F3A1"

    get:
      description: Retrieve a specific order for this customer
```

Let's trace through the `OrderId` pattern `^ORD-\\d{4}-[A-Z0-9]{6}$`:

| Part | Meaning |
|------|---------|
| `^` | Start of string |
| `ORD-` | Literal prefix "ORD-" |
| `\\d{4}` | Exactly 4 digits (year) |
| `-` | Literal hyphen separator |
| `[A-Z0-9]{6}` | Exactly 6 alphanumeric uppercase characters |
| `$` | End of string |

Valid: `ORD-2026-X9F3A1` — Invalid: `ORD-26-x9f3a1`, `ord-2026-ABCDEF`

---

### Step 8 — Validating Query Parameters and Headers

Query parameters and headers benefit enormously from pattern validation. They are the most common source of malformed input.

**Query parameters:**

```yaml
/orders:
  get:
    description: Search orders with filters
    queryParameters:
      status:
        type: string
        required: false
        pattern: "^(pending|confirmed|shipped|delivered|cancelled)$"
        description: Filter by order status
        example: "shipped"
      dateFrom:
        type: string
        required: false
        pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])$"
        description: Filter orders from this date (YYYY-MM-DD)
        example: "2026-01-15"
      dateTo:
        type: string
        required: false
        pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])$"
        description: Filter orders until this date (YYYY-MM-DD)
        example: "2026-05-05"
      sort:
        type: string
        required: false
        pattern: "^(createdAt|updatedAt|amount):(asc|desc)$"
        description: "Sort field and direction (e.g., createdAt:desc)"
        example: "createdAt:desc"
      page:
        type: string
        required: false
        pattern: "^[1-9]\\d*$"
        description: Page number (positive integer, no leading zeros)
        example: "1"
```

**Custom headers:**

```yaml
/orders:
  get:
    headers:
      X-Correlation-Id:
        type: string
        required: true
        pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
        description: UUID for request tracing
        example: "550e8400-e29b-41d4-a716-446655440000"
      X-Client-Id:
        type: string
        required: true
        pattern: "^[a-z][a-z0-9-]{2,29}$"
        description: |
          Registered client identifier.
          Starts with a letter, 3-30 chars, lowercase alphanumeric and hyphens.
        example: "order-service-prod"
      X-Request-Timestamp:
        type: string
        required: false
        pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])T([01]\\d|2[0-3]):[0-5]\\d:[0-5]\\dZ$"
        description: ISO 8601 UTC timestamp (e.g., 2026-05-05T14:30:00Z)
        example: "2026-05-05T14:30:00Z"
      Authorization:
        type: string
        required: true
        pattern: "^Bearer [A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+$"
        description: Bearer token in JWT format (three base64url segments)
        example: "Bearer eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ1c2VyMSJ9.signature"
```

> [!TIP]
> Validating headers like `Authorization` with a regex pattern documents the expected format for consumers and catches obvious mistakes (e.g., forgetting the "Bearer " prefix) without implementing full JWT verification in the spec layer.

---

### Step 9 — Real-World Pattern Library

Here is a reference library of battle-tested patterns ready to use in your RAML specs:

```yaml
types:
  # --- Identifiers ---
  UUID:
    type: string
    pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$"

  Slug:
    type: string
    pattern: "^[a-z0-9]+(-[a-z0-9]+)*$"
    description: URL-safe slug (e.g., my-resource-name)

  # --- Dates & Times ---
  ISODate:
    type: string
    pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])$"

  ISODateTime:
    type: string
    pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])T([01]\\d|2[0-3]):[0-5]\\d:[0-5]\\d(Z|[+-](0\\d|1[0-4]):[0-5]\\d)$"

  TimeOnly:
    type: string
    pattern: "^([01]\\d|2[0-3]):[0-5]\\d(:[0-5]\\d)?$"
    description: "24-hour time (HH:MM or HH:MM:SS)"

  # --- Network ---
  IPv4:
    type: string
    pattern: "^((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}(25[0-5]|2[0-4]\\d|[01]?\\d\\d?)$"

  Hostname:
    type: string
    pattern: "^([a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?\\.)+[a-zA-Z]{2,}$"

  PortNumber:
    type: string
    pattern: "^([1-9]\\d{0,3}|[1-5]\\d{4}|6[0-4]\\d{3}|65[0-4]\\d{2}|655[0-2]\\d|6553[0-5])$"
    description: Valid port (1-65535)

  # --- Business ---
  Email:
    type: string
    pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"

  CreditCardMasked:
    type: string
    pattern: "^\\*{4}-\\*{4}-\\*{4}-\\d{4}$"
    description: "Masked card number showing last 4 digits (****-****-****-1234)"

  CurrencyAmount:
    type: string
    pattern: "^-?[0-9]+(\\.[0-9]{1,2})?$"
    description: "Monetary amount with up to 2 decimal places"

  SemanticVersion:
    type: string
    pattern: "^(0|[1-9]\\d*)\\.(0|[1-9]\\d*)\\.(0|[1-9]\\d*)$"
    description: "SemVer without pre-release (e.g., 1.0.0, 2.3.11)"

  # --- MuleSoft Specific ---
  ClientId:
    type: string
    pattern: "^[0-9a-f]{32}$"
    description: Anypoint Platform client ID (32 hex chars)

  ApiVersion:
    type: string
    pattern: "^v[1-9]\\d*(\\.\\d+)?$"
    description: "API version (e.g., v1, v2.1)"
```

> [!NOTE]
> These patterns prioritize **readability and correctness over exhaustiveness**. For example, the `ISODate` pattern validates the format but allows dates like `2026-02-31`. Full calendar validation belongs in implementation logic, not in a spec pattern.

---

## Verification

After adding patterns to your RAML spec, verify them with these checks:

| Check | Expected Result |
|-------|-----------------|
| RAML file parses without errors | API Designer shows no red markers |
| Valid example values pass | Each `example` field satisfies its `pattern` |
| Invalid values are rejected | Mocking service returns 400 for malformed input |
| Double escaping is correct | No YAML parse warnings about unknown escape sequences |
| Anchors are present | Every pattern starts with `^` and ends with `$` |

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Pattern matches nothing | Single backslash in YAML (`\d`) | Double the backslash (`\\d`) |
| Pattern allows unexpected values | Missing `^` and/or `$` anchors | Add anchors: `"^pattern$"` |
| YAML parse error on pattern line | Unquoted pattern with special YAML chars (`{`, `:`) | Wrap pattern in double quotes |
| Pattern rejects valid input | Overly strict regex (e.g., exact length on variable fields) | Use `{min,max}` instead of `{n}` |
| API Designer shows "invalid regex" | Unsupported lookahead/lookbehind syntax | Replace with alternation or character classes |
| Pattern works locally but fails in Mocking | Regex flavor differences | Stick to basic POSIX/PCRE features; avoid `\p{}` Unicode classes |

---

## Summary

Regular expressions give you a powerful, declarative way to enforce input constraints directly in your API contracts. We covered:

- **Literals and metacharacters** for matching specific characters and character types
- **Character classes** for defining sets of allowed characters
- **Quantifiers** for controlling repetition (`+`, `*`, `?`, `{n,m}`)
- **Anchors** (`^`, `$`) for ensuring full-string validation
- **Groups and alternation** for complex structures and choices
- **RAML integration** using the `pattern` property in types, URI parameters, query parameters, and headers

The key takeaway: **push validation left**. By encoding format rules in your RAML spec, you document expectations for consumers, enable automatic validation in API gateways, and reduce defensive parsing code in your implementations.

---

## References

- [RAML 1.0 Specification — String Type and Pattern](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#string)
- [Regular-Expressions.info — Tutorial](https://www.regular-expressions.info/tutorial.html)
- [Regular Expressions 101](https://regex101.com/)
- [MDN Web Docs — Regular Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions)
- [Anypoint API Designer Documentation](https://docs.mulesoft.com/design-center/design-create-publish-api-specs)
- [RFC 4122 — UUID Format](https://www.rfc-editor.org/rfc/rfc4122)
- [Semantic Versioning 2.0.0](https://semver.org/)
