# Registration and Update
- The URL in a registration request resolves to a file of type "application/javascript"
- At registration time, the user agent retrieves this file (along with any associated icons) and stores it persistently
- The script is not updated automatically; however, it may call PaymentApp.register() itself, which will trigger a new download of the script
- _Open Issue: should we invoke the application automatically, periodically, to allow it to check for updates?_

# Invocation
_Note: This is based in part on the [WebRTC IdP instantiation process](http://w3c.github.io/webrtc-pc/#sec.create-identity-proxy)_

After Payment App Matching and Payment App selection:

- The user agent instantiates an isolated interpreted context, a JavaScript realm that operates in the origin of the payment app JavaScript.
- The realm is populated with a global that implements `WorkerGlobalScope` [[WEBWORKERS]](http://w3c.github.io/webrtc-pc/#bib-WEBWORKERS).
- The user agent creates an instance of `MessageChannel`, named `paymentPort`, in the global scope of the realm. This `MessageChannel` is used by the payment app to interact with the user agent.
- A global property can only be set by the user agent or the payment app itself. Therefore, the payment app can be assured that requests it receives originate from the user agent. This ensures that an arbitrary origin is unable to instantiate a payment app and impersonate this API.
