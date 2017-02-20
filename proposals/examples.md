# Example Code

The prupose of this file is to quickly write up some examples of code that would be written to perform the various fucntions of a payment app for different use cases.

## Flow

For each use case we should see code that demonstrates the following, with descriptions of user interactions, and browser behaviour etc for clarity.

  1. The origin getting permission to "handle payments"
  2. A web app in the origin installing at least 1 service worker that has an event listener for the `onpaymentrequest` event
  3. The web app making the browser aware of the payment methods it supports (with any specific capabilities, eg: basic-card > visa)
  4. The web app handling the `onpaymentrequest` event and returning a response.
  
Let's leave the `canMakePyament` fucntionality out for now.

## Use Cases

  1. App that handles `basic-card` payments
  2. App that handles multiple payment methods
  3. App that handles a payment by making a network request before responding
  4. App that renders UI to get user confirmaation for the payment
  5. App that has multiple SW
  6. App that is installed as a result of the browser processing an app manifest
  
## Example 1


