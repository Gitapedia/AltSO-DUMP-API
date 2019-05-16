# AltSO-Z4001 ( DUMP API )
The AltSO-Z4001 Standard is currently a concept being designed as an alternative to REST API with intentions to design an API interaction style that uses inferred methods for the data being provided allowing for more streamlined communications with an API endpoint without the need for many different routes.

## Why DUMP?
The name was a simple naming attempt for simple API design. Because of how you interact with the API by DUMPing data, the name just became apparent from how you work with the API as a DUMP of data.

## REST vs. DUMP
`REST` ( or if you prefer `RESTful` ) APIs are based on the concept of having resource based routes with URI structures to indicate identifiers. They also use HTTP standards such as METHODS and respective data as parameters or the content body.

`REST` has a place for being structured enough to lookup one or many records for a resource but with only the ability to work with a single resource record. When creating, updating, modifying and deleting, this leaves for no ability to bulk action and instead encourages a rinse and repeat approach by calling the same route endpoint many times to work with many records. The request could have been done more efficiently by calling a route endpoint once and providing many rows of data as a batch.

`DUMP` is designed to have only one route endpoint with one method, no multiple resources or methods with limiting handling for numerous records on one request. All data is contained within the request body and structured as either JSON or XML based on the supporting development flavor, and it would be recommended to support both and use a common parsing layer in the API to agnostically support at least these two structures.

`DUMP` will remove the complexity from interacting with APIs for different resources and simplify the inferred methods of CRUD allowing for a more straightforward approach to managing resource records.

## Where are the resource paths?
All data sent in the body of the request to the server is where the server will find all the information it will need to infer the resources along with other things. The records within the data are grouped into keyed arrays where the key represents the resource type, the namespace for the resource is up to the API designer, typically a slash based namespace or just the resource name is enough to build logic on the backend for handling resource identification.

## Where are the methods?
All record data sent in the body of the request to the server is where the service will find all the information it will need to infer the method along with other things. Each record will contain a structure that will help infer the intended method for that specific record.

This design is what makes DUMP APIs a robust solution as this means a single request can contain many different resources with each record inside each resource array can contain the right amount of detail and structure to infer one of the primary methods that would have otherwise needed alternative routes or route requests to achieve each action on each record if done via REST.

## Structure Example
```
[
  {
    "authors": [
      { // Create a new Author record
        "_reference": "author_record_request_id_1",
        "name": "Mr Author",
        "age": 27,
        "height": 187
      }
    ]
  },
  {
    "users": [
      { // Create a new User record
        "_reference": "user_record_request_id_1",
        "name": "Jane Smith",
        "age": 36,
        "height": 167
      }
    ]
  },
  {
    "books": [
      { // Create a new Book record
        "_reference": "book_record_request_id_1",
        "title": "How to DUMP API",
        "_attach": { // Attach possible relations to this book
          "author": {
            "_id": "author_record_request_id_1" // You can define the attachment as one of the newly created records in this request or define just "id" instead of "_id" and specify the actual record id for attaching
          },
          "user": {
            "_id": "user_record_request_id_1"
          }
        }
      }
    ]
  },
  {
    "users": [
      { // Get a User record
        "_reference": "user_record_request_id_2",
        "_id": "user_record_request_id_1",
        "_with": {
          "_attributes": [ // With specific attributes to be returned
            "name",
            "age"
          ],
          "_relations": { // And with specific relations to be returned
            "books": {
              "_take": 25, // With pagination for many relations
              "_skip": 50,
              "_with": {
                "_attributes": [ // While still supporting the request of specific attributes of the relation
                  "title"
                ],
                "_relations": { // And supporting the recurring relations of the relation
                  "author": {
                    "_with": {
                      "_attributes": [ // And still supporting the specific attributes to be returned.
                        "name",
                        "age"
                      ] // This could have also requested the relations of this relation and so on to the extent that the resource still has relations.
                    }
                  }
                }
              }
            }
          }
        }
      },
      { // Update a User record with new attribute values
        "_reference": "user_record_request_id_3",
        "_id": "user_record_request_id_2", // Defining "_id" or "id" is how we know what record you want to work with.
        "name": "Jane Smith",
        "height": 165,
        "_detach": { // Detach a relation from the User based on resource
          "book": {
            "_id": "book_record_request_id_1" // Same as above where you can define the request record id defined in this request using the "_id" or instead use "id" and specifiy a previously aware identifier for the record at the database.
          }
        }
      },
      { // Delete a User record by only defining a "_id" for use with the request record if or "id" for actual identifier.
        "_reference": "user_record_request_id_4",
        "_id": "user_record_request_id_1"
      }
    ]
  }
]
```

## Reserved request structure keys
Any reserved key will be prefixed with an underscore to help avoid conflicts with your resource attributes and to assist with identifying clear keys for other behaviours such as with, attributes, relations, attach and detaching etc.

Examples:

`_id` Used in substitution for the actual record `id` in the database where the identifier can be inferred by a request record `_reference`. You would use this in situations where you may want to create a user and their relation but also attach the user to that relationship in the one request without first knowing the `id` that will be assigned by the database. By referring to your request record `_reference`, the processing API logic will track these request record references for their true identifiers and use the correct identifier to execute the logiccal relationship `id`.

`_reference` Used to create a data reference for the given record and can be referred to elsewhere in the DUMP logic to maintain relationships for records not yet created and issued with a database identifier. This works with the `_id` key by allowing one DUMP payload to not only create individual records for various resources but to also allow setting up the relationships between relational models without needing to know their respective database identifiers which have not yet been assigned.

`_with` Used to indicate a `READ` of a record where it expects to have either an empty object meaning all attributes returned or the reserved key of `_attributes` and an array of defined attribute keys. Supports for the definition of the `_relations` key for retrieving the relationships for the record back.

`_attributes` Used to define an array of keys for the given record to influence the return data to only contain those attributes.

`_relations` Used to define additional resources where they are related to the record and returned as one or many other nested records for the current record. Relations are considered a new record query and support the use of `_with` and its respective `_attributes` and `_relations` for continuous chaining to get more relations.

`_take` and `_skip` Used with a many relationship where you need to define a limit on how many records to return.

`_attach` and `_detach` Used to add and remove records from their defined relationships with other resources where such a relationship can or does exist. Also supports the use of `_id` where the relationship is not yet known because it exists in the same request as its intended attachment or detachment.

## Work In Progress
I am still currently working through the designs of DUMP API and will be contributing more details to this Gitapedia page to help define the overall design and even open up to contributions for refining it more where it supports the DUMP concept.
