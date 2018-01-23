# Overview
The Batching API is a subset of the full Standard Order API limiting structure to just one Address, Product, and Detail.  These records are then eventually batched into a single Order based on internally configured business logic.

## Format enforcement
A batch record request is allowed only to have

- 1 Product
- 1 Address
- 1 Detail

## API Calls
[Swagger Documentation](/OrderAPIV2.pdf)

### GET Order

The GET Order call is the URL returned in the location header on a POST Order request.   This is the unique resource URI for the order that has just been created.  

The Detail's correlationId is used to match the status values returned in the Status Report API to details in a given batch record request.

`GET /api/order-service/v2/orders/{SBS Order ID}`

`GET /api/order-service/v2/orders/?orderId={customerReferenceId}&programId={programId for tenant}`

#### Sample Response

```json
{
    "id": "MARK-MB-201801-7505",
    "customerReference": "LAWFOO14",
    "createdAt": 1516319035.575,
    "tenantId": 1,
    "programId": 1,
    "statusHistory": [
        {
            "status": "NEW",
            "message": "New order created.",
            "updatedBy": "",
            "updatedAt": "2018-01-18T23:43:56Z"
        }
    ],
    "addresses": [
        {
            "first": "Cliff",
            "last": "Lewis",
            "address1": "968 Postal Rd Ste 210",
            "address2": "Foo Bar Baz",
            "zip": "18109",
            "city": "Allentown",
            "state": "PA",
            "country": "United States",
            "statusHistory": []
        }
    ],
    "productInstances": [
        {
            "product": {
                "skus": [
                    {
                        "id": "LPC01"
                    }
                ]
            },
            "properties": {
                "print": "44",
                "coating": "C2",
                "paper": "PT08",
                "boolean": true
            },
            "templates": [
                {
                    "sku": "LPC01",
                    "surface": "front",
                    "template": {
                        "preprocess": true,
                        "type": "application/x-fusion-template+zip",
                        "url": "http://hero/csbs/templates/SharperAgent/5.5x8.5_v1.zip"
                    },
                    "variables": {
                        "Front_Back": {
                            "type": "application/pdf",
                            "url": "http://172.18.186.51/s3/sbsdev01/test/path/LPC01_Crop_Page_Fix.pdf"
                        }
                    }
                }
            ]
        }
    ],
    "details": [
        {
            "address": 0,
            "correlationId": "821baa0a-d53e-4a3f-adc6-35100068f3ed",
            "prodInst": 0,
            "quantity": 10,
            "shipMethod": "UPS:GROUND",
            "statusHistory": [
                {
                    "status": "NEW",
                    "message": "New Line Item Created",
                    "updatedBy": "",
                    "updatedAt": "2018-01-18T23:43:56Z"
                }
            ]
        }
    ]
}
```


### GET Status Report
The GET Status Report API returns the status and shipments associated to correlationId's in a given batch order.  The API takes the correlationId of a record contained in a batch.  The Status Report API will only return data once a record has been batched for processing.  Until that point it will return 404.

`GET /api/order-service/v2/orders/statusreport/?orderId={correlationId}`

#### Sample Response
```json
[
    {
        "estimatedInHomeDate": "2018-01-16T08:00:00Z",
        "quantity": 20,
        "status": "CASS_COMPLETE",
        "message": "Successfully CASS'd 1 addresses for lineItem: 7672",
        "updatedBy": "",
        "updatedAt": "2018-01-16T20:25:24Z",
        "shipments": [
            {
                "shipmentNo": "1234567",
                "shipDate": "2008-07-05 12:30:56 PDT",
                "trackingNo": "1Z78RR270318054720",
                "carrier": "UPS",
                "method": "2_DAY",
                "packagingNo": "20-200-2000-20000",
                "weight": 2,
                "expectedShipDate": "2008-07-05 12:30:56 PDT ",
                "expectedCarrier": "UPS",
                "expectedMethod": "GROUND",
                "expectedTransitDays": 4,
                "readyToShipDate": "2008-07-05 12:30:56 PDT",
                "zoneNumber": "9",
                "createdDate": "2018-01-16 12:40:30.0"
            }
        ],
        "customerReference": "LAWFOO11",
        "correlationId": "8389f36e-73f1-4104-9660-81619db73077"
    },
    {
        "estimatedInHomeDate": "2018-01-16T08:00:00Z",
        "quantity": 20,
        "status": "CASS_COMPLETE",
        "message": "Successfully CASS'd 1 addresses for lineItem: 7672",
        "updatedBy": "",
        "updatedAt": "2018-01-16T20:25:24Z",
        "shipments": [
            {
                "shipmentNo": "1234567",
                "shipDate": "2008-07-05 12:30:56 PDT",
                "trackingNo": "1Z78RR270318054720",
                "carrier": "UPS",
                "method": "2_DAY",
                "packagingNo": "20-200-2000-20000",
                "weight": 2,
                "expectedShipDate": "2008-07-05 12:30:56 PDT ",
                "expectedCarrier": "UPS",
                "expectedMethod": "GROUND",
                "expectedTransitDays": 4,
                "readyToShipDate": "2008-07-05 12:30:56 PDT",
                "zoneNumber": "9",
                "createdDate": "2018-01-16 12:40:30.0"
            }
        ],
        "customerReference": "LAWFOO11",
        "correlationId": "d823e72e-4b79-4f81-ae37-97f2534a157b"
    },
    {
        "estimatedInHomeDate": "2018-01-16T08:00:00Z",
        "quantity": 10,
        "status": "CASS_COMPLETE",
        "message": "Successfully CASS'd 1 addresses for lineItem: 7673",
        "updatedBy": "",
        "updatedAt": "2018-01-16T20:25:36Z",
        "shipments": [],
        "customerReference": "LAWFOO11",
        "correlationId": "7aabc7c9-1849-4031-81e8-8e6b6502263c"
    },
    {
        "estimatedInHomeDate": "2018-01-16T08:00:00Z",
        "quantity": 10,
        "status": "CASS_COMPLETE",
        "message": "Successfully CASS'd 1 addresses for lineItem: 7674",
        "updatedBy": "",
        "updatedAt": "2018-01-16T20:25:46Z",
        "shipments": [],
        "customerReference": "LAWFOO11",
        "correlationId": "f2f4925d-436d-4337-b1f1-4ec2166885d0"
    }
]
```


### POST Order
The POST Order API takes the JSON format for Standard Orders and based on Program Configuration the record will be treated as a Batch Order record.  So the right programId and tenantId is required on the order for it to be treated as a batch order. 

#### Sample Playload

`POST /api/order-service/v2/orders/`

```json
{
  "tenantId": 1,
  "programId": 1,
  "customerReference": "LAWFOO14",
  "addresses": [
    {
      "first": "Cliff",
      "last": "Lewis",
      "address1": "968 Postal Rd Ste 210",
      "address2": "Foo Bar Baz",
      "city": "Allentown",
      "state": "PA",
      "zip": "18109",
      "country": "United States"
    }
  ],
  "productInstances": [
    {
      "product": {
        "skus": [
          {
            "id": "LPC01"
          }
        ]
      },
      "properties": {
        "print": "44",
        "coating": "C2",
        "paper": "PT08",
        "boolean": true
      },
      "templates": [
        {
          "sku": "LPC01",
          "surface": "front",
          "template": {
            "preprocess": true,
            "url": "http://hero/csbs/templates/SharperAgent/5.5x8.5_v1.zip",
            "type": "application/x-fusion-template+zip"
          },
          "variables": {
            "Front_Back": {
              "url": "http://172.18.186.51/s3/sbsdev01/test/path/LPC01_Crop_Page_Fix.pdf",
              "type": "application/pdf"
            }
          }
        }
      ]
    }
  ],
  "details": [
    {
      "address": 0,
      "prodInst": 0,
      "quantity": 10,
      "shipMethod": "UPS:GROUND"
    }
  ]
}
```
