title: Account
description: Get your SparkPost account information, including subscription status and quota usage.

# Group Account

This endpoint lets you retreive information regarding your SparkPost account and set account options.

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://app.getpostman.com/run-collection/5d9ae743a661a15d64bb)

## Account Properties

| Property             | Type   | Description |
|----------------------|--------|-------------|
| company_name         | string | Account holder company name |
| country_code         | string | Account holder 2-letter country code |
| anniversary_date     | string | ISO date of billing anniversary |
| created              | string | ISO date account was created |
| updated              | string | ISO date account details were last updated |
| status               | string | Account status. Possible values: `active`, `suspended`, `terminated` |
| service_level        | string | Account service level. Possible values: `priority`, `standard`, `premium`, `enterprise`, `regulated` |
| subscription         | object | Current subscription details. See [Subscription Properties](#header-subscription-properties) |
| pending_subscription | object | Pending subscription details representing an upgrade or downgrade. |
| options              | object | Account-level tracking settings. See [Options Properties](#header-options-properties) |
| usage                | object | Account quota usage details. Specify 'include=usage' in query string to include usage info. See [Usage Properties](#header-usage-properties) |
| support              | object | Support entitlement details. See [Support Properties](#header-support-properties) |

### Subscription Properties

| Property       | Type    | Description |
|----------------|---------|-------------|
| code           | string  | The account's plan code |
| name           | string  | The account's plan name |
| effective_date | string  | ISO date of when this subscription has been or will be effective |
| self_serve     | boolean | `true` if the subscription is managed through the UI |
| type           | string  | Type of subscription. Values include `aws` and `manual`. |

### Options Properties

| Property                    | Type    | Description |
|-----------------------------|---------|-------------|
| smtp_tracking_default       | boolean | Account-level default for SMTP engagement tracking |
| rest_tracking_default       | boolean | Account-level default for REST API engagement tracking |
| transactional_unsub         | boolean | Set to `true` to include `List-Unsubscribe` header for all transactional messages by default |
| transactional_default       | boolean | Set to `true` to send messages as transactional by default |
| initial_open_pixel_tracking | boolean | Account-level default for Initial Open tracking |

### Usage Properties

| Property  | Type   | Description |
|-----------|--------|-------------|
| timestamp | string | ISO date usage data was retrieved |
| day       | object | Daily usage report. See [Daily/Monthly Usage Properties](#header-daily-monthly-usage-properties) |
| month     | object | Monthly usage report. See [Daily/Monthly Usage Properties](#header-daily-monthly-usage-properties) |

### Daily/Monthly Usage Properties

| Property | Type   | Description |
|----------|--------|-------------|
| used     | number | Total messages sent in this period |
| limit    | number | Total allowance for this period |
| start    | string | ISO date when this period started |
| end      | string | ISO date when this period ends |

### Support Properties

| Property | Type    | Description |
|----------|---------|-------------|
| phone    | boolean | Whether account is entitled to phone support |
| online   | boolean | Whether account is entitled to online support |


## Retrieve [/account{?include}]

### Retrieve account information [GET]

Get your SparkPost account information, including subscription status and quota usage.
Usage details are not returned by default, specify 'include=usage' in the query string to include usage info.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Parameters

  + include (optional, `usage`, string) ... Additional parts of account details to include. The only valid value is currently `usage`.


+ Response 200 (application/json)

        {
            "results" : {
                "company_name": "Example Inc",
                "country_code": "US",
                "anniversary_date": "2017-01-11T08:00:00.000Z",
                "created": "2017-01-11T08:00:00.000Z",
                "updated": "2017-02-11T08:00:00.000Z",
                "status": "active",
                "status_reason_code": "",
                "subscription": {
                    "code": "150K-0817",
                    "name": "150K",
                    "plan_volume": 150000,
                    "self_serve": "true",
                    "type": "manual"
                },
                "support": {
                    "online": true,
                    "phone": false
                },
                "pending_subscription": {
                    "code": "2.5M-0817",
                    "name": "2.5M",
                    "effective_date": "2017-04-11T00:00:00.000Z"
                },
                "options": {
                    "smtp_tracking_default": false
                },
                "usage": {
                    "timestamp": "2016-03-17T05:19:00.932Z",
                    "day": {
                        "used": 22003,
                        "limit": 50000,
                        "start": "2016-03-16T05:30:00.932Z",
                        "end": "2016-03-17T05:30:00.932Z"
                    },
                    "month": {
                        "used": 122596,
                        "limit": 1500000,
                        "start": "2018-03-11T08:00:00.000Z",
                        "end": "2016-04-11T08:00:00.000Z"
                    }
                }
            }
        }

## Update [/account]

### Update account information [PUT]

Update your SparkPost account information and account-level options.

#### Request Body Attributes

| Field        | Type   | Required | Description |
|--------------|--------|----------|-------------|
| company_name | string | no       | Company name |
| options      | object | no       | Account-level options. |

#### Options Properties

| Property                    | Type    | Description |
|-----------------------------|---------|-------------|
| smtp_tracking_default       | boolean | Set to `true` to turn on SMTP engagement tracking by default |
| rest_tracking_default       | boolean | Set to `false` to turn off REST API engagement tracking by default |
| transactional_unsub         | boolean | Set to `true` to include `List-Unsubscribe` header for all transactional messages by default |
| transactional_default       | boolean | Set to `true` to send messages as transactional by default |
| initial_open_pixel_tracking | boolean | Set to `false` to exclude the initial open tracking pixel from top of emails |

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

    + Body

            {
                "company_name": "SparkPost",
                "options": {
                  "smtp_tracking_default": true
                }
            }

+ Response 200 (application/json)

        {
            "results": {
                "message": "Account has been updated"
            }
        }

+ Response 400 (application/json)

        {
          "errors": [
            {
              "message": "Incorrect type, expected boolean",
              "param": "smtp_tracking_default",
              "value": "bad value"
            }
          ]
        }

+ Response 500 (application/json)

        {
            "errors" : [
                {
                    "message" : "Cannot update Account"
                }
            ]
        }
