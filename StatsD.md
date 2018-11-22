

# Metrics styleguide

This styleguide is used so all microservices will send the same kind of statistics.

## Incoming API requests

Statistics for requests handled by the service.

Type: `statsd timing`

Stat name: `api.incoming_request`

Creates: count, 95p, max, min, avg

Tags:

- `service` // Name of the service sending the statistic
- `method`
- `path`
  Path should not include ID's of resources. In Hapi use `request.route.path`
  Example Hapi: /v0/product-lists/social-lists/{listID}
  Example Restify: /v4/product/:id
- `statuscode`
- `result`
  result of the request, (badrequest, failed, internal, success)
  The result is based on statuscode. 

## Outgoing requests

Statistics for requests going from the service to another service

Type: `statsd timing`

Stat name: `api.outgoing_request`

Creates: count, 95p, max, min, avg

Tags:

- `service` // Name of the service sending the statistic
- `external_service` // Name of external service 
- `method`
- `url`
- `statuscode`
- `result`
  result of the request, (badrequest, failed, internal, success)
  The result is based on statuscode. If no response is received (timeout or socket hangup) result should be failed.

## Queries

statsd timing for queries, (MongoDB, Couchbase, or other databases)

type: `statsd timing`

Stat name: `api.query`

Creates: count, 95p, max, min, avg

Tags:

- `service` // Name of the service sending the statistic
- `database_type`
- `database_name`
- `query_type`
  eq: find, create, select, getall, update, findAll. Different query names per different database
- `result` // succes, failed

## Uncaught Exceptions

Statsd Increment for measuring uncaught exceptions. This will allow us to make alerts for uncaught exceptions.

type: `statsd increment`

Stat name: `api.uncaught_exception`

Tags:

* `service` // Name of the service sending the statistic