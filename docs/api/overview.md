# What is a REST API?

REST stands for [representational state transfer](https://en.wikipedia.org/wiki/Representational_state_transfer). It's a particular type of API which employs HTTP to create, retrieve, update, and delete objects from a database. (This set of operations is commonly referred to as CRUD.) Each type of operation is associated with a particular HTTP verb:

* `GET`: Retrieve an object or list of objects
* `POST`: Create an object
* `PUT` / `PATCH`: Modify an existing object. `PUT` requires all mandatory fields to be specified, while `PATCH` only expects the field that is being modified to be specified.
* `DELETE`: Delete an existing object

The NetBox API represents all objects in [JavaScript Object Notation (JSON)](http://www.json.org/). This makes it very easy to interact with NetBox data on the command line with common tools. For example, we can request an IP address from NetBox and output the JSON using `curl` and `jq`. (Piping the output through `jq` isn't strictly required but makes it much easier to read.)

```
$ curl -s http://localhost:8000/api/ipam/ip-addresses/2954/ | jq '.'
{
  "custom_fields": {},
  "nat_outside": null,
  "nat_inside": null,
  "description": "An example IP address",
  "id": 2954,
  "family": 4,
  "address": "5.101.108.132/26",
  "vrf": null,
  "tenant": null,
  "status": {
    "label": "Active",
    "value": 1
  },
  "role": null,
  "interface": null
}
```

Each attribute of the NetBox object is expressed as a field in the dictionary. Fields may include their own nested objects, as in the case of the `status` field above. Every object includes a primary key named `id` which uniquely identifies it in the database.

# Interactive Documentation

Comprehensive, interactive documentation of all API endpoints is available on a running NetBox instance at `/api/docs/`. This interface provides a convenient sandbox for researching and experimenting with NetBox's various API endpoints and different request types.

# URL Hierarchy

NetBox's entire API is housed under the API root at `https://<hostname>/api/`. The URL structure is divided at the root level by application: circuits, DCIM, extras, IPAM, secrets, and tenancy. Within each application, each model has its own path. For example, the provider and circuit objects are located under the "circuits" application:

* /api/circuits/providers/
* /api/circuits/circuits/

Likewise, the site, rack, and device objects are located under the "DCIM" application:

* /api/dcim/sites/
* /api/dcim/racks/
* /api/dcim/devices/

The full hierarchy of available endpoints can be viewed by navigating to the API root in a web browser.

Each model generally has two views associated with it: a list view and a detail view. The list view is used to request a list of multiple objects or to create a new object. The detail view is used to retrieve, update, or delete an existing object. All objects are referenced by their numeric primary key (`id`).

* /api/dcim/devices/ - List devices or create a new device
* /api/dcim/devices/123/ - Retrieve, update, or delete the device with ID 123

Lists of objects can be filtered using a set of query parameters. For example, to find all interfaces belonging to the device with ID 123:

```
GET /api/dcim/interfaces/?device_id=123
```

# Serialization

The NetBox API employs three types of serializers to represent model data:

* Base serializer
* Nested serializer
* Writable serializer

The base serializer is used to represent the default view of a model. This includes all database table fields which comprise the model, and may include additional metadata. A base serializer includes relationships to parent objects, but **does not** include child objects. For example, the `VLANSerializer` includes a nested representation its parent VLANGroup (if any), but does not include any assigned Prefixes.

```
{
    "id": 1048,
    "site": {
        "id": 7,
        "url": "http://localhost:8000/api/dcim/sites/7/",
        "name": "Corporate HQ",
        "slug": "corporate-hq"
    },
    "group": {
        "id": 4,
        "url": "http://localhost:8000/api/ipam/vlan-groups/4/",
        "name": "Production",
        "slug": "production"
    },
    "vid": 101,
    "name": "Users-Floor1",
    "tenant": null,
    "status": {
        "value": 1,
        "label": "Active"
    },
    "role": {
        "id": 9,
        "url": "http://localhost:8000/api/ipam/roles/9/",
        "name": "User Access",
        "slug": "user-access"
    },
    "description": "",
    "display_name": "101 (Users-Floor1)",
    "custom_fields": {}
}
```

## Related Objects

Related objects (e.g. `ForeignKey` fields) are represented using a nested serializer. A nested serializer provides a minimal representation of an object, including only its URL and enough information to display the object to a user. When performing write API actions (`POST`, `PUT`, and `PATCH`), related objects may be specified by either numeric ID (primary key), or by a set of attributes sufficiently unique to return the desired object.

For example, when creating a new device, its rack can be specified by NetBox ID (PK):

```
{
    "name": "MyNewDevice",
    "rack": 123,
    ...
}
```

Or by a set of nested attributes used to identify the rack:

```
{
    "name": "MyNewDevice",
    "rack": {
        "site": {
            "name": "Equinix DC6"
        },
        "name": "R204"
    },
    ...
}
```

Note that if the provided parameters do not return exactly one object, a validation error is raised.

## Brief Format

Most API endpoints support an optional "brief" format, which returns only a minimal representation of each object in the response. This is useful when you need only a list of the objects themselves without any related data, such as when populating a drop-down list in a form.

For example, the default (complete) format of an IP address looks like this:

```
GET /api/ipam/prefixes/13980/

{
    "id": 13980,
    "family": 4,
    "prefix": "192.0.2.0/24",
    "site": null,
    "vrf": null,
    "tenant": null,
    "vlan": null,
    "status": {
        "value": 1,
        "label": "Active"
    },
    "role": null,
    "is_pool": false,
    "description": "",
    "tags": [],
    "custom_fields": {},
    "created": "2018-12-11",
    "last_updated": "2018-12-11T16:27:55.073174-05:00"
}
```

The brief format is much more terse, but includes a link to the object's full representation:

```
GET /api/ipam/prefixes/13980/?brief=1

{
    "id": 13980,
    "url": "https://netbox/api/ipam/prefixes/13980/",
    "family": 4,
    "prefix": "192.0.2.0/24"
}
```

The brief format is supported for both lists and individual objects.

## Static Choice Fields

Some model fields, such as the `status` field in the above example, utilize static integers corresponding to static choices. The available choices can be retrieved from the read-only `_choices` endpoint within each app. A specific `model:field` tuple may optionally be specified in the URL.

Each choice includes a human-friendly label and its corresponding numeric value. For example, `GET /api/ipam/_choices/prefix:status/` will return:

```
[
    {
        "value": 0,
        "label": "Container"
    },
    {
        "value": 1,
        "label": "Active"
    },
    {
        "value": 2,
        "label": "Reserved"
    },
    {
        "value": 3,
        "label": "Deprecated"
    }
]
```

Thus, to set a prefix's status to "Reserved," it would be assigned the integer `2`.

A request for `GET /api/ipam/_choices/` will return choices for _all_ fields belonging to models within the IPAM app.

# Pagination

API responses which contain a list of objects (for example, a request to `/api/dcim/devices/`) will be paginated to avoid unnecessary overhead. The root JSON object will contain the following attributes:

* `count`: The total count of all objects matching the query
* `next`: A hyperlink to the next page of results (if applicable)
* `previous`: A hyperlink to the previous page of results (if applicable)
* `results`: The list of returned objects

Here is an example of a paginated response:

```
HTTP 200 OK
Allow: GET, POST, OPTIONS
Content-Type: application/json
Vary: Accept

{
    "count": 2861,
    "next": "http://localhost:8000/api/dcim/devices/?limit=50&offset=50",
    "previous": null,
    "results": [
        {
            "id": 123,
            "name": "DeviceName123",
            ...
        },
        ...
    ]
}
```

The default page size derives from the [`PAGINATE_COUNT`](../configuration/optional-settings/#paginate_count) configuration setting, which defaults to 50. However, this can be overridden per request by specifying the desired `offset` and `limit` query parameters. For example, if you wish to retrieve a hundred devices at a time, you would make a request for:

```
http://localhost:8000/api/dcim/devices/?limit=100
```

The response will return devices 1 through 100. The URL provided in the `next` attribute of the response will return devices 101 through 200:

```
{
    "count": 2861,
    "next": "http://localhost:8000/api/dcim/devices/?limit=100&offset=100",
    "previous": null,
    "results": [...]
}
```

The maximum number of objects that can be returned is limited by the [`MAX_PAGE_SIZE`](../configuration/optional-settings/#max_page_size) setting, which is 1000 by default. Setting this to `0` or `None` will remove the maximum limit. An API consumer can then pass `?limit=0` to retrieve _all_ matching objects with a single request.

!!! warning
    Disabling the page size limit introduces a potential for very resource-intensive requests, since one API request can effectively retrieve an entire table from the database.

# Filtering

A list of objects retrieved via the API can be filtered by passing one or more query parameters. The same parameters used by the web UI work for the API as well. For example, to return only prefixes with a status of "Active" (`1`):

```
GET /api/ipam/prefixes/?status=1
```

The choices available for fixed choice fields such as `status` are exposed in the API under a special `_choices` endpoint for each NetBox app. For example, the available choices for `Prefix.status` are listed at `/api/ipam/_choices/` under the key `prefix:status`:

```
"prefix:status": [
    {
        "label": "Container",
        "value": 0
    },
    {
        "label": "Active",
        "value": 1
    },
    {
        "label": "Reserved",
        "value": 2
    },
    {
        "label": "Deprecated",
        "value": 3
    }
],
```

For most fields, when a filter is passed multiple times, objects matching _any_ of the provided values will be returned. For example, `GET /api/dcim/sites/?name=Foo&name=Bar` will return all sites named "Foo" _or_ "Bar". The exception to this rule is ManyToManyFields which may have multiple values assigned. Tags are the most common example of a ManyToManyField. For example, `GET /api/dcim/sites/?tag=foo&tag=bar` will return only sites tagged with both "foo" _and_ "bar".

## Custom Fields

To filter on a custom field, prepend `cf_` to the field name. For example, the following query will return only sites where a custom field named `foo` is equal to 123:

```
GET /api/dcim/sites/?cf_foo=123
```

!!! note
    Full versus partial matching when filtering is configurable per custom field. Filtering can be toggled (or disabled) for a custom field in the admin UI.
