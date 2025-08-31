# Unified Meta Diciontary

## Structure

```yaml
{dictionary}:
  {term}: {name}; {synonyms}; {examples}; {definition}
    {term}: ...
      {term}: ...
```

* **`{dictionary}`**: the name of the dictionary (string, root key).
* **`{term}`**: a unique key/slug for each entry.
* **Indentation**: expresses hierarchy (parent → child).

---

## Fields (semicolon-separated)

1. **name** *(required)*

   * Preferred label for the term.
   * Can include language tag suffix (e.g., `Car@en`, `Auto@de`).

2. **synonyms** *(optional)*

   * Comma-separated list of alternative labels.
   * May include language tags.
   * Leave empty if none.

3. **examples** *(optional)*

   * Comma-separated list of illustrative instances.
   * May include language tags.
   * Leave empty if none.

4. **definition** *(optional)*

   * Free-text explanation.
   * May include multiple values with language tags.
   * Leave empty if none.

---

## Syntax Rules

* Fields are separated by semicolons (`;`).
* If a field is empty, keep the semicolon (e.g., `Coupe; ; ; Two-door car`).
* Multiple values within a field are comma-separated.
* Language tags are optional and written as `@xx` (e.g., `Fahrzeug@de`).
* Whitespace around separators is ignored.

---

## Example

```yaml
vehicles:
  vehicle: Vehicle@en, Fahrzeug@de; Automobile@en, Transport@en, Auto@de; Sedan@en, SUV@en; A means of transporting people or goods@en, Ein Mittel zur Beförderung von Personen oder Gütern@de
    car: Car@en, Auto@de; Automobile@en; Coupe@en, Sedan@en; A road vehicle with four wheels@en, Ein vierrädriges Straßenfahrzeug@de
      sedan: Sedan@en, Limousine@de; Saloon@en; ; Passenger car in a three-box configuration@en, Personenkraftwagen in Dreibox-Bauweise@de
      coupe: Coupe@en, Coupé@fr; ; ; Two-door passenger car with a fixed roof@en
```
