# mitmproxy

Containerized version of [mitmproxy](https://mitmproxy.org/), an interactive SSL-capable intercepting HTTP proxy.

# Usage

```sh
$ docker run --rm -it [-v ~/.mitmproxy:/home/mitmproxy/.mitmproxy] -p 8080:8080 mitmproxy/mitmproxy
```
The *volume mount* is optional: It's to store the generated CA certificates.

Once started, mitmproxy listens as a HTTP proxy on `localhost:8080`:
```sh
$ http_proxy=http://localhost:8080/ curl http://example.com/
$ https_proxy=http://localhost:8080/ curl -k https://example.com/
```

You can also start `mitmdump` by just adding that to the end of the command-line:

```sh
$ docker run --rm -it -p 8080:8080 mitmproxy/mitmproxy mitmdump
```

## Reverse Proxy with mitmdump

Using as a reverse proxy, make sure to just change the port number of the target service. For instance, if you want to print all Request/Response headers, create a python script:

```python
cat parse_headers.py
def response(context, flow):
  request_headers = [{"name": k, "value": v} for k, v in flow.request.headers.iteritems()]
  response_headers = [{"name": k, "value": v} for k, v in flow.response.headers.iteritems()]
  print "################################"
  print "FOR: " + flow.request.url
  print flow.request.method + " " + flow.request.path + " " + flow.request.http_version
  print "HTTP REQUEST HEADERS"
  print request_headers
  print "HTTP RESPONSE HEADERS"
  print response_headers
  print ""
```

The, run the service with the parse_headers.py script as follows:

```sh
$ docker run -ti -v $PWD/parse_headers.py:/tmp/parse_headers.py -p 8080:8080 mitmproxy/mitmproxy \
    mitmdump -s /tmp/parse_headers.py -R http://pe2enpmas300.corp.company.net:8081 8080
```

The logs will display the following:

```
...
################################
FOR: http://pe2enpmas300.corp.company.net:8081/csv-stringify
GET /csv-stringify HTTP/1.1
HTTP REQUEST HEADERS
[{'name': 'accept-encoding', 'value': 'gzip'}, {'name': 'authorization', 'value': 'Bearer d2e0770656a9726dfb559ea2ddccff3078dba9a0'}, {'name': 'version', 'value': '2.11.2'}, {'name': 'accept', 'value': 'application/json'}, {'name': 'referer', 'value': 'install restify'}, {'name': 'npm-session', 'value': 'a9a4d805c6392599'}, {'name': 'user-agent', 'value': 'npm/2.11.2 node/v0.10.25 linux x64'}, {'name': 'if-none-match', 'value': 'W/"43fb-8/w7tzRZ9CvawCJo5Uiisg"'}, {'name': 'host', 'value': 'registry-e2e.npmjs.company.net'}, {'name': 'Connection', 'value': 'keep-alive'}, {'name': 'X-Forwarded-For', 'value': '10.181.70.43'}]
HTTP RESPONSE HEADERS
[{'name': 'X-Powered-By', 'value': 'Express'}, {'name': 'ETag', 'value': 'W/"43fb-8/w7tzRZ9CvawCJo5Uiisg"'}, {'name': 'Date', 'value': 'Tue, 18 Oct 2016 08:04:45 GMT'}, {'name': 'Connection', 'value': 'keep-alive'}]
```

# Tags

The available release tags can be seen [here](https://hub.docker.com/r/mitmproxy/mitmproxy/tags/).

---

Thanks to [Werner Beroux](https://github.com/wernight) and [David Weinstein](https://github.com/dweinstein) for their invaluable help with the mitmproxy Docker images!
