# Example Code

The prupose of this file is to quickly write up some examples of code that would be written to perform the various fucntions of a payment app for different use cases.

## Flow

For each use case we should see code that demonstrates the following, with descriptions of user interactions, and browser behaviour etc for clarity.

  1. [The origin getting permission to "handle payments"](#example2)
  1. [Adding](#example31), [Updating](#example33), [Removing](#example32) payment methods
  1. Handling `.canMakePayment()` - Let's leave this fucntionality out for now.
  1. [A web app in the origin installing at least 1 service worker that has an event listener for the `onpaymentrequest` event](#example1)
  1. The web app making the browser aware of the payment methods it supports (with any specific capabilities, eg: basic-card > visa)
  1. The web app handling the `onpaymentrequest` event and returning a response.
  1. Handling POSTing for payment (without a secure window)
  1. Getting the browser to open a secure window
  1. Handling PaymentRequest .abort(), .updateWith(), and whatever else PaymentRequest can throw at us.
  1. Creating a PaymentResponse - and showing how it coordinates between the secure window and merchant.
   
## Use Cases

  1. App that handles `basic-card` payments
  1. App that handles multiple payment methods
  1. App that handles a payment by making a network request before responding
  1. App that renders UI to get user confirmaation for the payment
  1. App that has multiple SW
  1. App that is installed as a result of the browser processing an app manifest
  
## Code Samples

<h3 id="example2">Getting permission to "handle payments"</h3>
```javascript
navigator.serviceWorker
  .register('/app.js', 'https://apps.domain.net/')
  .then(function(registration) {
    registration.paymentAppManager.options.set(
      "app-option-key",
      {
        name: "app XXXX",
        enabledMethods: ["https://apps.domain.net/app"],
      }
    );
});
```

<h3 id="example31">Adding payment methods</h3>
```javascript
navigator.serviceWorker
  .getRegistration('https://apps.domain.net/')
  .then(function(registration) {
    registration.paymentAppManager.options.set(
      "app-option2-key",
      {
        name: "Visa xxxx8200",
        enabledMethods: ["basic-card"],
      }
    );
});
```

<h3 id="example33">Updating payment methods</h3>
```javascript
navigator.serviceWorker
  .getRegistration('https://apps.domain.net/')
  .then(function(registration) {
    var walletKey = registration.paymentAppManager.options.get("app-option2-key");
    registration.paymentAppManager.options.set( walletKey,
      {
        name: "Visa xxxx8299",
        enabledMethods: ["basic-card"],
      }
    );
});
```

<h3 id="example32">Removing payment methods</h3>
```javascript
navigator.serviceWorker
  .getRegistration('https://apps.domain.net/')
  .then(function(registration) {
    registration.paymentAppManager.options.delete("app-option2-key");
});
```

<h3 id="example1">A service worker that has an event listener for the `onpaymentrequest` event</h3>
```javascript
self.addEventListener('paymentrequest', function (paymentrequestEvent) {
  paymentrequestEvent.respondWith(new Promise(function (resolve, reject) {
    resolve(response);
  }));
});
```


