# Network Errors with Cors
When browser javascript makes cors requests. If there is a network failure -- think of dns
not resolving or other transient network issues -- you see will be `CORS request did not succeed` in the js console.

MDN has a good [CORS Error Explanation](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSDidNotSucceed)
page with one of the reasons being

> The server did not respond to the actual request (even if it responded to the Preflight request). One scenario might be an HTTP service being developed that panicked without returning any data.
