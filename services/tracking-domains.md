title: Tracking Domains
description: Manage tracking domains, which are used to wrap links in engagement tracking to report opens and clicks in your messages.

# Group Tracking Domains
<a name="tracking-domains-api"></a>

Tracking domains are used in engagement tracking to report opens and clicks in your messages. When open and click tracking is enabled, you can set up a tracking domain which wraps the tracking pixel and all links in your messages.

As an example, for SparkPost.com, the tracking domain is spgo.io. Your message contains a link to http://www.some-website.com/some-article. That link gets wrapped and the resulting HTML would look something like this:

```html
<a href="http://spgo.io/e/nInKCLCf9wnO2brop7RTsg...">Check out this excellent article</a>
```

With a tracking domain you can replace the domain part of the link. So if your tracking domain was example.domain.com, your HTML would look like this:

```html
<a href="http://example.domain.com/e/nInKCLCf9wnO2brop7RTsg...">Check out this excellent article</a>
```

<div class="alert alert-info"><strong>Note</strong>: Use of a tracking domain requires modification of your DNS records to include a CNAME record.</div>

<div class="alert alert-info"><strong>Note</strong>: For SparkPost, in order to use HTTPS on tracking domains follow <a href="https://www.sparkpost.com/docs/tech-resources/enabling-https-engagement-tracking-on-sparkpost/">this guide</a>.</div>

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://www.getpostman.com/run-collection/81ee1dd2790d7952b76a)

## Tracking Domains Attributes

| Field   | Type   | Description | Required | Notes |
|------------|--------|-------------|----------|-------|
| domain | string | Name of the tracking domain | yes | Example: `example.domain.com` |
| port | integer | Determines the port to be used when constructing the tracking URL | no | Read only. |
| secure | boolean | Should the tracking URL should use https? | no | If `false` (the default), http will be used.<br/><a href="https://www.sparkpost.com/enterprise-email/"><span class="label label-warning"></span></a>  |
| default | boolean | Should the default tracking domain be used when not explicitly set | no | There can only be one default domain. Defaults to `false`. |
| status | JSON object| JSON object containing status details, including whether this domain's ownership has been verified  | no | Read only. For a full description, see the [Status Attributes](#header-status-attributes).|

### Port/Secure Attributes

Upon creation of a tracking domain, the values for port and secure are set according to the following table:

| secure (input) | port (value) | secure (value) |
|--------|--------|--------|
| not provided | 80 | false |
| false | 80 | false |
| true | 443 | true |

### Status Attributes

Detailed status for this tracking domain is described in a JSON object with the following fields:

| Field         | Type     | Description                           | Default   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
| verified | boolean | Whether domain has been verified | false | Read only. This field will return `true` if cname_status is `valid`.|
| cname_status | string | Verification status of CNAME configuration | unverified | Read only. Valid values are `unverified`, `pending`, `invalid` or `valid`.|
| compliance_status | string | Compliance status | | Read only. Valid values are `pending`, `valid`, or `blocked`.|


## Create and List [/tracking-domains]

### Create a Tracking Domain [POST]

Create a tracking domain. A tracking domain cannot be set as the default until it is verified.

<div class="alert alert-info"><strong>Note</strong>: The value for <tt>port</tt> is not configurable. <tt>port</tt> will be 80 for Non-Secure and 443 for Secure.</div>

+ Request (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
              "domain": "example.domain.com",
              "secure": false,
              "default": false
            }

+ Response 200

  + Body

            {
              "results": {
                "domain": "example.domain.com"
              }
            }

+ Response 400 (application/json)

  + Body

            {
              "errors" : [
                {
                  "code" : "7000",
                  "message" : "restricted domain",
                  "description" : "Please contact us at support@sparkpost.com to get this domain authorized for your account."
                }
              ]
            }

+ Response 409 (application/json)

  + Body

            {
              "errors" : [
                {
                  "code" : "1602",
                  "message" : "resource conflict",
                  "description" : "The tracking domain already exists."
                }
              ]
            }

+ Response 422 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1300",
                  "message": "invalid data format/type",
                  "description": "Tracking domain 'example.domain.com' unverified."
                }
              ]
            }


+ Response 422 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1300",
                  "message": "invalid data format/type",
                  "description": "Error validating domain name syntax for domain: 'example..domain.com'"
                }
              ]
            }

+ Response 422 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1300",
                  "message": "invalid data format/type",
                  "description": "Error domain name length too short for domain: 'ex'"
                }
              ]
            }

+ Response 422 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1300",
                  "message": "invalid data format/type",
                  "description": "Error domain name contains no '.'(s) for domain: 'exampledomaincom'"
                }
              ]
            }

+ Response 422 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1300",
                  "message": "invalid data format/type",
                  "description": "Error domain name contains invalid characters or spaces for domain: 'example*&#.domain.com'"
                }
              ]
            }

+ Response 422 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1400",
                  "message": "required field is missing",
                  "description": "field 'domain' is required"
                }
              ]
            }


### List all Tracking Domains [GET]

Retrieve a list of all tracking domains.


+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

  + Body

            {
              "results": [
                {
                  "port": 443,
                  "domain": "example.domain.com",
                  "secure": true,
                  "default": true,
                  "status": {
                    "verified": false,
                    "cname_status": "pending",
                    "compliance_status": "pending"
                  }
                },
                {
                  "port": 80,
                  "domain": "example2.domain.com",
                  "secure": false,
                  "default": false,
                  "status": {
                    "verified": true,
                    "cname_status": "valid",
                    "compliance_status": "valid"
                  }
                }
              ]
            }


## Retrieve, Update, and Delete [/tracking-domains/{domain}]

### Retrieve a Tracking Domain [GET]

Retrieve an existing tracking domain.

+ Parameters
  + domain (required, string, `example.domain.com`) ... domain name


+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

  + Body

            {
              "results": {
                "port": 443,
                "domain": "example.domain.com",
                "secure": true,
                "default": true,
                "status": {
                  "verified": false,
                  "cname_status": "pending",
                  "compliance_status": "pending"
                }
              }
            }

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1600",
                  "message": "resource not found",
                  "description": "Resource not found: example.domain.com"
                }
              ]
            }


### Update a Tracking Domain [PUT]

Update the attributes of an existing tracking domain.  A tracking domain cannot be
set as the default until it is verified.  If a tracking domain is set to the default,
and there is already a default domain, the default is changed.

<div class="alert alert-info"><strong>Note</strong>: <tt>port</tt> is not configurable.</div>

+ Parameters
  + domain (required, string, `example.domain.com`) ... domain name

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

        ```
        {
            "secure"  : true,
            "default" : true
        }
        ```

+ Response 200

  + Body

        ```
        {
            "results": {
              "domain": "example.domain.com"
            }
        }
        ```

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1600",
                  "message": "resource not found",
                  "description": "Resource not found: example.domain.com"
                }
              ]
            }

+ Response 422 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1300",
                  "message": "invalid data format/type",
                  "description": "Tracking domain 'example.domain.com' unverified."
                }
              ]
            }


### Delete a Tracking Domain [DELETE]

Delete an existing tracking domain.

+ Parameters
  + domain (required, string, `example.domain.com`) ... domain name

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

+ Response 204

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1600",
                  "message": "resource not found",
                  "description": "Resource not found: example.domain.com"
                }
              ]
            }


## Verify [/tracking-domains/{domain}/verify]

### Verify a Tracking Domain [POST]
Initiate a check against the configured redirect for the specified tracking domain. The domain's `status` object is returned on success.

+ Parameters
  + domain (required, string, `example.domain.com`) ... domain name


+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf


+ Response 200 (application/json; charset=utf-8)

        {
            "results": {
              "verified": true,
              "cname_status": "",
              "compliance_status": "valid"
            }
        }

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1600",
                  "message": "resource not found",
                  "description": "Resource not found: example.domain.com"
                }
              ]
            }

+ Response 400 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1404",
                  "message": "request to remote endpoint failed",
                  "description": "Unable to reach http://test.messagesystems.com:80. Please verify that your redirect is functioning."
                }
              ]
            }
