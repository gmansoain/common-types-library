# Common Types Library — RAML Fragment

![RAML 1.0](https://img.shields.io/badge/RAML-1.0-00A0DF?logo=mulesoft&logoColor=white)
![Anypoint Exchange](https://img.shields.io/badge/Exchange-Fragment-00A0DF?logo=mulesoft&logoColor=white)
![Types](https://img.shields.io/badge/Types-50+-2ecc71)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow)

A **reusable RAML Library fragment** containing 50+ regex-validated string types, organized by domain. Publish to Anypoint Exchange once, import from every API spec in your organization.

## Why?

- **Define once, reuse everywhere** — Single source of truth for validation patterns
- **Shift validation left** — Catch format errors at the API gateway, not in business logic
- **Self-documenting** — Types carry descriptions and examples alongside patterns
- **Type safety across integrations** — A Salesforce Account ID cannot be confused with a Contact ID

## Quick Start

### Import from Exchange

After [publishing to Exchange](#publishing-to-exchange), add the fragment as a dependency in Design Center (Exchange Dependencies → Add → search "Common Types Library"), then import the main library file:

```yaml
#%RAML 1.0

title: My API
version: v1

uses:
  Common: exchange_modules/{orgId}/common-types-library/1.0.0/common-types-library.raml

types:
  Order:
    type: object
    properties:
      id: Common.UUID
      createdAt: Common.ISODateTime
      currency: Common.CurrencyCode
      customerId: Common.SalesforceAccountId
```

All 127 types are accessible via single dot notation (`Common.TypeName`). The main library re-exports every type from the domain sub-files at the top level, avoiding RAML 1.0's chained reference and non-root dependency limitations.

## Project Structure

```
common-types-library/
├── common-types-library.raml      ← Main library entry point
├── exchange.json                  ← Exchange publishing metadata
├── types/
│   ├── identifiers.raml           ← UUID, ULID, Slug, PrefixedId, CorrelationId
│   ├── dates.raml                 ← ISODate, ISODateTime, Duration, YearMonth
│   ├── contact.raml               ← Email, Phone (E.164, NANP), Username
│   ├── geography.raml             ← Country codes, Locales, Coordinates, Postal codes (12 countries)
│   ├── financial.raml             ← Currency, Amounts, IBAN, BIC, Masked Cards
│   ├── network.raml               ← IPv4/6, CIDR, Hostname, Ports, MAC, URLs
│   ├── security.raml              ← Bearer, JWT, API Keys, Hashes, Base64
│   ├── versioning.raml            ← SemVer, API versions, MIME types, HTTP methods
│   ├── business.raml              ← Order IDs, Tracking, SKU, Cron, HexColor
│   ├── salesforce.raml            ← Object-specific IDs (Account, Contact, Case, etc.)
│   └── mulesoft.raml              ← Anypoint Org/Env IDs, Client credentials, App names
├── blog-part1-regex-fundamentals.md
├── blog-part2-building-type-library.md
└── README.md
```

## Type Domains

| Domain | File | Types | Highlights |
|--------|------|-------|------------|
| **Identifiers** | `identifiers.raml` | 6 | UUID (3 variants), ULID, Slug, PrefixedId, CorrelationId |
| **Dates & Times** | `dates.raml` | 9 | ISODate, DateTime (UTC/offset/millis), Duration, YearMonth, UnixTimestamp |
| **Contact** | `contact.raml` | 5 | Email, PhoneE164, PhoneNANP, Username, PersonName |
| **Geography** | `geography.raml` | 23 | Country codes (alpha-2/3/numeric), Languages, Locales, Lat/Long, 12 postal code formats |
| **Financial** | `financial.raml` | 9 | CurrencyCode, MonetaryAmount (3 variants), IBAN, BIC, CreditCardMasked, TaxId |
| **Network** | `network.raml` | 13 | IPv4/IPv6 + CIDR, Hostname, PortNumber, HostPort, MacAddress, HTTP/HTTPS URLs |
| **Security** | `security.raml` | 11 | BearerToken, JWT, ApiKey (prefixed/simple), Hashes (SHA256/MD5), Base64/Base64URL |
| **Versioning** | `versioning.raml` | 9 | SemanticVersion (full/strict), ApiVersion, MimeType, HttpMethod, HttpStatusCode, ETag |
| **Business** | `business.raml` | 11 | OrderId, InvoiceNumber, TrackingNumbers (UPS/FedEx/USPS), SKU, Cron, HexColor, Timezone |
| **Salesforce** | `salesforce.raml` | 14 | Generic + Account, Contact, Case, Opportunity, Lead, User, Task, Event, Product, Order, Campaign, Custom |
| **MuleSoft** | `mulesoft.raml` | 17 | OrgId, EnvId, ClientId/Secret, ExchangeAsset, AppName, CloudHub domain, Runtime versions, Trace/Span IDs |

## Salesforce Key Prefix Reference

The Salesforce types validate object-specific IDs by checking the 3-character key prefix:

| Object | Prefix | Type Name |
|--------|--------|-----------|
| Account | `001` | `SalesforceAccountId` |
| Contact | `003` | `SalesforceContactId` |
| Case | `500` | `SalesforceCaseId` |
| Opportunity | `006` | `SalesforceOpportunityId` |
| Lead | `00Q` | `SalesforceLeadId` |
| User | `005` | `SalesforceUserId` |
| Task | `00T` | `SalesforceTaskId` |
| Event | `00U` | `SalesforceEventId` |
| Product2 | `01t` | `SalesforceProductId` |
| Pricebook2 | `01s` | `SalesforcePriceBookId` |
| Order | `801` | `SalesforceOrderId` |
| Campaign | `701` | `SalesforceCampaignId` |
| Custom Object | `a__` | `SalesforceCustomObjectId` |


## Publishing to Exchange

### Using Anypoint CLI

```bash
anypoint-cli exchange asset upload \
  --organization "your-org-id" \
  --name "Common Types Library" \
  --classifier raml-fragment \
  common-types-library/1.0.0 .
```

### Using Maven

Add the Exchange Maven plugin to a `pom.xml`:

```xml
<project>
  <groupId>YOUR_ORG_ID</groupId>
  <artifactId>common-types-library</artifactId>
  <version>1.0.0</version>
  <packaging>raml-fragment</packaging>

  <build>
    <plugins>
      <plugin>
        <groupId>org.mule.tools.maven</groupId>
        <artifactId>exchange-mule-maven-plugin</artifactId>
        <version>0.0.22</version>
        <executions>
          <execution>
            <id>validate</id>
            <phase>validate</phase>
            <goals>
              <goal>exchange-pre-deploy</goal>
            </goals>
          </execution>
          <execution>
            <id>deploy</id>
            <phase>deploy</phase>
            <goals>
              <goal>exchange-deploy</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

Then run:

```bash
mvn deploy -DskipTests
```

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| **Format-only validation** | Patterns validate structure, not semantics (e.g., `2026-02-31` passes ISODate). Calendar/business rules belong in implementation. |
| **YAML double-escaping** | All `\d`, `\w`, etc. are written as `\\d`, `\\w` because YAML interprets single backslashes. |
| **Anchors on every pattern** | All patterns use `^...$` to enforce full-string matching. |
| **15/18-char Salesforce IDs** | Both formats accepted via `([a-zA-Z0-9]{3})?$` suffix to support legacy and modern systems. |
| **Split by domain** | Consumers can import the full library or individual domain files, keeping API specs lean. |

## Blog Posts

This library is accompanied by a two-part tutorial series:

1. **[Part 1: Regex Fundamentals for API Designers](./blog-part1-regex-fundamentals.md)** — Learn regex from scratch with RAML examples
2. **[Part 2: Building a Regex-Validated Type Library in RAML](./blog-part2-building-type-library.md)** — Build this library step by step

## Contributing

1. Add your type to the appropriate domain file in `types/`
2. Follow the existing format: `type`, `pattern`, `description`, `examples`
3. Always use `^...$` anchors
4. Always double-escape backslashes (`\\d` not `\d`)
5. Include at least one example that passes the pattern
6. Update this README's type count and domain table

## License

MIT
