# catbox-s3

Amazon S3 adapter for [catbox](https://github.com/hapijs/catbox).

[![Build Status](https://travis-ci.org/fhemberger/catbox-s3.svg?branch=master)](http://travis-ci.org/fhemberger/catbox-s3) ![Current Version](https://img.shields.io/npm/v/catbox-s3.svg)

### Options

- `bucket` - the S3 bucket. You need to have write access for it.
- `accessKeyId` - the Amazon access key.
- `secretAccessKey` - the Amazon secret access key.
- `region` - the Amazon S3 region. (If you don't specify a region, the bucket will be created in US Standard.)
- `endpoint` - the S3 endpoint URL. (If you don't specify an endpoint, the bucket will be created at Amazon S3 using the provided region if any)
- `setACL` - defaults to true, if set to false, not acl is set for the objects
- `ACL` - the ACL to set if setACL is not false, defaults to `public-read`


### Caching binary data

At the moment, Hapi doesn't support caching of non-JSONifiable responses (like Streams or Buffers, see [#1948](https://github.com/hapijs/hapi/issues/1948)).
If you want to use catbox-s3 for binary data, you have to handle it manually in your request handler:

```javascript
var Catbox = require('catbox');

// On hapi server initialization:
// 1) Create a new catbox client instance
var cache  = new Catbox.Client(require('catbox-s3'), {
    accessKeyId     : /* ... */,
    secretAccessKey : /* ... */,
    region          : /* ... (optional) */,
    bucket          : /* ... */
});

// 2) Inititalize the caching
cache.start(function (err) {

    if (err) { console.error(err); }
    /* ... */
});

// Your route's request handler
var handler = function (request, reply) {

    var cacheKey = {
        id      : /* cache item id */,
        segment : /* cache segment name */
    };

    cache.get(cacheKey, function (err, result) {

        if (result) {
            return reply(result.item).type(/* response content type */);
        }

        yourBusinessLogic(function (err, data) {

            cache.set(cacheKey, data, /* expiration in ms */, function (err) {

                /* ... */
            });
            reply(result.item).type(/* response content type */);
        });
    });
};

```

### Running tests

In order to run the tests, set the aforementioned options as environment variables:

```shell
S3_ACCESS_KEY=<YOURKEY> S3_SECRET_KEY=<YOURSECRET> S3_REGION=<REGION> S3_BUCKET=<YOURBUCKET> npm test
```


### License

[MIT](LICENSE.txt)
