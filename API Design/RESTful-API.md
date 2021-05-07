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

### Design Associations

Association or Relationship between resources should be used while designing your apis. /api/customers/123/invoices should give me back the invoices of customer 123 and not all the invoices. Similary there can be a URI for  other associations like shipments for a customer can be /api/customers/123/paymemts .

### Paging

Lists should support paging.
- Small and Fixed lists may work well without paging. For example list of countries will always return a fixed number of records.
- Long and dynamic lists should always support paging. For example list of all invoices. This helps in 
  - Stopping the client for pulling all the data because they did not know it better
  - Gives an option/control to the user 
- Query Strings are commonly used for paging an API response. /api/invoices?page=1&page_size=50 -> this means divide the response into pages of size 50 each and give back page 1 of that response.
- Response can also be formatted in a way that it contains some information about the paging that is being done and also contain links for the next page as well as the previous page - 
` {
    totalResults: 500,
    nextPage: "/api/sites?page=5",
    prevPage: "/api/sites?page=3",
    results: []
  }`
  
### Error Handling

- Not just Error Codes. ALso return the Error object. e.g. 400 Bad Request { error: "Failed to Supply ID""}
- Some Errors dont need a error info. like 404 Not Found

### Caching

- Not All API's need caching , but while designing a api caching should always be considered.
- Its not server side caching to be a restful API , it is HTTP Caching
  - Keep a version of the data and send it as a header. The Api will only fetch and return new data if the version does not matter. The API returns a 304 indicating it is cached and no changes are required. return 412 if the verion is not the same
  - This can also be used to provide consistency. Update only if the version of data that I have matches teh version on server.

### Asynchronous API
- Generally for long lived connections. The response is not immediate or needs polling etc. SignalR, Soccket.IO, WebSockets.
- Are not Restful by nature.

## Versioning

- Good if the API is exposed to internal or external consumers.
- The goal here is to evolve a api without breaking existing clients.
- Ways to version a api -
  - Versioning in the URI path . e.g http://foo.org/api/v2/customers
  - Query String Versioning . /api/customers?v=2.0 . Pros versioning is optional and can be defaulted to a specific version.
  - Header Versioning `X-Version: 2.0`

## API Security

### CORS

### App Authentication

- Allow Access only from authenticated apps

### Authentication Types

- Cookies
- Basic Auth
- Token Auth
- OAuth


### User Authentication
