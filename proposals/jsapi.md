# Registration and Update
- The URL in a [registration request](https://w3c.github.io/webpayments-payment-apps-api/#paymentapp.register) resolves to a file of type "application/javascript"
- At registration time, the user agent retrieves this file (along with any associated icons) and stores it persistently
- The script is not updated automatically; however, it may call PaymentApp.register() itself, which will trigger a new download of the script
- _Open Issue: should we invoke the application automatically, periodically, to allow it to check for updates?_

# Invocation
_Note: This is based in part on the [WebRTC IdP instantiation process](http://w3c.github.io/webrtc-pc/#sec.create-identity-proxy)_

After Payment App Matching and Payment App selection:

- The user agent instantiates an isolated interpreted context, a JavaScript realm that operates in the origin of the payment app JavaScript.
- The realm is populated with a global that implements `WorkerGlobalScope` [[WEBWORKERS]](http://w3c.github.io/webrtc-pc/#bib-WEBWORKERS).
- The user agent creates an instance of `MessageChannel`, named `paymentChannel`, in the global scope of the realm. This `MessageChannel` is used by the payment app to interact with the user agent.
- A global property can only be set by the user agent or the payment app itself. Therefore, the payment app can be assured that requests it receives originate from the user agent. This ensures that an arbitrary origin is unable to instantiate a payment app and impersonate this API.

At this point, the payment app is executed. It is required to add an event handler to `port1` of the `paymentChannel` during this initial execution. After the payment app is executed, the user agent queues a payment request message on `port1` of the `paymentChannel`. The payment app performs whatever actions are necessary to process the payment, and then sends a payment response on `port2` of the `paymentChannel`.

As a simple example, the script necessary to post a payment request to a remote server and receive a response with the payment response could look like something this (modulo additional error handling):

```js
paymentChannel.port1.onmessage = function(e) {
  fetch("https://www.example.com/bobpay/process", { method: "POST",  body: e.data })
  .then(function(response) {
    return response.text();
  }).then(function(body) {
    paymentChannel.port2.postMessage(body);
  }).catch(function(e) {
    paymentChannel.port2.postMessage(e.message);
  });
}
```

# User Interaction

Payment apps frequently need to be able to interact with users (e.g. to prompt for authentication information). To faciliate these situations, define a new interface, `PaymentWindow`:

```webidl
interface PaymentWindow {
  Promise<DOMString> openBody(DOMString body);
  Promise<DOMString> openUrl(URLString url);
  void close();
}
```

_Note: The actual rendering of the PaymentWindow is an implementation detail; while opening an entirely new window is possible, it is more likely that the contents will be rendered in a way that makes it more obvious that the interactions pertain to the payment transaction. This is an area for potential user agent experimentation and differentiation._

When the payment app context is instantiated, the user agent creates an instance of `PaymentWindow`, called `paymentWindow`, in the global scope of the realm. If the payment app calls `paymentWindow.openBody(body)`, a new window is created, and the value of "body" is loaded into the window. There is also a convenience method `paymentWindow.openUrl(url)`, which instead fetches the contents of the window from the indicated URL.

In the context of the PaymentWindow, there is an injected `promise<void>` at the global level, called paymentWindowPromise. Once the user interaction is complete, the content in the payment window calls `paymentWindowPromise.resolve(string)`, which causes payment window to close, and the promise returned from `openBody()` or `openUrl()` to successfully resolve. If the content in the payment window calls `paymentWindowPromise.reject()`, or if the payment window closes prior to calling `paymentWindowPromise.resolve()`, then the promise returned from `openBody()` or `openUrl()` is rejected.

The payment app can close the payment window at any time by calling `paymentWindow.close()`.

# Appendix: Using HTTP POST

The current version of the Payment Application specification sketches out a scheme by which a POST is sent to a URL with the payment request as a body. The response is allowed to be either `application/json` (which is inferred to contain a payment response), or `text/html` (which contains content to be rendered to the user). The following code demonstrates how this functionality could be achived using the API proposed above.

```js
paymentChannel.port1.onmessage = function(e) {
  fetch("https://www.example.com/bobpay/process", { method: "POST",  body: e.data })
  .then(function(response) {
    return response.text();
  }).then(function(body) {
      var contentType = response.headers.get("content-type");
      if (!contentType) {
        throw(new Error("No content type header"));
      }
      if(contentType.indexOf("application/json") !== -1) {
        paymentChannel.port2.postMessage(body);
      } else if (contentType.indexOf("text/html") !== -1) { {
        paymentWindow.openBody(body).then(function(response) {
          paymentChannel.port2.postMessage(response);
        });
      } else {
        throw(new Error("Unexpected value in content type header"));
      }
  }).catch(function(e) {
    paymentChannel.port2.postMessage(e.message);
  });
}
```
