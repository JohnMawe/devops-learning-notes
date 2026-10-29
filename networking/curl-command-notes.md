# `curl` Command-Line Notes

## 1. What is curl?

`curl` is a command-line tool for transferring data to and from servers
using supported network protocols, especially HTTP and HTTPS.

For backend development and DevOps, `curl` is useful for:

-   Testing APIs without a frontend or graphical API client.
-   Sending HTTP requests and JSON data.
-   Inspecting response headers and status codes.
-   Downloading and uploading files.
-   Checking websites and deployed services.
-   Troubleshooting connection, DNS, TLS, and HTTP problems.
-   Automating health checks in Bash scripts.

## 2. Install and inspect curl

On Ubuntu or Debian:

``` bash
sudo apt update
sudo apt install curl
```

Check the installed version, features, TLS implementation, and supported
protocols:

``` bash
curl --version
```

Get general help or the full manual:

``` bash
curl --help
curl --help all
man curl
```

## 3. Basic syntax

``` bash
curl URL
```

Example:

``` bash
curl http://127.0.0.1:5000/
```

This sends a GET request to the root endpoint of a service running on
the same computer at port `5000`.

-   `http://` is the protocol.
-   `127.0.0.1` is the loopback address (localhost).
-   `:5000` is the port.
-   `/` is the endpoint path.

The server must be running and listening on that address and port.
Change the URL to match your application.

## 4. Common HTTP methods

### GET: retrieve data

GET is curl's default method when no data or alternative method is
specified.

``` bash
curl http://127.0.0.1:5000/advice
```

### POST: create or submit data

Use `--json` to send JSON. It sets the usual JSON request headers and
uses POST by default.

``` bash
curl --json '{"advice":"Keep learning every day","author_id":1}' \
  http://127.0.0.1:5000/advice
```

The JSON keys must exactly match the fields expected by the API. For
example, `authro_id` is not the same key as `author_id`.

The API must support POST at this endpoint and accept the supplied
fields.

An alternative using `-X`, `-H`, and `-d` is:

``` bash
curl -X POST http://127.0.0.1:5000/advice \
  -H "Content-Type: application/json" \
  -d '{"advice":"Keep learning","author_id":1}'
```

### PUT and PATCH: update data

These methods are examples only; the API must implement them.

``` bash
curl -X PUT http://127.0.0.1:5000/advice/1 \
  -H "Content-Type: application/json" \
  -d '{"advice":"Practise Linux daily","author_id":1}'
```

``` bash
curl -X PATCH http://127.0.0.1:5000/advice/1 \
  -H "Content-Type: application/json" \
  -d '{"advice":"Practise Linux and networking"}'
```

Generally, PUT replaces a resource representation, while PATCH applies a
partial update. Exact behavior depends on the API.

### DELETE: request deletion

``` bash
curl -X DELETE http://127.0.0.1:5000/advice/1
```

Only use this against a test record or when you are certain deletion is
intended. The endpoint must support DELETE.

## 5. Request headers and query parameters

### Headers

`-H` adds a request header.

``` bash
curl http://127.0.0.1:5000/advice \
  -H "Accept: application/json"
```

-   `Content-Type` describes the format of the request body.
-   `Accept` indicates the response format the client prefers.
-   `Authorization` can carry credentials or a token.

For example, a Bearer token:

``` bash
curl https://api.example.com/profile \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

Replace the example URL and placeholder token. Never commit real tokens
to Git or share output that exposes credentials.

### Query parameters

``` bash
curl "http://127.0.0.1:5000/advice?category=motivation&limit=5"
```

Use `--get` with `--data-urlencode` to let curl encode query parameter
values:

``` bash
curl --get http://127.0.0.1:5000/advice \
  --data-urlencode "category=daily motivation"
```

## 6. Inspect response status codes and headers

### Include response headers

``` bash
curl -i http://127.0.0.1:5000/advice
```

`-i` displays response headers along with the response body.

### Show only the HTTP status code

``` bash
curl -s -o /dev/null -w "%{http_code}\n" \
  http://127.0.0.1:5000/advice
```

-   `-s` hides the progress meter.
-   `-o /dev/null` discards the response body on Linux.
-   `-w` prints selected information; `%{http_code}` is the HTTP status
    code.

### Common status codes

  -----------------------------------------------------------------------------
  Code                          Meaning                 Typical example
  ----------------------------- ----------------------- -----------------------
  `200 OK`                      Request succeeded       Data retrieved

  `201 Created`                 A resource was created  Successful creation
                                                        request

  `204 No Content`              Successful response     Successful deletion
                                with no body            

  `400 Bad Request`             Request is invalid      Missing or invalid
                                                        fields

  `401 Unauthorized`            Authentication is       Missing/invalid
                                missing or invalid      credentials

  `403 Forbidden`               Access is not permitted Insufficient
                                                        permissions

  `404 Not Found`               Route or resource not   Wrong URL or ID
                                found                   

  `405 Method Not Allowed`      Method is not supported POST sent to a GET-only
                                at this route           route

  `500 Internal Server Error`   Server-side error       Unhandled application
                                                        exception
  -----------------------------------------------------------------------------

These are common conventions. An API may use other valid status codes
depending on its design.

An HTTP error such as 404 means a server returned an HTTP response. A
connection error may happen before curl receives any HTTP response.

## 7. Debugging and connection options

### Verbose mode

``` bash
curl -v http://127.0.0.1:5000/advice
```

`-v` displays connection details and request/response headers. It can
help diagnose connection and HTTP problems. Be careful: verbose output
may expose sensitive headers or credentials.

### Follow redirects

``` bash
curl -L http://example.com
```

`-L` follows HTTP redirects.

### Inspect headers only

``` bash
curl -I https://example.com
```

`-I` makes a HEAD request to retrieve response headers without the
normal response body. Some servers handle HEAD differently or do not
support it.

### Set timeouts

``` bash
curl --connect-timeout 5 --max-time 15 \
  https://example.com
```

-   `--connect-timeout 5` limits the time spent establishing the
    connection.
-   `--max-time 15` limits the total transfer time.

### Test a particular IP for a hostname

``` bash
curl --resolve example.com:443:203.0.113.10 \
  https://example.com/
```

Replace the example hostname and documentation IP with your own test
values. This lets curl connect to a specified IP while using the
hostname in the URL for HTTP and TLS behavior.

### HTTPS certificate verification

Do not routinely use `-k` or `--insecure` to bypass certificate checks.
Investigate the certificate, hostname, and trust configuration instead.

## 8. Downloading files

### Save using the filename from the URL

``` bash
curl -O https://example.com/file.zip
```

`-O` uses the filename in the URL when available.

### Choose the local filename

``` bash
curl -o myfile.html https://example.com
```

`-o` lets you choose the output filename.

### Resume a download

``` bash
curl -C - -O https://example.com/large-file.zip
```

`-C -` attempts to resume from the existing local file when the server
supports it.

Download only from trusted sources. Verify checksums or signatures when
provided, especially before running downloaded programs or installers.

## 9. Uploading files

For an HTTP endpoint that accepts multipart form uploads:

``` bash
curl -F "file=@report.pdf" \
  https://example.com/upload
```

`-F` sends a multipart form field. `@report.pdf` tells curl to read the
local file.

The server must implement the upload endpoint and accept the field name
`file`.

For a server that supports FTP uploads:

``` bash
curl -T report.pdf ftp://example.com/uploads/
```

Use secure transfer methods for sensitive information and verify server
permissions and authentication requirements.

## 10. Cookies and authentication

### Basic authentication

``` bash
curl -u username:password https://example.com/private
```

This uses HTTP Basic Authentication. Avoid putting real passwords
directly in commands because they can appear in shell history or process
information.

### Save cookies

``` bash
curl -c cookies.txt https://example.com/login
```

`-c` saves cookies received from the server.

### Send saved cookies

``` bash
curl -b cookies.txt https://example.com/dashboard
```

`-b` sends cookies from a file. Real login flows may also require a POST
request, CSRF token, or other session-handling steps. Protect cookie
files because they can contain session credentials.

## 11. Other protocols

curl supports multiple protocols, depending on the build. Check
`curl --version` for the protocols available on your system. Depending
on the build, these can include HTTP/HTTPS, FTP, SFTP, SCP, SMTP, and
others.

Related Linux tools:

-   `curl`: transfers data over supported protocols.
-   `ssh`: opens a remote shell or runs a remote command.
-   `scp`: copies files over SSH.
-   `wget`: another tool commonly used for downloading files.

For regular SSH-based file transfers, `scp` is often more
straightforward than curl.

## 12. Automating an API health check with Bash

Create a file named `check_api.sh`:

``` bash
#!/usr/bin/env bash

URL="http://127.0.0.1:5000/"

STATUS=$(curl -s -o /dev/null \
  -w "%{http_code}" \
  --connect-timeout 5 \
  --max-time 10 \
  "$URL")
CURL_EXIT=$?

if [ "$CURL_EXIT" -ne 0 ]; then
    echo "API check failed: curl could not complete the request"
    exit 1
elif [ "$STATUS" = "200" ]; then
    echo "API is healthy"
else
    echo "API check failed: HTTP $STATUS"
    exit 1
fi
```

Make it executable and run it:

``` bash
chmod +x check_api.sh
./check_api.sh
```

This script checks both curl's exit status and the HTTP response code. A
successful curl transfer does not necessarily mean the server returned a
successful HTTP status, so check both.

The example considers only HTTP 200 healthy. Adjust the expected status
if your health endpoint uses a different successful response. The URL
must point to a running service.

This introduces useful DevOps concepts:

-   **Automation:** repeatable checks without manual effort.
-   **Health checks:** testing whether a service responds as expected.
-   **Exit codes:** reporting success or failure to other scripts and
    CI/CD systems.

## 13. Common problems and what to investigate

  -----------------------------------------------------------------------
  Symptom                             What to check
  ----------------------------------- -----------------------------------
  `Connection refused`                Is the server running and listening
                                      on the expected port?

  `Could not resolve host`            Hostname spelling, DNS, or network
                                      configuration

  HTTP `404`                          URL path, route registration, and
                                      resource ID

  HTTP `405`                          Whether the endpoint supports the
                                      requested method

  HTTP `400`                          JSON syntax, required fields, data
                                      types, and validation rules

  HTTP `401` or `403`                 Credentials, token, permissions,
                                      and authentication setup

  HTTP `500`                          Server logs and application
                                      traceback

  TLS/certificate error               Certificate validity, hostname,
                                      trust chain, and system time

  Request takes too long              Network connectivity, server
                                      responsiveness, and timeout
                                      settings
  -----------------------------------------------------------------------

When debugging a Flask application, inspect both curl's output and the
Flask server logs.

## 14. Practice checklist

-   [ ] Check the installed curl version and supported protocols.
-   [ ] Send GET requests to the root endpoint and `/advice`.
-   [ ] Send a valid JSON POST request.
-   [ ] Deliberately misspell a required JSON key and inspect the error.
-   [ ] Test an empty or invalid request body.
-   [ ] Use `-i` to inspect headers and status codes.
-   [ ] Use `-v` to investigate a connection.
-   [ ] Try query parameters and URL encoding.
-   [ ] Download a file using `-O` and `-o`.
-   [ ] Measure the response time of an endpoint.
-   [ ] Write and run the Bash health-check script.
-   [ ] Repeat the health check against a deployed API over HTTPS.

## 15. Recommended official documentation

-   curl tutorial: https://curl.se/docs/tutorial.html
-   HTTP scripting with curl: https://curl.se/docs/httpscripting.html
-   Complete curl manual: https://curl.se/docs/manpage.html

## Key takeaway

curl is not just an API-testing command. It is a flexible
network-transfer tool that helps developers inspect servers, transfer
files, debug HTTP and HTTPS connections, and automate service checks.
