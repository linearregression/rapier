# Welcome to Rapier - REST APIs from Entities and Relationships.

## Introduction

The goals of Rapier are to allow REST APIs to be specified with one tenth the effort required with other API specification languages, and to
produce specifications that describe higher quality APIs. \[1\]

Rapier takes a data-oriented approach to API design, which fits the model of REST and the world-wide-web. If your mental model of
a web API is network of HTTP resources identified and located using URLs, you should be confortable with Rapier. If your model of a web API
consists of 'end-points' with 'parameters' (i.e. a more traditional service-oriented model), you may find the Rapier approach does not 
fit with your mental model.

You specify an API with Rapier by specifying, in a YAML file, the entities and relationships that describe the resources of the API. The details of the API's 
HTTP messages are deduced from this specification using the standard patterns described in the HTTP specifications, plus a few conventions 
that we have added. In the future we will allow more options for these add-on conventions - for now they are mostly fixed.

Rapier is for specifying new APIs. You will not be able to describe existing APIs with Rapier unless that API used the same conventions that Rapier does
and was absolutely consistent in applying them.

Rapier documents are complete API spceicifications — you can give them directly to API developers to implement servers and to app developers to 
implement clients without additional documentation other than the Rapier spec and the HTTP specs themselves. Since the Rapier specification language is not yet widely 
known and understood, we provide a tool that will generate a 
Swagger document from a Rapier specification. The Swagger documents spell out the conventions used by Rapier in a way that is familiar to many.
Once you have seen a few examples of the generated Swagger, the conventions will become quickly obvious and you will stop looking at the Swagger. 
You can stop generating the Swagger
documents, which are not required, or you may continue to generate them for integrating with tools that are Swagger-based, or for communicating with
people who know Swagger but not Rapier. Swagger will likely remain important to you for
documenting APIs that are less consistent than Rapier APIs, follow different conventions to the ones Rapier currently understands, or which follow a service-oriented rather than a data-oriented design pattern. 

In the future we intend to work on test tools,
SDK generators and server implementation frameworks.  

## Examples

Rapier is very easy to understand and learn. The easiest way is by example.

### Hello World

Here is a 'Hello-world' example in Rapier:

    info:
        title: Hello World API
        version: "0.1"
    entities:
        Hello_message:
            well_known_URLs: /message
            properties:
                text:
                    type: string
                    
The API defined by this Rapier specification exposes a single resource whose type is `Hello_message` at the URL `/message`. This resource has a single declared property called `text`.
The API does not allow this resource to be deleted, because it is well-known, but it does allow it to be
retrieved using GET and modified using PATCH. You don't have to say this explicitly — it is implied by the standard HTTP patterns and our extensions. Rapier also assumes that a GET response
includes an ETag header that must be echoed in the 'If-Match' request header of the PATCH. This catches problems when two people try to update the resource at the same time.
In Rapier APIs, the server will add
a few standard properties to the `Hello-message` entity — the `Hello-message` at `/message` will actually look like this:

    {'self_link': 'http://example.org/message',
     'id': '1234567',
     'type': 'Hello_message',
     'message': 'Hello, world'
    }
 
The Swagger document generated for the 9-line Rapier sample above can be [found here](https://revision.aeip.apigee.net/mnally/rapier/raw/master/test/swagger-hello-message.yaml). 
It contains around 120 lines, which illustrates the efficiency of Rapier. 
The Swagger document is also more complex - it contains both JSON Refs and YAML anchors and aliases to try to avoid repetition, otherwise the Swagger would be even longer.

### To-do List

Traditionally, the next example after 'Hello world' is 'To-do List':

    info:
        title: To-do List API
        version: "0.1"
    entities:
        To_do_list:
            well_known_URLs: /to-dos
            query_paths: [items]
        Item:
            properties:
                description:
                    type: string
                due_date:
                    type: date
    relationships:
        list-to-items:
            one_end:
                entity: To_do_list
                property: items
                multiplicity: 0:n
            other_end:
                entity: Item
                
This API defines a single resource at the well_known_URL `/to-dos` whose type is `To_do_list`. In the relationships section, you can see that each `To_do_list` has a property
called `items` that represents a multi-valued relationship to the `Items` of the `To_do_list`. The value of the `items` property will be a URL that points to a Collection
resource that contains information on each item of the `To_do_list`. In JSON, the `To_do_list` at `/to-dos` will actually look like this:

    {'self_link': 'http://example.org/to-dos',
     'id': '987655443',
     'type': 'To_do_list',
     'items': 'http://example.org/xxxxx'
    }
    
The Collection at `http://example.org/xxxxx` will look like this in JSON:

    {'self_link': 'http://example.org/xxxxx',
     'type': 'Collection',
     'id': '5647382',
     'contents_type': 'Item',
     'contents': [{
         'self_link': 'http://example.org/yyyyy',
         'id': '10293847',
         'type': 'Item'
         'description': 'Get milk on the way home',
         'due': '1439228983'
         }
      ]
    }
 
The combination of the `well_known_URLS` and `query_paths` properties of `To_do_list` implies that the following URL and URL template are valid:

    /to-dos/items
    /to-dos/items/{Item_id}
    
The meaning of the first URL is "the resource that is referenced by the items property of the resource at `/todos`" — we are starting at `'/todos'`
and following the `items` relationship declared in the relationships section. From this, we know that `http://example.org/xxxxx`
and `http://example.org/todos/items` are the same resource, and although there is no requirement on the server to use the same URL, most will do so.
Since `items` is from a multi-valued relationship to the `Items` entity,
Rapier conventions say that we can tack on `Item_id` on to the end of `todos/items/` to resolve to a single `Item`. From this we know that
`http://example.org/yyyyy` is equivalant to `http://example.org/items/10293847`, and that will be the URL used by most servers. 
Rapier calls `{Item_id}` a selector. A Rapier option allows the selector value to be in a path parameter instead of 
a path segment - see the 'Property Tracker' example.
  
You can POST items to `http://example.org/to-dos/items` to create new items, you can PATCH items to change them, 
and you can DELETE items to remove them. You can also perform a GET on `http://example.org/yyyyy`, which will yield:
 
    {
     'self_link': 'http://example.org/yyyyy',
     'id': '10293847',
     'type': 'Item'
     'description': 'Get milk on the way home',
     'due': '1439228983'
    }
 
If you want to see the generated Swagger document for this API specification, [it is here](https://revision.aeip.apigee.net/mnally/rapier/raw/master/test/swagger-to-do-list.yaml)
 
### Dog Tracker
 
Another popular API example is the 'Dog Tracker' example. In Rapier, it looks lke this:
 
    info:
        title: Dog-tracker API
        version: "0.1"
    conventions:
        selector_location: path-parameter
    entities:
        Dog_tracker:
            well_known_URLs: /dog-tracker
            query_paths: [dogs, people, dogs/owner, people/dogs]
        Dog:
            properties:
                name:
                    type: string
                birth_date:
                    type: string
                fur_color:
                    type: string
        Person:
            properties:
                name:
                    type: string
                birth-date:
                    type: string
    relationships:
        tracker-to-dogs:
            one_end:
                entity: Dog_tracker
                property: dogs
                multiplicity: 0:n
            other_end:
                entity: Dog
        tracker-to-people:
            one_end:
                entity: Dog_tracker
                property: people
                multiplicity: 0:n
            other_end:
                entity: Person
        dogs-to-people:
            one_end:
                entity: Person
                property: dogs
                multiplicity: 0:n
            other_end:
                entity: Dog
                property: owner
                multiplicity: 0:1
                
This API defines a single resource at the URL `/dog-tracker` whose type is `Dog_tracker`. In the relationships section, you can see that each `Dog_tracker` has properties
called `dogs` and `people` that point to the Dogs and Persons that are tracked. The value of each of these will be a URL that points to a Collection
resource that contains information on each Dog or Property. You can POST to either of these Collections to create new \[resources for\] Dogs or Persons. From the `well_known_URLs` and `query_paths` 
properties of `Dog-tracker` we know that these Collections can also be accessed at `/dog-tracker/dogs` and `/dog-tracker/people` respectively.

The API also defines a relationship between Dogs and Persons, which is called `owner` on one side and `dogs` on the other. The `owner` property is settable on each Dog - this is in fact
the only way to change which Person owns a Dog. When a Dog is created by POSTing to `/dog-tracker/dogs`, the `owner` property may be set by the client. If a Dog is POSTed to the `dogs` Collection of a specific
Person, the server will set the `owner` property appropriately.

From the `well_known_URLs` and `query_paths` properties, you can infer that the following URLs and URL templates are part of the API:

    /dog-tracker
    /dog-tracker/dogs
    /dog-tracker/dogs;{Dog_id}
    /dog-tracker/dogs;{Dog_id}/owner
    /dog-tracker/people
    /dog-tracker/people;{Person_id}
    /dog-tracker/people;{Person_id}/dogs

Since you know the pattern, you already know what all these mean, but if you want to see a generated Swagger document for this API specification, [it is here](https://revision.aeip.apigee.net/mnally/rapier/raw/master/test/swagger-dog-tracker.yaml)

### Property Tracker

The next example shows a more complex set of relationships. In this example, a Dog can be owned by a Person or an Institution and People and Institutions can own Bicycles as well as Dogs.
The [source for this example is here](https://revision.aeip.apigee.net/mnally/rapier/raw/master/test/property-tracker.yaml). 
This example strains the expressive power of Swagger - you can see a generated [Swagger document here](https://revision.aeip.apigee.net/mnally/rapier/raw/master/test/swagger-property-tracker.yaml).

### Spec Repo

Not every resource has structured content that can be expressed as JSON. Even for resources whose content can be expressed as JSON, there is sometimes a requirement to preserve the exact document format, character-by-character.
Resources with this characteristic must be updated with PUT instead of PATCH, and their properties must be stored outside of the resource content. [This sample](https://revision.aeip.apigee.net/mnally/rapier/raw/master/test/spec-hub.yaml) 
shows an example of how Rapier handles this case. Here is the [corresponding generated Swagger document](https://revision.aeip.apigee.net/mnally/rapier/raw/master/test/swagger-spec-hub.yaml).


\[1\] Following Fred Brooks, we take consistency as being the primary measure of
quality of an API. “Blaauw and I believe that consistency underlies all principles. A good architecture is consistent in the sense that, given a partial knowledge of the system, one can predict the remainder”
