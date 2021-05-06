# Rest API design considerations

## What is REST

Rest Stands for 'REpresentational State Transfer'
Concepts include -
- Separation of Client and Server
- Stateless
- Cacheable
- Uniform Interface

## API Design

### URI

- Use Nouns instead of verbs. Example /getQuotes is a bad api name instead just use /quotes. Differentiate the operation using Verbs.
- Use Unique Identifiers in URI to get a Unique record. Example /quotes/70001 gives back the detail on QuoteID 70001.

### Query String

- Use query string for properties that are not associated with the resource. e.g. /quotes?records=100 (get first 100 records)

### Verbs

Most Commonly used -
- GET - Fetches a resource
- PUT - Update an existing resource
- POST - Add new resources
- DELETE - removes a resource

Examples:
| URI (Resource)  | GET | POST | PUT | DELETE |
|-----------------|-----|------|-----|--------|
| /cutomers | Get List | New Customer | Update Batch | Error |
| /customers/123 | Get Item | Error | Update Item | Delete Item |
