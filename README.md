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
  {entity}:
    {attribute}: {data_type} [{min_card},{max_card}] ({constraint}) # {description}
    {relation}:  {target}    [{min_card},{max_card}] ({constraint}) # {description}
````

### Elements

* **{model}** – The schema or namespace (e.g., a database name).
* **{entity}** – The entity or table name.
* **{attribute}** – A property with a data type.
* **{relation}** – A link to another entity.
* **{data\_type}** – The type for attributes (empty for relations).
* **{target}** – The target entity for relations (empty for attributes).
* **{min\_card}** / **{max\_card}** – Minimum and maximum cardinality.
* **{constraint}** – Optional, comma-separated labels or key–value pairs in parentheses (e.g., `(pk,unique)`).
* **{description}** – Optional human-readable comment after `#`.

### Example

```yaml
shop_db:
  Customer:
    customer_id: uuid     [1,1] (pk)                 # Primary key of Customer
    name:        string   [1,1]                      # Customer full name
    orders:      Order    [0,*]                      # All orders placed by this customer
  Order:
    order_id:    uuid     [1,1] (pk)                 # Primary key of Order
    customer_id: uuid     [1,1]                      # Foreign key column to Customer
    customer:    Customer.customer_id [1,1] (fk)     # Relation via dot-notation
```

### Naming Conventions

* `camelCase`
* `PascalCase`
* `snake_case`

### Data Types

* Always specify data types for attributes.

### Cardinality

* Optionally specify cardinalities in square brackets, comma-separated.
* `{min_card}` is an integer ≥ 0.
* `{max_card}` is an integer ≥ 0 or `*` (unbounded).

### Constraints

* Optionally specify constraints in parentheses, comma-separated.
* Common labels: `pk` (primary key), `fk` (foreign key), `unique`.
* Key–value examples: `(default=now())`, `(pattern=^[A-Z]{2}\d{4}$)`.

### Description

* Optionally add human-readable descriptions as YAML comments after `#`.

### Alignment Rules

* Use spaces only (no tabs); a two-space indent per level is recommended.
* Within each `{entity}` block, align the first character of `{data_type}` or `{target}` vertically, and align the opening `[` of cardinality vertically.
* Use a single space after the colon before `{data_type}` / `{target}`; apply alignment per entity block only.

### Target Forms

* `{target}` MAY be one of:

  * `{entity}`
  * `{entity}.{attribute}`
  * `{model}.{entity}.{attribute}`

### JSON Representation

```json
{
  "shop_db": {
    "Customer": {
      "customer_id": "uuid [1,1] (pk) # Primary key of Customer",
      "name": "string [1,1] # Customer full name",
      "orders": "Order [0,*] # All orders placed by this customer"
    },
    "Order": {
      "order_id": "uuid [1,1] (pk) # Primary key of Order",
      "customer_id": "uuid [1,1] # Foreign key column to Customer",
      "customer": "Customer.customer_id [1,1] (fk) # Relation via dot-notation"
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


### Schema Types

| Schema Type  | `{model}`                             | `{entity}` | `{attribute}` | `{relation}`            | `{data_type}` examples        | `{target}` examples      | `{min_card},{max_card}` meaning                                |
| ------------ | ------------------------------------- | ---------- | ------------- | ----------------------- | ----------------------------- | ------------------------ | -------------------------------------------------------------- |
| **Ontology** | ontology name / IRI                   | class      | attribute     | relation                | `xsd:string`, `xsd:dateTime`  | `ex:Person`, `ex:Order`  | Minimum/maximum property occurrences in class definition       |
| **Database** | database schema name (e.g., `public`) | table      | column        | foreign key / relation  | `uuid`, `varchar`, `integer`  | `Customer`, `Order`      | Min/max constraint on column value count per row (rarely used) |
| **Dataset**  | dataset schema name                   | table      | column        | relation / join         | `string`, `date`, `decimal`   | `Customer`, `Product`    | Min/max rows linked in relation                                |
| **JSON**     | schema identifier / IRI               | object     | property      | reference (`$ref`-like) | `string`, `number`, `boolean` | `#/definitions/Customer` | Min/max items or property occurrences                          |
| **Avro**     | Avro namespace or schema name         | record     | field         | relation (record ref)   | `string`, `long`, `bytes`     | `Customer`, `Order`      | Min/max occurrences in array/field constraints                 |
