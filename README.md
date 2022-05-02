## REST-API Cheat Sheet [see also 'REST design patterns'](https://medium.com/@patricksavalle/rest-api-design-as-a-craft-not-an-art-a3fd97ed3ef4)

Initially based on [this cheatsheet](https://github.com/RestCheatSheet/api-cheat-sheet).

- Build the API with consumers (developers) in mind--as a product in its own right.

  * Not for a specific front-end.
  * Use use-cases and scenarios to validate your APIs UX.

- A good API (as any other interface) is

  * Consistent (avoid surprises by being predictable) 
  * Cohesive (only lists endpoints with functional dependency)
  * Complete (has all necessary endpoints for its purpose)
  * Minimal (no more endpoints than neccessary to be complete, no featurism)
  * Encapsulating (hiding implementation details)
  * Self explaining
  * Documented (if self explanation is not sufficient)

- Use a domain model ([example domain model](https://i.imgur.com/55qxMz6h.png)), even if it needs to be reverse engineered (keep it simple)
  * Base resources and URLs on the entities and relationships of your domain model.

- Create an OpenAPI file for your API before you start implementing the REST-server

- Use a naming convention

  * Use plural forms for resources (```orders``` instead of ```order```), it's the datamodelling standard.
  * Use lowercase in constant parts of paths, e.g: ```/lowercase```, not ```/CamelCase``` or ```/UPPERCASE```.
  * Use camelCase field names, e.g.: ```fieldName```, not ```FieldName``` or ```field_name```
  * Use UpperCamelCase object names, e.g.: ```TicketObject```, not ```ticketObject``` or ```ticket_object```
  *	Avoid snake_case and kebab-case.


- Use the HTTP verbs to mean this:

    * POST - create and all other non-idempotent operations.
    * PUT - replace.
    * PATCH - (partial) update.
    * GET - read a resource or collection.
    * DELETE - remove a resource or collection.

- Ensure that your GET, PUT, PATCH and DELETE operations are all [idempotent](http://www.restapitutorial.com/lessons/idempotency.html).

- Use [HTTP status codes](https://httpstatuses.com/) to be meaningful.
  * 102 - Processing. Returned as long as a asynchronous response is still pending. See also 202 Accepted.
  * 200 - Success.
  * 201 - Created. Returned on successful creation of a new resource. Include a 'Location' header with a link to the newly-created resource.
  * 202 - Accepted. Returned to indicated an asynchronous response will be given. Include a 'Location' HTTP-header with URL of the future resource. See also 102 Processing.
  * 204 - No content (for empty responses)
  * 304 - Not modified. Returned when a client asks for an etag it already received (sent in the request headers). See _caching_ below.
  * 400 - Bad request. Data issues such as invalid JSON, etc.
  * 404 - Not found. Resource not found on GET.
  * 409 - Conflict. Duplicate data or invalid data state would occur.

- Use the Collection Metaphor.

    * Two URLs per public resource in the domain model:

      * The resource collection (e.g. ```/orders```)
      * Individual resource within the collection (e.g. ```/orders/{keytype}/{key}```).

            e.g.

            POST /orders                          
            GET /orders                           
            GET /orders/orderid/$id
            PUT /orders/orderid/$id
            PATCH /orders/orderid/$id
            DELETE /orders/orderid/$id
            
- Reflect the hierarchy of the domain model in the URLs, use the same names

  * The first path-segment is the type of the resource in the response
  
        e.g.

        GET /wholes/$id           // returns a 'whole'
        GET /parts/wholeid/$id    // returns 'parts' for a specified whole
        
  * Don't be dogmatic, flat and non-domain-model URLs are sometimes needed. 
  
        e.g.
        
        GET /parts/[subpartid}]
        GET /frontpage

- Use [Simple Versioning](https://simver.org/)
  * A normal version number MUST take the form X.Y where X is the major version and Y is the minor version.
  * Minor version MUST be incremented for any release which maintains backwards compatibility to the public API.
  * Major version MUST be incremented if any backwards incompatible changes are introduced to the public API.
  * Keep the API backward compatible as long as possible / avoid breaking changes
  * API's based on a domain model are the most stable
  * Versioning via the URL signifies a 'platform' version and the entire platform must be versioned at the same time.

        e.g.

        https://api.example.com/v1.0/orders

  * Versioning via the Accept header is versioning the resource, avoid this.

        e.g.

        GET /jobs HTTP/1.1
        Host: api.example.com
        Accept: application/vnd.example.api+json;version=2

  * Additions to a response do not require versioning. However, additions to a request body that are 'required' are troublesome--and may require versioning (breaking changes).
  
- Responses are part of the interface, don't expose implementationdetails in them.

  * Return domain entities not database entities (use a (logical) domain model not a (technical) data model).
  * Don’t expose internal / coupling tables as two IDs.

- Support sorting and pagination on collections (```?offset=100&limit=50&order=id```).

- For large responses, allow clients to select the fields that come back in the response (with query-arguments, ```?fields=name&fields=address&fields=city```)

- Use UTF-8 character encoding.

- Use ISO 8601 for dates. 

      E.g. 

      1997-07-16T19:20:30+01:00   # time-offset
      1997-07-16T19:20:30Z        # Zulu-time
      1997-07-16T19:20:30EST      # time-zone
      1997-07-16T19:20:30         # local datetime

- Use ISO 4217 for currency codes.

- Use ISO 3166 for country codes.

- Use [RFC7807](https://tools.ietf.org/html/rfc7807) for errormessages.

- Avoid HATEOAS, while elegent hypermedia linking (HATEOAS) and versioning is troublesome no matter what--minimize it.

-	Don’t use OData, in particular avoid OData function calls as they violate REST-principles (remodel functions calls to resource manipulations)

- Use [OAuth2](http://oauth.net/2/) to secure your API.
  * Use an auto-expiring Bearer token for authentication (```Authorisation: Bearer f0ca4227-64c4-44e1-89e6-b27c62ac2eb6```).
  * Require HTTPS.
  * Copnsider using [JSON Web Tokens](https://jwt.io/).

- Enforce use of the Content-Type and Accept-Type headers even if you use JSON as default for both requests and responses.

      e.g.

      Content-Type: application/json
      Accept-Type: application/json

- Responses contain header: ```X-Content-Type-Options: nosniff```

- Responses contain header: ```X-Frame-Options: deny```

- For multi-lingual APIs, use the Accept-Language header for locale setting (```Accept-Language: nl, en-gb;q=0.8, en;q=0.7```)

- All endpoint return the Date header
    * Date - Date and time the response was returned (in RFC1123 format). (```Date: Sun, 06 Nov 1994 08:49:37 GMT```)

- Allow for strong caching in (client, transport, proxy, etc.) through the cache-control response-header. As a minimum have public GET-endpoints return the following response headers:
    * [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) - The maximum number of seconds (ttl) a response can be cached. (```Cache-Control: public, 360``` or ```Cache-Control: no-store```)
    * Strong caching minimizes the number of requests a server receives

- Allow for weak caching through the ETag respons-header
    * [ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) - Use a SHA1 hash for the version of a resource. Make sure to include the media type in the hash value, because that makes a different representation. (```ETag: "2dbc2fd2358e1ea1b7a6bc08ea647b9a337ac92d"```). The client needs to send a **[If-None-Match](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match)** header for this mechanism to work.
    * Weak caching minimizes the work a server needs to do (but not the number of requests it receives)

- No privacy or security compromising data in URL's or responses

- Use the **X-Signing-Algorithm** header to communicate the type of [content signing](https://datatracker.ietf.org/doc/html/rfc7518#appendix-A.3) (```X-Signing-Algorithm: RS256```) 

- Use the **X-SHA256-Checksum** header to communicate the SHA256 hash value of the content (```X-SHA256-Checksum: e1d58ba0a1810d6dca5f086e6e36a9d81a8d4bb00378bdab30bdb205e7473f87```) 

- Use the **X-Encryption-Algorithm** header to communicate the type of [content encryption](https://datatracker.ietf.org/doc/html/rfc7518#appendix-A.3) (```X-Encryption-Algorithm: A128CBC-HS256```) 

- When content signing is used, this is done after the content is (optionally) encrypted.

- Incoming asynchronous requests require a [V4 UUID](https://www.uuidtools.com/v4) in the **X-Request-ID** request header which will be echoed in the **X-Response-ID** response header. This enables clients to correlate requests to responses. (```X-Request-ID: f62e77ae-c2ba-4988-884d-e9de8f3ff1b1```, ```X-Response-ID: f62e77ae-c2ba-4988-884d-e9de8f3ff1b1```) 

