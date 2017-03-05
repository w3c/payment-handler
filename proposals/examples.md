# Example Code

The prupose of this file is to quickly write up some examples of code that would be written to perform the various fucntions of a payment app for different use cases.

## Flow

For each use case we should see code that demonstrates the following, with descriptions of user interactions, and browser behaviour etc for clarity.

  1. The origin [getting permission](#example2) to "handle payments"
  1. [Adding](#example31), [Updating](#example33), [Removing](#example32) payment methods
  1. Handling `.canMakePayment()` (*Let's leave this fucntionality out for now*).
  1. A web app in the origin installing at least 1 service worker that has [an event listener](#example1) for the `onpaymentrequest` event
  1. The web app [making the browser aware](#example31) of the payment methods it supports (with any specific capabilities, eg: basic-card > visa)
  1. The web app handling the `onpaymentrequest` event and [returning a response](#example4).
  1. Handling POSTing for payment (without a secure window)
  1. Getting the browser to open a secure window
  1. Handling PaymentRequest .abort(), .updateWith(), and whatever else PaymentRequest can throw at us.
  1. Creating a PaymentResponse - and showing how it coordinates between the secure window and merchant.
   
## Use Cases

  1. App that handles [`basic-card` payments](#app1)
  1. App that handles multiple payment methods
  1. App that handles a payment by making a network request before responding
  1. App that renders UI to get user confirmation for the payment
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
  console.log("got it! " + JSON.stringify(paymentrequestEvent.data));
});
```

<h3 id="example4">Returning a response to the `onpaymentrequest` event</h3>
```javascript
self.addEventListener('paymentrequest', function (paymentrequestEvent) {
  var accepted = true;
  var responseDetails = { reason: "a good reason" }; 
  paymentrequestEvent.respondWith(new Promise(function (resolve, reject) {
    if (accepted == true) resolve(responseDetails); else reject(responseDetails);
  }));
});
```

## App Samples

<h3 id="app1">`basic-card` payment</h3>
#### registration
```javascript
navigator.serviceWorker
    .register('/app-bc.js')
    .then(function(registration) {
      console.log("[app-bc] registration " + JSON.stringify(registration));

      // Set Registration methods & options
      if (registration.paymentAppManager) {
        registration.paymentAppManager.options.set("app-bc_key", {
          enabledMethods: ["basic-card"],
          name: "app-bc for MyCard",
          id: "M. JAUNE D'EAU;4111111111111111;12;25;987"
        });
      }

      // Forward registration object to the next function
      return (registration);

    }).then(function(sw) {
      // Previous step is successfully completed
      console.log("[app-bc] Payment-App Service Worker is " + (sw.installing || sw.waiting || sw.active).state);

    }).catch(function(exception) {
      alert("[app-bc] Dev.Exception: " + exception);
    });
```

#### execution (app-bc.js)
Actual running full project [here](https://github.com/pjbazin/wapp-examples)
```javascript
self.addEventListener('install', function(event) {
  // 'install' event is fired when the SW registration is successfully completed.
  console.log("[app-bc] is installed");
});


self.addEventListener('activate', function(event) {
  // 'activate' event is fired after 'install' event.
  console.log("[app-bc] is activated");
});


self.addEventListener('paymentrequest', function(paymentrequestEvent) {
  console.log("[app-bc] Got paymentrequest " + JSON.stringify(paymentrequestEvent.data));

  // Send back selected option to the payment requester (Payee/Merchant, PISP,...)
  paymentrequestEvent.respondWith(new Promise(function(resolve, reject) {

    try {
      // Retrive payment card options
      var optionIds = paymentrequestEvent.data.optionId.split(';');

      // Build Basic-Card response according to the specific BasicCardResponse dictionary model
      var basicCardResponse = {
        methodName: "basic-card",
        details: {
          cardholderName: optionIds[0],
          cardNumber: optionIds[1],
          expiryMonth: optionIds[2],
          expiryYear: optionIds[3],
          cardSecurityCode: optionIds[4],
          billingAddress: null
        }
      };

      // Callback Promise
      resolve(basicCardResponse);

    } catch (exception) { reject(exception) }

  }));

});

```
