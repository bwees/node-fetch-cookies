# node-fetch-cookies

A [node-fetch](https://github.com/node-fetch/node-fetch) wrapper with support for cookies.
It supports reading/writing from/to a JSON cookie jar and keeps cookies in memory until you call `CookieJar.save()` to reduce disk I/O.

## Usage Examples

### with file...

```javascript
import {fetch, CookieJar} from "node-fetch-cookies";

(async () => {
    // creates a CookieJar instance
    const cookieJar = new CookieJar("jar.json");

    // usual fetch usage, except with one or multiple cookie jars as first parameter
    const response = await fetch(cookieJar, "https://example.com");

    // save the received cookies to disk
    await cookieJar.save();
})();
```

On the next start [`cookieJar.load()`](#async-loadfile--thisfile) can be used to load the cookies from the saved file.

### ...or without

```javascript
import {fetch, CookieJar} from "node-fetch-cookies";

(async () => {
    const cookieJar = new CookieJar();

    // log in to some api
    let response = await fetch(cookieJar, "https://example.com/api/login", {
        method: "POST",
        body: "credentials"
    });

    // do some requests you require login for
    response = await fetch(
        cookieJar,
        "https://example.com/api/admin/drop-all-databases"
    );

    // and optionally log out again
    response = await fetch(cookieJar, "https://example.com/api/logout");
})();
```

## Exports

This module exports the following classes/functions:

-   [`fetch`](#async-fetchcookiejars-url-options) _(default)_
-   [`CookieJar`](#class-cookiejar)
-   [`Cookie`](#class-cookie)
-   [`CookieParseError`](#class-cookieparseerror)
-   [`nodeFetch`](https://github.com/node-fetch/node-fetch#fetchurl-options)
-   [`Headers`](https://github.com/node-fetch/node-fetch#class-headers)
-   [`Request`](https://github.com/node-fetch/node-fetch#class-request)
-   [`Response`](https://github.com/node-fetch/node-fetch#class-response)
-   [`FetchError`](https://github.com/node-fetch/node-fetch#class-fetcherror)
-   `isRedirect`: A function that accepts a number, more precisely an http status code as input, and returns, whether the status code is a redirect status code as a boolean.  
    It is implemented in `node-fetch` and used by `node-fetch-cookies`. It is also exported here, because `node-fetch` exports it.

## Documentation

### async fetch(cookieJars, url[, options])

-   `cookieJars` A [CookieJar](#class-cookiejar) instance, an array of CookieJar instances or null, if you don't want to send or store cookies.
-   `url` and `options` as in https://github.com/node-fetch/node-fetch#fetchurl-options

Returns a Promise resolving to a [Response](https://github.com/node-fetch/node-fetch#class-response) instance on success.

### Class: CookieJar

A class that stores cookies.

#### Properties

-   `flags` The read/write flags as specified below.
-   `file` The path of the cookie jar on the disk.
-   `cookies` A [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) mapping hostnames to maps, which map cookie names to the respective [Cookie](#class-cookie) instance.
-   `cookieIgnoreCallback` The callback function passed to `new CookieJar()`, that is called whenever a cookie couldn't be parsed.

#### new CookieJar([file, flags = `rw`, cookies, cookieIgnoreCallback])

-   `file` An optional string containing a relative or absolute path to the file on the disk to use.
-   `flags` An optional string specifying whether cookies should be read and/or written from/to the jar when passing it as parameter to [fetch](#async-fetchcookiejars-url-options). Default: `rw`
    -   `r`: only read from this jar
    -   `w`: only write to this jar
    -   `rw` or `wr`: read/write from/to this jar
-   `cookies` An optional initializer for the cookie jar - either an array of [Cookie](#class-cookie) instances or a single Cookie instance.
-   `cookieIgnoreCallback(cookie, reason)` An optional callback function which will be called when a cookie is ignored instead of added to the cookie jar.
    -   `cookie` The cookie string
    -   `reason` A string containing the reason why the cookie has been ignored

#### addCookie(cookie[, fromURL])

Adds a cookie to the jar.

-   `cookie` A [Cookie](#class-cookie) instance to add to the cookie jar.
    Alternatively this can also be a string, for example a serialized cookie received from a website.
    In this case `fromURL` must be specified.
-   `fromURL` The url a cookie has been received from.

Returns `true` if the cookie has been added successfully. Returns `false` otherwise.  
If the parser throws a [CookieParseError](#class-cookieparseerror), it will be caught and `cookieIgnoreCallback` will be called with the respective cookie string and error message.

#### domains()

Returns an iterator over all domains currently stored cookies for.

#### \*cookiesDomain(domain)

Returns an iterator over all cookies currently stored for `domain`.

#### \*cookiesValid(withSession)

Returns an iterator over all valid (non-expired) cookies.

-   `withSession`: A boolean. Iterator will include session cookies if set to `true`.

#### \*cookiesAll()

Returns an iterator over all cookies currently stored.

#### \*cookiesValidForRequest(requestURL)

Returns an iterator over all cookies valid for a request to `url`.

#### deleteExpired(sessionEnded)

Removes all expired cookies from the jar.

-   `sessionEnded`: A boolean. Also removes session cookies if set to `true`.

#### async load([file = this.file])

Reads cookies from `file` on the disk and adds the contained cookies.

-   `file`: Path to the file where the cookies should be saved. Default: `this.file`, the file that has been passed to the constructor.

#### async save([file = this.file])

Saves the cookie jar to `file` on the disk. Only non-expired non-session cookies are saved.

-   `file`: Path to the file where the cookies should be saved. Default: `this.file`, the file that has been passed to the constructor.

### Class: Cookie

An abstract representation of a cookie.

#### Properties

-   `name` The identifier of the cookie.
-   `value` The value of the cookie.
-   `expiry` A [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) object of the cookies expiry date or `null`, if the cookie expires with the session.
-   `domain` The domain the cookie is valid for.
-   `path` The path the cookie is valid for.
-   `secure` A boolean value representing the cookie's secure attribute. If set the cookie will only be used for `https` requests.
-   `subdomains` A boolean value specifying whether the cookie should be used for requests to subdomains of `domain` or not.

#### new Cookie(str, requestURL)

Creates a cookie instance from the string representation of a cookie as send by a webserver.

-   `str` The string representation of a cookie.
-   `url` The url the cookie has been received from.

Will throw a `CookieParseError` if `str` couldn't be parsed.

#### static fromObject(obj)

Creates a cookie instance from an already existing object with the same properties.

#### serialize()

Serializes the cookie, transforming it to `name=value` so it can be used in requests.

#### hasExpired(sessionEnded)

Returns whether the cookie has expired or not.

-   `sessionEnded`: A boolean that specifies whether the current session has ended, meaning if set to `true`, the function will return `true` for session cookies.

#### isValidForRequest(requestURL)

Returns whether the cookie is valid for a request to `url`.

### Class: CookieParseError

The Error that is thrown when the cookie parser located in the constructor of the [Cookie](#class-cookie) class is unable to parse the input.

## License

This project is licensed under the MIT license, see [LICENSE](LICENSE).
