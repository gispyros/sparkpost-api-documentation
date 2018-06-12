title: Sending IPs
description: Manage the sending IPs for your account and assign them to IP pools.

# Group Sending IPs

<div class="alert alert-info"><strong><a href="https://www.sparkpost.com/enterprise-email/">SparkPost Enterprise</a></strong> customers: IPs are managed through your TAM, please contact them directly for assistance.</div>

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://app.getpostman.com/run-collection/5d9ae743a661a15d64bb)

#### Sending IP Properties

| Property   | Type    | Description | Notes |
|------------|---------|-------------|-------|
| external_ip | string | Public-facing IP address of this sending IP | |
| hostname | string | Reverse DNS hostname associated with this IP | |
| ip_pool | string | IP pool this sending IP is held in | |
| customer_provided | boolean | `true` if this sending IP is customer owned | |

## Sending IPs [/sending-ips]

## Get all sending IPs [GET]

Gets all IP addresses.

+ Request

    + Headers

            Authorization: aHR0cDovL2kuaW1ndXIuY29tL293UndTR3AucG5n
            Accept: application/json

+ Response 200 (application/json)

    ```json
    {
        "results": [{
          "external_ip": "123.45.67.89",
          "hostname": "mta472a.sparkpostmail.com",
          "ip_pool": "marketing",
          "customer_provided": false
        }, {
          "external_ip": "123.45.67.80",
          "hostname": "mta474a.sparkpostmail.com",
          "ip_pool": "default",
          "customer_provided": false
        }]
    }
    ```

## Sending IP Resource [/sending-ips/{external_ip}]

## Get a Sending IP [GET]

Retrieves a specific sending IP.

+ Request

    + Headers

            AUTHORIZATION: aHR0cDovL2kuaW1ndXIuY29tL293UndTR3AucG5n
            Accept: application/json

+ Parameters

  + external_ip (required, string, `123.45.67.89`) ... The external IP of the sending IP


+ Response 200 (application/json)

    ```json
    {
        "results": {
          "external_ip": "123.45.67.89",
          "hostname": "mta472a.sparkpostmail.com",
          "ip_pool": "cool_kids",
          "customer_provided": false
        }
    }

    ```

+ Response 400 (application/json)

    ```json
    {
        "errors": [{"message": "external ip must be a valid IPv4 address"}]
    }
    ```

+ Response 404 (application/json)

    ```json
    {
        "errors": [{"message": "Sending IP does not exist"}]
    }
    ```

## Update a Sending IP [PUT]

Updates the IP Pool of a sending IP.

#### Request Body Attributes

| Field          | Type           | Description                                | Required      |
|----------------|----------------|--------------------------------------------|---------------|
| ip_pool        | string         | The IP pool to add this sending IP to.     | yes           |


+ Request

    + Headers

            AUTHORIZATION: aHR0cDovL2kuaW1ndXIuY29tL293UndTR3AucG5n
            Accept: application/json

    + Body

            {
                "ip_pool": "too_cool_for_pool"
            }

+ Parameters

  + external_ip (required, string, `123.45.67.89`) ... The external IP of the sending IP to update


+ Response 200 (application/json)

    ```json
    {
        "results": {
          "message": "Updated IP Pool."
        }
    }
    ```

+ Response 400 (application/json)

    ```json
    {
        "errors": [{"message": "IP Pool glibglob does not exist."}]
    }
    ```

+ Response 404 (application/json)

    ```json
    {
        "errors": [{"message": "Sending IP does not exist"}]
    }
    ```
