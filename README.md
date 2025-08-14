# meta-schema
A unified, technology-agnostic meta-schema for representing ontologies, database schemas, dataset schemas, JSON Schemas, and Avro Schemas in a single compact format. It standardizes core modeling concepts — {model}, {entity}, {attribute}, {relation}, {data_type}, {target}, {min_card}, {max_card} — enabling consistent validation, transformation, and interoperability across diverse schema paradigms.

# Specification

## Yaml

### Level Structure
1. model
2. entity
3. attribute or relation

### Syntax
```yaml
{model}:
  {entity}:
    {attribute}: {data_type} [{min_card},{max_card}]
    {relation}:  {target} [{min_card},{max_card}]
```

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

## Naming Conventions
* camelCase
* PascalCase
* snake_case

## Elements
| Schema Type     | `{model}`                            | `{entity}` | `{attribute}` | `{relation}`           | `{data_type}` examples        | `{target}` examples      | `{min_card},{max_card}` meaning                                |
| --------------- | ------------------------------------ | ---------- | ------------- | ---------------------- | ----------------------------- | ------------------------ | -------------------------------------------------------------- |
| **Ontology**    | ontology name / IRI                  | class      | attribute     | relation               | `xsd:string`, `xsd:dateTime`  | `ex:Person`, `ex:Order`  | Minimum/maximum property occurrences in class definition       |
| **Database**    | database schema name (e.g. `public`) | table      | column        | foreign key / relation | `uuid`, `varchar`, `integer`  | `Customer`, `Order`      | Min/max constraint on column value count per row (rarely used) |
| **Dataset**     | dataset schema name                  | table      | column        | relation / join        | `string`, `date`, `decimal`   | `Customer`, `Product`    | Min/max rows linked in relation                                |
| **JSON** | schema identifier / IRI              | object     | property      | reference (\$ref-like) | `string`, `number`, `boolean` | `#/definitions/Customer` | Min/max items or property occurrences                          |
| **Avro** | Avro namespace or schema name        | record     | field         | relation (record ref)  | `string`, `long`, `bytes`     | `Customer`, `Order`      | Min/max occurrences in array/field constraints                 |

