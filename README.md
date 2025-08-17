<p align="center">
  <img src="ums_logo_200.png" alt="Project Logo" width="200"/>
</p>

# Unified Meta Schema (UMS)

The Unified Meta Schema (UMS) is a technology-agnostic specification that bridges diverse data modeling paradigms through a single, compact representation format. UMS unifies concepts from ontologies, database schemas, dataset schemas, JSON Schema, GraphQL, OpenAPI, and other modeling frameworks into a coherent YAML-based syntax.

By standardizing core modeling concepts — `{schema}`, `{entity}`, `{attribute}`, `{relation}`, `{data_type}`, `{target}`, `{min}`, and `{max}` — UMS enables seamless transformation, validation, and interoperability between previously incompatible schema ecosystems.

## Scope

This specification:

- Defines a concise YAML structure for expressing entities, their attributes, and relations
- Provides explicit representation of datatypes and cardinality constraints
- Supports metadata annotations at schema and entity levels
- Enables bidirectional relation mapping with rich target expression syntax
- Facilitates transformation between different schema languages and paradigms
- Serves as both a documentation format and an executable specification

## Purpose & Benefits

UMS addresses the "Tower of Babel" problem in data modeling by:

- **Reducing Integration Complexity**: Convert between schema types without custom translators
- **Preserving Semantic Richness**: Maintain constraints and relationships across transformations
- **Enabling Knowledge Transfer**: Apply patterns across traditionally siloed domains
- **Simplifying Documentation**: Document diverse systems using consistent terminology
- **Supporting Schema Evolution**: Track changes consistently across technologies


## Specification

This specification defines how UMS uses YAML syntax to represent schema structures across multiple modeling paradigms.

### Structure
```
{schema}
  ├── {@metadata}
  └── {entity}
       ├── {@metadata}
       ├── {attribute}: {data_type}
       └── {relation}:  {target}
```
**Note:** Curly brackets `{variable}` are used to denote template variables throughout the document.

### Syntax
```yaml
{schema}:
  "{@metadata}": "..."
  {entity}:
    "{@metadata}": "..."
    {attribute}: {data_type} [{min},{max}] ({constraint}) | {description}
    {relation}:  {target}    [{min},{max}] ({constraint}) | {description}
````
**Note:**
* Use 2 spaces per indentation level (no tabs).
* Add always a space after colons (`key: value`, not `key:value`).
* Simple identifiers and data types (e.g. `Book`, `title`, `uuid`, `decimal`) can remain unquoted.
* Quote keys and values containing special characters (`@`, `:`, `/`, `#`, or spaces).
* Prefer double quotes as the default for safety; use single quotes only for verbatim strings.
* Indicate (directed) relations and with the symbols `->` (forward) and `<-` (backward).

### Schema Elements
The following YAML keys define the accepted schema elements in UMS:

| Element | Description | Required | Example |
|---------|-------------|:--------:|---------|
| **{schema}** | The schema or namespace | Yes | `bookstore` |
| **{entity}** | The entity, class, or table name | Yes | `Book`, `Author` |
| **{attribute}** | A property with a data type | * | `title: string`, `price: decimal` |
| **{relation}** | A link to another entity | * | `author: -> Author`, `books: <- Book.author` |

*\* Each entry must be either an attribute with a data type OR a relation with a target, never both.*

**Syntax:**
```yaml
{schema}:
  {entity}:
    {attribute}: ...
    {relation}:  ...
```

### Definition
Each `{attribute}` and `{relation}` key MUST have a YAML value that specifies its definition.

| Element         | Description                               | Notation           | Order | Required              | Example                 |
|-----------------|-------------------------------------------|--------------------|-------|-----------------------|-------------------------|
| **{data_type}** | The type for attributes                   | type name          | 1     | Yes (for attributes)  | `string`                |
| **{target}**    | The target entity for relations           | `->` or `<-`       | 1     | Yes (for relations)   | `-> Author`             |
| **{min}/{max}** | Minimum and maximum cardinality           | `[min,max]`        | 2     | No                    | `[1,1]`, `[0,*]`        |
| **{constraint}**| Labels or key-value pairs in parentheses  | `( … )`            | 3     | No                    | `(pk)`                  |
| **{description}** | Human-readable description              | `\|` (pipe)        | 4     | No                    | `\| Book title`         |

**Syntax:**
```yaml
    {attribute}: {data_type} [{min},{max}] ({constraint}) | {description}
    {relation}:  {target}    [{min},{max}] ({constraint}) | {description}
```
**Note:**
* Additional whitespace MAY be used within a YAML value for readability; it is not significant for parsing.
* Write YAML values without quotes for readability, and add quotes when needed.

#### Parsing Logic

A YAML value MUST be parsed from **right to left**: first strip the `| description` (if present), then the `(constraint)`, then the `[min,max]`. The remaining head is interpreted as `{data_type}` (for attributes) or `{target}` (for relations). Syntactically, the order of elements is left-to-right. Parsing SHOULD proceed right-to-left for deterministic extraction.

### Example

```yaml
bookstore:
  "@title": "Book Shop"
  
  Book:
    id:        uuid    [1,1] (pk)       | Unique identifier
    title:     string  [1,1]            | Book title
    price:     decimal [1,1]            | Retail price
    author:    -> Author [1,*]          | Book author(s)
    
  Author:
    id:        uuid    [1,1] (pk)       | Unique identifier
    name:      string  [1,1]            | Author's name
    books:     <- Book.author [0,*]     | Books by this author
```

### Naming Conventions
UMS supports multiple naming conventions to accommodate different programming and modeling traditions. Choose a consistent style throughout your schema:

* **PascalCase**: Capitalize first letter of each word, no separators
* **camelCase**: First word lowercase, capitalize subsequent words
* **snake_case**: All lowercase with underscores between words
* **kebab-case**: All lowercase with hyphens between words

**Note:** Be consistent within a single schema for better readability.

### Metadata
* Optionally declare metadata at the `{schema}` or `{entity}` level using quoted `@`-prefixed keys (e.g., `"@id"` or `"@title"`).
* Metadata values can be single or nested values (e.g.,  `"@id": "123"` or `"@prefix": {...}`).
* Examples: `@id`, `@name`, `@description`, `@version`, `@prefixes`, `@creator`, `@creation_date`, `@update_date`

**Syntax:**
```yaml
{schema}:
  "{@metadata}": "..."
  {entity}:
    "{@metadata}": "..."
````

### Data Types

* Always specify data types for attributes.

#### Enums
Attributes MAY declare an enumerated type using the enum(...) constructor, where the allowed values are listed inside parentheses (comma-separated, no spaces):
```yaml
    {attribute}: enum(value1,value2,…) [min,max] (constraint) | description
```

#### Multiple Types
Attributes MAY allow a union of multiple data types (comma-separated, no spaces):
```yaml
    {attribute}: {data_type1},{data_type1}
```

### Target Syntax

The `{target}` MUST resolve unambiguously to an entity or one of its attributes using dot-notation.
Each relation MUST specify a direction symbol:

* `->` for forward relations
* `<-` for reverse (inverse) relations

#### Dot-Notation
Dot-notation is used to qualify names across different scopes:

* **`{entity}.{attribute}`** — reference within a schema, scoped to an entity
* **`{schema}.{entity}.{attribute}`** — fully qualified reference across schema, entity, and attribute

#### Entity Relation

Reference to the entire entity:

```yaml
    {relation}: -> {entity}
```

#### Union Relation

A union relation allows a single relation to target multiple entities. Entities (and optionally their attributes) MUST be listed comma-separated without spaces.

```yaml
    {relation}: -> {entity1},{entity2}
    {relation}: -> {entity1}.{attr1},{entity2}.{attr1}
```


#### Qualified Relation

Reference to a specific attribute of the entity (typically a primary/foreign key):

```yaml
    {relation}: {attr} -> {entity}.{attr}
```

#### Composite Relation

Reference to multiple attributes in the relation:

```yaml
    {relation}: {attr1},{attr2} -> {entity}.{attr1},{attr2}
```

**Note:**
* Composite relations list multiple entity attributes separated by commas (no spaces).  

#### External Relation

Fully qualified reference across schema, entity, and attribute:

```yaml
    {relation}: {attr} -> {schema}.{entity}.{attr}
```


#### Composite External Relation

Fully qualified reference with multiple attributes:

```yaml
    {relation}: {attr1},{attr2} -> {schema}.{entity}.{attr1},{attr2}
```

### Cardinality

Cardinality specifies the minimum and maximum number of values allowed.

* Cardinality is written as `[{min},{max}]` (whitespace allowed).
* `{min}` is the minimum number of values; `{max}` is the maximum (or `*` for unbounded).
* If cardinality is omitted, the default is `[0,*]` (optional, unbounded).
* Keyword shorthands MAY be used as aliases for common numeric patterns.

**Keywords:**

| Numeric | Meaning         | Keywords / Shorthand          |
| ------- | --------------- | ----------------------------- |
| `[1,1]` | exactly one     | `[required]`, `[mandatory]`   |
| `[0,1]` | zero or one     | `[optional]`                  |
| `[0,*]` | zero or more    | `[optional*]`                 |
| `[1,*]` | one or more     | `[required*]`, `[mandatory*]` |
| `[n,m]` | between n and m | *(no shorthand)*              |

### Constraints

Constraints specify additional rules that refine the values of attributes or relations.

* They are written in parentheses `( … )` immediately after the data type or target.
* Multiple constraints MAY be comma-separated; whitespace is ignored.

| Constraint       | Meaning                                    | Example                                        |
| ---------------- | ------------------------------------------ | ---------------------------------------------- |
| `(pk)`           | Primary key (implies required + unique)    | `id: uuid [1,1] (pk)`                          |
| `(fk)`           | Foreign key / reference to another entity  | `author_id: uuid [1,1] (fk)`                   |
| `(unique)`       | Value must be distinct across all entities | `email: string [1,1] (unique)`                 |
| `(index)`        | Marks attribute for indexing            )  | `id: uuid [1,1] (index)`                       |
| `(default=…)`    | Default value if none provided             | `created: date [1,1] (default=now())`          |
| `(pattern=…)`    | Regex or format pattern constraint         | `code: string [1,1] (pattern=^[A-Z]{2}\d{4}$)` |
| `(check=…)`      | General condition expression               | `age: int [0,1] (check=>=0)`                   |
| `(min=… ,max=…)` | Numeric or length boundaries               | `qty: int [0,1] (min=1,max=100)`               |

**Note:**
* Nullability MUST be expressed via cardinality (`[1,1]`, `[0,1]`, etc.), not as a constraint (e.g., `(not null)`).


### Description

* Optionally add human-readable descriptions after `|`.

### Prefixes

Prefixes provide namespace abbreviations for IRIs and MAY be used anywhere an identifier is expected.

* Prefixes MUST be declared at the `{schema}` level using `@prefix`.
* Prefixed names MAY be used in `{schema}`, `{entity}`, `{attribute}`, `{relation}`, `{data_type}`, and `{target}`, including in `{target}` dot-notation.
* Prefixes MUST be expanded to absolute IRIs before validation and before parsing cardinality/constraints.
* A colon (`:`) MUST directly follow the prefix alias (e.g., `ex:Person`, not `ex: Person`).
* Keys containing `/` or `#` in YAML SHOULD be quoted (e.g., `"http:Thing"`). Quoting is RECOMMENDED when unsure.

**Syntax:**

```yaml
  "@prefix":
    {alias}: {link}
```

**Example:**

```yaml
  "@prefix":
    ex:  "https://example.org/onto#"
    xsd: "http://www.w3.org/2001/XMLSchema#"
  
  "ex:Customer":
    customerId:  xsd:string [1,1] (pk) | Customer identifier
    orders:      -> ex:Order [0,*]     | All orders of this customer
```

### Alignment Rules

* Add whitespace after `:` so the first character of `{data_type}` / `{target}` starts at a single global column across the file.

### JSON Representation

```json
{
  "bookstore": {
    "Book": {
      "id": "uuid [1,1] (pk) | Unique identifier",
      "title": "string [1,1] | Book title",
      "price": "decimal [1,1] | Retail price",
      "author": "-> Author [1,*] | Book author(s)"
    },
    "Author": {
      "id": "uuid [1,1] (pk) | Unique identifier",
      "name": "string [1,1] | Author's name",
      "books": "<- Book.author [0,*] | Books by this author"
    }
  }
}
```

**Note:**
* In JSON representation, all values are quoted strings; this preserves the full UMS value expression.

### Table Representation

| schema    | entity | attribute | relation | data\_type | target      | min | max | constraint | description          |
| --------- | ------ | --------- | -------- | ---------- | ----------- | --- | --- | ---------- | -------------------- |
| bookstore | Book   | id        |          | uuid       |             | 1   | 1   | pk         | Unique identifier    |
| bookstore | Book   | title     |          | string     |             | 1   | 1   |            | Book title           |
| bookstore | Book   | price     |          | decimal    |             | 1   | 1   |            | Retail price         |
| bookstore | Book   |           | author   |            | Author      | 1   | \*  |            | Book author(s)       |
| bookstore | Author | id        |          | uuid       |             | 1   | 1   | pk         | Unique identifier    |
| bookstore | Author | name      |          | string     |             | 1   | 1   |            | Author’s name        |
| bookstore | Author |           | books    |            | Book.author | 0   | \*  |            | Books by this author |

**Note:** In each row, exactly one of *(attribute & data\_type)* or *(relation & target)* is populated.


### Schema Types

#### **Semantic Web & Knowledge Graphs**

| Schema Type  | `{schema}`           | `{entity}`     | `{attribute}`                               | `{relation}`                            | `{data_type}` examples       | `{target}` examples     |
| ------------ | ------------------- | -------------- | ------------------------------------------- | --------------------------------------- | ---------------------------- | ----------------------- |
| **Ontology** | ontology name / IRI | `owl:Class`    | `owl:DatatypeProperty`                      | `owl:ObjectProperty`                    | `xsd:string`, `xsd:dateTime` | `ex:Person`, `ex:Order` |
| **RDFS**     | ontology IRI        | `rdfs:Class`   | `rdf:Property` with `rdfs:range` (datatype) | `rdf:Property` with class range         | `xsd:string`, `xsd:dateTime` | `ex:Person`, `ex:Order` |
| **SHACL**    | shapes graph IRI    | `sh:NodeShape` | `sh:property` with `sh:datatype`            | `sh:property` with `sh:node`/`sh:class` | `xsd:string`, `xsd:integer`  | target `ex:Customer`    |

---

#### **Databases & Datasets**

| Schema Type  | `{schema}`            | `{entity}` | `{attribute}` | `{relation}`           | `{data_type}` examples       | `{target}` examples   |
| ------------ | -------------------- | ---------- | ------------- | ---------------------- | ---------------------------- | --------------------- |
| **Database** | database schema name | table      | column        | foreign key / relation | `uuid`, `varchar`, `integer` | `Customer`, `Order`   |
| **Dataset**  | dataset schema name  | table      | column        | relation / join        | `string`, `date`, `decimal`  | `Customer`, `Product` |

---

#### **Data Serialization & Storage Formats**

| Schema Type          | `{schema}`                | `{entity}`                       | `{attribute}`                              | `{relation}`                     | `{data_type}` examples                     | `{target}` examples               |
| -------------------- | ------------------------ | -------------------------------- | ------------------------------------------ | -------------------------------- | ------------------------------------------ | --------------------------------- |
| **Avro**             | Avro namespace or schema | record                           | field                                      | relation (record ref)            | `string`, `long`, `bytes`                  | `Customer`, `Order`               |
| **XSD (XML Schema)** | schema namespace         | `complexType` / global `element` | `xs:attribute` or simple-content `element` | `element` with `@ref` / `keyref` | `xsd:string`, `xsd:date`, `xsd:int`        | `CustomerType`, `OrderType`       |
| **Protobuf**         | `package`                | `message`                        | scalar `field`                             | `field` of `message` type        | `int32`, `string`, `bytes`, `bool`         | `Customer`, `Order`               |
| **Parquet**          | file schema              | group (logical struct)           | primitive field                            | group-typed field                | `INT32`, `DOUBLE`, `BYTE_ARRAY`, `BOOLEAN` | nested `group` (e.g., `Customer`) |
| **Thrift IDL**       | `namespace`              | `struct`                         | field with base type                       | field with `struct`/`union` type | `i32`, `string`, `bool`, `binary`          | `Customer`, `Order`               |

---

#### **APIs & Service Contracts**

| Schema Type | `{schema}`               | `{entity}`      | `{attribute}`                | `{relation}`                           | `{data_type}` examples                   | `{target}` examples             |
| ----------- | ----------------------- | --------------- | ---------------------------- | -------------------------------------- | ---------------------------------------- | ------------------------------- |
| **JSON**    | schema identifier / IRI | object          | property                     | reference (`$ref`-like)                | `string`, `number`, `boolean`            | `#/definitions/Customer`        |
| **GraphQL** | schema / namespace      | `type` (object) | field with scalar type       | field with object/union/interface type | `Int`, `String`, `Boolean`, `ID`         | `Customer`, `Order`             |
| **OpenAPI** | components/schemas      | schema (object) | property with primitive type | `$ref` property or array of `$ref`     | `string`, `integer`, `number`, `boolean` | `#/components/schemas/Customer` |

---

