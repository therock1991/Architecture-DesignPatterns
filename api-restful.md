# API

## An overview of HTTP
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview
## Resources on web
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Identifying_resources_on_the_Web
## What is REST

## Principles of REST

## REST Contraints

## REST Resource Naming Guide

## Caching

## Compression

## Content Negotiation

## HATEOAS

## Idempotence

## Security Essentials

## Versioning

## Stalessness

## PUT vs PATCH

- https://rapidapi.com/blog/put-vs-patch/

## GET vs POST

|                                   | GET                                                                                                                                                                  | POST                                                                                                            |
|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
|                           History | Parameters remain in browser history because they are part of the URL                                                                                                | Parameters are not saved in browser history.                                                                    |
|                        Bookmarked | Can be bookmarked.                                                                                                                                                   | Can not be bookmarked.                                                                                          |
|   BACK button/re-submit behaviour | GET requests are re-executed but may not be re-submitted to server if the HTML is stored in the browser cache.                                                       | The browser usually alerts the user that data will need to be re-submitted.                                     |
| Encoding type (enctype attribute) | application/x-www-form-urlencoded                                                                                                                                    | multipart/form-data or application/x-www-form-urlencoded Use multipart encoding for binary data.                |
|                        Parameters | can send but the parameter data is limited to what we can stuff into the request line (URL). Safest to use less than 2K of parameters, some servers handle up to 64K | Can send parameters, including uploading files, to the server.                                                  |
|                            Hacked | Easier to hack for script kiddies                                                                                                                                    | More difficult to hack                                                                                          |
|    Restrictions on form data type | Yes, only ASCII characters allowed.                                                                                                                                  | No restrictions. Binary data is also allowed.                                                                   |
|                          Security | GET is less secure compared to POST because data sent is part of the URL. So it's saved in browser history and server logs in plaintext.                             | POST is a little safer than GET because the parameters are not stored in browser history or in web server logs. |
|  Restrictions on form data length | Yes, since form data is in the URL and URL length is restricted. A safe URL length limit is often 2048 characters but varies by browser and web server.              | No restrictions                                                                                                 |
|                         Usability | GET method should not be used when sending passwords or other sensitive information.                                                                                 | POST method used when sending passwords or other sensitive information.                                         |
|                        Visibility | GET method is visible to everyone (it will be displayed in the browser's address bar) and has limits on the amount of information to send.                           | POST method variables are not displayed in the URL.                                                             |
|                            Cached | Can be cached                                                                                                                                                        | Not cached                                                                                                      |

## Define operations in terms of HTTP methods

### GET

- Idempotent
- Retrieves a representation of the resource at the specified URI. The body of the response message contains the details of the requested resource.

### POST

- No idempotent
- Creates a `new` resource at the specified URI.
- The body of the request message provides the details of the new resource. Note that POST can also be used to trigger operations that don't actually create resources.

### PUT
- Idempotent
- Replace an object, or create a named object, when applicable
### PATCH

- No idempotent
- Apply a partial update to an object
### Delete
- Idempotent
- Delete an object

### HEAD
- Idempotent
- Return metadata of an object for a GET response. Resources that support the GET method MAY support the HEAD method as well
### OPTION
- Idempotent
 - Get information about a request; see below for details.
## Web API Checklist
- https://mathieu.fenniak.net/the-api-checklist/

##
## References:

- API design best practices: https://github.com/MicrosoftDocs/architecture-center/blob/master/docs/best-practices/api-design.md
- Microsoft API guidelines: https://github.com/stephennp/api-guidelines/blob/vNext/Guidelines.md
- Rest API Tutorial: https://restfulapi.net
- Web API versioning in real world applications: https://vladsukhachev.wordpress.com/2016/12/12/web-api-versioning-in-real-world-applications/
