# Flask-CDN

[![Version](https://img.shields.io/pypi/v/flask-cdn-auto.svg)](https://pypi.org/project/Flask-CDN-Auto)
[![Build Status](https://travis-ci.org/NullYing/flask-cdn.png)](https://travis-ci.org/NullYing/flask-cdn)
[![Coverage](https://coveralls.io/repos/NullYing/flask-cdn/badge.svg)](https://coveralls.io/github/NullYing/flask-cdn)
[![License](https://img.shields.io/pypi/l/flask-cdn-auto.svg)](https://github.com/NullYing/flask-cdn/blob/master/LICENSE.txt)

Flask-CDN allows you to easily serve all your [Flask](http://flask.pocoo.org/) application’s static assets from a CDN (like [Amazon Cloudfront](https://aws.amazon.com/cloudfront/)), without having to modify your templates.

修改了[flask-cdn](https://github.com/libwilliam/flask-cdn)的核心, 删除了CDN_ENDPOINTS, 并且修复了原有版本的缺陷, 并对新版flask做了兼容

Ps: 原作者好像消失了, pull request都没人理

## How it works
Flask-CDN replaces the URLs that Flask’s [`flask.url_for`](http://flask.pocoo.org/docs/latest/api/#flask.url_for) function would insert into your templates, with URLs that point to your CDN. This makes setting up an origin pull CDN extremely easy.

Internally, every time `url_for` is called in one of your application’s templates, `flask_cdn.url_for` is instead invoked. If the endpoint provided is deemed to refer to static assets, then the CDN URL for the asset specified in the filename argument is instead returned. Otherwise, `flask_cdn.url_for` passes the call on to [`flask.url_for`](http://flask.pocoo.org/docs/latest/api/#flask.url_for).


## Installation
If you use pip then installation is simply:
```shell
$ pip install flask-cdn
```

or, if you want the latest github version:
```shell
$ pip install git+git://github.com/NullYing/flask-cdn
```

You can also install Flask-CDN via Easy Install:
```shell
$ easy_install flask-cdn
```

## Dependencies

There are no additional dependencies besides Flask itself. Note: Flask-CDN currently only supports applications that use the [jinja2](http://jinja.pocoo.org/docs/) templating system.


## Using Flask-CDN
Flask-CDN is incredibly simple to use. In order to start serving your Flask application’s assets from Amazon CDN, the first thing to do is let Flask-CDN know about your [`flask.Flask`](http://flask.pocoo.org/docs/latest/api/#flask.Flask) application object.

```python
from flask import Flask
from flask_cdn import CDN

app = Flask(__name__)
app.config['CDN_DOMAIN'] = 'mycdnname.cloudfront.net'
CDN(app)
```

In many cases, however, one cannot expect a Flask instance to be ready at import time, and a common pattern is to return a Flask instance from within a function only after other configuration details have been taken care of. In these cases, Flask-CDN provides a simple function, `flask_cdn.CDN.init_app`, which takes your application as an argument.

```python
from flask import Flask
from flask_cdn import CDN

cdn = CDN()

def start_app():
    app = Flask(__name__)
    cdn.init_app(app)
    return app
```

In terms of getting your application to use external CDN URLs when referring to your application’s static assets, passing your Flask object to the CDN object is all that needs to be done. Once your app is running, any templates that contained relative static asset locations, will instead be pulled from your CDN.


## Static Asset URLs
URLs generated by Flask-CDN will look like the following:

`/static/foo/style.css` becomes `https://mycdnname.cloudfront.net/static/foo/style.css`, assuming that mycdnname.cloudfront.net is the domain of your CDN, and you have chosen to have assets served over HTTPS.


# Flask-CDN Options
Within your Flask application’s settings you can provide the following settings to control the behaviour of Flask-CDN. None of the settings are required.

| Option | Description | Default |
| ------ | ----------- | ------- |
| `CDN_DEBUG` | Activate Debug mode to return relative url rather than CDN enabled ones. | `app.debug` |
| `CDN_DOMAIN` | Set the base domain for your CDN here. | `None` |
| `CDN_HTTPS` | Specifies whether or not to serve your assets over HTTPS. If not specified the asset will be served by the same method the request comes in as. | `None` |
| `CDN_TIMESTAMP` | Specifies whether or not to add a timestamp to the generated urls. | `True` |
| `CDN_VERSION` | The version string to add to the generated urls. Useful when the timestamps of your files are different across servers or if you just want a more stable cache key. | `None` |


## Serve Static Assets with CloudFront
CloudFront is a very simple way to seamlessly serve your static assets with it’s CDN. When a request comes into CloudFront, if the asset is not on the CDN or has expired, then CloudFront can get the asset from an “origin server”. This type of setup is called an origin pull CDN.

To setup a new CloudFront “Distribution”:

- Signup for an [AWS Account](https://aws.amazon.com/)
- Open the [Cloudfront Management Console](https://console.aws.amazon.com/cloudfront/)
- Select Create Distribution
- Leave Download selected as the delivery method and select Continue
- In the Origin Domain Name field enter the domain name for your application
- Change **Forward Query Strings** to **Yes**
- Keep the other default values as-is and select Create Distribution

It will now take a few minutes for AWS to create the CloudFront distribution.

- Set `CDN_DOMAIN` in your Flask app to the newly created ‘Domain Name’ of your Cloudfront CDN.

Then you are done! Next time someone visits your site Cloudfront will cache and serve everything under the /static/ directory.
