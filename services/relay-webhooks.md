title: Relay Webhooks
description: Manage relay webhooks, a way to instruct SparkPost to accept inbound email on your behalf and forward it to you over HTTP.

# Group Relay Webhooks

Relay Webhooks are a way to instruct SparkPost to accept inbound email on your behalf and forward it to you over HTTP for your own consumption.

By configuring a relay webhook for a specified inbound domain, those inbound messages can be forwarded to a specified target over HTTP.  Before you create a relay webhook, be sure to first create an inbound domain that is properly configured. To create an inbound domain for your account, please use our Inbound Domains API. The Relay Webhooks API provides the means to create, list, retrieve, update, and delete a relay webhook.

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://www.getpostman.com/run-collection/81ee1dd2790d7952b76a)

## Relay Webhooks Object Properties

| Property  | Type   | Description                          | Required | Notes
|-----------|--------|--------------------------------------|----------|----------------------|
| name      | string | User-friendly name                   | no       | example: `Inbound Customer Replies` |
| target    | string | URL of the target to which to POST relay batches | yes | example: `https://webhooks.customer.example/replies` |
| auth_token | string | Authentication token to present in the X-MessageSystems-Webhook-Token header of POST requests to target | no | Use this token in your target application to confirm that data is coming from the Relay Webhooks API. example: `5ebe2294ecd0e0f08eab7690d2a6ee69` |
| match     | object | Restrict which inbound messages will be relayed to the target | yes | See [Match Object Properties](#header-match-object-properties). example: `"match": { "protocol": "SMTP", "domain": "replies.customer.example" }` |
| custom_headers | JSON | Object of custom HTTP headers to be used during POST requests to target ( **Note:** SparkPost only ) | no | See [Custom HTTP Header Properties](#header-custom-http-headers-properties). example: `"custom_headers": { "x-api-key" : "abcd" }` |

## Match Object Properties

| Property  | Type   | Description                                                 | Required               | Notes
|-----------|--------|-----------------------------------------------------------------------|--------------|----------------------|
| protocol  | string | Inbound messaging protocol associated with this webhook | no - defaults to "SMTP" |                      |
| domain    | string | Inbound domain associated with this webhook             | yes, when protocol is "SMTP" | To create an inbound domain for your account, please use the Inbound Domains API. |
| esme_address | string | ESME address binding associated with this webhook    | yes, when protocol is "SMPP" | <a href="https://www.sparkpost.com/enterprise-email/"><span class="label label-warning"><strong>Enterprise</strong></span></a> Please speak with your TAM to create an ESME address. |

## Custom HTTP Headers Properties
The custom headers JSON object allows you to add up to five custom headers to your relay webhook. The custom_headers object may only be up to 3,000 bytes in size, and must be formatted as an object with keys as strings or numbers. Headers already used by SparkPost will not be allowed, and SparkPost may also disallow some HTTP headers for security reasons.

**Adding Custom Headers**

When creating (POST) or updating (PUT) a Relay Webhook:
```json
{
  "custom_headers": {
    "header1": "value1",
    "header2": "value2"
  }
}
```

**Removing Custom Headers**

When updating (PUT) a Relay Webhook:
```json
{
  "custom_headers": {}
}
```

## Field Definitions

**SMTP**

The following fields will be included in the JSON object posted to the SMTP relay webhooks target:

| Field     | Type   | Description                                                 | Notes
|-----------|--------|-----------------------------------------------------------------------|--------------|
| content   | object | Content that will be used to construct a relay message           | For a full description, see the [Content Attributes](#header-content-attributes). |
| friendly_from | string | Email address used to compose the "From" header |
| msg_from | string | [SMTP envelope](http://www.rfcreader.com/#rfc5321_line817) "MAIL FROM", matches "Return-Path" header address |
| rcpt_to | string | [SMTP envelope](http://www.rfcreader.com/#rfc5321_line817) "RCPT TO" |
| webhook_id | string | ID of the relay webhook which triggered this relay message |
| protocol | string | Protocol of the originating inbound message | For smtp payloads, this string will be "smtp" |

**SMPP**

<div class="alert alert-info">SMPP is available to <a href="https://www.sparkpost.com/enterprise-email/">SparkPost Enterprise</a> customers.</div>
The following fields will be included in the JSON object posted to the SMPP relay webhooks target:

| Field       | Type   | Description                                                           | Notes
|-------------|--------|-----------------------------------------------------------------------|--------------|
| text        | string | Contents of the first text/plain part of the message                  | For a full description, see the [Content Attributes](#header-content-attributes). |
| to          | string | SMPP message recipient                                                |              |
| from        | string | SMPP message sender                                                   |              |
| date        | string | Date that Sparkpost recieved the SMPP message                         |              |
| webhook_id  | string | ID of the relay webhook which triggered this relay message            |              |
| protocol    | string | Protocol of the originating inbound message                           | For smpp payloads, this string will be "smpp" |
| customer_id | string Customer ID of the customer that created the relay webhook            |              |

## Content Attributes

Content for an SMTP relay webhook is described in a JSON object with the following fields:

| Field     | Type   | Description                                                 | Notes
|-----------|--------|-------------------------------------------------------------|----------------|
| html      | string | Contents of the first text/html part of the message |
| text      | string | Contents of the first text/plain part of the message |
| subject   | string | "Subject" header value (decoded from email) |
| to        | array of strings | "To" header value (decoded from email), RFC2822 address list |
| cc        | array of strings | "CC" header value (decoded from email), RFC2822 address list |
| headers   | array of objects | Ordered array of email top-level headers | This array preserves ordering and allows for multiple occurrences of a header (e.g.: to support trace headers such as "Received"). |
| email_rfc822 | string | Raw MIME content for an email | If the Raw MIME content contains at least one non UTF-8 encoded character, the entire "email_rfc822" JSON value will be base64 encoded and the "email_rfc822_is_base64" JSON boolean value will be set to true |
| email_rfc822_is_base64 | boolean | Whether the "email_rfc822" value is base64 encoded |

## Example Payloads

**SMTP**

Once registered, your relay webhook HTTP endpoint will receive inbound emails in the JSON form described above. Here is an example of the payload which your endpoint can expect to receive:

```json
[
  {
    "msys": {
      "relay_message": {
        "content": {
          "email_rfc822": "Return-Path: <me@here.com>\r\nMIME-Version: 1.0\r\nFrom: me@here.com\r\nReceived: by 10.114.82.10 with HTTP; Mon, 4 Jul 2016 07:53:14 -0700 (PDT)\r\nDate: Mon, 4 Jul 2016 15:53:14 +0100\r\nMessage-ID: <484810298443-112311-xqxbby@mail.there.com>\r\nSubject: Relay webhooks rawk!\r\nTo: you@there.com\r\nContent-Type: multipart/alternative; boundary=deaddeaffeedf00fall45dbhail980dhypnot0ad\r\n\r\n--deaddeaffeedf00fall45dbhail980dhypnot0ad\r\nContent-Type: text/plain; charset=UTF-8\r\nHi there SparkPostians.\r\n\r\n--deaddeaffeedf00fall45dbhail980dhypnot0ad\r\nContent-Type: text/html; charset=UTF-8\r\n\r\n<p>Hi there <strong>SparkPostians</strong></p>\r\n\r\n--deaddeaffeedf00fall45dbhail980dhypnot0ad--\r\n",
          "email_rfc822_is_base64": false,
          "headers": [
            {
              "Return-Path": "<me@here.com>"
            },
            {
              "MIME-Version": "1.0"
            },
            {
              "From": "me@here.com"
            },
            {
              "Received": "by 10.114.82.10 with HTTP; Mon, 4 Jul 2016 07:53:14 -0700 (PDT)"
            },
            {
              "Date": "Mon, 4 Jul 2016 15:53:14 +0100"
            },
            {
              "Message-ID": "<484810298443-112311-xqxbby@mail.there.com>"
            },
            {
              "Subject": "Relay webhooks rawk!"
            },
            {
              "To": "you@there.com"
            }
          ],
          "html": "<p>Hi there <strong>SparkPostians</strong>.</p>",
          "subject": "We come in peace",
          "text": "Hi there SparkPostians.",
          "to": [
            "your@yourdomain.com"
          ]
        },
        "customer_id": "1337",
        "friendly_from": "me@here.com",
        "msg_from": "me@here.com",
        "rcpt_to": "you@there.com",
        "webhook_id": "4839201967643219"
      }
    }
  }
]
```

**SMPP**

Once registered, your relay webhook HTTP endpoint will receive inbound SMPP messages in the JSON form described above. Here is an example of the payload which your endpoint can expect to receive:

```json
[
  {
    "msys": {
      "relay_message": {
        "protocol": "smpp",
        "to": "12345",
        "date": "Wed, 14 Sep 2016 15:41:13 -0400",
        "from": "54321",
        "text": "Hi, I'm a text message",
        "customer_id": "1337",
        "webhook_id": "12363818881515528"
      }
    }
  }
]
```


## Create and List [/relay-webhooks]

### Create an SMTP Relay Webhook [POST]

Create a relay webhook by providing a **relay webhooks object** as the POST request body.

+ Request (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
              "name": "Replies Webhook",
              "target": "https://webhooks.customer.example/replies",
              "auth_token": "5ebe2294ecd0e0f08eab7690d2a6ee69",
              "match":
                {
                  "protocol": "SMTP",
                  "domain": "email.example.com"
                }
            }

+ Response 200 (application/json)

  + Body

            {
              "results":
                {
                  "id": "12013026328707075"
                }
            }

+ Response 400 (application/json)

  + Body

          ```
          { "errors": [
              {
                "message": "invalid params",
                "description": "Domain '(domain)' is not a registered inbound domain",
                "code": "1200"
              }
            ]
          }
          ```

+ Response 401 (application/json)

  + Body

          ```
          { "errors": [
              {
                "message": "Unauthorized Tenant",
                "code": "1303"
              }
            ]
          }
          ```

+ Response 409 (application/json)

  + Body

          ```
          { "errors": [
              {
                "message": "resource conflict",
                "description": "Domain '(domain)' is already in use",
                "code": "1602"
              }
            ]
          }
          ```

+ Response 422 (application/json)

  + Body

          ```
            {
              "errors" : [
                {
                  "message" : "required field is missing",
                  "description" : "field '(field_name)' is required",
                  "code" : "1400"
                }
              ]
            }
          ```

+ Response 422 (application/json)

  + Body

          ```
            {
              "errors" : [
                {
                  "message": "invalid data format/type",
                  "description": "Error validating domain name syntax for domain: '(domain)'",
                  "code": "1300"
                }
              ]
            }
          ```

+ Response 422 (application/json)

  + Body

          ```
            {
              "errors" : [
                {
                  "message": "Invalid custom header",
                  "description": "Header '(header_name)' is reserved and cannot be used",
                  "code": "10000"
                }
              ]
            }
          ```

+ Response 413 (application/json)

  + Body

          ```
            {
              "errors" : [
                {
                  "message": "custom_headers exceeds permitted size",
                  "description": "custom_headers cannot be larger than 3000 bytes",
                  "code": "10001"
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
                  "message": "custom_headers exceeded maximum headers allowed",
                  "description": "A maximum of 5 custom headers are permitted",
                  "code": "10002"
                }
              ]
            }
          ```

### Create an SMPP Relay Webhook [POST]

<div class="alert alert-info">SMPP relay webhooks are available to <strong><a href="https://www.sparkpost.com/enterprise-email/">SparkPost Enterprise</a></strong> customers.</div>

+ Request (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
              "name": "Replies Webhook",
              "target": "https://webhooks.customer.example/replies",
              "auth_token": "5ebe2294ecd0e0f08eab7690d2a6ee69",
              "match":
                {
                  "protocol": "SMPP",
                  "esme_address": "12345"
                }
            }

+ Response 200 (application/json)

  + Body

            {
              "results":
                {
                  "id": "12013026328707075"
                }
            }

+ Response 400 (application/json)

  + Body

            { "errors": [
                {
                  "message": "invalid params",
                  "description": "esme address not configured",
                  "code": "9000"
                }
              ]
            }


### List all Relay Webhooks [GET]

List all your relay webhooks.

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

  + Body

        ```
         { "results": [
              {
                "id": "12013026328707075",
                "name": "Replies Webhook",
                "target": "https://webhooks.customer.example/replies",
                "auth_token": "5ebe2294ecd0e0f08eab7690d2a6ee69",
                "match":
                  {
                    "protocol": "SMTP",
                    "domain": "email.example.com"
                  }
              }
            ]
          }
        ```

+ Response 401 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "Unauthorized Tenant",
                "code": "1303"
              }
            ]
          }
        ```

## Retrieve, Update, and Delete [/relay-webhooks/{webhook_id}]

### Retrieve a Relay Webhook [GET]

Retrieve a specific relay webhook by specifying the webhook ID in the URI path.

+ Parameters
  + webhook_id (required, string, `12013026328707075`) ... Webhook ID

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

  + Body

        ```
         {
            "results": {
              "name": "Replies Webhook",
              "target": "https://webhooks.customer.example/replies",
              "auth_token": "5ebe2294ecd0e0f08eab7690d2a6ee69",
              "match": {
                  "protocol": "SMTP",
                  "domain": "email.example.com"
              }
            }
         }
        ```

+ Response 401 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "Unauthorized Tenant",
                "code": "1303"
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
                "code": "1600"
              }
            ]
          }
        ```

### Update a Relay Webhook [PUT]

Update a relay webhook by specifying the webhook ID in the URI path.

+ Parameters
  + webhook_id (required, string, `12013026328707075`) ... Webhook ID

+ Request (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

        ```
            {
              "name": "New Replies Webhook",
              "target": "https://webhook.customer.example/replies",
              "auth_token": "A different auth token",
              "match":
                {
                  "protocol": "SMTP",
                  "domain": "email.a-different-domain.com"
                }
            }
        ```

+ Response 200 (application/json)

  + Body

        ```
            {
              "results":
                {
                  "id": "12013026328707075"
                }
            }
        ```

+ Response 400 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "invalid params",
                "description": "Domain ('domain') is not a registered inbound domain",
                "code": "1200"
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
                "message": "invalid params",
                "description": "esme address not configured",
                "code": "1200"
              }
            ]
          }
        ```

+ Response 401 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "Unauthorized Tenant",
                "code": "1303"
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
                "description": "UPDATE requires a webhook_id in the URI",
                "code": "1600"
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
                "code": "1600"
              }
            ]
          }
        ```

+ Response 409 (application/json)

  + Body

        ```
          { "errors": [
              {
                "message": "resource conflict",
                "description": "Domain '(domain)' is already in use",
                "code": "1602"
              }
            ]
          }
        ```

+ Response 422 (application/json)

  + Body

        ```
            {
              "errors" : [
                {
                  "message": "invalid data format/type",
                  "description": "Error validating domain name syntax for domain: '(domain)'",
                  "code": "1300"
                }
              ]
            }
        ```

+ Response 422 (application/json)

  + Body

          ```
            {
              "errors" : [
                {
                  "message": "Invalid custom header",
                  "description": "Header '(header_name)' is reserved and cannot be used",
                  "code": "10000"
                }
              ]
            }
          ```

+ Response 413 (application/json)

  + Body

          ```
            {
              "errors" : [
                {
                  "message": "custom_headers exceeds permitted size",
                  "description": "custom_headers cannot be larger than 3000 bytes",
                  "code": "10001"
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
                  "message": "custom_headers exceeded maximum headers allowed",
                  "description": "A maximum of 5 custom headers are permitted",
                  "code": "10002"
                }
              ]
            }
          ```

### Delete a Relay Webhook [DELETE]

Delete a relay webhook by specifying the webhook ID in the URI path.

+ Parameters
  + webhook_id (required, string, `12013026328707075`) ... Webhook ID

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

+ Response 200

+ Response 401 (application/json)

  + Body

        ```
          {
            "errors": [
              {
                "message": "Unauthorized Tenant",
                "code": "1303"
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
                "description": "DELETE requires a webhook_id in the URI",
                "code": "1600"
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
                "code": "1600"
              }
            ]
          }
        ```
