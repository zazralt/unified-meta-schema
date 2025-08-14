# meta-schema
A unified, technology-agnostic meta-schema for representing ontologies, database schemas, dataset schemas, JSON Schemas, and Avro Schemas in a single compact format. It standardizes core modeling concepts — {model}, {entity}, {attribute}, {relation}, {data_type}, {target}, {min_card}, {max_card} — enabling consistent validation, transformation, and interoperability across diverse schema paradigms.


# Overview

| Schema Type     | `{model}`                            | `{entity}` | `{attribute}` | `{relation}`           | `{data_type}` examples        | `{target}` examples      | `{min_card},{max_card}` meaning                                |
| --------------- | ------------------------------------ | ---------- | ------------- | ---------------------- | ----------------------------- | ------------------------ | -------------------------------------------------------------- |
| **Ontology**    | ontology name / IRI                  | class      | attribute     | relation               | `xsd:string`, `xsd:dateTime`  | `ex:Person`, `ex:Order`  | Minimum/maximum property occurrences in class definition       |
| **Database**    | database schema name (e.g. `public`) | table      | column        | foreign key / relation | `uuid`, `varchar`, `integer`  | `Customer`, `Order`      | Min/max constraint on column value count per row (rarely used) |
| **Dataset**     | dataset schema name                  | table      | column        | relation / join        | `string`, `date`, `decimal`   | `Customer`, `Product`    | Min/max rows linked in relation                                |
| **JSON Schema** | schema identifier / IRI              | object     | property      | reference (\$ref-like) | `string`, `number`, `boolean` | `#/definitions/Customer` | Min/max items or property occurrences                          |
| **Avro Schema** | Avro namespace or schema name        | record     | field         | relation (record ref)  | `string`, `long`, `bytes`     | `Customer`, `Order`      | Min/max occurrences in array/field constraints                 |
