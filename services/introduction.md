description: Documentation for sending via SMTP or HTTP with the SparkPost API.

# SparkPost API

SparkPost presents a unified core API to all customers with a few noted exceptions.

* Features not available to SparkPost Enterprise customers are marked **Not available on SparkPost Enterprise**
* Features available only to SparkPost Enterprise customers are marked **SparkPost Enterprise customers only**.

In addition, SparkPost is available in multiple regions. "SparkPost" refers to the SparkPost service hosted in North America. "SparkPost EU" refers to the SparkPost service hosted in Western Europe. [SparkPost](https://app.sparkpost.com/) and [SparkPost EU](https://app.eu.sparkpost.com/) operate independently. An account created with SparkPost cannot be used with SparkPost EU, and vice-versa. Customers may use accounts in both regions.

## API Endpoints

| Endpoint   | Use for |
|------------|---------|
| `https://api.sparkpost.com/api/v1` | SparkPost and SparkPost Premium |
| `https://api.eu.sparkpost.com/api/v1` | SparkPost EU and SparkPost Premium EU |
| `https://api.sparkpost.com/api/labs` | SparkPost Labs |
| `https://yourdomain.api.e.sparkpost.com/api/v1` | SparkPost Enterprise API |

## API Conventions
* API versioning is handled using a major version number in the URL, e.g. /api/v1/endpoint.
* /something is equivalent to /something/.
* URL paths, URL query parameter names, and JSON field names are case sensitive.
* URL paths use lower case, with dashes separating words.
* Query parameters and JSON fields use lower case, with underscores separating words.
* The HTTP status indicates whether an operation failed or succeeded, with extra information included in the HTTP response body.
* All APIs return standard error code formats.
* Unexpected query parameters are ignored.
* Unexpected JSON fields in the request body are ignored.
* The JSON number type is bounded to a signed 32-bit integer.

## Authentication
* To authenticate with the APIs, specify the "Authorization" header with each request. The value of the Authorization header must be a valid API key or basic auth with the API key as username and an empty password.
* Administrators can [generate an API key](https://app.sparkpost.com/account/credentials) ([EU](https://app.eu.sparkpost.com/account/credentials)). Please take care to record and safeguard your API keys at all times. You cannot retrieve an API key after it has been created.
* For examples of supplying the Authorization header, refer to the cURL example below or any of the individual API request examples.

## API examples
SparkPost Enterprise and SparkPost EU customers should make sure to change the API endpoint domain in all the example calls to the domain for their specific environment.

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://app.getpostman.com/run-collection/5d9ae743a661a15d64bb)

In order to use most of the requests in the collection, you must define your SparkPost API key as a Postman variable called `API_KEY`.

Additionally, if you are a SparkPost Enterprise or SparkPost EU customer, you must set the Postman variable `BASE_URL` to the appropriate host (e.g. `https://api.eu.sparkpost.com` for SparkPost EU). More information on setting up Postman environments and variables can be found [here](https://www.getpostman.com/docs/v6/postman/environments_and_globals/variables).

## Using cURL
If you are using cURL to call the API, you must include the resource URI in quotes when you pass in multiple query parameters separated by an `&`.

For example:

```
curl -v \
-H "Content-Type: application/json" \
-H "Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf" \
-X GET "https://api.sparkpost.com/api/v1/metrics/deliverability/aggregate?campaigns=testjob&from=2014-01-23T14:00&metrics=count_targeted,count_sent,count_accepted&timezone=America%2FNew_York&to=2014-06-23T15:50"
```

or

```
curl -v \
-H "Content-Type: application/json" \
-u <APIKey>: \
-X GET "https://api.sparkpost.com/api/v1/metrics/deliverability/aggregate?campaigns=testjob&from=2014-01-23T14:00&metrics=count_targeted,count_sent,count_accepted&timezone=America%2FNew_York&to=2014-06-23T15:50"
```

## SMTP Relay Endpoints
<a name="smtp-relay-endpoints"></a>

<a name="header-sparkpost-smtp-endpoint"></a>
**SparkPost SMTP Endpoint**

To use SparkPost as an SMTP relay you need to point your SMTP client (or local MTA) to the following endpoint:

<a name="header-sparkpost-enterprise-smtp-endpoint"></a>
<div class="alert alert-info"><strong>Note</strong>: SparkPost Enterprise customers should contact their Technical Account Manager for SMTP details.</div>

* Host: smtp.sparkpostmail.com (SparkPost) or smtp.eu.sparkpostmail.com (SparkPost EU)
* Port: 587 or 2525
* Encryption: STARTTLS
* Authentication: AUTH LOGIN
* User: SMTP_Injection
* Password: Any API key with Send via SMTP permission
* To inject mail to an SMTP relay endpoint on behalf of a subaccount, modify your SMTP injection username to include the subaccount ID.
  * For example, use `SMTP_Injection:X-MSYS-SUBACCOUNT=123` to send as a Subaccount, having an ID of 123.
  * The Master Account's API key is still used as the password when sending on behalf of a Subaccount.
  * When sending on behalf of a Subaccount, the Subaccount's Sending Domain must be used.

<div class="alert alert-info"><strong>Note</strong>: Port 2525 is provided as an alternate port for cases where port 587 is blocked (such as a Google Compute Engine environment).</div>

The SMTP relay optionally supports advanced API features using the [SMTP API](smtp-api.html).  To create an API key, login to your SparkPost [Account Credentials](https://app.sparkpost.com/account/credentials) page ([EU](https://app.eu.sparkpost.com/account/credentials)).

## SMTP Security

<div class="alert alert-danger"><strong>Note</strong>: Disabling TLS will cause all data sent through SparkPost to be sent over the public internet unencrypted.</div>
SparkPost strongly recommends using TLS with SMTP to protect your message content, recipient information and API keys in transmission. This includes API keys and any details such as recipient email addresses and message content.

If TLS is not supported by your application, SparkPost recommends using API keys with _only_ the `Send via SMTP` privilege enabled. It is also good practice to regularly cycle your API keys to limit exposure of keys sent in the clear.

<div class="alert alert-danger"><strong>Note</strong>: API keys should be treated like passwords, and as stated in <a href="https://www.sparkpost.com/policies/tou/">our Terms of Use</a>, you "are solely responsible for all use of [your account]." That includes use of your account by someone who sniffed your API key on the unsecured wifi at your favorite coffee shop because you weren't using TLS.</div>

## Rate Limiting
<div class="alert alert-info"><strong>Note</strong>: To prevent abuse, our servers enforce request rate limiting, which may trigger responses with HTTP status code 429.</div>

SparkPost implements rate limiting on the following API endpoints:

- `/api/v1/message-events`
- `/api/v1/metrics/*`

The limits imposed here are dynamic but as a general rule, polling these endpoints more than once in 2 minutes may encounter rate limiting and a 429 status code.

**Alternatives To Polling:** For some common use cases, the SparkPost API offers more efficient alternatives to polling, especially of the message events endpoint. For instance, A single call to the [metrics deliverability summary](api/metrics.html#metrics-deliverability-metrics-get) endpoint offers a summary of deliveries, bounces, opens, clicks and more for some time period. If your application requires low latency access to each message event, using a [webhook-based](/api/webhooks.html) process will be more efficient than polling message events and will avoid rate limiting.

**Sandbox Domain Limits (sparkpostbox.com):** If you use the sandbox domain for testing you are limited to 5 emails for the lifetime of your SparkPost account.

## Account Suspension

If your account has been suspended due to concern about a possible violation of our [Messaging Policy](https://www.sparkpost.com/policies) please reply to the email you should have received from [compliance@sparkpost.com](mailto:compliance@sparkpost.com). If you have not received an email, please write to [compliance@sparkpost.com](mailto:compliance@sparkpost.com), ideally from the email address you used to sign up, including your Account ID, Company Name (if you set one), and Username which you can find under `Account > Profile` in our app UI.

## Errors

When you make an API call you may receive an error message in response. Either there is something wrong with your request or something went wrong on our end. Errors respond with an error code and JSON that contains a more precise message, description and API code.
### Example Error
```
422 Unprocessable Entity
```
```
{
  "errors": [
    {
      "message": "required field is missing",
      "description": "content object or template_id required",
      "code": "1400"
    }
  ]
}
```

### Error Table
|Code|Status Name           |Description                                                                   |Suggested Action|
|----|----------------------|------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
|400 |Bad Request           |There is a problem with your request.                                         |Check your request follows the API documentation and uses correct syntax.                                                                           |
|401 |Unauthorized          |You don't have the needed authorization to make the request.                  |Make sure you are using a valid API key with the necessary permissions for your request.                                                            |
|403 |Forbidden             |The server understood the request but refused to fulfill it.                  |See if your SparkPost plan includes the resource you are requesting and your API key has the necessary premissions.                                 |
|404 |Not Found             |The server couldn't find the requested file.                                  |Change your request URL to match a valid API endpoint.                                                                                              |
|405 |Method Not Allowed    |The resource does not have the specified method. (e.g. PUT on transmissions)  |Change the method to follow the documentation for the resource.                                                                                     |
|409 |Conflict              |A conflict arose from your request. (e.g. user already exists with that email)|Modify the payload to clear the conflict.                                                                                                           |
|415 |Unsupported Media Type|The request is not in a supported format.                                     |Check that your Content-Type header is a supported type and that your request adheres to the documentation.                                         |
|422 |Unprocessable Entity  |The request was syntactically correct but failed due to semantic errors.      |Make sure that your request includes all of the required fields and your data is valid.                                                             |
|429 |Exceed Sending Limit  |You sent too many requests in a given time period.                            |Check that you are with in the limits of your SparkPost plan. (If you are using the sandbox domain you'll need to add a sending domain to continue.)|
|500 |Internal Server Error |Something went wrong on our end.                                              |Try the request again later. If the error does not resolve, [contact support](https://support.sparkpost.com/).                                      |
|503 |Service Unavailable   |We are experiencing higher than normal levels of traffic.                     |Try the request again.                                                                                                                              |
