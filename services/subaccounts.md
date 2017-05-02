title: Subaccounts
description: Manage subaccounts, a way for service providers to provision and manage customers.

# Group Subaccounts
Subaccounts are a way for service providers to provision and manage their customers separately from each other and to provide assets and reporting data.

## Introduction

With the introduction of subaccounts, some of the APIs are now able to store and retrieve information at a more granular level.
Subaccounts are a way for service providers to provision and manage their customers separately from each other and to provide assets and reporting data.

The following APIs have subaccount support:

* [Metrics](metrics.html) <span class="label label-info"><strong>Note</strong></span> Not available for Subaccount API keys
* [Message Events](message-events.html)
* [Sending Domains](sending-domains.html)
* [Suppression List](suppression-list.html)
* [SMTP API](smtp-api.html)
* [Transmissions](transmissions.html)
* [Tracking Domains](tracking-domains.html)

<div class="alert alert-info"><strong>Note</strong>: all subaccount-level transmissions must use <tt>inline</tt> recipients and content. Recipient lists and stored templates do not support subaccounts.</div>

### Terminology
* Master Account - This refers to a Service Provider and their data
* Subaccounts - This refers to a Service Provider's customer(s), and that customer's data

### Managing subaccount data as a Service Provider
* Master Accounts can set the `X-MSYS-SUBACCOUNT` HTTP header with the ID of their subaccount to manage subaccount data on their behalf
  * For example, on a GET request to `/api/v1/sending-domains`, setting `X-MSYS-SUBACCOUNT` to `123` will only return sending domains which belong to Subaccount `123`
  * The same applies to data management, setting `X-MSYS-SUBACCOUNT` to `123` on a POST request to `/api/v1/sending-domains` will create a sending domain belonging to Subaccount `123`
* `X-MSYS-SUBACCOUNT` is not required, but if provided, must be a number

### Managing master account data as a Service Provider
* Setting `X-MSYS-SUBACCOUNT` to `0` will retrieve or manage Master Account data only
* For POST/PUT/DELETE requests, omitting `X-MSYS-SUBACCOUNT` will result in the same behavior as setting `X-MSYS-SUBACCOUNT` to `0`
* For GET requests, omitting `X-MSYS-SUBACCOUNT` will result in Master Account and Subaccount data in the response
  * Subaccount data will have the key `subaccount_id` in the response object
* Metrics and Message Events APIs do not use `X-MSYS-SUBACCOUNT`. Instead, setting the query parameter `subaccounts` to `0` will return only Master Account reporting data

## Subaccounts Collection [/subaccounts]

### List subaccounts [GET]

Endpoint for retrieving a list of your subaccounts. This endpoint only returns information about the subaccounts themselves, not the data associated with the subaccount.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results": [
            {
              "id": 123,
              "name": "Joe's Garage",
              "status": "active",
              "ip_pool": "my_ip_pool",
              "compliance_status": "active"
            },
            {
              "id": 456,
              "name": "SharkPost",
              "status": "active",
              "compliance_status": "active"
            },
            {
              "id": 789,
              "name": "Dev Avocado",
              "status": "suspended",
              "compliance_status": "active"
            }
          ]
        }

### Create a new Subaccount [POST]

Provisions a new subaccount and an initial subaccount API key. Subaccount API keys are only allowed very specific grants, which are limited to: `smtp/inject`, `sending_domains/manage`, `tracking_domains/view`, `tracking_domains/manage`, `message_events/view`, `suppression_lists/manage`, `transmissions/view`, and `transmissions/modify`.

Subaccounts are allowed to send mail using the SMTP protocol or Transmissions API, retrieve sending statistics via the Message Events API, manage their Sending Domains, manage their Suppression List.

<div class="alert alert-info"><strong>Note</strong>: Stored recipients lists and stored templates are currently not supported for subaccounts sending mail using the Transmissions API.</div>

#### Request Body Attributes

| Field         | Required   | Type    | Description                                                               | Notes                                                                                                                                                         |
| ------------  | ---------- | ------- | --------------------------------------------------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name          | yes        | string  | User friendly identifier for a specific subaccount                        |                                                                                                                                                               |
| key_label     | yes        | string  | User friendly identifier for the initial subaccount api key               |                                                                                                                                                               |
| key_grants    | yes        | Array   | List of grants to give to the initial subaccount api key                  | Valid values are `smtp/inject`, `sending_domains/manage`, `tracking_domains/view`, `tracking_domains/manage`, `message_events/view`, `suppression_lists/manage`, `transmissions/view`, and `transmissions/modify` |
| key_valid_ips | no         | Array   | List of IP's that the initial subaccount api key can be used from         | If the supplied `key_valid_ips` is an empty array, the api key is usable by any IP address                                                                    |
| ip_pool       | no         | string  | The ID of the default IP Pool assigned to this subaccount's transmissions | If the supplied `ip_pool` is an empty string or not present, no default `ip_pool` will be assigned<br/><a href="https://www.sparkpost.com/enterprise-email/"><span class="label label-warning"><strong>Enterprise</strong></span></a></strong> customers: IPs are managed through your TAM. |


+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

            {
              "name": "Sparkle Ponies",
              "key_label": "API Key for Sparkle Ponies Subaccount",
              "key_grants": ["smtp/inject", "sending_domains/manage", "message_events/view", "suppression_lists/manage", "tracking_domains/view", "tracking_domains/manage"],
              "key_valid_ips": [],
              "ip_pool": ""
            }

+ Response 200 (application/json)

        {
          "results": {
            "subaccount_id": 888,
            "key": "cf806c8c472562ab98ad5acac1d1b06cbd1fb438",
            "label": "API Key for Sparkle Ponies Subaccount",
            "short_key": "cf80"
          }
        }

+ Response 400 (application/json)

        {
          "errors": [
            {
              "message": "`name` is a required field",
              "param": "name",
              "value": null
            },
            {
              "message": "`key_label` is a required field",
              "param": "key_label",
              "value": null
            },
            {
              "message": "`key_grants` is a required field",
              "param": "key_grants",
              "value": null
            },
            {
              "message": "Invalid `key_grants value`. Supported values are: 'smtp/inject', 'sending_domains/manage', 'message_events/view', 'suppression_lists/manage'",
              "param": "key_grants",
              "value": null
            },
            {
              "message": "`key_valid_ips` must be an Array",
              "param": "key_valid_ips",
              "value": null
            },
            {
              "message": "`key_valid_ips` must have valid netmask values",
              "param": "key_valid_ips",
              "value": null
            },
            {
              "message": "ip_pool must be 20 characters or less",
              "param": "ip_pool",
              "value": "an_ip_pool_name_that_is_too_long"
            },
            {
              "message": "ip_pool must be alphanumeric and underscore",
              "param": "ip_pool",
              "value": "$invalid chars"
            }
          ]
        }

## Subaccounts Summary [/subaccounts/summary]

### Retrieve Subaccounts Summary [GET]

Retrieve the total number of subaccounts for an account.

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results": {
            "total": 46
          }
        }

## Subaccounts Entity [/subaccounts/{subaccount_id}]

### List specific subaccount [GET]

Endpoint for retrieving information about a specific subaccount.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Parameters

    + subaccount_id (required, integer, `123`) ... Identifier of subaccount

+ Response 200 (application/json)

            {
              "results": {
                "id": 123,
                "name": "Joes Garage",
                "status": "active",
                "compliance_status": "active",
                "ip_pool": "assigned_ip_pool"
              }
            }

### Edit a subaccount [PUT]
Update an existing subaccount's information. You can update the following information associated with a subaccount:

#### Request Body Attributes

| Field   | Required   | Type   | Description                                        | Notes |
| ------- | ---------- | ------ | -------------------------------------------------- | ----- |
| name    | no         | string | User friendly identifier for a specific subaccount |       |
| status  | no         | string | Status of the account                              | Value is one of `active`, `suspended`, or `terminated` |
| ip_pool | no         | string | The ID of the default IP Pool assigned to this subaccount's transmissions | If the supplied `ip_pool` is an empty string, it will clear any currently specified `ip_pool` |

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

            {
              "name": "Hey Joe! Garage and Parts",
              "status": "suspended",
              "ip_pool": ""
            }

+ Parameters

    + subaccount_id (required, integer, `123`) ... Identifier of subaccount

+ Response 200 (application/json)

            {
              "results": {
                "message": "Successfully updated subaccount information"
              }
            }

+ Response 400 (application/json)

        {
          "errors": [
            {
              "message": "ip_pool must be 20 characters or less",
              "param": "ip_pool",
              "value": "an_ip_pool_name_that_is_too_long"
            }
          ]
        }
