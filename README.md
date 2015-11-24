# Proxy Switch

Test [mitmproxy](https://mitmproxy.org) via rewriting their [mitmwrapper.py example](https://github.com/mitmproxy/mitmproxy/blob/master/examples/mitmproxywrapper.py)

## Install

```sh
pip install "git+https://github.com/ustwo/proxyswitch.git@v0.4.4#egg=proxyswitch"
```

## Proxyswitch

Check the help:

```sh
proxyswitch --help
```


## Mastermind

Mastermind combines `mitmdump` and `proxyswitch` making sure the proxy settings
are enabled when the mitmproxy starts and disabling them when the proxy stops.

There are three forms you can use, "Driver", "Simple" and "Script".  They can't
be mixed.

### Driver

The driver mode will mount a thin HTTP server listening for actions and a set
of rules to apply.

```sh
sudo mastermind --with-driver \
                --source-dir $(pwd)/test/records
```

In the example above, `mastermind` will expect to find one or more YAML ruleset
files.  [Check the example](test/records).

A ruleset file is an array of rules.  Each rule is composed by at least a `url`
and a `response.body`.  The body must be a _relative_ path to an existing file.

```yaml
---
- url: https://api.github.com/users/octocat/orgs
  response:
    body: fake.json
```

A more elaborated case will have headers to add or remove:

```yaml
---
- name: bar
  url: https://api.github.com/users/arnau/orgs
  request:
    headers:
      remove:
        - 'If-None-Match'
  response:
    body: arnau-orgs.json
    headers:
      remove:
        - 'ETag'
      add:
        Cache-Control: no-cache
        X-ustwo-intercepted: 'Yes'
```

Assuming the two examples above were named `a.yaml` and `b.yaml` a running
`mastermind` in driver mode would load the first with:

```sh
curl --proxy http://localhost:8080 \
     -XGET http://proxapp:5000/a/start
```

The second with:

```sh
curl --proxy http://localhost:8080 \
     -XGET http://proxapp:5000/b/start
```

And cleaning any ruleset with:

```sh
curl --proxy http://localhost:8080 \
     -XGET http://proxapp:5000/stop
```


### Simple

The simple mode expects a response body filepath and a URL to intercept:

```sh
sudo mastermind --response-body $(pwd)/test/records/fake.json" \
                https://api.github.com/users/octocat/orgs
```

### Script

The script mode expects a mitmproxy Python script:

```sh
sudo mastermind --script $(pwd)/myscript.py
```

Or pass parameters to your script the same way mitmproxy does:

```sh
sudo mastermind --script "$(pwd)/myscript.py param1 param2"
```

If you go for the `--script` approach, you have to explicitly manage proxyswitch
yourself. Adding the following will do the trick:

```python
from proxyswitch import enable, disable

def start(context, argv):
    enable('127.0.0.1', '8080')

def done(context):
    disable()
```


Check the help for more.

```sh
mastermind --help
```


## Maintainers

* [Arnau Siches](mailto:arnau@ustwo.com)
