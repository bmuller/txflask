# txflask

Working with [Python Twisted](http://twistedmatrix.com)'s [Web](http://twistedmatrix.com/trac/wiki/TwistedWeb) library is a bit of a pain.  This library makes it look like a [Flask](http://flask.pocoo.org) application.

Yes, you can run a [Flask app under Twisted](http://flask.pocoo.org/docs/deploying/wsgi-standalone/#twisted-web) using it in a WSGI container, but if you want to be able to handle deferreds in your web app then something else is needed.

## Installation

```shell
pip install txflask
```

## Usage
Routes are handled a bit differently than Flask, in that they use the [routes](https://github.com/bbangert/routes) library, but the functionality is roughly similar.  A ```TxFlask``` instance is just a [twisted.web.resource.Resource](http://twistedmatrix.com/documents/current/api/twisted.web.resource.Resource.html), so it can be put in a ```web.Site``` or run directly by calling the ```run``` method.

```python
from txflask.app import TxFlask

app = TxFlask()

@app.route('/{name}')
def index(request, name):
    return "Hi %s" % name

app.run()
```

The request that's passed in is just a [twistd.web.http.Request](http://twistedmatrix.com/documents/current/api/twisted.web.http.Request.html) object (with a few helper methods to make it look like a Flask request).  You can return either a string from a route, or a deferred that eventually returns a string.  For instance, this is valid:

```python
from twisted.internet import reactor, defer

@app.route('/{name}/{id}', methods=['GET', 'POST'])
def params(request, name, id):
    request.setHeader('content-type', 'application/json')
    d = defer.Deferred()
    reactor.callLater(1, d.callback, "{\"%s\": %s}" % (name, id))
    return d
```
