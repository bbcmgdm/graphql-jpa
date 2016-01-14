GraphQL for JPA
===============

If you're already using JPA, then you already have a schema defined...don't define it again just for GraphQL!

This is a simple project to extend [graphql-java!](https://github.com/andimarek/graphql-java) and have it derive the
schema from a JPA model.  It also implements an execution platform to generate and run JPA queries based on
GraphQL queries.

While limited, there is a lot of power bundled within.  This project takes a somewhat opinionated view of GraphQL, by
introducing things like Pagination, Aggregations, and Sorting.

This project is intended on depending on very little: graphql-java, and some javax annotation packages.  The tests depend
on Spring (with Hibernate for JPA), and Spock for testing.  These tests are a good illustration of how this library might
be used, but any stack (with JPA) should be able to be utilized.

Schema Generation
-----------------

Using a JPA Entity Manager, the models are introspected, and a GraphQL Schema is built.  With this GraphQL schema,
graphql-java does most of the work, except for querying.

Pagination
----------

GraphQL does not specify any language or idioms for performing Pagination.  Therefore, this library takes an opinionated
view, similar to that of Spring.

Each model (say Human or Droid - see tests) will have two representations in the generated schema:

- One that models the Entities directly (Human or Droid)
- One that wraps the Entity in a page request (HumanPage or DroidPage)

This allows you to query for the "Page" version of any Entity, and return metadata (like total count) alongside of the
actual requested data.  For example:

    {
        HumanConnection(paginationRequest: { page: 1, size: 2 }) {
            totalPages
            totalElements
            content {
                name
            }
        }
    }

Will return:

    {
        HumanConnection: {
            totalPages: 3,
            totalElements: 5,
            content: [
                { name: 'Luke Skywalker' },
                { name: 'Darth Vader' }
            ]
        }
    }

Of course, an extra query is needed to get the total elements, so if you have not requested 'totalPages' or 'totalElements'
this query will not be executed.

NOTE: The "Connection" name is used here for further extension (Aggregations, Sorting, etc...).  The name is borrowed
from suggestions by Facebook developers: https://github.com/facebook/graphql/issues/4

Aggregations
------------

Not yet implemented, but will be similar to Pagination

Sorting
-------

Sorting is supported on any field.  Simply pass in an 'orderBy' argument with the value of ASC or DESC.  Here's an example
of sorting by name for Human objects:

    {
        Human {
            name(orderBy: DESC)
            homePlanet
        }
    }

Query Injectors
---------------

Not yet implemented.  Main use case would be to intercept query execution for security purposes.