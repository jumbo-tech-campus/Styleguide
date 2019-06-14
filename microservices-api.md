---
title: Microservices API
subtitle: Abstract Guidelines
---

# Introduction

This document proposes a set of guidelines for structuring the APIs of our microservices, regardless of the underlying implementation platform. The expectation is that a common implementation platform is not always likely nor desirable. Still, regardless of the underlying implementation platform, we can still agree on the way services are expected to behave from the outside. This document is aiming to extract these common properties. Other documents will detail the implication of the guidelines below for specific implementation platforms.

These guidelines are further refined in implementation specific guidelines. The words MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are used in accordance with [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

# HTTP API

1. **MUST** URIs to identify resources;
2. **MUST** allow you to operate on those resources with HTTP methods;
3. **MUST** implement these methods in accordance with the semantics defined in [section 4.3 of RFC 7231](https://tools.ietf.org/html/rfc7231#section-3.3);
4. **MUST** use relevant status codes to communicate the outcome of a request, as defined in [section 6 of RFC 7231](https://tools.ietf.org/html/rfc7231#section-6);
5. **MUST** Support `application/json` as the representation of a data structure when one is expected as a payload or when the endpoint is expected to return a data structure. Other representations **MAY** be supported, and in those cases, the endpoints **MAY** rely on content negotiation to return a representation according to the preferences of the User Agent.
6. Payloads of type `application/json` **MUST** use camelcase attribute names.
6. If a resource does *not* exist, then the endpoint **MUST** return a *404 Not Found*.
7. If a resource *does* exist, but the given method is not supported, then the endpoint **MUST** return a *405 Method Not Allowed*.
8. *If* an endpoint implements content negotiation, then it **MUST** return a *406 Not Acceptable* if the requested represenation cannot be produced.
9. If a resource *does* exist and the endpoint *does* support the given method, yet the payload does not comply with expectations of the endpoint, then that endpoint **MUST** return *400 Bad Request*.
10. In case of a *POST* or a *PUT*, the endpoint should return with a *201 Created*, and set the *Location* header to the URI of a representation of the resource that was created.
10. In any of the above error situations, an endpoint **MUST** (together with the relevant status code) return a JSON document containing three attributes: `statusCode`, `error` and `message`. The `statusCode` should hold a number corresponding to the status code returned in the HTTP request. The `error` should be the standardized error message associated to that status code. The `message` should contain a message that is specific to the conditions encountered by this endpoint, and that should be helpful of callers to identify the cause of the error or at least be helpful in assisting the caller to deal with the error.

# Events

In general, we should aim to minimize blocking dependencies between services. 

1. When a service publishes an event, then the event **MUST** be published in a durable way.
2. Every event **MUST** include the type of event, and **MAY** include a payload specifying details of the event, and an identifier of the producer including its version number.
3. Consumers of events **MUST** explicitly acknowledge receiving the event.
4. Events **MUST** be published in such a way that *does not* preclude responding to events by various consumers.
5. Events that have relevance outside the boundaries of the microservices architecture discussed in this document **MUST** be published on an internal persistent queue before an attempt is made to hand it off to the external service. 
6. In case of delivery failures, the underlying architecture **MUST** implement a retry mechanism with exponential back-off. 

# Versioning

1. A service **MUST** have a version number composed out of three version numbers, in accordance with [Semantic Versioning 2.0.0](https://semver.org/): a major and minor version number, and a patch number.
2. The *major* version number of the API **MUST* be encoded in the base URI of the service, e.g. `v1/catalog`.
3. In case of breaking API changes, the major version **MUST** be increased and raised by one, e.g. `v1/catalog` to `v2/catalog`

# Documentation

1. A service **MUST** publish a swagger document detailing the usage of an endpoint, it's pre- and postconditions.
2. If an endpoint requires an access token with a specific scope or set of scopes, then that **MUST** be included in the swagger file. (The specific ways of how to do that have not been defined yet.)
3. Services **MAY** use markdown for textual snippets included in swagger. (Even though swagger itself does not make any specific guarantees on markdown processing, all tools we examined in the past *do* support markdown.)
4. Services **MUST** document the different status codes that it expects to return, with the exception of status codes that can *always* be expected to be returned in case of programming errors or system failure, such as a `500 Internal Server Error`.


# Access Control

Not all of our microservices are necessarily exposed and accessible to everything on the Internet. In fact, in many cases, they will target a very specific subset of all potential clients. Some services will only enable other services in the same *zone*. Others will be exposed to Jumbo-only services of another department. Others will be exposed to our partners, or to mobile clients. And then there is a subset of services and endpoints facilitating our web clients. 

![Circles of trust](./circles.svg)

That also mean there is no single authorization and authentication model accommodating all of these different scenarios. Detailing the specifics for each of these scenarios is still in the works. Consequently, this section will grow over time. The access control model for services within our own domain is outside the scope of this document, since it's linked specifically to our deployment architecture. 

Having said that, [OpenID Connect](https://en.wikipedia.org/wiki/OpenID_Connect) has a large role to play in addressing these concerns, specifically for web clients, mobile clients and partner clients. For interdepartmental dependencies, it's not precluded that there will be an alternative way to establish trust. For internal service dependencies, there will *surely* be another way to enforce access control.

1. Callers of type 1, 2 and 3 **MUST** pass a JWT based access token in the `Authorization header of a request.
2. The access token **MUST** be provided by an OpenID Connect compliant Identity Provider, in accordance with [RFC 6749](https://tools.ietf.org/html/rfc6749). (This applies both to situation in which the API call is made *on behalf* of a user, be it a customer or employee, but also the case where the caller is a trusted service, in which case the access token would be retrieved using the Client Credentials Grant.)
3. Services **MUST** validate the signature of the JWT passed in, and return a `401 Unauthorized` if it fails validation. Implementations are free to rely on an API gateway to do the JWT signature validation, however this is not required and may complicated testing the service locally.
4. Services **MUST** check if the scopes passed with the JWT are sufficient to grant access to the resource accessed, and if not return a `401 Unauthorized`. Implementations are encouraged to build the access control into the endpoints itself, since that's the place the entire context will be available and can be checked against service specific policies.
5. Services **MAY** forward the JWT used to make the original call  when they are delegating some of the work to other services. Since we want to *prevent* blocking calls between services as much as possible, this has specifial significance in situations where services to an asynchronous hand-off to a bus. 
