# LeadPages HTTP API Standards

This document has been adapted from [a repository](https://github.com/WhiteHouse/api-standards)
originally published by The White House, and will be continually updated to
reflect API Standards at LeadPages. We are thankful to be able to use their
work as a guide. Please refer to the [credits](#credits) for further
information.

## Contents

- [Guidelines](#guidelines)
- [Pragmatic REST](#pragmatic-rest)
- [RESTful URLs](#restful-urls)
- [HTTP Verbs](#http-verbs)
- [Responses](#responses)
- [Error handling](#error-handling)
- [Versions](#versions)
- [Record Limits](#record-limits)
- [Mock Responses](#mock-responses)
- [JSONP](#jsonp)
- [Contributing](#contributing)
- [Credits](#credits)

## Guidelines

This document provides guidelines and examples for LeadPages HTTP APIs,
encouraging consistency, maintainability, and best practices across
applications. LeadPages APIs aim to balance a truly RESTful API interface with
a positive developer experience (DX).

## Pragmatic REST

These guidelines aim to support a truly RESTful API. Here are a few exceptions:

- Put the version number of the API in the URL (see examples below). Don't
accept any requests that do not specify a version number.
- Allow users to request formats like JSON or XML like this:
    - `http://example.com/api/v1/magazines.json`
    - `http://example.com/api/v1/magazines.xml`

## RESTful URLs

### General guidelines for RESTful URLs
- A URL identifies a resource.
- URLs should include nouns, not verbs.
- Use plural nouns only for consistency (no singular nouns).
- Use HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`, etc.) to operate on the
collections and elements.
- You shouldn’t need to go deeper than `resource/identifier/resource`.
- Put the version number at the base of your URL, for example
`http://example.com/v1/path/to/resource`.
- URL v. header:
    - If it changes the logic you write to handle the response, put it in the
    URL.
    - If it doesn’t change the logic for each response, like OAuth info, put
    it in the header.
- Specify optional fields in a comma separated list.
- Formats should be in the form of `api/v2/resource/{id}.json`

### Good URL examples
- List of magazines:
    - `GET http://www.example.com/api/v1/magazines.json`
- Filtering is a query:
    - `GET http://www.example.com/api/v1/magazines.json?year=2011&sort=desc`
    - `GET http://www.example.com/api/v1/magazines.json?topic=economy&year=2011`
- A single magazine in JSON format:
    - `GET http://www.example.com/api/v1/magazines/1234.json`
- All articles in (or belonging to) this magazine:
    - `GET http://www.example.com/api/v1/magazines/1234/articles.json`
- All articles in this magazine in XML format:
    - `GET http://www.example.com/api/v1/magazines/1234/articles.xml`
- Specify optional fields in a comma separated list:
    - `GET http://www.example.com/api/v1/magazines/1234.json?fields=title,subtitle,date`
- Add a new article to a particular magazine:
    - `POST http://www.example.com/api/v1/magazines/1234/articles`

### Bad URL examples
- Non-plural noun:
    - `http://www.example.com/magazine`
    - `http://www.example.com/magazine/1234`
    - `http://www.example.com/publisher/magazine/1234`
- Verb in URL:
    - `http://www.example.com/magazine/1234/create`
- Filter outside of query string
    - `http://www.example.com/magazines/2011/desc`

## HTTP Verbs

HTTP verbs, or methods, should be used in compliance with their definitions
under the [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)
standard.
The action taken on the representation will be contextual to the media type
being worked on and its current state. Here's an example of how HTTP verbs
map to create, read, update, delete operations in a particular context:

| HTTP Method     | `POST`          | `GET`     | `PUT`       | `DELETE` |
| --------------- | --------------- | --------- | ----------- | -------- |
| CRUD equivalent | CREATE          | READ      | UPDATE      | DELETE |
| `/dogs`           | Create new dogs | List dogs | Bulk update | Delete all dogs |
| `/dogs/1234`      | Error           | Show Bo   | If exists, update Bo; If not, error | Delete Bo |

(Example from Web API Design, by Brian Mulloy, Apigee.)


## Responses

- No values in keys
- No internal-specific names (e.g. "node" and "taxonomy term")
- Metadata should only contain direct properties of the response set, not
properties of the members of the response set

### Good examples

No values in keys:

```json
"tags": [
    {"id": "125", "name": "Environment"},
    {"id": "834", "name": "Water Quality"}
]
```

### Bad examples

Values in keys:

```json
"tags": [
    {"125": "Environment"},
    {"834": "Water Quality"}
],
```

## Error handling

Error responses should include a common HTTP status code, message for the
developer, message for the end-user (when appropriate), internal error code
(corresponding to some specific internally determined ID), links where
developers can find more info. For example:

```json
{
    "status" : 400,
    "developerMessage" : "Verbose, plain language description of the problem. Provide developers suggestions about how to solve their problems here",
    "userMessage" : "This is a message that can be passed along to end-users, if needed.",
    "errorCode" : "444444",
    "moreInfo" : "http://www.example.com/developer/path/to/help/for/444444, http://drupal.org/node/444444",
}
```

Use three simple, common response codes indicating success, failure due to
client-side problem, or failure due to server-side problem (respectively):

- `200 OK`
- `400 Bad Request`
- `500 Internal Server Error`


## Versions

- Never release an API without a version number.
- Versions should be integers, not decimal numbers, prefixed with `v`. For
example:
    - Good: `v1`, `v2`, `v3`
    - Bad: `v-1.1`, `v1.2`, `1.3`
- Maintain APIs at least one version back.


## Record limits

- If no limit is specified, return results with a default limit.
- To get records 51 through 75, request `http://example.com/magazines?limit=25&offset=50`
where:
    - `offset=50` means "skip the first 50 records"
    - `limit=25` means, "return a maximum of 25 records"

Information about record limits and total available count should also be included in the response. Example:

```json
{
    "metadata": {
        "resultset": {
            "count": 227,
            "offset": 25,
            "limit": 25
        }
    },
    "results": []
}
```

## Mock Responses

It is suggested that each resource accept a 'mock' parameter on the testing
server. Passing this parameter should return a mock data response (bypassing
the backend).

Implementing this feature early in development ensures that the API will
exhibit consistent behavior, supporting a test driven development methodology.

Note that if the mock parameter is included in a request to the production
environment, an error should be raised.

## JSONP

JSONP is most easily explained with an example, like [this one](http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about?answertab=votes#tab-top)
on Stack Overflow.

## Contributing

Contributing follows a fork, pull request & issue format. All ideas,
improvements, critiques, and fixes are welcome. All contributers, including
maintainers, must open pull requests, other than exceptional cases or basic
repository chores.

- If you have something that you feel is best represented **concretely**, just
fork the repository and **create a pull request** with your changes.
- If you have something that you feel is an **idea**, **brainstorm topic**, or
is otherwise **still cookin'**, feel free to **open an issue** and use the
"idea" tag.

## Credits

This document has been adapted from [The White House's API standards](https://github.com/WhiteHouse/api-standards).
Additional credits from that document are reproduced here.

This document borrows heavily from:
- [Fielding's Dissertation on REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
- [API Facade Pattern](http://apigee.com/about/resources/ebooks/api-fa%C3%A7ade-pattern) by **Brian Mulloy** Apigee
- [Web API Design](http://pages.apigee.com/web-api-design-ebook.html) by **Brian Mulloy**, Apigee
- [Designing HTTP Interfaces and RESTful Web Services](https://www.youtube.com/watch?v=zEyg0TnieLg)
