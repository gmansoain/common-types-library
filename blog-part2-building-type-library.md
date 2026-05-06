---
title: "Building a Regex-Validated Type Library in RAML"
slug: "building-regex-validated-type-library-raml"
description: "Create a reusable RAML Library fragment packed with regex-validated types — from UUIDs to Salesforce IDs — ready to share across all your API specs."
author: "Gonzalo Marcos"
date: 2026-05-05
status: draft
lang: en
category: api-management
tags:
  - raml
  - api-designer
  - anypoint-platform
  - anypoint-exchange
  - best-practices
  - reference
  - english
type: tutorial
difficulty: intermediate
read_time: 20
platform:
  - anypoint-platform
canonical_url: ""
---

![Draft](https://img.shields.io/badge/Status-Draft-7f8c8d) ![English](https://img.shields.io/badge/Lang-English-4a4a4a) ![Anypoint Platform](https://img.shields.io/badge/Platform-Anypoint_Platform-00A0DF?logo=mulesoft&logoColor=white) ![Intermediate](https://img.shields.io/badge/Level-Intermediate-f39c12) ![Tutorial](https://img.shields.io/badge/Type-Tutorial-8e44ad) ![20 min](https://img.shields.io/badge/Read_Time-20_min-lightgrey)

# Building a Regex-Validated Type Library in RAML

In [Part 1](regex-fundamentals-pattern-validation-raml) we learned how regular expressions work and how to apply them in RAML using the `pattern` property. Now it is time to take that knowledge and build something you can actually ship: a **reusable RAML Library fragment** containing dozens of validated types covering identifiers, dates, networking, finance, security, Salesforce, MuleSoft, and more.

The goal is simple — define the pattern once, publish it to Exchange, and reference it from every API spec in your organization. No more copy-pasting regex patterns across projects. No more inconsistencies where one spec accepts `2026-5-5` and another demands `2026-05-05`.

By the end of this tutorial you will have a production-ready `common-types-library.raml` file with **50+ validated types**, structured for maximum reuse across your entire API portfolio.

---

## Prerequisites

- Completed [Part 1: Regex Fundamentals for API Designers](regex-fundamentals-pattern-validation-raml) (or equivalent regex knowledge)
- Familiarity with RAML 1.0 type syntax
- Access to Anypoint API Designer or a local RAML editor
- (Optional) Access to Anypoint Exchange for publishing the fragment

## Overview

We will build the library incrementally, domain by domain:

1. **Project structure** — setting up the RAML Library fragment
2. **Identifiers** — UUID, ULID, Slug, prefixed IDs, Salesforce IDs
3. **Dates & Times** — ISO 8601 dates, timestamps, durations
4. **People & Contact** — email, phone, usernames
5. **Geography & Locale** — country codes, language tags, postal codes
6. **Financial** — currencies, amounts, IBAN, BIC
7. **Network & Infrastructure** — IPs, hostnames, ports, CIDR
8. **Security & Auth** — tokens, API keys, base64
9. **Versioning & Metadata** — SemVer, MIME types, API versions
10. **Business & Enterprise** — order IDs, tracking, cron, colors
11. **Salesforce** — object-specific ID validation
12. **MuleSoft / Anypoint** — platform-specific identifiers
13. **Assembling and consuming the library**

## Table of Contents

- [Step 1 — Setting Up the RAML Library Fragment](#step-1--setting-up-the-raml-library-fragment)
- [Step 2 — Identifiers](#step-2--identifiers)
- [Step 3 — Dates and Times](#step-3--dates-and-times)
- [Step 4 — People and Contact](#step-4--people-and-contact)
- [Step 5 — Geography and Locale](#step-5--geography-and-locale)
- [Step 6 — Financial](#step-6--financial)
- [Step 7 — Network and Infrastructure](#step-7--network-and-infrastructure)
- [Step 8 — Security and Auth](#step-8--security-and-auth)
- [Step 9 — Versioning and Metadata](#step-9--versioning-and-metadata)
- [Step 10 — Business and Enterprise](#step-10--business-and-enterprise)
- [Step 11 — Salesforce IDs](#step-11--salesforce-ids)
- [Step 12 — MuleSoft and Anypoint Platform](#step-12--mulesoft-and-anypoint-platform)
- [Step 13 — Assembling and Consuming the Library](#step-13--assembling-and-consuming-the-library)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)

---

### Step 1 — Setting Up the RAML Library Fragment

A RAML Library is a reusable fragment that can be published to Anypoint Exchange and imported by any API specification. It uses the `#%RAML 1.0 Library` header.

Create the following project structure:

```
common-types-library/
├── common-types-library.raml      ← Main library entry point
├── types/
│   ├── identifiers.raml
│   ├── dates.raml
│   ├── contact.raml
│   ├── geography.raml
│   ├── financial.raml
│   ├── network.raml
│   ├── security.raml
│   ├── versioning.raml
│   ├── business.raml
│   ├── salesforce.raml
│   └── mulesoft.raml
└── exchange.json                  ← Exchange metadata (for publishing)
```

The main entry point imports all domain sub-files and **re-exports every type at the top level**. This is critical — RAML 1.0 does not support chained references (`Common.Identifiers.UUID`) and Design Center warns against importing non-root dependency files. By re-exporting types flat, consumers get clean single dot notation (`Common.UUID`):

```yaml
#%RAML 1.0 Library

usage: |
  Common regex-validated types for reuse across all API specifications.
  Import this library to get consistent validation for identifiers, dates,
  contact info, financial data, networking, security, and more.

uses:
  _Identifiers: types/identifiers.raml
  _Dates: types/dates.raml
  _Contact: types/contact.raml
  _Geography: types/geography.raml
  _Financial: types/financial.raml
  _Network: types/network.raml
  _Security: types/security.raml
  _Versioning: types/versioning.raml
  _Business: types/business.raml
  _Salesforce: types/salesforce.raml
  _MuleSoft: types/mulesoft.raml

types:
  # Re-export all types at the top level
  UUID:
    type: _Identifiers.UUID
  ISODate:
    type: _Dates.ISODate
  Email:
    type: _Contact.Email
  SalesforceAccountId:
    type: _Salesforce.SalesforceAccountId
  # ... (all 127 types follow this pattern)
```

> [!TIP]
> The underscore prefix on `uses` aliases (`_Identifiers`, `_Dates`) is a convention to signal these are internal namespaces — not intended for direct consumer access. Consumers only see the flat type names: `Common.UUID`, `Common.ISODate`, `Common.SalesforceAccountId`, etc.

---

### Step 2 — Identifiers

These are the most commonly reused types across any API portfolio. Every resource needs an ID, and enforcing format consistency prevents integration bugs.

📄 `types/identifiers.raml`

```yaml
#%RAML 1.0 Library

usage: Universal identifier types with regex validation.

types:
  UUID:
    type: string
    pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$"
    description: |
      RFC 4122 UUID (lowercase, hyphenated).
      Validates version (1-5) and variant (8, 9, a, b) bits.
    examples:
      v4: "550e8400-e29b-41d4-a716-446655440000"
      v1: "6ba7b810-9dad-11d1-80b4-00c04fd430c8"

  UUIDUppercase:
    type: string
    pattern: "^[0-9A-F]{8}-[0-9A-F]{4}-[1-5][0-9A-F]{3}-[89AB][0-9A-F]{3}-[0-9A-F]{12}$"
    description: |
      RFC 4122 UUID in uppercase (some legacy systems output uppercase).
    example: "550E8400-E29B-41D4-A716-446655440000"

  UUIDAnyCase:
    type: string
    pattern: "^[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[1-5][0-9a-fA-F]{3}-[89abAB][0-9a-fA-F]{3}-[0-9a-fA-F]{12}$"
    description: |
      RFC 4122 UUID accepting both upper and lowercase.
      Use when you cannot control the casing of upstream systems.
    example: "550e8400-E29B-41d4-A716-446655440000"

  ULID:
    type: string
    pattern: "^[0-7][0-9A-HJKMNP-TV-Z]{25}$"
    description: |
      Universally Unique Lexicographically Sortable Identifier.
      26 characters, Crockford Base32 encoded. Time-sortable.
      First character is limited to 0-7 to prevent overflow.
    example: "01ARZ3NDEKTSV4RRFFQ69G5FAV"

  Slug:
    type: string
    pattern: "^[a-z0-9]+(-[a-z0-9]+)*$"
    minLength: 1
    maxLength: 128
    description: |
      URL-safe slug. Lowercase alphanumeric segments separated by single hyphens.
      Cannot start or end with a hyphen, no consecutive hyphens.
    examples:
      simple: "hello"
      multi_word: "my-resource-name"
      with_numbers: "order-123-details"

  PrefixedId:
    type: string
    pattern: "^[a-z]{2,10}_[a-zA-Z0-9]{8,32}$"
    description: |
      Stripe-style prefixed identifier.
      A lowercase prefix (2-10 chars) followed by underscore and an alphanumeric value (8-32 chars).
    examples:
      user: "usr_A1b2C3d4E5f6G7h8"
      customer: "cus_9a8B7c6D5e4F3g2H"
      payment: "pay_Xk9mN2pQ4rS6tU8w"

  CorrelationId:
    type: string
    pattern: "^[a-zA-Z0-9][a-zA-Z0-9._-]{0,127}$"
    description: |
      Distributed tracing correlation identifier.
      Starts with alphanumeric, allows dots, underscores, hyphens. Max 128 chars.
      Flexible enough for UUID-based, timestamp-based, or custom formats.
    examples:
      uuid_style: "550e8400-e29b-41d4-a716-446655440000"
      custom: "corr.20260505.order-svc.a1b2c3d4"
      simple: "req-001-abc"
```

> [!NOTE]
> The `UUID` type validates the version nibble (position 13: 1–5) and variant bits (position 17: 8, 9, a, b) per RFC 4122. If you only care about format and not spec compliance, replace `[1-5]` with `[0-9a-f]` and `[89ab]` with `[0-9a-f]`.

---

### Step 3 — Dates and Times

Date/time validation is critical. APIs that accept freeform date strings inevitably break when consumers send `5/5/2026` instead of `2026-05-05`.

📄 `types/dates.raml`

```yaml
#%RAML 1.0 Library

usage: ISO 8601 date and time types with format validation.

types:
  ISODate:
    type: string
    pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])$"
    description: |
      ISO 8601 date: YYYY-MM-DD.
      Validates month (01-12) and day (01-31) ranges.
      Note: does not validate calendar correctness (e.g., Feb 30 passes).
    examples:
      standard: "2026-05-05"
      first_of_year: "2026-01-01"
      end_of_year: "2026-12-31"

  ISODateTime:
    type: string
    pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])T([01]\\d|2[0-3]):[0-5]\\d:[0-5]\\dZ$"
    description: |
      ISO 8601 UTC timestamp: YYYY-MM-DDTHH:MM:SSZ.
      Always in UTC (Z suffix). Use ISODateTimeOffset for non-UTC.
    examples:
      noon: "2026-05-05T12:00:00Z"
      midnight: "2026-01-01T00:00:00Z"
      late: "2026-12-31T23:59:59Z"

  ISODateTimeOffset:
    type: string
    pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])T([01]\\d|2[0-3]):[0-5]\\d:[0-5]\\d([+-](0\\d|1[0-4]):[0-5]\\d|Z)$"
    description: |
      ISO 8601 timestamp with timezone offset: YYYY-MM-DDTHH:MM:SS±HH:MM or Z.
      Supports offsets from -14:00 to +14:00.
    examples:
      utc: "2026-05-05T14:30:00Z"
      positive_offset: "2026-05-05T14:30:00+02:00"
      negative_offset: "2026-05-05T08:30:00-05:00"

  ISODateTimeMillis:
    type: string
    pattern: "^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])T([01]\\d|2[0-3]):[0-5]\\d:[0-5]\\d\\.\\d{3}([+-](0\\d|1[0-4]):[0-5]\\d|Z)$"
    description: |
      ISO 8601 timestamp with millisecond precision.
      Format: YYYY-MM-DDTHH:MM:SS.mmmZ or YYYY-MM-DDTHH:MM:SS.mmm±HH:MM.
    examples:
      utc_millis: "2026-05-05T14:30:00.123Z"
      offset_millis: "2026-05-05T14:30:00.456+02:00"

  TimeOnly:
    type: string
    pattern: "^([01]\\d|2[0-3]):[0-5]\\d(:[0-5]\\d)?$"
    description: |
      24-hour time: HH:MM or HH:MM:SS.
      Hours 00-23, minutes and seconds 00-59.
    examples:
      with_seconds: "14:30:00"
      without_seconds: "09:15"
      midnight: "00:00:00"

  ISODuration:
    type: string
    pattern: "^P(\\d+Y)?(\\d+M)?(\\d+W)?(\\d+D)?(T(\\d+H)?(\\d+M)?(\\d+S)?)?$"
    description: |
      ISO 8601 duration.
      Format: PnYnMnWnDTnHnMnS (all components optional, at least one required).
    examples:
      full: "P3Y6M4DT12H30M5S"
      days_only: "P30D"
      hours_minutes: "PT2H30M"
      one_week: "P1W"

  YearMonth:
    type: string
    pattern: "^\\d{4}-(0[1-9]|1[0-2])$"
    description: |
      Year and month: YYYY-MM.
      Useful for billing periods, monthly reports, partition keys.
    examples:
      current: "2026-05"
      january: "2026-01"

  Year:
    type: string
    pattern: "^\\d{4}$"
    description: |
      Four-digit year: YYYY.
      Useful for fiscal years, vintages, annual reports.
    examples:
      current: "2026"
      past: "1999"

  UnixTimestamp:
    type: string
    pattern: "^[1-9]\\d{9,12}$"
    description: |
      Unix timestamp as string (seconds or milliseconds since epoch).
      10 digits = seconds, 13 digits = milliseconds.
    examples:
      seconds: "1778000000"
      milliseconds: "1778000000000"
```

> [!WARNING]
> ISO date patterns validate **format**, not **calendar correctness**. A pattern will happily accept `2026-02-31` or `2026-04-31`. Full date validation (leap years, days-in-month) belongs in implementation logic — the spec pattern catches format violations, which represent 95% of real-world errors.

---

### Step 4 — People and Contact

📄 `types/contact.raml`

```yaml
#%RAML 1.0 Library

usage: Contact and personal information types.

types:
  Email:
    type: string
    pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    maxLength: 254
    description: |
      Email address (simplified validation per RFC 5321 length limit).
      Validates general structure but does not cover all edge cases of RFC 5322.
      For production, combine with a verification email flow.
    examples:
      simple: "user@example.com"
      with_dots: "first.last@company.co.uk"
      with_plus: "user+tag@gmail.com"

  PhoneE164:
    type: string
    pattern: "^\\+[1-9]\\d{6,14}$"
    description: |
      International phone number in E.164 format.
      Starts with +, followed by country code and subscriber number.
      Total 7-15 digits (not counting the +).
    examples:
      us: "+14155552671"
      uk: "+442071234567"
      short: "+3912345678"

  PhoneNANP:
    type: string
    pattern: "^\\+1[2-9]\\d{2}[2-9]\\d{6}$"
    description: |
      North American phone number (US, Canada) in E.164.
      Validates area code and exchange rules (no 0 or 1 as first digit).
    example: "+12025551234"

  Username:
    type: string
    pattern: "^[a-zA-Z][a-zA-Z0-9._-]{2,29}$"
    description: |
      Login username. Starts with a letter, 3-30 total characters.
      Allows letters, digits, dots, underscores, and hyphens.
    examples:
      simple: "gmarcos"
      with_numbers: "user123"
      with_separators: "gonzalo.marcos_91"

  PersonName:
    type: string
    pattern: "^[\\p{L}][\\p{L}\\s'.,-]{0,99}$"
    minLength: 1
    maxLength: 100
    description: |
      Display name for a person. Starts with a letter (Unicode-aware).
      Allows letters, spaces, apostrophes, dots, commas, hyphens.
      Note: \\p{L} may not be supported in all RAML parsers — use the
      ASCII fallback if needed.

  PersonNameASCII:
    type: string
    pattern: "^[a-zA-Z][a-zA-Z\\s'.,-]{0,99}$"
    minLength: 1
    maxLength: 100
    description: |
      ASCII-only display name (fallback when Unicode regex is unsupported).
      Starts with a letter, allows spaces, apostrophes, dots, commas, hyphens.
    examples:
      simple: "Gonzalo Marcos"
      hyphenated: "Mary-Jane O'Brien"
      with_suffix: "James Smith, Jr."
```

---

### Step 5 — Geography and Locale

📄 `types/geography.raml`

```yaml
#%RAML 1.0 Library

usage: Geographic, locale, and address-related types.

types:
  CountryCodeAlpha2:
    type: string
    pattern: "^[A-Z]{2}$"
    description: |
      ISO 3166-1 alpha-2 country code (uppercase).
      Two letters representing a country (e.g., US, GB, IT, AR).
    examples:
      usa: "US"
      italy: "IT"
      uk: "GB"

  CountryCodeAlpha3:
    type: string
    pattern: "^[A-Z]{3}$"
    description: |
      ISO 3166-1 alpha-3 country code (uppercase).
      Three letters representing a country (e.g., USA, GBR, ITA, ARG).
    examples:
      usa: "USA"
      italy: "ITA"
      argentina: "ARG"

  CountryCodeNumeric:
    type: string
    pattern: "^\\d{3}$"
    description: |
      ISO 3166-1 numeric country code.
      Three-digit code (e.g., 840 for USA, 380 for Italy).
    examples:
      usa: "840"
      italy: "380"

  LanguageCode:
    type: string
    pattern: "^[a-z]{2}$"
    description: |
      ISO 639-1 two-letter language code (lowercase).
    examples:
      english: "en"
      spanish: "es"
      italian: "it"

  LanguageCode3:
    type: string
    pattern: "^[a-z]{3}$"
    description: |
      ISO 639-2/3 three-letter language code (lowercase).
    examples:
      english: "eng"
      spanish: "spa"
      italian: "ita"

  Locale:
    type: string
    pattern: "^[a-z]{2}(-[A-Z]{2})?$"
    description: |
      BCP 47 language tag: language with optional region.
      Format: ll or ll-RR (language-REGION).
    examples:
      language_only: "en"
      with_region: "en-US"
      italian: "it-IT"
      spanish_argentina: "es-AR"

  LocaleExtended:
    type: string
    pattern: "^[a-z]{2,3}(-[A-Z][a-z]{3})?(-[A-Z]{2})?(-([A-Za-z]{5,8}|\\d{3}))*$"
    description: |
      Extended BCP 47 language tag with optional script and variants.
      Format: language[-Script][-REGION][-variant].
    examples:
      simple: "en-US"
      with_script: "zh-Hans-CN"
      serbian_latin: "sr-Latn-RS"

  Latitude:
    type: string
    pattern: "^-?(([0-8]?\\d)(\\.\\d{1,8})?|90(\\.0{1,8})?)$"
    description: |
      Geographic latitude: -90.0 to 90.0.
      Up to 8 decimal places (~1mm precision).
    examples:
      nyc: "40.71280000"
      south: "-33.86880000"
      equator: "0.0"

  Longitude:
    type: string
    pattern: "^-?((1[0-7]\\d|0?\\d{1,2})(\\.\\d{1,8})?|180(\\.0{1,8})?)$"
    description: |
      Geographic longitude: -180.0 to 180.0.
      Up to 8 decimal places (~1mm precision).
    examples:
      nyc: "-74.00600000"
      east: "151.20930000"
      prime_meridian: "0.0"

  LatLong:
    type: string
    pattern: "^-?(([0-8]?\\d)(\\.\\d{1,8})?|90(\\.0{1,8})?),-?((1[0-7]\\d|0?\\d{1,2})(\\.\\d{1,8})?|180(\\.0{1,8})?)$"
    description: |
      Comma-separated latitude,longitude pair.
      Format: lat,long (no spaces).
    examples:
      nyc: "40.7128,-74.0060"
      london: "51.5074,-0.1278"

  # --- Postal Codes ---

  PostalCodeUS:
    type: string
    pattern: "^\\d{5}(-\\d{4})?$"
    description: |
      United States ZIP code.
      Five digits, with optional ZIP+4 extension.
    examples:
      five_digit: "10001"
      zip_plus_4: "10001-1234"

  PostalCodeUK:
    type: string
    pattern: "^[A-Z]{1,2}\\d[A-Z\\d]?\\s?\\d[A-Z]{2}$"
    description: |
      United Kingdom postcode.
      Format varies: A9 9AA, A99 9AA, A9A 9AA, AA9 9AA, AA99 9AA, AA9A 9AA.
      Space between outward and inward code is optional.
    examples:
      london: "SW1A 1AA"
      manchester: "M1 1AE"
      no_space: "EC1A1BB"

  PostalCodeCA:
    type: string
    pattern: "^[A-Z]\\d[A-Z]\\s?\\d[A-Z]\\d$"
    description: |
      Canadian postal code.
      Format: A9A 9A9 (letter-digit alternating). Space is optional.
    examples:
      with_space: "K1A 0B1"
      without_space: "V6B3K9"

  PostalCodeDE:
    type: string
    pattern: "^\\d{5}$"
    description: |
      German postal code (Postleitzahl). Exactly 5 digits.
    examples:
      berlin: "10115"
      munich: "80331"

  PostalCodeFR:
    type: string
    pattern: "^\\d{5}$"
    description: |
      French postal code (Code postal). Exactly 5 digits.
    examples:
      paris: "75001"
      marseille: "13001"

  PostalCodeIT:
    type: string
    pattern: "^\\d{5}$"
    description: |
      Italian postal code (CAP - Codice di Avviamento Postale). Exactly 5 digits.
    examples:
      rome: "00100"
      milan: "20100"

  PostalCodeES:
    type: string
    pattern: "^(0[1-9]|[1-4]\\d|5[0-2])\\d{3}$"
    description: |
      Spanish postal code (Código postal).
      5 digits, first two represent the province (01-52).
    examples:
      madrid: "28001"
      barcelona: "08001"

  PostalCodeBR:
    type: string
    pattern: "^\\d{5}-?\\d{3}$"
    description: |
      Brazilian postal code (CEP). Format: 99999-999 (hyphen optional).
    examples:
      with_hyphen: "01001-000"
      without_hyphen: "01001000"

  PostalCodeAR:
    type: string
    pattern: "^[A-Z]\\d{4}[A-Z]{3}$"
    description: |
      Argentine CPA (Código Postal Argentino).
      Format: A9999AAA (letter + 4 digits + 3 letters).
    example: "C1420ABC"

  PostalCodeJP:
    type: string
    pattern: "^\\d{3}-?\\d{4}$"
    description: |
      Japanese postal code (郵便番号). Format: 999-9999 (hyphen optional).
    examples:
      with_hyphen: "100-0001"
      without_hyphen: "1000001"

  PostalCodeAU:
    type: string
    pattern: "^\\d{4}$"
    description: |
      Australian postcode. Exactly 4 digits.
    examples:
      sydney: "2000"
      melbourne: "3000"

  PostalCodeIN:
    type: string
    pattern: "^[1-9]\\d{5}$"
    description: |
      Indian PIN code. 6 digits, first digit is 1-9 (zone).
    examples:
      delhi: "110001"
      mumbai: "400001"
```

> [!TIP]
> If your API serves a specific region, only include the relevant postal code types. If it is global, offer a generic `PostalCode` type with a lenient pattern (`^[A-Z0-9\\s-]{3,10}$`) and validate per-country in implementation logic.

---

### Step 6 — Financial

📄 `types/financial.raml`

```yaml
#%RAML 1.0 Library

usage: Financial, currency, and payment-related types.

types:
  CurrencyCode:
    type: string
    pattern: "^[A-Z]{3}$"
    description: |
      ISO 4217 currency code. Three uppercase letters.
    examples:
      dollar: "USD"
      euro: "EUR"
      pound: "GBP"
      yen: "JPY"

  MonetaryAmount:
    type: string
    pattern: "^-?[0-9]+(\\.[0-9]{1,2})?$"
    description: |
      Monetary amount with up to 2 decimal places.
      Allows negative values for refunds/credits.
      No thousand separators, no currency symbol.
    examples:
      positive: "1234.56"
      integer: "100"
      negative: "-99.00"
      cents: "0.99"

  MonetaryAmountStrict:
    type: string
    pattern: "^-?[1-9]\\d*\\.\\d{2}$|^-?0\\.\\d{2}$"
    description: |
      Strict monetary amount: always exactly 2 decimal places, no leading zeros
      (except for values less than 1).
    examples:
      standard: "1234.56"
      small: "0.99"
      negative: "-50.00"

  MonetaryAmountCrypto:
    type: string
    pattern: "^-?[0-9]+(\\.[0-9]{1,18})?$"
    description: |
      Monetary amount for cryptocurrency (up to 18 decimal places for Wei precision).
    examples:
      bitcoin: "0.00012345"
      ethereum: "1.234567890123456789"

  CreditCardMasked:
    type: string
    pattern: "^\\*{4}-\\*{4}-\\*{4}-\\d{4}$"
    description: |
      PCI-DSS compliant masked card number.
      Shows only the last 4 digits, all others are asterisks.
    example: "****-****-****-4242"

  CreditCardBIN:
    type: string
    pattern: "^\\d{6,8}$"
    description: |
      Bank Identification Number (first 6-8 digits of a card).
      Used for routing and identifying the issuing bank.
    examples:
      six_digit: "424242"
      eight_digit: "42424242"

  IBAN:
    type: string
    pattern: "^[A-Z]{2}\\d{2}[A-Z0-9]{4}[A-Z0-9]{7,27}$"
    description: |
      International Bank Account Number (ISO 13616).
      Format: 2 country letters + 2 check digits + 4 bank code + 7-27 account chars.
      Total length varies by country (max 34 characters).
    examples:
      germany: "DE89370400440532013000"
      uk: "GB29NWBK60161331926819"
      spain: "ES9121000418450200051332"

  BIC:
    type: string
    pattern: "^[A-Z]{4}[A-Z]{2}[A-Z0-9]{2}([A-Z0-9]{3})?$"
    description: |
      SWIFT/BIC code (ISO 9362).
      Format: 4 bank code + 2 country + 2 location + optional 3 branch.
      8 or 11 characters total.
    examples:
      eight_char: "NWBKGB2L"
      eleven_char: "DEUTDEFF500"

  TaxId:
    type: string
    pattern: "^[A-Z]{2}[A-Z0-9]{5,20}$"
    description: |
      Generic tax identification number.
      Country prefix (2 uppercase letters) followed by alphanumeric identifier.
      Specific formats vary by country — use country-specific types for strict validation.
    examples:
      eu_vat: "DE123456789"
      uk_vat: "GB123456789"
```

---

### Step 7 — Network and Infrastructure

📄 `types/network.raml`

```yaml
#%RAML 1.0 Library

usage: Network, infrastructure, and connectivity types.

types:
  IPv4:
    type: string
    pattern: "^((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}(25[0-5]|2[0-4]\\d|[01]?\\d\\d?)$"
    description: |
      IPv4 address. Four octets (0-255) separated by dots.
    examples:
      private: "192.168.1.1"
      public: "8.8.8.8"
      localhost: "127.0.0.1"

  IPv4CIDR:
    type: string
    pattern: "^((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}(25[0-5]|2[0-4]\\d|[01]?\\d\\d?)/(3[0-2]|[12]?\\d)$"
    description: |
      IPv4 address with CIDR notation (network range).
      Suffix /0 to /32.
    examples:
      class_c: "192.168.1.0/24"
      single_host: "10.0.0.5/32"
      large_range: "10.0.0.0/8"

  IPv6:
    type: string
    pattern: "^([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$"
    description: |
      Full IPv6 address (expanded form, 8 groups of 4 hex digits).
      For abbreviated forms with ::, use IPv6Any.
    example: "2001:0db8:85a3:0000:0000:8a2e:0370:7334"

  IPv6Any:
    type: string
    pattern: "^([0-9a-fA-F]{0,4}:){2,7}[0-9a-fA-F]{0,4}$"
    description: |
      IPv6 address allowing abbreviated notation (::).
      Matches both full and shortened forms.
    examples:
      full: "2001:0db8:85a3:0000:0000:8a2e:0370:7334"
      short: "2001:db8::1"
      loopback: "::1"

  IPv6CIDR:
    type: string
    pattern: "^([0-9a-fA-F]{0,4}:){2,7}[0-9a-fA-F]{0,4}/(12[0-8]|1[01]\\d|\\d{1,2})$"
    description: |
      IPv6 address with CIDR notation.
      Suffix /0 to /128.
    examples:
      subnet: "2001:db8::/32"
      single: "::1/128"

  Hostname:
    type: string
    pattern: "^([a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?\\.)+[a-zA-Z]{2,}$"
    maxLength: 253
    description: |
      Fully qualified domain name (FQDN).
      Each label: 1-63 chars, starts/ends with alphanumeric.
      Total max 253 characters.
    examples:
      simple: "api.example.com"
      subdomain: "us-east-1.api.example.com"
      short: "localhost.localdomain"

  HostnameOrIP:
    type: string
    pattern: "^(([a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?\\.)+[a-zA-Z]{2,}|((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}(25[0-5]|2[0-4]\\d|[01]?\\d\\d?))$"
    description: |
      Either a valid hostname (FQDN) or an IPv4 address.
      Useful for configuration fields that accept both.
    examples:
      hostname: "api.example.com"
      ip: "192.168.1.100"

  PortNumber:
    type: string
    pattern: "^([1-9]\\d{0,3}|[1-5]\\d{4}|6[0-4]\\d{3}|65[0-4]\\d{2}|655[0-2]\\d|6553[0-5])$"
    description: |
      Valid TCP/UDP port number (1–65535).
      Does not allow 0 (reserved) or leading zeros.
    examples:
      http: "80"
      https: "443"
      custom: "8443"
      max: "65535"

  HostPort:
    type: string
    pattern: "^([a-zA-Z0-9]([a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?\\.)+[a-zA-Z]{2,}:\\d{1,5}$"
    description: |
      Hostname:port combination.
    examples:
      standard: "api.example.com:443"
      custom: "mule-worker.internal:8081"

  MacAddress:
    type: string
    pattern: "^([0-9A-Fa-f]{2}[:-]){5}[0-9A-Fa-f]{2}$"
    description: |
      MAC address in colon or hyphen-separated format.
      Six groups of two hex digits.
    examples:
      colon: "00:1A:2B:3C:4D:5E"
      hyphen: "00-1A-2B-3C-4D-5E"

  HttpUrl:
    type: string
    pattern: "^https?://[a-zA-Z0-9]([a-zA-Z0-9-]*[a-zA-Z0-9])?(\\.[a-zA-Z0-9]([a-zA-Z0-9-]*[a-zA-Z0-9])?)*(:\\d{1,5})?(/[a-zA-Z0-9._~:/?#\\[\\]@!$&'()*+,;=%-]*)?$"
    description: |
      HTTP or HTTPS URL. Validates scheme, host, optional port, and path.
    examples:
      simple: "https://api.example.com"
      with_path: "https://api.example.com/v1/orders"
      with_port: "http://localhost:8081/health"

  HttpsUrl:
    type: string
    pattern: "^https://[a-zA-Z0-9]([a-zA-Z0-9-]*[a-zA-Z0-9])?(\\.[a-zA-Z0-9]([a-zA-Z0-9-]*[a-zA-Z0-9])?)*(:\\d{1,5})?(/[a-zA-Z0-9._~:/?#\\[\\]@!$&'()*+,;=%-]*)?$"
    description: |
      HTTPS-only URL (secure connections required).
    example: "https://api.example.com/v1/orders"
```

---

### Step 8 — Security and Auth

📄 `types/security.raml`

```yaml
#%RAML 1.0 Library

usage: Security, authentication, and authorization types.

types:
  BearerToken:
    type: string
    pattern: "^Bearer [A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+$"
    description: |
      Bearer token in JWT format (Authorization header value).
      Three Base64URL-encoded segments separated by dots, prefixed with "Bearer ".
    example: "Bearer eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ1c2VyMSJ9.dGhpcyBpcyBhIHNpZ25hdHVyZQ"

  JWTStructure:
    type: string
    pattern: "^[A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+\\.[A-Za-z0-9_-]+$"
    description: |
      JWT token structure (without Bearer prefix).
      Three Base64URL-encoded segments: header.payload.signature.
    example: "eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ1c2VyMSJ9.dGhpcyBpcyBhIHNpZ25hdHVyZQ"

  ApiKey:
    type: string
    pattern: "^[a-z]{2,10}_(live|test|sandbox)_[A-Za-z0-9]{20,64}$"
    description: |
      Prefixed API key with environment indicator.
      Format: prefix_environment_secret.
    examples:
      live: "sk_live_A1b2C3d4E5f6G7h8I9j0K1l2M3n4O5p6"
      test: "pk_test_Xk9mN2pQ4rS6tU8wY0zA1bC2dE3fG4hI"
      sandbox: "ak_sandbox_R7s8T9u0V1w2X3y4Z5a6B7c8D9e0F1g2"

  ApiKeySimple:
    type: string
    pattern: "^[A-Za-z0-9]{32,128}$"
    description: |
      Simple API key (no prefix structure). Alphanumeric, 32-128 chars.
    example: "a1B2c3D4e5F6g7H8i9J0k1L2m3N4o5P6"

  ClientId:
    type: string
    pattern: "^[0-9a-f]{32}$"
    description: |
      OAuth2 client identifier (32 lowercase hexadecimal characters).
      Standard format for Anypoint Platform client applications.
    example: "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6"

  ClientSecret:
    type: string
    pattern: "^[A-Za-z0-9]{32,64}$"
    description: |
      OAuth2 client secret. Alphanumeric, 32-64 characters.
      WARNING: Never log or expose in responses.
    example: "A1b2C3d4E5f6G7h8I9j0K1l2M3n4O5p6"

  Base64:
    type: string
    pattern: "^[A-Za-z0-9+/]+=*$"
    minLength: 4
    description: |
      Standard Base64-encoded string (RFC 4648).
      Allows padding characters (=).
    examples:
      short: "SGVsbG8="
      longer: "SGVsbG8gV29ybGQh"

  Base64URL:
    type: string
    pattern: "^[A-Za-z0-9_-]+$"
    minLength: 1
    description: |
      Base64URL-encoded string (RFC 4648 §5).
      URL-safe variant: uses - instead of + and _ instead of /.
      No padding characters.
    example: "SGVsbG8gV29ybGQh"

  SHA256Hash:
    type: string
    pattern: "^[0-9a-f]{64}$"
    description: |
      SHA-256 hash digest (lowercase hex, 64 characters).
    example: "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"

  MD5Hash:
    type: string
    pattern: "^[0-9a-f]{32}$"
    description: |
      MD5 hash digest (lowercase hex, 32 characters).
      Note: MD5 is cryptographically broken — use only for checksums, never for security.
    example: "d41d8cd98f00b204e9800998ecf8427e"

  BasicAuthHeader:
    type: string
    pattern: "^Basic [A-Za-z0-9+/]+=*$"
    description: |
      HTTP Basic Authentication header value.
      Format: "Basic " followed by Base64-encoded "username:password".
    example: "Basic dXNlcjpwYXNzd29yZA=="
```

> [!IMPORTANT]
> Never use `pattern` to validate the *content* of a secret (e.g., ensuring a password has uppercase, lowercase, numbers). Pattern validation happens in the spec/gateway layer — secrets should flow through opaquely. These patterns validate *format* (length, encoding, prefix) to catch malformed requests early.

---

### Step 9 — Versioning and Metadata

📄 `types/versioning.raml`

```yaml
#%RAML 1.0 Library

usage: Versioning, content type, and metadata types.

types:
  SemanticVersion:
    type: string
    pattern: "^(0|[1-9]\\d*)\\.(0|[1-9]\\d*)\\.(0|[1-9]\\d*)(-[a-zA-Z0-9]+(\\.[a-zA-Z0-9]+)*)?(\\+[a-zA-Z0-9]+(\\.[a-zA-Z0-9]+)*)?$"
    description: |
      Full Semantic Versioning 2.0.0.
      Format: MAJOR.MINOR.PATCH[-prerelease][+build].
      No leading zeros in numeric segments.
    examples:
      stable: "1.0.0"
      prerelease: "2.1.0-beta.1"
      with_build: "1.0.0+20260505"
      complex: "3.2.1-rc.1+build.123"

  SemanticVersionStrict:
    type: string
    pattern: "^(0|[1-9]\\d*)\\.(0|[1-9]\\d*)\\.(0|[1-9]\\d*)$"
    description: |
      Strict Semantic Version without pre-release or build metadata.
      Use when you only accept released versions.
    examples:
      initial: "1.0.0"
      mature: "12.3.45"

  ApiVersionPrefix:
    type: string
    pattern: "^v[1-9]\\d*$"
    description: |
      API major version prefix (e.g., v1, v2, v10).
      No minor/patch — API versions use major-only.
    examples:
      v1: "v1"
      v2: "v2"
      v12: "v12"

  ApiVersionFull:
    type: string
    pattern: "^v[1-9]\\d*(\\.\\d+){0,2}$"
    description: |
      API version with optional minor and patch.
      Format: vMAJOR[.MINOR[.PATCH]].
    examples:
      major_only: "v1"
      major_minor: "v2.1"
      full: "v3.0.1"

  MimeType:
    type: string
    pattern: "^(application|audio|font|image|message|model|multipart|text|video)/[a-zA-Z0-9][a-zA-Z0-9!#$&\\-^_.+]*$"
    description: |
      IANA media type (Content-Type without parameters).
      Format: type/subtype.
    examples:
      json: "application/json"
      xml: "application/xml"
      html: "text/html"
      custom: "application/vnd.api+json"

  MimeTypeWithParams:
    type: string
    pattern: "^(application|audio|font|image|message|model|multipart|text|video)/[a-zA-Z0-9][a-zA-Z0-9!#$&\\-^_.+]*(;\\s?[a-zA-Z0-9-]+=[a-zA-Z0-9-]+)*$"
    description: |
      Media type with optional parameters (e.g., charset).
    examples:
      with_charset: "application/json; charset=utf-8"
      plain: "text/plain"

  HttpMethod:
    type: string
    pattern: "^(GET|POST|PUT|PATCH|DELETE|HEAD|OPTIONS|TRACE)$"
    description: |
      Standard HTTP method (uppercase).
    examples:
      get: "GET"
      post: "POST"
      delete: "DELETE"

  HttpStatusCode:
    type: string
    pattern: "^[1-5]\\d{2}$"
    description: |
      HTTP status code (100-599).
    examples:
      ok: "200"
      not_found: "404"
      server_error: "500"

  ETag:
    type: string
    pattern: "^(W/)?\"[^\"]*\"$"
    description: |
      HTTP ETag header value. Optionally weak (W/ prefix), quoted string.
    examples:
      strong: "\"33a64df551425fcc55e4d42a148795d9f25f89d4\""
      weak: "W/\"0815\""
```

---

### Step 10 — Business and Enterprise

📄 `types/business.raml`

```yaml
#%RAML 1.0 Library

usage: Business domain identifiers and enterprise patterns.

types:
  OrderId:
    type: string
    pattern: "^ORD-\\d{4}-[A-Z0-9]{6}$"
    description: |
      Order identifier with year prefix.
      Format: ORD-YYYY-XXXXXX (6 uppercase alphanumeric chars).
    examples:
      current: "ORD-2026-A1B2C3"
      past: "ORD-2024-X9F3A1"

  InvoiceNumber:
    type: string
    pattern: "^INV-\\d{6}-\\d{4}$"
    description: |
      Invoice number with sequence and year-month.
      Format: INV-YYYYMM-NNNN (sequential number).
    examples:
      may: "INV-202605-0042"
      january: "INV-202601-0001"

  TrackingNumberUPS:
    type: string
    pattern: "^1Z[A-Z0-9]{16}$"
    description: |
      UPS tracking number.
      Starts with 1Z, followed by 16 alphanumeric characters.
    example: "1Z999AA10123456784"

  TrackingNumberFedEx:
    type: string
    pattern: "^\\d{12,22}$"
    description: |
      FedEx tracking number. 12 to 22 digits.
    examples:
      twelve: "123456789012"
      fifteen: "123456789012345"

  TrackingNumberUSPS:
    type: string
    pattern: "^(94|93|92|94|95)\\d{18,22}$"
    description: |
      USPS tracking number. Starts with 92-95, followed by 18-22 digits.
    example: "9400111899223100001234"

  SKU:
    type: string
    pattern: "^[A-Z]{2,5}-[A-Z0-9]{4,12}(-[A-Z0-9]{2,6})?$"
    description: |
      Stock Keeping Unit. Category prefix, product code, optional variant.
      Format: CAT-PRODUCT[-VARIANT].
    examples:
      simple: "ELEC-IPHONE15PRO"
      with_variant: "SHOE-NKE4532-BLK42"
      short: "BK-A1B2C3"

  CronExpression:
    type: string
    pattern: "^(\\*|\\d{1,2}|\\d{1,2}-\\d{1,2}|\\d{1,2}/\\d{1,2}|\\d{1,2}(,\\d{1,2})*)\\s+(\\*|\\d{1,2}|\\d{1,2}-\\d{1,2}|\\d{1,2}/\\d{1,2}|\\d{1,2}(,\\d{1,2})*)\\s+(\\*|\\d{1,2}|\\d{1,2}-\\d{1,2}|\\d{1,2}/\\d{1,2}|\\d{1,2}(,\\d{1,2})*)\\s+(\\*|\\d{1,2}|\\d{1,2}-\\d{1,2}|\\d{1,2}/\\d{1,2}|\\d{1,2}(,\\d{1,2})*)\\s+(\\*|\\d{1,2}|\\d{1,2}-\\d{1,2}|\\d{1,2}/\\d{1,2}|\\d{1,2}(,\\d{1,2})*)$"
    description: |
      Standard 5-field cron expression: minute hour day-of-month month day-of-week.
      Supports: * (any), ranges (1-5), steps (*/5), lists (1,3,5).
    examples:
      every_minute: "* * * * *"
      daily_10am: "0 10 * * *"
      weekdays_9am: "0 9 * * 1-5"
      every_5min: "*/5 * * * *"

  HexColor:
    type: string
    pattern: "^#([0-9a-fA-F]{6}|[0-9a-fA-F]{3})$"
    description: |
      CSS hex color code. # followed by 3 or 6 hex digits.
    examples:
      six_digit: "#FF5733"
      three_digit: "#F00"
      lowercase: "#00a0df"

  TagList:
    type: string
    pattern: "^[a-z0-9-]+(,[a-z0-9-]+)*$"
    description: |
      Comma-separated list of lowercase tags (no spaces).
      Each tag: lowercase alphanumeric and hyphens.
    examples:
      single: "api-design"
      multiple: "raml,best-practices,validation"

  Percentage:
    type: string
    pattern: "^(100(\\.0{1,2})?|\\d{1,2}(\\.\\d{1,2})?)$"
    description: |
      Percentage value 0.00 to 100.00 (up to 2 decimal places).
    examples:
      whole: "75"
      decimal: "33.33"
      max: "100.00"
      zero: "0"

  TimeZoneIANA:
    type: string
    pattern: "^[A-Z][a-zA-Z]+(/[A-Z][a-zA-Z_]+)+$"
    description: |
      IANA timezone identifier.
      Format: Region/City (e.g., America/New_York).
    examples:
      eastern: "America/New_York"
      utc: "Etc/UTC"
      tokyo: "Asia/Tokyo"
      rome: "Europe/Rome"
```

---

### Step 11 — Salesforce IDs

Salesforce uses 15 or 18-character IDs where the first three characters (the "Key Prefix") identify the object type. This makes regex validation uniquely powerful — you can enforce not just format but also **object type correctness** at the API contract level.

📄 `types/salesforce.raml`

```yaml
#%RAML 1.0 Library

usage: |
  Salesforce ID types with object-specific key prefix validation.
  Salesforce IDs are 15 (case-sensitive) or 18 (case-insensitive) characters.
  The first 3 characters identify the object type (Key Prefix).

types:
  # --- Generic Salesforce ID ---

  SalesforceId:
    type: string
    pattern: "^[a-zA-Z0-9]{15}([a-zA-Z0-9]{3})?$"
    description: |
      Generic Salesforce record ID (any object).
      Accepts both 15-character (case-sensitive) and 18-character (case-insensitive) formats.
      Use object-specific types below when the expected object type is known.
    examples:
      fifteen_char: "001000000000001"
      eighteen_char: "001000000000001AAA"

  SalesforceId18:
    type: string
    pattern: "^[a-zA-Z0-9]{18}$"
    description: |
      Salesforce 18-character ID only (case-insensitive).
      Recommended for API integrations to avoid case-sensitivity issues.
    example: "001000000000001AAA"

  # --- Standard Object IDs (by Key Prefix) ---

  SalesforceAccountId:
    type: string
    pattern: "^001[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Account record ID.
      Key Prefix: 001. Accepts 15 or 18 characters.
    examples:
      fifteen: "001xx000003DGb2"
      eighteen: "001xx000003DGb2AAG"

  SalesforceContactId:
    type: string
    pattern: "^003[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Contact record ID.
      Key Prefix: 003. Accepts 15 or 18 characters.
    examples:
      fifteen: "003xx000004TmiQ"
      eighteen: "003xx000004TmiQAAS"

  SalesforceCaseId:
    type: string
    pattern: "^500[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Case record ID.
      Key Prefix: 500. Accepts 15 or 18 characters.
    examples:
      fifteen: "500xx000001Svf0"
      eighteen: "500xx000001Svf0AAC"

  SalesforceOpportunityId:
    type: string
    pattern: "^006[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Opportunity record ID.
      Key Prefix: 006. Accepts 15 or 18 characters.
    examples:
      fifteen: "006xx000001Svf0"
      eighteen: "006xx000001Svf0AAC"

  SalesforceLeadId:
    type: string
    pattern: "^00Q[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Lead record ID.
      Key Prefix: 00Q. Accepts 15 or 18 characters.
    examples:
      fifteen: "00Qxx000001Svf0"
      eighteen: "00Qxx000001Svf0AAC"

  SalesforceUserId:
    type: string
    pattern: "^005[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce User record ID.
      Key Prefix: 005. Accepts 15 or 18 characters.
    examples:
      fifteen: "005xx000001Svf0"
      eighteen: "005xx000001Svf0AAC"

  SalesforceTaskId:
    type: string
    pattern: "^00T[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Task record ID.
      Key Prefix: 00T. Accepts 15 or 18 characters.
    example: "00Txx000003gHw0AAE"

  SalesforceEventId:
    type: string
    pattern: "^00U[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Event record ID.
      Key Prefix: 00U. Accepts 15 or 18 characters.
    example: "00Uxx000002HJKAAE"

  SalesforceProductId:
    type: string
    pattern: "^01t[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Product2 record ID.
      Key Prefix: 01t. Accepts 15 or 18 characters.
    example: "01txx0000006iIk"

  SalesforcePriceBookId:
    type: string
    pattern: "^01s[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Pricebook2 record ID.
      Key Prefix: 01s. Accepts 15 or 18 characters.
    example: "01sxx0000002LKT"

  SalesforceOrderId:
    type: string
    pattern: "^801[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Order record ID.
      Key Prefix: 801. Accepts 15 or 18 characters.
    example: "801xx000000001a"

  SalesforceCampaignId:
    type: string
    pattern: "^701[a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Campaign record ID.
      Key Prefix: 701. Accepts 15 or 18 characters.
    example: "701xx000000001a"

  SalesforceCustomObjectId:
    type: string
    pattern: "^a[0-9][0-9A-Za-z][a-zA-Z0-9]{12}([a-zA-Z0-9]{3})?$"
    description: |
      Salesforce Custom Object record ID.
      Custom object key prefixes start with 'a' followed by two alphanumeric characters.
      The exact prefix depends on the org and object definition order.
    examples:
      custom1: "a00xx000001Svf0"
      custom2: "a0Bxx000002ABCD"
```

> [!NOTE]
> **Key Prefix reference for common objects:**
>
> | Object | Prefix | | Object | Prefix |
> |--------|--------|-|--------|--------|
> | Account | `001` | | Lead | `00Q` |
> | Contact | `003` | | User | `005` |
> | Case | `500` | | Task | `00T` |
> | Opportunity | `006` | | Event | `00U` |
> | Product2 | `01t` | | Order | `801` |
> | Pricebook2 | `01s` | | Campaign | `701` |
> | Custom Object | `a__` | | | |
>
> You can find any object's prefix in Salesforce Setup → Object Manager → select object → Details → "Key Prefix" field.

> [!TIP]
> When building integrations with Salesforce, using object-specific ID types in your RAML spec prevents a common class of bugs — passing an Account ID where a Contact ID is expected. The API will reject the request at the gateway/validation layer, long before it reaches your Salesforce connector.

---

### Step 12 — MuleSoft and Anypoint Platform

📄 `types/mulesoft.raml`

```yaml
#%RAML 1.0 Library

usage: |
  MuleSoft and Anypoint Platform-specific identifier types.
  Use these for internal platform integrations, automation scripts,
  and admin APIs.

types:
  # --- Platform Identifiers ---

  AnypointOrgId:
    type: string
    pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    description: |
      Anypoint Platform Organization ID.
      UUID format (lowercase, hyphenated). Found in Access Management → Organization.
    example: "a1b2c3d4-e5f6-7890-abcd-ef1234567890"

  AnypointEnvironmentId:
    type: string
    pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    description: |
      Anypoint Platform Environment ID.
      UUID format. Found in Access Management → Environments.
    example: "f1e2d3c4-b5a6-7890-fedc-ba0987654321"

  AnypointClientId:
    type: string
    pattern: "^[0-9a-f]{32}$"
    description: |
      Anypoint Platform client application ID.
      32 lowercase hexadecimal characters (no hyphens).
      Used for client_id in OAuth2 client credentials and API autodiscovery.
    example: "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6"

  AnypointClientSecret:
    type: string
    pattern: "^[A-Za-z0-9]{32,64}$"
    description: |
      Anypoint Platform client application secret.
      Alphanumeric, typically 32 or 64 characters.
      WARNING: Treat as sensitive — never log or return in responses.
    example: "A1b2C3d4E5f6G7h8I9j0K1l2M3n4O5p6"

  # --- API & Asset Identifiers ---

  ExchangeAssetId:
    type: string
    pattern: "^[a-z][a-z0-9-]*[a-z0-9]$"
    minLength: 2
    maxLength: 64
    description: |
      Anypoint Exchange asset identifier (artifact ID).
      Lowercase alphanumeric and hyphens, starts with letter, ends with alphanumeric.
    examples:
      api: "order-management-api"
      connector: "salesforce-connector"
      fragment: "common-types-library"

  ExchangeGroupId:
    type: string
    pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$"
    description: |
      Exchange group ID (matches the Organization ID).
    example: "a1b2c3d4-e5f6-7890-abcd-ef1234567890"

  ExchangeVersion:
    type: string
    pattern: "^(0|[1-9]\\d*)\\.(0|[1-9]\\d*)\\.(0|[1-9]\\d*)(-SNAPSHOT)?$"
    description: |
      Anypoint Exchange asset version.
      Semantic versioning with optional -SNAPSHOT suffix for development.
    examples:
      release: "1.2.3"
      snapshot: "2.0.0-SNAPSHOT"

  ApiInstanceId:
    type: string
    pattern: "^\\d{5,10}$"
    description: |
      API Manager instance ID (numeric).
      Found in API Manager → API instance details.
    example: "18436572"

  ApiAutoDiscoveryId:
    type: string
    pattern: "^\\d{5,10}$"
    description: |
      API autodiscovery ID used in Mule application properties.
      Same format as API instance ID.
    example: "18436572"

  # --- Runtime & Deployment ---

  MuleAppName:
    type: string
    pattern: "^[a-z][a-z0-9-]{1,40}[a-z0-9]$"
    description: |
      CloudHub 2.0 / Runtime Fabric application name.
      Lowercase, starts with letter, ends with alphanumeric, hyphens allowed.
      3-42 characters total.
    examples:
      simple: "order-api"
      env_prefix: "dev-order-management-sapi"

  CloudHubDomain:
    type: string
    pattern: "^[a-z][a-z0-9-]{1,61}[a-z0-9]\\.([a-z0-9-]+\\.)?cloudhub\\.io$"
    description: |
      CloudHub 1.0 application domain (FQDN).
      Format: appname[.region].cloudhub.io.
    examples:
      default: "my-order-api.cloudhub.io"
      regional: "my-order-api.us-e1.cloudhub.io"

  RuntimeFabricTarget:
    type: string
    pattern: "^[a-z][a-z0-9-]{1,62}[a-z0-9]$"
    description: |
      Runtime Fabric target/cluster name.
    example: "prod-rtf-cluster-east"

  # --- DataWeave & Configuration ---

  DataWeaveVersion:
    type: string
    pattern: "^\\d+\\.\\d+$"
    description: |
      DataWeave language version.
    examples:
      current: "2.0"
      legacy: "1.0"

  MuleRuntimeVersion:
    type: string
    pattern: "^4\\.(\\d+)\\.(\\d+)(-hf(\\d+))?$"
    description: |
      Mule Runtime 4.x version with optional hotfix.
      Format: 4.MINOR.PATCH[-hfN].
    examples:
      standard: "4.6.0"
      with_hotfix: "4.4.0-hf2"
      latest: "4.9.0"

  MavenCoordinates:
    type: string
    pattern: "^[a-zA-Z0-9._-]+:[a-zA-Z0-9._-]+:(0|[1-9]\\d*)\\.(0|[1-9]\\d*)\\.(0|[1-9]\\d*)(-SNAPSHOT)?$"
    description: |
      Maven GAV coordinates (groupId:artifactId:version).
    examples:
      mule_connector: "com.mulesoft.connectors:mule-salesforce-connector:10.18.0"
      custom: "com.example:order-api:1.0.0-SNAPSHOT"

  # --- Monitoring & Logging ---

  AnypointTraceId:
    type: string
    pattern: "^[0-9a-f]{32}$"
    description: |
      Anypoint Monitoring / Titanium trace ID (32 hex chars, W3C format).
    example: "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6"

  AnypointSpanId:
    type: string
    pattern: "^[0-9a-f]{16}$"
    description: |
      Anypoint Monitoring span ID (16 hex chars, W3C format).
    example: "a1b2c3d4e5f6a7b8"

  WorkerLogLevel:
    type: string
    pattern: "^(TRACE|DEBUG|INFO|WARN|ERROR|FATAL|OFF)$"
    description: |
      Log4j2 log level for Mule worker configuration.
    examples:
      production: "WARN"
      debug: "DEBUG"
```

> [!TIP]
> When building Platform APIs (admin tools, automation scripts, CI/CD pipelines), these MuleSoft-specific types prevent common mistakes like passing an Environment ID where an Org ID is expected — both are UUIDs, but context matters!

---

### Step 13 — Assembling and Consuming the Library

Now let's see how to import and use the library in an actual API specification.

**Publishing to Exchange:**

Create the `exchange.json` file for your fragment. This file is **required** by Design Center and must include the full asset coordinates:

```json
{
  "main": "common-types-library.raml",
  "name": "Common Types Library",
  "groupId": "your-org-id",
  "assetId": "common-types-library",
  "version": "1.0.0",
  "classifier": "raml-fragment",
  "tags": ["types", "validation", "regex", "patterns"]
}
```

> [!IMPORTANT]
> The `groupId`, `assetId`, and `version` fields are mandatory. Without them, Design Center will reject the fragment with validation errors. The `groupId` must match your Anypoint Organization ID (found in Access Management → Organization → Settings).

Publish using the Exchange Maven plugin or the Anypoint CLI:

```bash
anypoint-cli exchange asset upload \
  --organization "your-org-id" \
  --name "Common Types Library" \
  --classifier raml-fragment \
  common-types-library/1.0.0 common-types-library.zip
```

**Consuming from an API specification:**

Before you can reference the library types, you need to add the fragment as a dependency in your Design Center project:

1. Open your API specification project in **Anypoint Design Center**
2. In the left panel, click the **+** button (Add Dependencies) under **Exchange Dependencies**
3. Search for **"Common Types Library"** in the search box
4. Select the library and click **Add**
5. Design Center downloads the fragment into the `exchange_modules/` folder automatically

> [!NOTE] 📸 **Screenshot needed:** Design Center left panel showing the Exchange Dependencies section with the "Add" button and the search dialog for adding a fragment.

Once the dependency is added, you will see the fragment appear under `exchange_modules/{orgId}/common-types-library/1.0.0/` in the project file tree.

**Using the library in your spec:**

Import the main library file and reference types with single dot notation — `Common.TypeName`:

```yaml
#%RAML 1.0

title: Order Management API
version: v1
baseUri: https://{environment}.api.example.com/{version}

uses:
  Common: exchange_modules/your-org-id/common-types-library/1.0.0/common-types-library.raml

baseUriParameters:
  environment:
    type: string
    pattern: "^(dev|staging|prod)$"
  version:
    type: Common.ApiVersionPrefix

types:
  Order:
    type: object
    properties:
      id: Common.UUID
      customerId: Common.SalesforceAccountId
      contactId: Common.SalesforceContactId
      status:
        type: string
        pattern: "^(pending|confirmed|shipped|delivered|cancelled)$"
      createdAt: Common.ISODateTime
      updatedAt: Common.ISODateTime
      currency: Common.CurrencyCode
      amount: Common.MonetaryAmount
      trackingNumber?: Common.TrackingNumberUPS

/orders:
  get:
    description: List orders with filters
    headers:
      X-Correlation-Id: Common.CorrelationId
      X-Client-Id: Common.AnypointClientId
      Authorization: Common.BearerToken
    queryParameters:
      dateFrom?:
        type: Common.ISODate
        description: Filter from this date
      dateTo?:
        type: Common.ISODate
        description: Filter until this date
      page?:
        type: string
        pattern: "^[1-9]\\d*$"
    responses:
      200:
        body:
          application/json:
            type: Order[]

  /{orderId}:
    uriParameters:
      orderId: Common.UUID
    get:
      description: Get a specific order
      responses:
        200:
          body:
            application/json:
              type: Order
```

> [!NOTE]
> All types are accessed with a single dot (`Common.UUID`, `Common.ISODate`, `Common.SalesforceAccountId`). Internally, the main library re-exports every type from its domain sub-files at the top level. This avoids two RAML 1.0 limitations:
> - **Chained references** (`Common.Identifiers.UUID`) are not supported
> - **Non-root file references** (importing `types/identifiers.raml` directly) trigger Design Center warnings
>
> The type names are unique across all domains, so there are no collisions.

> [!TIP]
> The `uses` alias is up to you — `Common`, `Types`, `Lib`, or even just `T`. Pick a short name that reads well in your type references: `T.UUID`, `Types.ISODate`, etc.

---

## Verification

After assembling the library, verify everything works end-to-end:

| Check | Expected Result |
|-------|-----------------|
| Each `.raml` file parses independently | No syntax errors when opened in API Designer |
| Main library resolves all `uses` references | No "cannot resolve" warnings |
| All `examples` pass their own `pattern` | No validation errors on example values |
| Consuming API resolves library types | Dot-notation references show type details on hover |
| Mocking service validates patterns | Sending invalid data returns 400 Bad Request |
| Exchange publish succeeds | Fragment visible in org Exchange with correct metadata |
| Multiple APIs can import the same version | No conflicts or resolution errors |

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| "Cannot resolve reference" in consuming API | Wrong exchange_modules path or version | Verify asset GAV coordinates in Exchange |
| Pattern works in isolation but fails in library | Missing double-escape in YAML (`\d` vs `\\d`) | Ensure all backslashes are doubled |
| Type not accessible via dot notation | Missing or misspelled `uses` key in main library | Check namespace aliases match file names |
| Exchange upload rejects the fragment | Invalid `exchange.json` or missing `main` entry | Ensure `classifier` is `raml-fragment` and `groupId`, `assetId`, `version` are present |
| "Chained reference" errors in Design Center | Using double dot notation (`Common.Identifiers.UUID`) | RAML 1.0 does not support chained refs — import the main library which re-exports all types flat (`Common.UUID`) |
| Pattern allows values outside expected range | Regex validates format, not semantics | Add `minLength`/`maxLength` or `enum` constraints alongside pattern |
| Salesforce ID type rejects valid record | Key prefix is case-sensitive (`00Q` vs `00q`) | Verify the prefix matches Salesforce's actual prefix for that object |
| API Designer shows "pattern too complex" | Catastrophic backtracking in nested quantifiers | Simplify alternations, avoid nested `(a+)+` patterns |
| Library version conflict between APIs | Two APIs importing different versions | Align on a single version; use SemVer for backward-compatible updates |

---

## Summary

We built a **production-ready RAML Library fragment** containing 50+ regex-validated types across 11 domains:

- **Identifiers**: UUID, ULID, Slug, prefixed IDs, correlation IDs
- **Dates & Times**: ISO dates, timestamps with milliseconds/offsets, durations
- **Contact**: Email, phone (E.164, NANP), usernames
- **Geography**: Country codes, languages, locales, coordinates, postal codes for 10 countries
- **Financial**: Currencies, amounts, masked cards, IBAN, BIC, tax IDs
- **Network**: IPv4/IPv6 with CIDR, hostnames, ports, MAC addresses, URLs
- **Security**: Bearer tokens, JWTs, API keys, hashes, Base64
- **Versioning**: SemVer, API versions, MIME types, HTTP methods
- **Business**: Order/invoice IDs, tracking numbers, SKUs, cron, colors
- **Salesforce**: Object-specific IDs (Account, Contact, Case, Opportunity, Lead, User, and more)
- **MuleSoft**: Org/Environment IDs, client credentials, app names, Exchange assets, runtime versions

The key benefits of this approach:

1. **Define once, reuse everywhere** — Single source of truth for validation patterns
2. **Shift validation left** — Catch format errors at the API gateway, not in business logic
3. **Self-documenting** — Types carry descriptions and examples alongside patterns
4. **Type safety across integrations** — Salesforce Account IDs cannot be confused with Contact IDs

Publish this library to Exchange, pin it to a version, and import it from every API spec in your organization. When patterns need updating, release a new minor version and let teams adopt at their own pace.

---

## References

- [RAML 1.0 Specification — Libraries](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#libraries)
- [RAML 1.0 Specification — String Type Pattern](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#string)
- [Anypoint Exchange — Publishing RAML Fragments](https://docs.mulesoft.com/exchange/to-create-an-asset)
- [Anypoint API Designer — Using Fragments](https://docs.mulesoft.com/design-center/design-create-publish-api-fragment)
- [Salesforce Object Key Prefixes](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_objects_list.htm)
- [RFC 4122 — UUID Format](https://www.rfc-editor.org/rfc/rfc4122)
- [ISO 8601 — Date and Time Format](https://www.iso.org/iso-8601-date-and-time-format.html)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [E.164 Phone Number Format](https://www.itu.int/rec/T-REC-E.164)
- [IANA Media Types Registry](https://www.iana.org/assignments/media-types/)
