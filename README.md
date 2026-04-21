# AWS Lambda + SQS Demo

A minimal AWS Lambda handler that processes messages from an SQS queue.
Demonstrates the event-driven, decoupled pattern I've used extensively in
production for async workflow processing.

## Tech stack

- Java 17
- AWS SDK v2
- Maven
- AWS Lambda + SQS

## Handler code

```java
package com.loganathan.demo;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.lambda.runtime.events.SQSEvent;

public class OrderEventHandler implements RequestHandler<SQSEvent, String> {

    @Override
    public String handleRequest(SQSEvent event, Context context) {
        int processed = 0;
        for (SQSEvent.SQSMessage msg : event.getRecords()) {
            try {
                processMessage(msg.getBody(), context);
                processed++;
            } catch (Exception e) {
                context.getLogger().log("Failed to process message: " + e.getMessage());
                throw e; // send to DLQ
            }
        }
        return "Processed " + processed + " messages";
    }

    private void processMessage(String payload, Context ctx) {
        ctx.getLogger().log("Processing payload: " + payload);
        // business logic here
    }
}
```

## Design notes

- Throws on failure to push the message to the Dead Letter Queue (DLQ)
- Batch-size tuning handled at the SQS trigger level
- Stateless — suitable for high-throughput async processing
