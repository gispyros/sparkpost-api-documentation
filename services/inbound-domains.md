title: Inbound Domains
description: Manage inbound domains, which enable you to customize the address to which inbound messages are sent.

# Group Inbound Domains

Specifying an inbound domain enables you to customize the address to which inbound messages are sent.
Inbound domains are used in conjunction with Relay Webhooks.
You can have multiple inbound domains but each domain must be globally unique.

Before you can use your inbound domain (e.g. `inbounddomain.test.com`),
you will need to add MX records to your DNS settings.
The following DNS settings are for all plans **except SparkPost Enterprise:**

| Name                     | Type | Data                  | Priority |
|--------------------------|------|-----------------------|----------|
| `inbounddomain.test.com` | MX   | rx1.sparkpostmail.com | 10       |
| `inbounddomain.test.com` | MX   | rx2.sparkpostmail.com | 10       |
| `inbounddomain.test.com` | MX   | rx3.sparkpostmail.com | 10       |

<div class="alert alert-info"><strong><a href="https://www.sparkpost.com/enterprise-email/">SparkPost Enterprise</a> customers</strong>, you will need to use MX records similar to your existing domains.  In many cases
the MX records for existing domains point at <tt>inbound.<main-bounce-domain></tt>.
The following DNS settings assume that your existing domains point
at <tt>inbound.<main-bounce-domain></tt>.  Please check with your TAM
if you are unsure of the setting in your own environment.</div>

| Name                     | Type | Data                  | Priority |
|--------------------------|------|-----------------------|----------|
| `inbounddomain.test.com` | MX   | `inbound.<main-bounce-domain>` | 10       |


#### Inbound Domains Attributes

| Field   | Type   | Description | Required | Notes |
|------------|--------|-------------|----------|-------|
| domain     | string | Domain (or subdomain) name for which SparkPost will receive inbound emails | yes | Your DNS provider's MX record for this domain must point back to SparkPost. (Example: `inbounddomain.test.com`) |

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://www.getpostman.com/run-collection/81ee1dd2790d7952b76a)

## Create and List [/inbound-domains]

### Create an Inbound Domain [POST]

Create an inbound domain by providing an **inbound domains object** as the POST request body.

+ Request (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
              "domain": "inbounddomain.test.com"
            }

+ Response 200

+ Response 400 (application/json)

  + Body

            {
              "errors" : [
                {
                  "message" : "Restricted domain",
                  "description" : "Please contact us at support@sparkpost.com to get this domain authorized for your account.",
                  "code" : "7000"
                }
              ]
            }

+ Response 401 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "Unauthorized Tenant",
                  "code": "1303"
                }
              ]
            }

+ Response 409 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "resource conflict",
                  "description": "An inbound domain with similar attributes already exists",
                  "code": "1602"
                }
              ]
            }

+ Request Missing Domain (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {}

+ Response 422 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "required field is missing",
                  "description": "Missing required field: (domain) is required",
                  "code": "1400"
                }
              ]
            }

+ Request Invalid Domain (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
                "domain": "inbounddomain"
            }

+ Response 422 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "invalid data format\/type",
                  "description": "Error domain name contains no '.'(s) for domain: {domain}",
                  "code": "1300"
                }
              ]
            }

### List all Inbound Domains [GET]

List all your inbound domains.

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

  + Body

            {
              "results": [
                {
                  "domain": "inbounddomain.test.com"
                },
                {
                  "domain": "inbounddomain2.test.com"
                }
              ]
            }

+ Response 401 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "Unauthorized Tenant",
                  "code": "1303"
                }
              ]
            }

## Retrieve and Delete [/inbound-domains/{domain}]

### Retrieve an Inbound Domain [GET]

Retrieve an inbound domain by specifying its domain name in the URI path.

+ Parameters
  + domain (required, string, `inbounddomain.test.com`) ... Domain name

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

  + Body

            {
              "results": {
                "domain": "inbounddomain.test.com"
              }
            }

+ Response 401 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "Unauthorized Tenant",
                  "code": "1303"
                }
              ]
            }

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "resource not found",
                  "code": "1600"
                }
              ]
            }


### Delete an Inbound Domain [DELETE]

Delete an inbound domain by specifying its domain name in the URI path.

+ Parameters
  + domain (required, string, `inbounddomain.test.com`) ... Domain name

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

+ Response 200

+ Response 401 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "Unauthorized Tenant",
                  "code": "1303"
                }
              ]
            }

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "resource not found",
                  "code": "1600"
                }
              ]
            }

+ Response 409 (application/json)

  + Body

            {
              "errors": [
                {
                  "message": "resource conflict",
                  "description": "Domain currently being used in a relay-webhook. Please delete the relay-webhook first.",
                  "code": "1602"
                }
              ]
            }
