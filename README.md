# HealthEngine Booking Webhooks

Documentation for booking webhooks from the HealthEngine platform.

## Synopsis

Webhooks can be utilised to notify external parties of booking events that have occurred within the HealthEngine platform.

The table below shows the events that currently support webhooks.

| Event Name                                                            | Description                                                                    |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| [`booking-submitted`](#booking-submitted)                             | A booking was made using the HealthEngine platform.                            |
| [`booking-cancelled`](#booking-cancelled)                             | A booking has been cancelled on the HealthEngine platform.                     |
| [`booking-pre-screening-submitted`](#booking-pre-screening-submitted) | A booking has had a pre-screening form submitted on the HealthEngine platform. |
| [`booking-marked-attended`](#booking-marked-attended)                 | A booking has been marked as attended on the HealthEngine platform.            |

## Event Details

Webhooks are configured per-practice and will be delivered to the practice’s configured URL.

Some general notes and limitations of event webhooks:

- Due to the highly available and redundant architecture, duplicate webhooks may be sent, including simultaneously,
- If the HTTP response code is not 200, the webhook will be retried,
- A maximum of 3 retries are made before the webhook is marked as failed,
- The retries have a backoff strategy of 5 minutes, 1 hour and 12 hours,
- Webhooks will only be delivered to HTTPS,
- A webhook subscription must be confirmed before events are sent ([see below](#subscribing-to-an-event)),
- A response must be received within 5 seconds or the request will be terminated. Any long running process should be triggered asynchronously,

### `booking-submitted`

This event is triggered after a booking has been made within the HealthEngine platform. An example payload is shown below.

```json
{
  "version": 1,
  "type": "booking-submitted",
  "data": {
    "appointment": {
      "datetime": 1577808000,
      "type": "General Appointment"
    },
    "booking_id": "1234",
    "practice": {
      "id": "9876",
      "timezone": "Australia/Perth"
    },
    "patient": {
      "address": {
        "postcode": "6000",
        "state": "WA",
        "street": "Wellington St",
        "suburb": "Perth"
      },
      "dob": "1970-01-01",
      "email": "noreply@healthengine.com.au",
      "firstname": "Jane",
      "lastname": "Blogs",
      "medicare":
        null |
        {
          "expiry": "01/2022",
          "number": "1",
          "reference": "1234 56789 0"
        },
      "mobile_phone": "0412345678"
    }
  }
}
```

### `booking-cancelled`

This event is triggered after a booking has been cancelled on the HealthEngine platform. An example payload is shown below.

```json
{
  "version": 1,
  "type": "booking-cancelled",
  "data": {
    "booking_id": "1234",
    "practice": {
      "id": "9876"
    },
    "patient": {
      "address": {
        "postcode": "6000",
        "state": "WA",
        "street": "Wellington St",
        "suburb": "Perth"
      },
      "dob": "1970-01-01",
      "email": "noreply@healthengine.com.au",
      "firstname": "Jane",
      "lastname": "Blogs",
      "mobile_phone": "0412345678"
    },
    "cancelled_time": 1537809000
  }
}
```

### `booking-pre-screening-submitted`

This event is triggered after a pre-screening form has been submitted on the HealthEngine platform. An example payload is shown below.

```json
{
  "version": 1,
  "type": "booking-pre-screening-submitted",
  "data": {
    "booking_id": "1234",
    "pre_screening_form": {
      "question_answers": [
        {
          "question_id": "aeb110ab-6446-481b-b383-908be888fe37",
          "answer": {
            "value": true | false
          },
          "question_wording": "You have had any vaccine in the past month",
          "variant": "YES_NO"
        },
        {
          "question_id": "f98ae1df-b110-4e0f-83a6-fe6b69a28c48",
          "answer": {
            "value": true | false,
            "specification": "I am a nurse." | null
          },
          "question_wording": "You have an occupation or lifestyle factor(s) for which vaccination may be needed (discuss with doctor/pharmacist/nurse). Please specify below:",
          "variant": "YES_NO_SPECIFY"
        },
        {
          "question_id": "0b039374-5f98-4078-917d-659505528723",
          "question_wording": "Before any vaccination takes place, clinic staff should ask you:
- Is this a bullet point?
- Is **this** in bold?
- Is _this_ italicised?
",
          "answer": null,
          "variant": "INFORMATION"
        }
      ]
    },
    "practice": {
      "id": "9876"
    },
    "submitted_at": 1537809000
  }
}
```

### `booking-marked-attended`

This event is triggered after a booking has been marked as attended on the HealthEngine platform. An example payload is shown below.

```json
{
  "version": 1,
  "type": "booking-marked-attended",
  "data": {
    "appointment": {
      "datetime": 1577808000,
    },
    "booking_id": "1234",
    "practice": {
      "id": "9876",
      "name": "Practice Name"
    },
    "patient": {
      "address": {
        "postcode": "6000",
        "state": "WA",
        "street": "Wellington St",
        "suburb": "Perth"
      },
      "dob": "1970-01-01",
      "email": "noreply@healthengine.com.au",
      "firstname": "Jane",
      "lastname": "Blogs",
      "medicare":
        null |
        {
          "expiry": "01/2022",
          "number": "1",
          "reference": "1234 56789 0"
        },
      "mobile_phone": "0412345678"
    }
  }
}
```

## Subscribing to an event

Before real events are sent, the webhook URL subscription must be confirmed. This ensures that the URL is valid and correct.  
When a new webhook is configured, an example payload is sent to the URL with the JSON payload below.  
The third party must retrieve the confirmation_url from the body and make a HTTPS request to that endpoint to confirm the subscription.  
Once we receive this request the webhook will be enabled and events will begin being delivered to the endpoint.

```json
{
  "version": 1,
  "type": "subscription-confirmation",
  "data": {
    "confirmation_url": "https://healthengine.com.au/api/4/practice-webhook-subscription?id=1234&signature=9023a0ef234"
  }
}
```

_Example:_ Here is an example cURL request to confirm a webhook subscription.

`curl --request GET --url 'https://healthengine.com.au/api/4/practice-webhook-subscription?id=1234&signature=9023a0ef234' --header 'Accept: application/json'`

_Note:_ Since the configured endpoint can receive payloads of different formats, the third party can inspect the type parameter in the JSON body to determine which type of payload has been received. This can be used to infer the structure of the data parameter.

_Note:_ The `confirmation_url` is only valid for 1 hour.

_Note:_ When making the confirmation request, you must set the Accept header to application/json. The response has the format: `{"success": true}`.

## Whitelist IPs

To secure your endpoint it is suggested to whitelist HealthEngine's IP addresses being:

- 13.55.48.1
- 52.62.53.70
- 13.54.122.64
