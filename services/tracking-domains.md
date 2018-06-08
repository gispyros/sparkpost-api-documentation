title: Tracking Domains
description: Manage tracking domains, which are used to wrap links in engagement tracking to report opens and clicks in your emails.

# Group Tracking Domains
<a name="tracking-domains-api"></a>

Tracking domains are used in engagement tracking to report email opens and link clicks.
They wrap the open pixel and all links in your emails.
When clicked, these links report the event to SparkPost, then quickly forward to the intended destination.
By default, the tracking domain on your account is `spgo.io` (`eu.spgo.io` for EU accounts). This API allows you to set up and manage custom tracking domains.
You can also set one up from <a href="https://app.sparkpost.com/account/tracking-domains/create" target="_blank">your account</a>.

As an example, say your email contains this link:

```HTML
<a href="https://www.heml.io">Build responsive emails</a>
```

In order to track clicks on that link, we wrap it. The resulting HTML would look something like this:

```HTML
<a href="http://spgo.io/e/nInKCLCf9wnO2brop7RTsg...">Build responsive emails</a>
```

With a custom tracking domain you can replace the domain part of the link. So if your tracking domain was `click.heml.io`, your HTML would look like this:

```HTML
<a href="http://click.heml.io/e/nInKCLCf9wnO2brop7RTsg...">Build responsive emails</a>
```

### Association to Sending Domains and Defaults
In order for a tracking domain to be used, it must be associated to a [sending domain](/api/sending-domains) or set as a default.
Once that is done, templates and transmissions that have open or click tracking on will automatically use the domain to wrap links.
If a tracking domain is set as default, it will automatically be used with sending domains that aren't associated with a different tracking domain.
Default tracking domains do not override associated tracking domains. You can set a default for each subaccount as well.
SparkPost picks the first tracking domain it can find to use in this order: 1. Associated tracking domain. 2. Subaccount default. 3. Account default.

### DNS
Tracking domains need to be verified via DNS. You'll need to add a CNAME record with the value of `spgo.io` (`eu.spgo.io` for EU accounts) to the domain's DNS settings before it can be used or set as the default.
For more information and Enterprise settings <a target="_blank" href="https://www.sparkpost.com/docs/tech-resources/enabling-multiple-custom-tracking-domains/">go here</a>.

### Security
Tracking domains use HTTP by default. Using HTTPS requires the set up and management of SSL Certificates,
for more information see <a target="_blank" href="https://www.sparkpost.com/docs/tech-resources/enabling-https-engagement-tracking-on-sparkpost/">this guide</a>.

### Subaccounts
Tracking domains can only be assigned to subaccounts on creation. They cannot be re-assigned at a later point.
If assigned and verified, a tracking domain can also be set as the default for that subaccount.
In order to be associated to a subaccount's sending domain, a tracking domain has to be assigned to the same subaccount.

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://app.getpostman.com/run-collection/5d9ae743a661a15d64bb)

## Tracking Domains Attributes

| Field         | Type    | Default | Description |
|---------------|---------|---------|-------------|
| domain        | string  |         | Name of the tracking domain. Example: `example.domain.com` |
| port          | integer |         | The port used when constructing the tracking URL. `80` if secure is `false`, `443` if secure is `true` |
| secure        | boolean | `false` | Whether the tracking URL should use HTTPS. HTTP will be used if `false`. |
| default       | boolean | `false` | Whether the tracking domain should be used when not explicitly associated with a sending domain. Only one default domain per account and subaccount can be set. A domain has to be verified to be set as the default. |
| status        | object  |         | Contains status details like verification and DNS. For a full description, see the [Status Attributes](#header-status-attributes). |
| subaccount_id | integer |         | ID of the subaccount the tracking domain is assigned to. Not returned unless set and can only be set on creation. |

### Status Attributes

The detailed status of a tracking domain is described in a JSON object with the following *read only* fields:

| Field             | Type    | Default      | Description |
|-------------------|---------|--------------|-------------|
| verified          | boolean | `false`      | Whether the domain's ownership has been verified. `true` if cname_status is `valid`. |
| cname_status      | string  | `unverified` | Verification status of CNAME record configuration. Possible values: `unverified`, `pending`, `invalid` or `valid`. |
| compliance_status | string  | `pending`    | Status of the domain's compliance review. Possible values: `pending`, `valid`, or `blocked`. |

## Create and List [/tracking-domains]

### Create a Tracking Domain [POST]

#### Request Body Attributes

| Field         | Type    | Required | Default | Description |
|---------------|---------|----------|---------|-------------|
| domain        | string  | yes      |         | Name of the tracking domain. Example: `example.domain.com` |
| secure        | boolean | no       | `false` | Whether the tracking URL should use HTTPS. HTTP will be used if `false`. |
| subaccount_id | integer | no       |         | ID of the subaccount the tracking domain should be assigned to. |

+ Request (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

            {
              "domain": "example.domain.com"
            }

+ Response 200 (application/json)

  + Body

            {
              "results": {
                "domain": "example.domain.com"
              }
            }

+ Request (application/json)

  + Body

            {
              "domain": "example.domain.com",
              "secure": true
            }

+ Response 200 (application/json)

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
                  "description" : "Please contact SparkPost support to get this domain authorized for your account."
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

### List All Tracking Domains [GET]

Returns an array with all tracking domains. Each item in the array is a full tracking domain object.

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
                  },
                  "subaccount_id": 215
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

Update the attributes of an existing tracking domain. The master account and each subaccount can only have one default tracking domain.
Setting a new default automatically unsets the current relevant default, if it exists.

#### Request Body Attributes

| Field   | Type    | Required | Description |
|---------|---------|----------|-------------|
| secure  | boolean | no       | Whether the tracking URL should use HTTPS. HTTP will be used if `false`. |
| default | boolean | no       | Whether the tracking domain should be used when not explicitly associated with a sending domain. A domain has to be verified to be set as the default. |


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


+ Response 200 (application/json)

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
