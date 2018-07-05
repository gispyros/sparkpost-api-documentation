title: Message Events
description: Query detailed message event data for further analysis and processing.

# Group Message Events
<a name="message-events-api"></a>

The Message Events API allows searching for recent events, and supports various types of result filtering. Available events include message status - injection, delivery, bounce - as well as recipient engagement - clicks and opens. Message Events is an on-demand (pull) interface to the same underlying event data that gets delivered in a near-real-time (push) fashion via [Webhooks](/api/webhooks).

You can get all event types or only specific ones, such as bounces, deliveries, or clicks. You can filter by date range, campaign, or just about any other field. Event data is retained for 10 days and is generally available within 1 minute. Of course, aggregate reporting data is available via our metrics endpoint or the UI reports for much longer.

## Data retention

Message Events API data is retained for 10 days.

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://app.getpostman.com/run-collection/5d9ae743a661a15d64bb)


## Copyrights
__**The SparkPost message events API uses MaxMind software [MaxMind License](https://www.maxmind.com/download/geoip/database/LICENSE.txt)**__

## Events Documentation [/message-events/events/documentation]

### Documentation [GET]

List descriptions of the event fields that could be included in a response from the Message Events search endpoint. Fields will vary by event type.

+ Request

  + Headers

            Accept: application/json




## Events Samples [/message-events/events/samples{?events}]

### Samples [GET]

List an example of the event data that will be included in a response from the Message Events search endpoint.

+ Parameters
  + events (optional, string, `bounce`) ... Event types for which to get a sample payload, use the Webhooks Events endpoint to list the available event types, defaults to all event types.

+ Request

  + Headers

            Accept: application/json



## Message Events [/message-events{?bounce_classes,campaign_ids,delimiter,events,friendly_froms,from,message_ids,page,per_page,reason,recipients,subaccounts,template_ids,timezone,to,transmission_ids,ab_test_ids}]

### Search for Message Events [GET]

Perform a filtered search for message event data. The response is sorted by descending timestamp.

<div class="alert alert-info"><strong>Hint!</strong> Use the <tt>delv_method</tt> key to differentiate between Email, Push, and SMS type events.</div>

+ Parameters
    + bounce_classes (optional, number, `1`) ... delimited list of bounce classification codes to search. (See [Bounce Classification Codes.](https://support.sparkpost.com/customer/portal/articles/1929896))
    + campaign_ids (optional, string, `Example Campaign Name`) ... delimited list of campaign ID's to search (i.e. the campaign id used during creation of a transmission).
        
        **Notes:** Not available for `sms_status` type.
        
    + delimiter = `,` (optional, string, `,`) ... Specifies the delimiter for query parameter lists
    + events (optional, list, `delivery,injection,bounce,delay,policy_rejection,out_of_band,open,initial_open,click,generation_failure,generation_rejection,spam_complaint,list_unsubscribe,link_unsubscribe`) ... delimited list of event types to search. Defaults to all event types.
    + friendly_froms (optional, list, `sender@mail.example.com`) ... delimited list of friendly from emails to search. 
        
        **Notes:** Not available for `sms_status` type.
        
    + from: `2014-07-20T08:00` (datetime, optional) - Datetime in format of YYYY-MM-DDTHH:MM.
        + Default: `24 hours ago`
    + message_ids (optional, list, `0e0d94b7-9085-4e3c-ab30-e3f2cd9c273e`) ... delimited list of message ID's to search.
    + page = `1` (optional, number, `1`) ... The results page number to return. Used with per_page for paging through results.
    + per_page = `1000` (optional, number, `1000`) ... Number of results to return per page. Must be between 1 and 10,000 (inclusive).
    + reason (optional, string, `bounce`) ... Bounce/failure/rejection reason that will be matched using a wildcard (e.g., %reason%).
    + recipients (optional, list, `recipient@example.com`) ... delimited list of recipients to search.
    + subaccounts (optional, list, `101`) ... delimited list of subaccount ID's to search.

    + template_ids (optional, list, `templ-1234`) ... delimited list of template ID's to search.
    + timezone =`UTC` (optional, string, `America/New_York`) ... Standard timezone identification string.
    + to: `2014-07-20T09:00` (datetime, optional) - Datetime in format of YYYY-MM-DDTHH:MM.
        + Default: `now`
    + transmission_ids (optional, list, `65832150921904138`) ... delimited list of transmission ID's to search (i.e. id generated during creation of a transmission).
    + ab_test_ids (optional, list, `ab-12341`) ... delimited list of A/B test ID's to search.
+ Request

  + Headers

            Accept: application/json
            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
