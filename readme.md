# LeadPages HTTP API Standards

This document has been adapted from [a repository](https://github.com/WhiteHouse/api-standards)
originally published by The White House, and will be continually updated to
reflect API Standards at LeadPages. We are thankful to be able to use their
work as a guide. Please refer to the [credits](#credits) for further
information.

## Contents

- [Overview](#overview)
- [Base URL](#base-url)
- [Resources](#resources)
- [HTTP Verbs](#http-verbs)
- [Responses](#responses)
- [Error handling](#error-handling)
- [Versions](#versions)
- [Record Limits](#record-limits)
- [Mock Responses](#mock-responses)
- [JSONP](#jsonp)
- [Contributing](#contributing)
- [Credits](#credits)

## Overview

LeadPages APIs embrace several core ideas that guide how our APIs present
themselves. The aim of this document is to document those ideas, whether it be
through concrete examples or prose. The rest of the overview provides some
detail on what those core ideas are.

### APIs Are User Experiences
We place an emphasis on developer experience, both for others and ourselves.
We are our own API consumers, and we must be efficient and concerted in both
our creation and use of APIs.

### APIs Are Designed
An API presents itself as the best representation of something. It is
structured, thought-out, canonical, and has semantic importance. An API could
be [considered a contract](http://apievangelist.com/2014/07/15/an-api-definition-as-the-truth-in-the-api-contract/),
or putting your best foot forward. These are justifications for utilizing a
design process. This document helps in the design process by guiding the process
of deciding "what goes where and why" and moving the "why" out of the core
development process. This helps keep the API design process measured and
repeatable.

### APIs Are Opinionated
This document provides guidelines and examples for LeadPages HTTP APIs,
encouraging consistency, maintainability, and best practices across
applications. Best practices may come with scenarios that pit The Right Way™
against the pragmatic way. LeadPages APIs aim to balance the rigor and
structure of a REST API with a positive developer experience. We'll always go
for the best solution when it's practical to do so.

### APIs Are Interfaces

It's in the name. APIs are a developer's user interface to a service. When
tough questions arise about the design of the API, we can defer to experts.
Jakob Nielsen's "Usability Heuristics for User Interface Design" have provided
designers with research-based common ground and apply well to developers using
an API.

Reading [the heuristics themselves](http://www.nngroup.com/articles/ten-usability-heuristics/)
is the ideal starting point, but they are summarized here.

- Visibility of system status
- Match between system and the real world
- User control and freedom
- Consistency and standards
- Error prevention
- Recognition rather than recall
- Flexibility and efficiency of use
- Aesthetic and minimalist design
- Help users recognize, diagnose, and recover from errors
- Help and documentation

## Base URL

The base URL of an API resource reflects a few key considerations. The first
example base URL is the primary case.

Examples of acceptable base URLs:

- `https://api.company.com/data/v1`
- `https://product.company.com/api/v1`
- `https://company.com/api/v1`

For clarification, here are the base URLs in the context of a full resource
URL. These are **not** base URLs:

- `https://api.company.com/data/v1/widgets/a1b2c3`
- `https://product.company.com/api/v1/widgets/a1b2c3`
- `https://company.com/api/v1/widgets/a1b2c3`


What follows are high-level things to notice in these URLs, and they are
ordered by first appearance.

### SSL/TLS
The protocol (`https`) is specified in the base URL. This is purposeful.
SSL/TLS is a minimum requirement and specifying a base URL beginning with
`http` is invalid. There are [lots](http://googleonlinesecurity.blogspot.com/2014/08/https-as-ranking-signal_6.html)
of [resources](http://stackoverflow.com/questions/548029/how-much-overhead-does-ssl-impose)
and [discussions](http://code.flickr.net/2014/04/30/flickr-api-going-ssl-only-on-june-27th-2014/)
about [why](https://blog.cloudflare.com/how-cloudflare-is-making-ssl-fast/)
SSL/TLS only is a good choice and why others choose that path, and addressing
concerns about speed and overhead.

### Domain Scheme
The API in this example is hosted on a subdomain dedicated to a set of APIs.
This distinction is important, because it necessitates the idea of an API
name, discussed below.
  - If there are multiple APIs, and they make sense grouped under the company
  as an entity, having several named APIs is a logical grouping.
  - If the subdomain is a specific product, service, or similar, and there are
  very few of them, the second example may make more sense. In this case, the
  API name is replaced with the `api` fragment to indicate that this is the
  sole, self-contained API for this item. This API should be comprehensive and
  should not overlap functionality with other APIs located on other subdomains.
  - If the company, product, and API can be considered a singular unit, the
  third example, with no subdomain, is sensible.

### API Names
This API is named `data`. You would call this the "Data API" based on the
fragment in the URL. You could also have API names like `admin` or `tasks`
that you would call the "Administration API" and "Tasks API," respectively.
The goal of having an API name in the path is to provide a sensible grouping
of top-level objects. In an Admin API, you might have top-level objects like
"users" that would not make sense if placed next to "tasks" in a Tasks API.
In the full resource URL example above, the top-level objects are `widgets`.

### Versioning
A version (`v1`) is specified in the URL. API versioning helps ease
transitions when there are breaking changes to the interface put forward by
an API, and makes for smoother and more straightforward deprecation plans. If
an API update will break clients and implementations, at a minimum, we want to
know how many will be affected, and a versioned API is the place to
start for finding that information.

Version tags begin with a `v` and end with a a positive integer and have
nothing in between. Requests without a version tag are invalid and must be
rejected.

There is a semantic "shortcut" version that is also valid. An API must always
provide a version called `latest` that maps to the latest version of the API,
though when the API returns responses that contain the base URL, they must
replace the `latest` token with the latest version (for example, `v3`). This
helps establish the idea that the record returned is canonical for a given
version. Think of this in a concrete example: if a record is returned and
stored, and then a `self` URL (containing `latest`) is later accessed, the
returned resource's schema could have changed since the last access. Thus, the
idea that the stored record is the "latest" is incorrect.

Some valid examples of versions:
- `v1`
- `v2`
- `v3`
- `latest`

Some invalid examples of versions:
- `v1.0`
- `ver1`
- `current`
- `v1beta`

## Resources

Resources are represented by a path that follows a base URL. You can consider
the resource path as the canonical path for a resource, despite the base URL.

These are the full URLs from the base URL section:

- `https://api.company.com/data/v1/widgets/a1b2c3`
- `https://product.company.com/api/v1/widgets/a1b2c3`
- `https://company.com/api/v1/widgets/a1b2c3`

Note that if you strip the base URL from the front, they are now all the same:

- `/widgets/a1b2c3`
- `/widgets/a1b2c3`
- `/widgets/a1b2c3`

What follows are some more specific guidelines around resource paths.

### General

Here are some basics for RESTful URLs:

- A URL identifies a resource.
- URLs must use plural nouns, not verbs.
- Do not use "formats" in the URL. Some APIs place a type, like `.json`, at
the end of the URL.

### Collections, Resources, and Nesting

Given a resource path like...

`/widgets/a1b2c3/sprockets`

...you can note several things:

- `widgets` and `sprockets` are collections. They are plural nouns.
Collections hold resources and relate to the "type" of something.
- Some specific resources, like this one (`a1b2c3`) may themselves hold
collections (in this case, `sprockets`). Only in extrodinary situations should
you consider nesting any further, and you should not specify a resource after
the secondary collection (like `/widgets/a1b2c3/sprockets/d4f5g6`).
- If you're using a nested resource, like `/widget/a1b2c3/sprockets`, note that
this is a strong implication that `sprockets` **must** be created as a child of
a single `widget` (`a1b2c3`). What this then implicates is that you may not,
in general, create a `sprocket` at the root `sprocket` endpoint. The root `sprocket`
endpoint may list all of the current context's `sprocket`s, or similar, but in
general should not accept creation requests. In concrete examples:
  - `POST /sprockets` is invalid.
  - `POST /widgets/a1b2c3/sprockets` will create a `sprocket` that may be
  returned as a property within `/widgets/a1b2c3`.
  - `GET /widgets/a1b2c3/sprockets` will return all the `sprocket`s that are
  in the collection for the `a1b2c3` widget.
  - `GET /sprockets/d4f5g6` will return a specific `sprocket`.
  - `GET /sprockets` may return all `sprocket`s, regardless of `widget`
  collection.

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

Additionally, this document has been created and improved thanks to the
following people:

- [bendemaree](https://github.com/bendemaree)
- [brechin](https://github.com/brechin)
- [kristenwomack](https://github.com/kristenwomack)
- [rskm1](https://github.com/rskm1)
