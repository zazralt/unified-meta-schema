# meta-schema

A unified, technology-agnostic meta-schema for representing ontologies, database schemas, dataset schemas, JSON Schemas, and Avro Schemas in a single compact YAML format. It standardizes core modeling concepts — `{model}`, `{entity}`, `{attribute}`, `{relation}`, `{data_type}`, `{target}`, `{min_card}`, `{max_card}` — enabling consistent validation, transformation, and interoperability across diverse schema paradigms.

**Scope:** This specification defines a concise YAML structure for expressing entities, their attributes, and relations across multiple schema paradigms with explicit datatypes and cardinalities.

## Specification

### YAML Level Structure
1. `{model}`
2. `{entity}`
3. `{attribute}` or `{relation}`

### Syntax
```yaml
{model}:
  '@metadata': '...'
  {entity}:
    '@metadata': '...'
    {attribute}: {data_type} [{min_card},{max_card}] ({constraint}) | {description}
    {relation}:  {target}    [{min_card},{max_card}] ({constraint}) | {description}
````

### Elements

* **{model}** – The schema or namespace (e.g., a database name).
* **{entity}** – The entity or table name.
* **{attribute}** – A property with a data type.
* **{relation}** – A link to another entity.
* **{data\_type}** – The type for attributes (mandatory for attributes, empty for relations).
* **{target}** – The target entity for relations (mandatory for relations, empty for attributes).
* **{min\_card}** / **{max\_card}** – Optional, minimum and maximum cardinality in square brackets (e.g. `[1,1]`).
* **{constraint}** – Optional, comma-separated labels or key–value pairs in parentheses (e.g., `(pk,unique)`).
* **{description}** – Optional human-readable description after `|`.

Note that an entry MUST have exactly one of {data_type} or {target}, never both.

### Example

```yaml
shop_db:
  "@id":         "schema_123"
  "@title":      "My Shop"
  Customer:
    customer_id: uuid     [1,1] (pk)                 | Primary key of Customer
    name:        string   [1,1]                      | Customer full name
    orders:      Order    [0,*]                      | All orders placed by this customer
  Order:
    order_id:    uuid     [1,1] (pk)                 | Primary key of Order
    customer_id: uuid     [1,1]                      | Foreign key column to Customer
    customer:    Customer.customer_id [1,1] (fk)     | Relation via dot-notation
```

### Naming Conventions

* `camelCase`
* `PascalCase`
* `snake_case`

### Metadata
* Optionally declare metadata at the `{model}` or `{entity}` level using quoted `@`-prefixed keys (e.g., `"@id"` or `"@title"`).
* Metadata values can be single or nested values (e.g.,  `"@id": "123"` or `"@prefix": {...}`).

### Data Types

* Always specify data types for attributes.

### Target Dot-Notation
`{target}` may be one of:
* `{entity}`
* `{entity}.{attribute}`
* `{model}.{entity}.{attribute}`

### Cardinality

* Optionally specify cardinalities as `[{min_card},{max_card}]` (whitespaces allowed).
* `{min_card}` is the minimum number of values allowed; `{max_card}` is the maximum (or * for unbounded).
* If the entire cardinality is omitted, default is `[0,*]` (optional, unbounded).

### Constraints

* Optionally specify constraints in parentheses, comma-separated `(,)` (whitespaces allowed and ignored).
* Common labels: `pk` (primary key), `fk` (foreign key), `unique`.
* Key–value examples: `(default=now())`, `(pattern=^[A-Z]{2}\d{4}$)`.

### Description

* Optionally add human-readable descriptions after `|`.

### Prefixes

* Declare prefixes at the `{model}` level using `@prefix` (e.g., `@prefix: { ex: "https://example.org/onto#", xsd: "http://www.w3.org/2001/XMLSchema#" }`).
* Use prefixed names anywhere—`{model}`, `{entity}`, `{attribute}`, `{relation}`, `{data_type}`, `{target}`—including `{target}` dot-notation.
* Expand prefixes before validation and before parsing cardinality/constraints.
* Do not insert a space after the prefix colon (write `ex:Person`, not `ex: Person`).
* Quote keys that contain `/` or `#` in YAML (e.g., `"http:Thing"`), and prefer quoting when unsure.

```yaml
my_model:
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

| model    | entity   | attribute    | relation | data\_type | target                | min\_card | max\_card | constraint | description                        |
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

| Schema Type  | `{model}`           | `{entity}`     | `{attribute}`                               | `{relation}`                            | `{data_type}` examples       | `{target}` examples     |
| ------------ | ------------------- | -------------- | ------------------------------------------- | --------------------------------------- | ---------------------------- | ----------------------- |
| **Ontology** | ontology name / IRI | class          | attribute                                   | relation                                | `xsd:string`, `xsd:dateTime` | `ex:Person`, `ex:Order` |
| **RDFS**     | ontology IRI        | `rdfs:Class`   | `rdf:Property` with `rdfs:range` (datatype) | `rdf:Property` with class range         | `xsd:string`, `xsd:dateTime` | `ex:Person`, `ex:Order` |
| **SHACL**    | shapes graph IRI    | `sh:NodeShape` | `sh:property` with `sh:datatype`            | `sh:property` with `sh:node`/`sh:class` | `xsd:string`, `xsd:integer`  | target `ex:Customer`    |

---

#### **Databases & Datasets**

| Schema Type  | `{model}`            | `{entity}` | `{attribute}` | `{relation}`           | `{data_type}` examples       | `{target}` examples   |
| ------------ | -------------------- | ---------- | ------------- | ---------------------- | ---------------------------- | --------------------- |
| **Database** | database schema name | table      | column        | foreign key / relation | `uuid`, `varchar`, `integer` | `Customer`, `Order`   |
| **Dataset**  | dataset schema name  | table      | column        | relation / join        | `string`, `date`, `decimal`  | `Customer`, `Product` |

---

#### **Data Serialization & Storage Formats**

| Schema Type          | `{model}`                | `{entity}`                       | `{attribute}`                              | `{relation}`                     | `{data_type}` examples                     | `{target}` examples               |
| -------------------- | ------------------------ | -------------------------------- | ------------------------------------------ | -------------------------------- | ------------------------------------------ | --------------------------------- |
| **Avro**             | Avro namespace or schema | record                           | field                                      | relation (record ref)            | `string`, `long`, `bytes`                  | `Customer`, `Order`               |
| **XSD (XML Schema)** | schema namespace         | `complexType` / global `element` | `xs:attribute` or simple-content `element` | `element` with `@ref` / `keyref` | `xsd:string`, `xsd:date`, `xsd:int`        | `CustomerType`, `OrderType`       |
| **Protobuf**         | `package`                | `message`                        | scalar `field`                             | `field` of `message` type        | `int32`, `string`, `bytes`, `bool`         | `Customer`, `Order`               |
| **Parquet**          | file schema              | group (logical struct)           | primitive field                            | group-typed field                | `INT32`, `DOUBLE`, `BYTE_ARRAY`, `BOOLEAN` | nested `group` (e.g., `Customer`) |
| **Thrift IDL**       | `namespace`              | `struct`                         | field with base type                       | field with `struct`/`union` type | `i32`, `string`, `bool`, `binary`          | `Customer`, `Order`               |

---

#### **APIs & Service Contracts**

| Schema Type | `{model}`               | `{entity}`      | `{attribute}`                | `{relation}`                           | `{data_type}` examples                   | `{target}` examples             |
| ----------- | ----------------------- | --------------- | ---------------------------- | -------------------------------------- | ---------------------------------------- | ------------------------------- |
| **JSON**    | schema identifier / IRI | object          | property                     | reference (`$ref`-like)                | `string`, `number`, `boolean`            | `#/definitions/Customer`        |
| **GraphQL** | schema / namespace      | `type` (object) | field with scalar type       | field with object/union/interface type | `Int`, `String`, `Boolean`, `ID`         | `Customer`, `Order`             |
| **OpenAPI** | components/schemas      | schema (object) | property with primitive type | `$ref` property or array of `$ref`     | `string`, `integer`, `number`, `boolean` | `#/components/schemas/Customer` |

---
