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
    {attribute}: {data_type} [{min_card},{max_card}] ({constraint}) # description
    {relation}:  {target}    [{min_card},{max_card}] ({constraint}) # description
```

### Elements

* **{model}** – The schema or namespace (e.g., a database name).
* **{entity}** – The entity or table name.
* **{attribute}** – A property with a data type.
* **{relation}** – A link to another entity.
* **{data\_type}** – The type for attributes (empty for relations).
* **{target}** – The target entity for relations (empty for attributes).
* **{min\_card}** / **{max\_card}** – Minimum and maximum cardinality.


### Example

```yaml
shop_db:
  Customer:
    customer_id: uuid     [1,1]
    name:        string   [1,1]
    orders:      Order    [0,*]
  Order:
    order_id:    uuid     [1,1]
    customer:    Customer [1,1]
```

### Naming Conventions

* `camelCase`
* `PascalCase`
* `snake_case`

### Data Type

* write always data types for attributes

### Cardinality

* write optionally cardinalities in square brackets and comma separated
* `{min_card}` is an integer ≥ 0.
* `{max_card}` is an integer ≥ 0 or `*` (unbounded).

### Constraints

* write optionally constraint in round brackets and comma separated, if necessary
* pk: primar key
* fk: forgein key
* unique: unique value
* default: default value, syntax: (default: ...)
* pattern: regex pattern, syntax: (pattern: ...)

### Description

* write optionally human-readable descriptions or comments after a hash #

### Alignment Rules

Within each `{entity}` block, use spaces (no tabs) to align the first character of `{data_type}` or `{target}` and the opening `[` of cardinality in vertical columns, with a single space after the colon. Alignment is applied per entity block only.

### Target Forms

* **{target}** may be either `{entity}` or `{entity}.{attribute}` (dot notation) to reference a specific attribute/key in the target entity.



### JSON Representation

```json
{
  "shop_db": {
    "Customer": {
      "customer_id": "uuid [1,1]",
      "name": "string [1,1]",
      "orders": "Order [0,*]"
    },
    "Order": {
      "order_id": "uuid [1,1]",
      "customer": "Customer [1,1]"
    }
  }
}
```

### CSV Representation

| model    | entity   | attribute    | relation | data\_type | target   | min\_card | max\_card |
| -------- | -------- | ------------ | -------- | ---------- | -------- | --------- | --------- |
| shop\_db | Customer | customer\_id |          | uuid       |          | 1         | 1         |
| shop\_db | Customer | name         |          | string     |          | 1         | 1         |
| shop\_db | Customer |              | orders   |            | Order    | 0         | \*        |
| shop\_db | Order    | order\_id    |          | uuid       |          | 1         | 1         |
| shop\_db | Order    |              | customer |            | Customer | 1         | 1         |




### Schema Types

| Schema Type  | `{model}`                            | `{entity}` | `{attribute}` | `{relation}`           | `{data_type}` examples        | `{target}` examples      | `{min_card},{max_card}` meaning                                |
| ------------ | ------------------------------------ | ---------- | ------------- | ---------------------- | ----------------------------- | ------------------------ | -------------------------------------------------------------- |
| **Ontology** | ontology name / IRI                  | class      | attribute     | relation               | `xsd:string`, `xsd:dateTime`  | `ex:Person`, `ex:Order`  | Minimum/maximum property occurrences in class definition       |
| **Database** | database schema name (e.g. `public`) | table      | column        | foreign key / relation | `uuid`, `varchar`, `integer`  | `Customer`, `Order`      | Min/max constraint on column value count per row (rarely used) |
| **Dataset**  | dataset schema name                  | table      | column        | relation / join        | `string`, `date`, `decimal`   | `Customer`, `Product`    | Min/max rows linked in relation                                |
| **JSON**     | schema identifier / IRI              | object     | property      | reference (\$ref-like) | `string`, `number`, `boolean` | `#/definitions/Customer` | Min/max items or property occurrences                          |
| **Avro**     | Avro namespace or schema name        | record     | field         | relation (record ref)  | `string`, `long`, `bytes`     | `Customer`, `Order`      | Min/max occurrences in array/field constraints                 |

---
