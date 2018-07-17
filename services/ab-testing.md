title: A/B Testing
description: A/B Testing of templates.

# Group A/B Testing

<a name="ab-testing-api"></a>

An A/B test is a method of comparing templates against a default template to see how their performance compares.  Users specify a default template and up to twenty template variants to compare, the comparison is based on the user selected metric.  Currently there are two supported modes of audience selection (which recipients receive the variant templates): a fixed number of recipients per variant can be specified, alternatively a percentage of recipients per variant can be specified.  There are two supported modes of behavior selection once the A/B test completes.  In Learning Mode once the test has completed subsequent transmissions will revert to using the default template.  In Bayesian mode the best performing template (the "winner") as determined by a Bayesian algorithm will be used in subsequent transmissions.

#### A/B Test Properties

<div class="alert alert-info"><strong>Note</strong>: Bayesian test mode is coming soon!</div>

| Property   | Type    | Description | Notes |
|------------|---------|-------------|-------|
| id | string | The identifier for this A/B test | |
| name | string | A human readable name for this A/B test | |
| status | string | The current state of the test.  Possible values: `draft`, `scheduled`, `running`, `completed`, `cancelled` | GET only |
| winning_template_id | string | The "winner" of the A/B test (only present if the state is `completed`) | GET only |
| version | integer | The current version number of the test.  The version increments each time the A/B test is modified. | GET only |
| default_template | object | Details for the default template. See [Template Properties](#header-template-properties) | |
| variants | array | Specifies which variants to test, as well as how messages are distributed to each variant. See [Template Properties](#header-template-properties) | |
| metric | string | One of `count_unique_clicked`, `count_unique_confirmed_opened` | |
| audience_selection | string | Determines how to distribute messages for templates. Each template will receive either a percent of the total of all messages or a set number of messages determined by the template's sample_size. Options are `percent`, `sample_size` | |
| test_mode | string | Either `bayesian` or `learning` | |
| start_time | string | ISO Date specifying when the test should begin | |
| end_time | string | (Optional) ISO Date specifying when the test should end | Defaults to (30 days from start_time - engagement timeout) |
| total_sample_size | int | (Optional) Total number of messages to send as part of the test | |
| confidence_level | float | (Optional) Specify a confidence level at which point the test should end | Defaults to 0.95 |
| engagement_timeout | int | (Optional) The amount of time, in hours, a test waits to collect results after the end_time to make a decision on winner and/or mark test as completed | Defaults to 24 hours |
| created_at | string | ISO Date of A/B Test Creation | GET only |
| updated_at | string | ISO Date of the last time the A/B test was updated | GET only |

### Template Properties
| Property   | Type    | Description | Notes |
|------------|---------|-------------|-------|
| template_id | string | The template id | |
| sample_size | integer | The number of injections to send using this template | Required if A/B test has `total_sample_size` defined |
| percent | integer | The percent of injections to send using this template | Required if A/B test does not have `total_sample_size` defined |
| count_unique_clicked or count_unique_confirmed_opened | integer | The number of unique clicks or confirmed opens. | Read only. Based on `metric` type |
| count_accepted | integer | Messages an ISP or other remote domain accepted (less Out-of-Band Bounces) | Read only. |

<div class="alert alert-info"><strong>Note</strong>: Tests will run until one of the following criteria are met: The end_time has passed, messages equal to the total_sample_size have been sent, or the confidence_level (Bayesian mode only) has been reached. In Bayesian mode, reaching the specified confidence_level for a template will cause it to become the "winner". If a test ends and the confidence_level has not been reached, the default template will be considered the "winner". </div>

## A/B Tests [/ab-test]

## Create an A/B Test [POST]

+ Request A/B Test using a percentage for distribution

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

  + Body
    ```json
        {
          "id": "payment-confirmation",
          "name": "Payment Confirmation",
          "metric": "count_unique_confirmed_opened",
          "audience_selection": "percent",
          "start_time": "2018-04-03T22:08:33Z",
          "test_mode": "bayesian",
          "confidence_level": 0.99,
          "default_template": {
            "template_id": "default_payment_confirmation_template",
            "percent": 50
          },
          "variants": [
            {
              "template_id": "payment_confirmation_variant1",
              "percent": 25
            },
            {
              "template_id": "payment_confirmation_variant2",
              "percent": 25
            }
          ]
        }
    ```

+ Response 200 (application/json)

    ```json
    {
      "results": {
        "id": "payment-confirmation"
      }
    }
    ```

+ Response 400 (application/json)

    ```json
    {
      "errors": [{"message": "Variants must have a template_id"}]
    }
    ```

+ Request create a test using sample_size for distribution

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

  + Body
    ```json
        {
          "id": "payment-confirmation",
          "name": "Payment Confirmation",
          "metric": "count_unique_confirmed_opened",
          "audience_selection": "sample_size",
          "start_time": "2018-04-03T22:08:33+00:00",
          "test_mode": "learning",
          "total_sample_size": 60000,
          "default_template": {
            "template_id": "default_payment_confirmation_template",
            "sample_size": 40000
          },
          "variants": [
            {
              "template_id": "payment_confirmation_variant1",
              "sample_size": 10000
            },
            {
              "template_id": "payment_confirmation_variant2",
              "sample_size": 10000
            }
          ]
        }
    ```

+ Response 200 (application/json)

     ```json
    {
      "results": {
        "id": "payment-confirmation"
      }
    }
    ```

+ Response 400 (application/json)

    ```json
    {
      "errors": [{"message": "Variants must have a template_id"}]
    }
    ```

## List All A/B Tests [GET]

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

  ```json
    {
      "results": [
        {
          "id": "payment-confirmation",
          "name": "Payment Confirmation",
          "version": 2,
          "status": "running",
          "metric": "count_unique_confirmed_opened",
          "audience_selection": "percent",
          "start_time": "2018-04-03T22:08:33+00:00",
          "test_mode": "bayesian",
          "confidence_level": 0.99,
          "engagement_timeout": 24,
          "default_template": {
            "template_id": "default_payment_confirmation_template",
            "percent": 60,
            "count_unique_confirmed_opened": 1000,
            "count_accepted": 100000
          },
          "variants": [
            {
              "template_id": "payment_confirmation_variant1",
              "percent": 10,
              "count_unique_confirmed_opened": 489,
              "count_accepted": 9000
            },
            {
              "template_id": "payment_confirmation_variant2",
              "percent": 30,
              "count_unique_confirmed_opened": 320,
              "count_accepted": 68933
            }
          ],
          "created_at": "2018-04-02T22:08:33+00:00",
          "updated_at": "2018-04-02T22:08:33+00:00"
        },
        {
          "id": "password-reset",
          "name": "Password Reset",
          "version": 2,
          "status": "completed",
          "winning_template_id": "password_reset_variant2",
          "metric": "count_unique_clicked",
          "audience_selection": "percent",
          "start_time": "2018-04-03T22:08:33+00:00",
          "test_mode": "bayesian",
          "confidence_level": 0.99,
          "engagement_timeout": 24,
          "default_template": {
            "template_id": "default_password_reset_template",
            "percent": 70,
            "count_unique_clicked": 8909,
            "count_accepted": 3423230
          },
          "variants": [
            {
              "template_id": "password_reset_variant1",
              "percent": 15,
              "count_unique_clicked": 398,
              "count_accepted": 90302
            },
            {
              "template_id": "password_reset_variant2",
              "percent": 15,
              "count_unique_clicked": 231,
              "count_accepted": 73039
            }
          ],
          "created_at": "2018-04-02T22:08:33+00:00",
          "updated_at": "2018-04-02T22:08:33+00:00"
        }
      ]
    }
  ```

## A/B Tests Resource [/ab-test/{id}{?version}]

## Get an A/B Test [GET]

+ Parameters

  + id (required, string, `password-reset`) ... A/B Test ID
  + version (optional, integer) ... If passed return information about the specific version of the A/B test. If not specified, return information about the latest version.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

     ```json
     {
     "results": {
        "id": "password-reset",
        "name": "Password Reset",
        "version": 2,
        "status": "scheduled",
        "metric": "count_unique_confirmed_opened",
        "audience_selection": "sample_size",
        "start_time": "2018-04-03T22:08:33+00:00",
        "test_mode": "bayesian",
        "confidence_level": 0.99,
        "total_sample_size": 60000,
        "engagement_timeout": 24,
        "default_template": {
          "template_id": "default_password_reset_template",
          "sample_size": 20000,
          "count_unique_confirmed_opened": 1398,
          "count_accepted": 20321
        },
        "variants": [
          {
            "template_id": "password_reset_variant1",
            "sample_size": 20000,
            "count_unique_confirmed_opened": 343,
            "count_accepted": 18908
          },
          {
            "template_id": "password_reset_variant2",
            "sample_size": 20000,
            "count_unique_confirmed_opened": 890,
            "count_accepted": 22987
          }
         ],
          "created_at": "2018-04-02T22:08:33+00:00",
          "updated_at": "2018-04-02T22:08:33+00:00"
       }
     }
    ```

+ Response 404 (application/json)

    ```json
    {
      "errors": [{"message": "A/B test password-reset does not exist"}]
    }
    ```

## Update an A/B Test [/ab-test/{id}]

### Update an A/B Test [PUT]

Modify an A/B test properties

<div class="alert alert-info"><strong>Note</strong>: Updating an A/B test creates a new version of the test if its latest version is cancelled or completed.  This effectively causes the test to restart. Tests in `running` state must be cancelled before updating. The winning_template_id will be reset for the new version. If winning_template_id existed in the previous version, the template_id of the new version will default to that value, unless overridden by the update.</div>

+ Parameters

  + id (required, string, `password-reset`) ... A/B Test ID

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json
    + Body
      ```json
      {
        "total_sample_size": 100000,
        "default_template": {
          "template_id": "default_password_reset_template",
          "sample_size": 70000
        },
        "variants": [
          {
            "template_id": "password_reset_variant1",
            "sample_size": 10000
          },
          {
            "template_id": "password_reset_variant2",
            "sample_size": 20000
          }
        ]
      }
      ```


+ Response 200 (application/json)
    ```json
    {
      "results": {
        "version": 2
      }
    }
    ```

+ Response 409 (application/json)

    ```json
    {
      "errors": [{"message": "A/B test password-reset is running"}]
    }
    ```

## Cancel an A/B Test [/ab-test/{id}/cancel]

### Cancel an A/B Test [POST]

+ Parameters

  + id (required, string, `password-reset`) ... A/B Test ID

+ Request Cancel an A/B test

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json


+ Response 200 (application/json)

    ```json
    {
      "results": {
        "status": "cancelled"
      }
    }
    ```

+ Response 404 (application/json)

    ```json
    {
      "errors": [{"message": "A/B test password-reset does not exist"}]
    }
    ```

## A/B Test Drafts [/ab-test/draft]
A/B Test drafts allow a user to set a name and default template on create, and configure tests over several updates before setting a start time.

<div class="alert alert-info"><strong>Note</strong>: Only the id, name and default_template are allowed when creating a draft.</div>

### Create an A/B Test draft [POST]

+ Request create a draft with only default_template

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

  + Body
    ```json
        {
          "id": "payment-confirmation",
          "name": "Payment Confirmation",
          "default_template": {
            "template_id": "default_payment_confirmation_template",
            "percent": 50
          }
        }
    ```

+ Response 200 (application/json)

    ```json
    {
      "results": {
        "id": "payment-confirmation"
      }
    }
    ```

+ Response 400 (application/json)

    ```json
    {
      "errors": [{"message": "default_template must have a template_id"}]
    }
    ```

## A/B Test Draft Resource [/ab-test/draft/{id}]

### Get an A/B Test Draft [GET]

+ Parameters

  + id (required, string, `my-draft-test`) ... A/B Test ID

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

    ```json
    {
      "results": {
        "id": "my-draft-test",
        "name": "my draft",
        "version": 1,
        "status": "draft",
        "default_template": {
            "template_id": "my-test-temp",
            "percent": 50
        },
        "created_at": "2018-07-10T21:55:34.960Z",
        "updated_at": "2018-07-11T21:55:47.176Z"
      }
    }
    ```

### Update an A/B Test Draft [PUT]

+ Parameters

  + id (required, string, `payment-confirmation`) ... A/B Test ID

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

  + Body
    ```json
        {
          "metric": "count_unique_confirmed_opened",
          "audience_selection": "percent",
          "test_mode": "bayesian",
          "confidence_level": 0.99,
          "variants": [
            {
              "template_id": "payment_confirmation_variant1",
              "percent": 25
            },
            {
              "template_id": "payment_confirmation_variant2",
              "percent": 25
            }
          ]
        }
    ```

+ Response 200 (application/json)

    ```json
    {
      "results": {
        "id": "payment-confirmation"
      }
    }
    ```

+ Response 400 (application/json)

    ```json
    {
      "errors": [{"message": "Variants must have a template_id"}]
    }
    ```

## Schedule an A/B Test Draft [/ab-test/draft/{id}/schedule]

### Schedule an A/B Test [POST]

+ Parameters

  + id (required, string, `payment-confirmation`) ... A/B Test ID

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

  + Body
    ```json
        {
          "start_time": "2018-04-03T22:08:33+00:00",
          "end_time": "2018-04-15T22:08:33+00:00",
          "engagement_timeout": 4
        }
    ```

+ Response 200 (application/json)

    ```json
    {
      "results": {
        "id": "payment-confirmation"
      }
    }
    ```

+ Response 400 (application/json)

    ```json
    {
      "errors": [{"message": "default_template must have a template_id"}]
    }
    ```
