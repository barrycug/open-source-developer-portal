---
layout: twoColumn
section: guides
type: guide
guide:
    name: webhooks
    step: '3'
title: Webhooks | Validating Webhooks
description: Validating webhooks for payments within your application by utilizing our open API with no per transaction fees.
---

# Step 3: Validating webhooks

Assume that your integration is an online marketplace, and that a customer just placed an order on your site. A few days after the customer initiated their payment, your application receives this webhook.

The `topic` field of an event holds [a description](http://docsv2.dwolla.com/#events) of the event, which is similar the subject of an e-mail message.  The `webhook` itself contains `_links` to the resource impacted by the event that can be used to retrieve more information about the webhook you have received. 

**NOTE**: The `event` must be retrieved with a `client_credentials` granted access_token.

##### JSON

```jsonnoselect
{
  "id": "cac95329-9fa5-42f1-a4fc-c08af7b868fb",
  "resourceId": "cdb5f11f-62df-e611-80ee-0aa34a9b2388",
  "topic": "customer_transfer_created",
  "timestamp": "2017-01-20T22:45:12.790Z",
  "_links": {
    "self": {
      "href": "https://api-sandbox.dwolla.com/events/cac95329-9fa5-42f1-a4fc-c08af7b868fb"
    },
    "account": {
      "href": "https://api-sandbox.dwolla.com/accounts/ad5f2162-404a-4c4c-994e-6ab6c3a13254"
    },
    "resource": {
      "href": "https://api-sandbox.dwolla.com/transfers/cdb5f11f-62df-e611-80ee-0aa34a9b2388"
    },
    "customer": {
      "href": "https://api-sandbox.dwolla.com/customers/e358a488-6699-4d79-bbfb-c5bf58100ea4"
    }
  },
  "created": "2017-01-20T22:45:12.790Z"
}
```

## Step 3A. Authenticating the webhook request

Before we process any data from the webhook we’ll want to validate that the request really came from Dwolla and not someone pretending to be Dwolla. Dwolla signs each webhook request with the `secret` you passed in when you created the webhook subscription. The signature is contained in the `X-Request-Signature-Sha-256` header and is a SHA256 HMAC hash of the request body with the key being your webhook secret.

You can validate the webhook by generating the same SHA256 HMAC hash and comparing it to the signature sent with the payload.

```ruby
def verify_signature(payload_body, request_signature)
  signature = OpenSSL::HMAC.hexdigest(OpenSSL::Digest.new("sha256"),
ENV["DWOLLA_WEBHOOK_SECRET"],
payload_body)
  unless Rack::Utils.secure_compare(signature, request_signature)
    halt 500, "Signatures didn't match!"
  end
end
```

```raw
not available
```

```javascript
var verifyGatewaySignature = function(proposed_signature, webhook_secret, payload_body) {
  var crypto    = require('crypto');

  var hash = crypto.createHmac('sha256', webhook_secret).update(payload_body).digest('hex');

return proposed_signature === hash;
}
```

```python
def verify_gateway_signature(proposed_signature, webhook_secret, payload_body):
  import hmac
  from hashlib import sha256

  signature = hmac.new(webhook_secret, payload_body, sha256).hexdigest()

  return True if (signature == proposed_signature) else False
```

```php
<?php
function verifyGatewaySignature($proposedSignature, $webhookSecret, $payloadBody) {
    $signature = hash_hmac("sha256", $payloadBody, $webhookSecret);
    return $signature == $proposedSignature;
}
?>
```

## Step 3B. Check for duplicate events

It is important to consider that multiple webhooks are fired for the same action on certain events. For example, multiple webhooks are fired for `Transfer` events, that is, two `transfer_created` events with different resource IDs (and, by extension, resource URLs) will be fired, one for each customer. To avoid doing any business logic twice, you would have to check if you have already received a webhook relating to the `Transfer` resource responsible for the event.

To do this, keep a queue of events in a database and check to see if an `Event` has the same `self` resource location in `_links` as another event. If not, process the logic for that event. To illustrate, this is how a developer would implement this using Ruby and the ActiveRecord ORM.

##### Ruby/ActiveRecord
```rubynoselect
check_db = ActiveRecord::Base.connection.execute("SELECT * FROM EVENTS WHERE SELF = #{event[:_links][:self].to_s}")

# check_db will be an array of rows returned
unless check_db.length() == 0
    # do something
end
```

<nav class="pager-nav">
    <a href="create-subscription.html">Back: Create a webhook subscription</a>
    <a href="process-webhooks.html">Next step: Processing webhooks</a>
</nav>
