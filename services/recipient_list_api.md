# Group Recipient Lists

A recipient list is a collection of recipients that can be used in a transmission.  The Recipient
List API provides the means to manage recipient lists.  When creating a new
transmission using the Transmissions API, the recipients may be submitted "inline" as part of the
transmission data, or a stored recipient list id attribute can be specified.

The Recipient List API operates on lists as a whole and does not currently support management of individual recipients.

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://www.getpostman.com/run-collection/81ee1dd2790d7952b76a)

## Recipient List Attributes
| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|id |string       |Short, unique, recipient list identifier |no  |If an id is not specified, one is generated.  Maximum length - 64 bytes |
|name    |string       |Short, pretty/readable recipient list display name, not required to be unique |no  |If a name is not specified, then defaults to the same value as id.  Maximum length - 64 bytes |
|description |string |Detailed description of the recipient list|no | Maximum length - 1024 bytes|
|attributes |JSON object  |Recipient list attribute object  |no  |This JSON object allows users to store arbitrary metadata related to this list.  This data is not used by the API.  It is only for the user. |
|recipients |JSON array  |Array of recipient objects |yes |For a full description, see the Recipient Attributes. |

### Recipient Attributes

Recipients are described in a JSON array with the following fields:

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|address | JSON object or string | Address information for a recipient  | At a minimum, address or devices is required | See the Address Attributes. Required for stored recipient lists. |
|devices | JSON array | Array of device objects | At a minimum, address or devices is required. | Currently only supported for inline recipient lists. See Device Attributes. |
|return_path | string |Email to use for envelope FROM ( **Note:** SparkPost Elite only )| no | To support Variable Envelope Return Path (VERP), this field provides a specific recipient a unique envelope MAIL FROM. |
|tags | JSON array |Array of text labels associated with a recipient | no | Tags are available in Webhook events.  Maximum number of tags - 10 per recipient, 100 system wide.  Any tags over the limits are ignored.|
|metadata | JSON object| Key/value pairs associated with a recipient |no | Metadata is available during events through the Webhooks and is provided to the substitution engine.  A maximum of 1000 bytes of merged metadata (transmission level + recipient level) is available with recipient metadata taking precedence over transmission metadata when there are conflicts.  |
|substitution_data | JSON object | Key/value pairs associated with a recipient that are provided to the substitution engine |no | Recipient substitution data takes precedence over transmission substitution data.  Unlike metadata, substitution data is not included in Webhook events.|

#### Address Attributes
If the "address" field is a string type, it is interpreted as the email address.  If it is a JSON
object, it is described with the following fields:  

| Field         | Type     | Description                           | Required   |
|------------------------|:-:       |---------------------------------------|-------------|
|email    |string       |Valid email address   |yes  |
|name |string |User-friendly name for the email address |no |
|header_to|string       |Email address to display in the "To" header instead of _address.email_ (for BCC)|no|

**Constructing Headers using the Address Attributes**

The _address.email_ attribute is used as the envelope RCPT TO value.

If the address attribute is specified as a JSON string instead of a JSON address object, the address JSON string is used as the envelope RCPT TO value.

The _address.name_ attribute, in conjuction with the _address.email_ attribute, is used to construct the
content "To" header.

`To: "address.name" <address.email>`

If the _address.name_ attribute is not specified, the "To" header uses the _address.email_ attribute in contructing the header.

`To: address.email`

If the address is specified as a JSON string instead of a JSON address object, the "To" header is constructed using the address JSON string.

`To: address`

If the _address.header_to_ attribute is specified, then the "To" header uses
the _address.header_to_ attribute in constructing the header.
_address.header_to_ can be used to BCC (blind carbon copy) recipients,
by hiding the envelope RCPT TO address and replacing it
with an alternative address in the "To" header.

`To: address.header_to`

or:

`To: "address.name" <address.header_to>`

The "To" header is only constructed for messages built from email part content.  The "To" header is not built for email_rfc822 content.

#### Device Attributes
| Field         | Type     | Description                           | Required   | Notes |
|------------------------|:-:       |---------------------------------------|-------------| ------------|
|token    |string       |Token used to uniquely identify a device   |yes  |Device token in APN, Registration token in GCM |
|os |string |Operating system of device identified by token |yes |Valid values are "iOS" and "Android" |

## Create [/recipient-lists{?num_rcpt_errors}]

### Create a Recipient List [POST]

Create a recipient list by providing a **recipient list object** as the POST request body.

At a minimum, the "recipients" array is required, which must contain a valid "address".  If the
recipient list "id" is not provided in the POST request body, one will be generated and returned
in the results body.  Use the **num_rcpt_errors** parameter to limit the number of recipient errors
returned.

**Note:** The "return_path" in the POST request body applies to SparkPost Elite only.

+ Parameters
  + num_rcpt_errors (optional, number, `3`) ... Maximum number of recipient errors that this call can return, otherwise all validation errors are returned.

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
          "id": "unique_id_4_graduate_students_list",
          "name": "graduate_students",
          "description": "An email list of graduate students at UMBC",
          "attributes": {
              "internal_id": 112,
              "list_group_id": 12321
          },
          "recipients": [
              {
                  "return_path": "return-path-wilmaflin@tstone.com",
                  "address": {
                      "email": "wilmaflin@yahoo.com",
                      "name": "Wilma"
                  },
                  "metadata": {
                      "place": "Bedrock"
                  },
                  "substitution_data": {
                      "subrcptkey": "subrcptvalue"
                  },
                  "tags": [
                      "greeting",
                      "prehistoric",
                      "fred",
                      "flintstone"
                  ]
              },
              {
                  "return_path": "return-path-abc@tstone.com",
                  "address": {
                      "email": "abc@flintstone.com",
                      "name": "ABC"
                  },
                  "metadata": {
                      "place": "MD"
                  },
                  "tags": [
                      "driver",
                      "computer science",
                      "fred",
                      "flintstone"
                  ]
              },
              {
                  "return_path": "return-path-def@tstone.com",
                   "address": {
                      "email": "fred.jones@flintstone.com",
                      "name": "Grad Student Office",
                      "header_to": "grad-student-office@flintstone.com"
                  },
                  "tags": [
                      "driver",
                      "computer science",
                      "fred",
                      "flintstone"
                  ]
              }
          ]
        }
        ```

+ Response 200 (application/json)

  + Body

        ```
        {
        "results": {
            "total_rejected_recipients": 0,
            "total_accepted_recipients": 3,
            "id": "unique_id_4_graduate_students_list",
            "name": "graduate_students"
        }
        }
        ```

+ Response 400 (application/json)

  + Body

        ```
        {
          "errors": [
            {
              "message": "At least one valid recipient is required",
              "code": "5002"
            }
          ]
        }
        ```

+ Response 400 (application/json)

  + Body

        ```
        {
          "errors": [
            {
              "message": "List already exists",
              "code": "5001",
              "description": "List 'unique_id_4_graduate_students_list' already exists"
            }
          ]
        }
        ```

+ Response 422 (application/json)

  + Body

        ```
        {
          "errors": [
            {
              "message": "invalid data format/type",
              "code": "1300",
              "description": "List id 'rcptlist_id_students_list' cannot start with 'rcptlist_'"
            }
          ]
        }
        ```


## Retrieve [/recipient-lists/{id}{?show_recipients}]

### Retrieve a Recipient List [GET]

Retrieve details about a specified recipient list by specifying its id in the URI path.  To
retrieve the recipients contained in a list, the list must be specified and the **show_recipients** parameter must be set to true.

**Note:** The "return_path" in the POST request body applies to SparkPost Elite only.

+ Parameters
    + id (required, string, `unique_id_4_graduate_students`) ... Identifier of the recipient list
    + show_recipients (optional, boolean, `true`) ... If set to true, return attributes for all recipients.
                                              If not specified, return only recipient list attributes.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

    + Body

        ```
        {
            "results": {
                "id": "unique_id_4_graduate_students_list",
                "name": "graduate_students",
                "description": "An email list of graduate students at UMBC",
                "attributes": {
                    "internal_id": 112,
                    "list_group_id": 12321
                },
                "total_accepted_recipients": 2,
                "recipients": [
                    {
                        "address": {
                            "email": "wilmaflin@yahoo.com",
                            "name": "Wilma"
                        },
                        "return_path": "return-path-wilmaflin@tstone.com",
                        "tags": [
                            "greeting",
                            "prehistoric",
                            "fred",
                            "flintstone"
                        ],
                        "metadata": {
                            "place": "Bedrock"
                        },
                        "substitution_data": {
                            "subrcptkey": "subrcptvalue"
                        }
                    },
                    {
                        "address": {
                            "email": "abc@flintstone.com",
                            "name": "ABC"
                        },
                        "return_path": "return-path-abc@tstone.com",
                        "tags": [
                            "driver",
                            "computer science",
                            "fred",
                            "flintstone"
                        ],
                        "metadata": {
                            "place": "MD"
                        }
                    }
                ]
            }
        }
        ```

+ Response 404 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "resource not found",
                "code": "1600",
                "description": "List 'unique_id_4_graduate_students_list' does not exist"
              }
            ]
          }
        ```

## List [/recipient-lists]

### List all Recipient Lists [GET]

List a summary of all recipient lists.  The recipients for each list are not included in the
results.  To retrieve recipient details, use the RETRIEVE API for a specified recipient list.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

    + Body

        ```
        {
        "results": [
            {
                "id": "unique_id_4_graduate_students_list",
                "name": "graduate_students",
                "description": "An email list of graduate students at UMBC",
                "attributes": {
                    "internal_id": 112,
                    "list_group_id": 12321
                },
                "total_accepted_recipients": 2
            },
            {
                "id": "unique_id_4_undergraduates",
                "name": "undergraduate_students",
                "description": "An email list of undergraduate students at UMBC",
                "attributes": {
                    "internal_id": 111,
                    "list_group_id": 11321
                },
                "total_accepted_recipients": 8
            }
        ]
        }
        ```

## Update [/recipient-lists/{id}{?num_rcpt_errors}]

### Update a Recipient List [PUT]

Update an existing recipient list by specifying its ID in the URI path and use a
**recipient list object** as the PUT request body. Use the **num_rcpt_errors** parameter to limit the number of recipient errors
returned.

The following are key points about updating your recipient lists:

* If a non-scheduled transmission contains a recipient list, the recipient list cannot
be updated if the transmission is submitted or generating.
* If a scheduled transmission contains a recipient list, the recipient list cannot be updated if the transmission is
generating or submitted and within 10 minutes of the scheduled generation time.  
* The "id" field is read only and cannot be changed.  If the recipient list "id" is provided in
the **recipient list object**, it must match the id parameter.
* If a "recipients" array is provided in the update request, it must contain the complete recipient
list and all relevant recipient fields whether they are being changed or not.  The new recipients
will completely replace the existing recipients.  The number of accepted recipients and the
number of rejected recipients will only be returned if a "recipients" array is provided in the request.
* If a "name" field is provided in the update request, it will replace the existing
"name" field for the recipient list.
* If a "description" field is provided in the update request, it will replace the existing
"description" field for the recipient list.
* If an "attributes" object is provided in the update request, it will completely replace the existing
"attributes" object for the recipient list.

+ Parameters
  + id (required, string, `unique_id_4_graduate_students_list`) ... Identifier of the recipient list
  + num_rcpt_errors (optional, number, `3`) ... Maximum number of recipient errors that this call can return, otherwise all validation errors are returned.

+ Request (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

        ```
          {
              "name": "updated_graduate_students",
              "description": "An email list of graduate students at UMBC",
              "attributes": {
                  "internal_id": 112,
                  "list_group_id": 12321
              },
              "recipients": [
                  {
                      "address": {
                          "email": "wilmaflin@yahoo.com",
                          "name": "Wilma"
                      },
                      "metadata": {
                          "place": "Bedrock"
                      },
                      "substitution_data": {
                          "subrcptkey": "subrcptvalue"
                      },
                      "tags": [
                          "greeting",
                          "prehistoric",
                          "fred",
                          "flintstone"
                      ]
                  },
                  {
                      "address": {
                          "email": "fred.jones@flintstone.com",
                          "name": "Grad Student Office",
                          "header_to": "grad-student-office@flintstone.com"
                      },
                      "tags": [
                          "driver",
                          "computer science",
                          "fred",
                          "flintstone"
                      ]
                  }
              ]
          }
        ```

+ Response 200 (application/json)

  + Body

        ```
        {
            "results": {
                "total_rejected_recipients": 0,
                "total_accepted_recipients": 2,
                "id": "unique_id_4_graduate_students_list",
                "name": "updated_graduate_students"
            }
        }
        ```

+ Response 400 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "invalid uri",
                "code": "1101",
                "description": "PUT requires a recipient list id in the URI"
              }
            ]
          }
        ```

+ Response 400 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "At least one valid recipient is required",
                "code": "5002"
              }
            ]
          }
        ```

+ Response 404 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "resource not found",
                "code": "1600",
                "description": "List 'unique_id_4_graduate_students_list' does not exist"
              }
            ]
          }
        ```

+ Response 409 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "resource conflict",
                "code": "1602",
                "description": "List 'unique_id_4_graduate_students_list' is in use by msg generation"
              }
            ]
          }
        ```

+ Response 409 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "resource conflict",
                "code": "1602",
                "description": "List 'unique_id_4_graduate_students_list' is in use by another request"
              }
            ]
          }
        ```

+ Response 422 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "invalid data format/type",
                "code": "1300",
                "description": "List id 'unique_id_4_graduate_students_list' does not match the list being updated"
              }
            ]
          }
        ```


## Delete [/recipient-lists/{id}]

### Delete a Recipient List [DELETE]

Permanently delete the specified recipient list.

Once a recipient list is deleted, it
cannot be recovered.  Before deleting a list, ensure that it is no longer needed and keep a backup copy.  If a deleted
list is needed again, the list must be resubmitted with the CREATE API.

If a transmission contains a recipient list, the recipient list cannot be deleted if the transmission is
submitted or generating.  

+ Parameters
    + id (required, string, `unique_id_4_graduate_students_list`) ... Identifier of the recipient list

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

+ Response 200 (application/json)

            {
            }

+ Response 400 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "invalid uri",
                "code": "1101",
                "description": "DELETE requires a recipient list id in the URI"
              }
            ]
          }
        ```

+ Response 404 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "resource not found",
                "code": "1600",
                "description": "List 'unique_id_4_graduate_students_list' does not exist"
              }
            ]
          }
        ```

+ Response 409 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "resource conflict",
                "code": "1602",
                "description": "List 'unique_id_4_graduate_students_list' is in use by msg generation"
              }
            ]
          }
        ```
