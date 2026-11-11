# Overview of Sample flows for handling Rich Messages 

- The flows enclosed in this folder serve as samples for the below supported Rich Message features from flow to the customer:
  - Time picker
  - List picker
- These flows can be used as reference to send out rich messages to the customer for automated / bot scenarios.
- Ensure that the `request identifier` field is set to a static string, such as `system`, if rich message response flows are also made live.
  - ![AppleSetReqId](../../../images/AppleSetReqId.png)
  - If the business does not require rich messages to be sent out by agents, and handling of the corresponding responses from the customer, setting of request identifier is not required.
  - Refer to the [Rich message response flow Readme](/Webex%20Connect%20Flows/v3.3/Sample/Apple%20Business%20Messages/Rich%20message%20response%20flows/README.MD) to understand how to use the request identifier set here.
- Further details about each of these rich message features can be found on Apple's documentation
- https://register.apple.com/resources/messages/messaging-documentation/customer-journey#interactive-features
- 

