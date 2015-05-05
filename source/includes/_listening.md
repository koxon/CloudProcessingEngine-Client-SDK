# Listening for updates

```ruby
/* Keep polling for messages! Forever ... */
while (42)
{
    try {
        /* Will poll for 10 seconds then return */
        if ($msg = $CTComSDK->receive_message(10))
        {
            if (!($decoded = json_decode($msg['Body'])))
                throw new Exception("JSON data is invalid!");
            else
            {
                /* Custom function that will parse the JSON data and do something */
                parse_output($decoded);
            }

            /* Message polled. We delete it from SQS */
            $CTComSDK->delete_message($msg);
        }
    } catch (Exception $e) {
        print("[ERROR] " . $e->getMessage() . "\n");
    }
}
```

<aside class="notice">
Call this method to retrieve a possible message from the stack. This method polls messages from the 'output' SQS channel configured for your client.
</aside>

The method will poll the 'output' SQS queue. If no message are present it returns 'false'.
If there is a message, it returns the String containing this message.
