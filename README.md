# Public Order API (alpha)

[Details](#specific-details)
[Order Status](#order-status)

## Introduction

If you're interested in being able to efficiently, cost-effectively, and
automatically print and ship personalized items to your customers or prospects,
then Shutterfly Business Services is creating a platform designed to serve your
needs.

We're designing an API-first platform that exposes the ability to set up and
ship customized products to serve the personalized printing or marketing needs
of most organizations, and to allow customers to integrate directly into their
automated systems to maximize your ROI.

This documentation covers the initial public alpha API we'll be releasing for
existing clients to programatically create new print orders. We designed the API
and formats to be RESTful (stateless and resource-oriented), and flexible so
that they can serve multiple use cases, including a flexible many-to-many
mapping of print products to customer shipping destinations.

## Principles behind the data format

- Flexibility
  - We've allowed for both inline and externalized resource files in most
      places. If you'd like to specify a file for addresses, or send a JSON
      array, either are acceptable as long as they contain the required data
      fields.
- Many-to-many mapping of products+data to destination addresses
  - Define product instances (product reference + any variable config)
  - Define address list(s)
  - Map between specific final products (or groups) and individual ship
      addresses

The fundamental top-level structure is as follows:
```json
{
  ... // top-level order-specific details (e.g. your tenant id, program id, unique customer reference, etc.)
  "addresses": [], // Inline list or {url, type} specification
  "productInstances": [], // Define products, mfg props, template data (and/or sources)
  "details": [] // Map individual or groups of product instances to individual or groups of destinations.
}
```

## Specific details

### Top-level fields

- `tenantId` is the tenant id assigned to you by the engineering team during
    onboarding. This must match your assigned tenancy from the security token,
    or the system will reject your order.
- `programId` is the id of the program for which you want to print. Depending on
    your use cases, this may mean different things, but each program will have
    specific configuration and possibly specific pricing details, usually based
    on negotiated contractual requirements.
- `customerReference` is a required unique field. We require this field for
    legacy reasons, but exists to allow you to associate an order with an
    origination point in your system, should you need to. If you have no
    originating unique ID, we recommend using a sufficiently random value to
    avoid collisions, or an auto-incrementing value, depending on your use case.

### Reference conventions
We have defined our data format to flexibly support many ways of specifying the
same things. For many fields in our objects, you may specify inline JSON data or
refer to an outside data source via an object with `url` and `type` (mimeType)
fields (`{"url": "http://foo.bar/file.zip", "type":
"application/sbs-template+zip"}`).

For example, the `addresses` top-level field may be either a list of inline JSON
address data, or a reference to an external url/type pair that our system will
load and treat the same as the inline list. This is useful for lazy-loading data
at the last possible moment for composition or print quantities.

For another example, in the `details` objects, you may specify `address` as a
number, which we treat as an index into the top-level `addresses` array, or you
may send a url/type object, which refers to an externalized resource containing
address data.

### Address format

For external address URLs, we currently only support the mime type
`text/sbsaddr+csv`. Our formatter requires specification of the header names
must match [the example document](/addresstest.csv). The acceptable JSON fields
match exactly (case-sensitive) the header names from the CSV format.

For inline JSON, `addresses` should be an array of JSON objects containing
address data fields. See the headers in the above example document, or the
inline [example](#example-bulk-order) [documents](#direct-mail-example) below to
see the acceptable field names for address data.


### Product specifications

We only currently support valid SBS skus as the `product` specification. A
Shutterfly employee will help you set up any SKUs required for your program. The
SKU ids will vary by tenant and program.  The following is an example of how to
specify your product sku in the `productInstance`.

```
{
  "product": {
    "skus": [ { "id": "LPC01" } ]
  }
}
```

Eventually, we expect to create a product service that will encapsulate these
details, and pass references to the specific product, as configured by you,
rather than requiring the back-and-forth of setting up internal SKU codes that
is currently required.

To specify template data or metadata, use the  `templates` field in the
`productInstance` object, and specify the sku ID and insert any variable data
under the `variables` object. The key should be the template field name, and the
value the replacement value. To refer to a file, e.g. for inserting personalized
art assets, the value must use our common `{url:, type:}` JSON format to specify
the URL and MIME type of your asset.

### Many-to-many `details` mapping

The `details` section of the JSON object is for mapping product instances, along
with their template data, to specific destinations, along with a quantity and
shipping speed. If the `address` specified in the detail object is a CSV
reference, then the `quantity` applies per recipient. That is, the total
quantity will be `quantity` * `len(addressList)`.

This general format should feel very familiar if you've created many-to-many
relationships in SQL before, where normalized data modeling requires a junction
table. You may also recognize this as an adjacency list.

The end result is that users can map any composed/printed final product
(`productInstance`) to any specified destination, in any quantity.

### Simple 1:1 product/address example order

```
{
  "tenantId": 1,
  "programId": 1,
  "customerReference": "SomeUniqueReference",
  "addresses": [{
    "first": "John",
    "last": "Doe",
    "address1": "1234 Test Street",
    "address2": "Foo Bar Baz",
    "city": "Allentown",
    "state": "PA",
    "zip": "18109",
    "country": "United States"
  }],
  "productInstances": [{
    "product": {
      "skus": [ { "id": "LPC01" } ]
    },
    "properties": {
      "print": "44",
      "coating": "C2",
      "paper": "PT08"
    },
    "templates": [{
      "sku": "LPC01",
      "variables": {
        "Front_Back": {
          "url": "http://127.0.0.1/s3/example_art_asset.pdf",
          "type": "application/pdf"
        }
      }
    }]
  }],
  "details": [{
    "address": 0,
    "prodInst": 0,
    "quantity": 1,
    "shipMethod": "UPS:GROUND"
  }]
}
```



### Example bulk order
```json
{
  "tenantId": 1,
  "programId": 1,
  "customerReference": "SomeUniqueReference",
  "addresses": [{
    "first": "John",
    "last": "Doe",
    "address1": "1234 Test Street",
    "address2": "Foo Bar Baz",
    "city": "Allentown",
    "state": "PA",
    "zip": "18109",
    "country": "United States"
  }, {
    "first": "John",
    "last": "Lewis",
    "address1": "1234 Test Street",
    "address2": "Foo Bar Baz",
    "city": "Allentown",
    "state": "PA",
    "zip": "18109",
    "country": "United States"
  } ],
  "productInstances": [{
    "product": {
      "skus": [ { "id": "LPC01" } ]
    },
    "properties": {
      "print": "44",
      "coating": "C2",
      "paper": "PT08"
    },
    "templates": [{
      "sku": "LPC01",
      "variables": {
        "Front_Back": {
          "url": "http://127.0.0.1/s3/example_art_asset.pdf",
          "type": "application/pdf"
        }
      }
    }]
  }, {
    "product": {
      "skus": [ { "id": "LPC02" } ]
    },
    "properties": {
      "print": "44",
      "coating": "C2",
      "paper": "PT08",
      "boolean": true
    },
    "templates": [{
      "SKU": "LPC02",
      "template": {
        "preprocess": true,
        "url": "http://foo.test/override-template.zip",
        "type": "application/x-fusion-template+zip"
      },
      "variables": {
        "Front_Back": {
          "url": "http://127.0.0.1/s3/example_art_asset.pdf",
          "type": "application/pdf"
        }
      }
    }]
  }],
  "details": [{
    "address": 0,
    "prodInst": 0,
    "quantity": 10,
    "shipMethod": "UPS:GROUND"
  }, {
    "address": 0,
    "prodInst": 1,
    "quantity": 10,
    "shipMethod": "UPS:GROUND"
  }, {
    "address": 1,
    "prodInst": 0,
    "quantity": 10,
    "shipMethod": "UPS:GROUND"
  }, {
    "address": 1, // doesn't matter that address/prodInst combo repeats
    "prodInst": 0,
    "quantity": 10,
    "shipMethod": "UPS:GROUND"
  }, {
    "address": 1,
    "prodInst": 0,
    "quantity": 10,
    "shipMethod": "UPS:2DAY"
  }]
}
```

### Direct mail example

```json
{
  "tenantId": 2,
  "programId": 2,
  "customerReference": "askldjfasdf",
  "productInstances": [
    {
      "product": { "skus": [ { "id": "LPC01" } ] },
      "properties": {
        "print": "44",
        "coating": "C2",
        "paper": "PT08"
      },
      "templates": [{
        "sku": "LPC01",
        "variables": {
          "Front_Back": {
            "url": "http://somehost.test/LPC01_Crop_Page_Fix.pdf",
            "type": "application/pdf"
          }
        }
      }]
    }
  ],
  "details": [{
    "address": {
      "url": "https://raw.githubusercontent.com/sfly-sbs/standard-ingestion-docs/master/addresstest.csv",
      "type": "text/sbsaddr+csv"
    },
    "prodInst": 0,
    "quantity": 10,
    "shipMethod": "UPS:GROUND"
  }]
}
```

## Order status

The current implementation does expose status details at a high level, in most
cases. When you create an order successfully, the response body will indicate
both the order ID and the URL you can query to get the current status.
Eventually, status should be granularly returned at the level of each detail
object. Currently, however, we only return the order-level statuses (which will
not be granular to each recipient, etc.), and we duplicate the same events
across each object in the details array. This should simulate the eventual
behavior, as eventually the same status API will be available, but at a
correctly-granular level, and will actually correspond to each detail level
rather than being broadly applicable to the entire order and duplicated.
