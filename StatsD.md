

# Metrics styleguide

## Incoming API requests

Statistics for requests handled by the service.

Type: `statsd timing`

Stat name: `request`

Creates: count, 95p, max, min, avg

Tags:

- env // Will be added automatically
- service: Name of the service sending the statistic
- method
- path // Path should not include ID's of resources. In Hapi use `request.route.path`
- statuscode

## Outgoing requests

Statistics for requests going from the service to another service

Type: `statsd timing`

Stat name: `external` // Up for discussion, currently httprequest

Creates: count, 95p, max, min, avg

Tags:

- env // Will be added automatically
- service: Name of the service sending the statistic
- external_service: Name of external service 
- method
- url
- statuscode
- result
  result of the request, (badrequest, failed, internal, success)
  The result is based on statuscode. If no response is received (timeout or socket hangup) result should be failed.

## Queries

statsd timing for queries, (MongoDB, Couchbase, or other databases)

type: `statsd timing`

Stat name: `query`

Creates: count, 95p, max, min, avg

Tags:

- env // Will be added automatically
- service: Name of the service sending the statistic
- Database type
- Query Type
  eq: find, create, select, getall, update, findAll
- Result