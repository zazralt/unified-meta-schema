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

### Structure
```
{schema}
  ├── {@metadata}
  └── {entity}
       ├── {@metadata}
       ├── {attribute}: {data_type}
       └── {relation}: > {target}
```
**Note:** Curly brackets `{variable}` are used to denote template variables throughout the document (like `{schema}`, `{entity}`, `{attribute}`, etc.).

### Syntax
```yaml
{schema}:
  '{@metadata}': '...'
  {entity}:
    '{@metadata}': '...'
    {attribute}: {data_type} [{min},{max}] ({constraint}) | {description}
    {relation}:  > {target}  [{min},{max}] ({constraint}) | {description}
    {relation}:  < {target}  [{min},{max}] ({constraint}) | {description}
````
**Note:** The symbols `>` and `<` indicate relation direction (forward and backward respectively).

### Elements

* **{schema}** – The schema or namespace (e.g., a database name).
* **{entity}** – The entity or table name.
* **{attribute}** – A property with a data type.
* **{relation}** – A link to another entity.
* **{data\_type}** – The type for attributes (mandatory for attributes, empty for relations).
* **{target}** – The target entity for relations (mandatory for relations, empty for attributes).
* **{min}** / **{max}** – Optional, minimum and maximum cardinality in square brackets (e.g. `[1,1]`).
* **{constraint}** – Optional, comma-separated labels or key–value pairs in parentheses (e.g., `(pk,unique)`).
* **{description}** – Optional human-readable description after `|`.

**Important:** An entry MUST have exactly one of `{data_type}` or `{target}`, never both.

### Example

```yaml
shop_db:
  "@id":         "schema_123"
  "@title":      "My Shop"
  Customer:
    customer_id: uuid     [1,1] (pk)                 | Primary key of Customer
    name:        string   [1,1]                      | Customer full name
    orders:      > Order  [0,*]                      | All orders placed by this customer
  Order:
    order_id:    uuid     [1,1] (pk)                 | Primary key of Order
    customer_id: uuid     [1,1] (fk)                 | Foreign key column to Customer
    customer:    customer_id > Customer.customer_id [1,1]
```

### Naming Conventions

* `camelCase`
* `PascalCase`
* `snake_case`

### Metadata
* Optionally declare metadata at the `{schema}` or `{entity}` level using quoted `@`-prefixed keys (e.g., `"@id"` or `"@title"`).
* Metadata values can be single or nested values (e.g.,  `"@id": "123"` or `"@prefix": {...}`).
* Examples: `@id`, `@name`, `@description`, `@version`, `@prefixes`, `@creator`, `@creation_date`

### Data Types

* Always specify data types for attributes.

### Target Syntax Dot-Notation
* Each relation must have one direction symbol `>` for forward and `<` for reverse in the `{target}`.

**Entity Relation:**
* `> {entity}`
* `< {entity}`

**Single Relation:**
* `{attr} > {entity}.{attr}`
* `{attr} < {entity}.{attr}`

**Composite Relation:**
* `({attr1},{attr2}) > {entity}.({attr1},{attr2}`)
* `({attr1},{attr2}) < {entity}.({attr1},{attr2}`)

**External Relation:**
* `{attr} > {schema}.{entity}.{attr}`
* `{attr} < {schema}.{entity}.{attr}`

**Composite External Relation:**
* `({attr1},{attr2}) > {schema}.{entity}.({attr1},{attr2}`)
* `({attr1},{attr2}) < {schema}.{entity}.({attr1},{attr2}`)

### Cardinality

* Optionally specify cardinalities as `[{min},{max}]` (whitespaces allowed).
* `{min}` is the minimum number of values allowed; `{max}` is the maximum (or * for unbounded).
* If the entire cardinality is omitted, default is `[0,*]` (optional, unbounded).

### Constraints

* Optionally specify constraints in parentheses, comma-separated `(,)` (whitespaces allowed and ignored).
* Common labels: `pk` (primary key), `fk` (foreign key), `unique`.
* Key–value examples: `(default=now())`, `(pattern=^[A-Z]{2}\d{4}$)`.

### Description

* Optionally add human-readable descriptions after `|`.

### Prefixes

* Declare prefixes at the `{schema}` level using `@prefix` (e.g., `@prefix: { ex: "https://example.org/onto#", xsd: "http://www.w3.org/2001/XMLSchema#" }`).
* Use prefixed names anywhere—`{schema}`, `{entity}`, `{attribute}`, `{relation}`, `{data_type}`, `{target}`—including `{target}` dot-notation.
* Expand prefixes before validation and before parsing cardinality/constraints.
* Do not insert a space after the prefix colon (write `ex:Person`, not `ex: Person`).
* Quote keys that contain `/` or `#` in YAML (e.g., `"http:Thing"`), and prefer quoting when unsure.

```yaml
my_schema:
  "@prefix":
    ex:  "https://example.org/onto#"
    xsd: "http://www.w3.org/2001/XMLSchema#"
  "ex:Customer":
    customer_id: xsd:string [1,1] (pk)                 | Customer identifier
    name:        xsd:string [1,1]                      | Full name
    orders:      ex:Order   [0,*]                      | All orders of this customer
```

### Alignment Rules

* Use spaces only (no tabs); 2-space indent per level.
* Add whitespace after `:` so the first character of `{data_type}` / `{target}` starts at a single global column across the file.

### JSON Representation

```json
{
  "shop_db": {
    "Customer": {
      "customer_id": "uuid [1,1] (pk) | Primary key of Customer",
      "name": "string [1,1] | Customer full name",
      "orders": "Order [0,*] | All orders placed by this customer"
    },
    "Order": {
      "order_id": "uuid [1,1] (pk) | Primary key of Order",
      "customer_id": "uuid [1,1] | Foreign key column to Customer",
      "customer": "Customer.customer_id [1,1] (fk) | Relation via dot-notation"
    }
  }
}
```

### Table Representation

| schema    | entity   | attribute    | relation | data\_type | target                | min | max | constraint | description                        |
| -------- | -------- | ------------ | -------- | ---------- | --------------------- | --------- | --------- | ---------- | ---------------------------------- |
| shop\_db | Customer | customer\_id |          | uuid       |                       | 1         | 1         | pk         | Primary key of Customer            |
| shop\_db | Customer | name         |          | string     |                       | 1         | 1         |            | Customer full name                 |
| shop\_db | Customer |              | orders   |            | Order                 | 0         | \*        |            | All orders placed by this customer |
| shop\_db | Order    | order\_id    |          | uuid       |                       | 1         | 1         | pk         | Primary key of Order               |
| shop\_db | Order    | customer\_id |          | uuid       |                       | 1         | 1         |            | Foreign key column to Customer     |
| shop\_db | Order    |              | customer |            | Customer.customer\_id | 1         | 1         | fk         | Relation via dot-notation          |

Note: In each row, exactly one of attribute & data_type or relation & target is populated.

---

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

