# Cross-Site-Scripting


## Cross-Site Scripting: Content Sniffing

### Abstract
Sending unvalidated data to a web browser may result in certain browsers executing malicious code.
### Explanation
Cross-site scripting (XSS) vulnerabilities occur when:

1. Data enters a web application through an untrusted source. In the case of reflected XSS, the untrusted source is typically a web request, while in the case of persisted (also known as stored) XSS it is typically a database or other back-end data store.


2. The data is included in dynamic content that is sent to a web user without validation.

The malicious content sent to the web browser often takes the form of a JavaScript segment, but may also include HTML, Flash or any other type of code that the browser executes. The variety of attacks based on XSS is almost limitless, but they commonly include transmitting private data such as cookies or other session information to the attacker, redirecting the victim to web content controlled by the attacker, or performing other malicious operations on the user's machine under the guise of the vulnerable site.

For the browser to render the response as HTML, or other document that may execute scripts, it has to specify a text/html MIME type. Therefore, XSS is only possible if the response uses this MIME type or any other that also forces the browser to render the response as HTML or other document that may execute scripts such as SVG images (image/svg+xml), XML documents (application/xml), etc.

Most modern browsers will not render HTML, nor execute scripts when provided a response with MIME types such as application/json. However, some browsers such as Internet Explorer perform what is known as Content Sniffing. Content Sniffing involves ignoring the provided MIME type and attempting to infer the correct MIME type by the contents of the response.
It is worth noting however, a MIME type of text/html is only one such MIME type that may lead to XSS vulnerabilities. Other documents that may execute scripts such as SVG images (image/svg+xml), XML documents (application/xml), as well as others may lead to XSS vulnerabilities regardless of whether the browser performs Content Sniffing.

Therefore, a response such as <html><body><script>alert(1)</script></body></html>, could be rendered as HTML even if its content-type header is set to application/json.

Example 1: The following AWS Lambda function reflects user data in an application/json response.

```
def mylambda_handler(event, context):
    name = event['name']
    response = {
        "statusCode": 200,
        "body": "{'name': name}",
        "headers": {
            'Content-Type': 'application/json',
        }
    }
    return response
```

If an attacker sends a request with the name parameter set to <html><body><script>alert(1)</script></body></html>, the server will produce the following response:

```
HTTP/1.1 200 OK
Content-Length: 88
Content-Type: application/json
Connection: Closed

{'name': '<html><body><script>alert(1)</script></body></html>'}
```

Even though, the response clearly states that it should be treated as a JSON document, an old browser may still try to render it as an HTML document, making it vulnerable to a Cross-Site Scripting attack.
