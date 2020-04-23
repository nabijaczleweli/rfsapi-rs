# rfsapi-rs [![Build status](https://travis-ci.org/nabijaczleweli/rfsapi-rs.svg?branch=master)](https://travis-ci.org/nabijaczleweli/rfsapi-rs) [![Licence](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](LICENSE) [![Crates.io version](https://meritbadge.herokuapp.com/rfsapi)](https://crates.io/crates/rfsapi-rs)
Raw Filesystem API for Rust — enable simpler browsing with ease

## [Documentation](https://rawcdn.githack.com/nabijaczleweli/rfsapi-rs/doc/rfsapi/index.html)

<!-- Update relevant sexion in toplevel doc, too -->
## Format brief

RFSAPI requests are made by setting a GET request's `X-Raw-Filesystem-API` header to `1`

If the server supports RFSAPI, the response will have its `X-Raw-Filesystem-API` header also set to `1`.

RFSAPI responses do *not* return file contents. Use an undecorated GET instead.

The response body is a JSON object of type [`FilesetData`](https://rawcdn.githack.com/nabijaczleweli/rfsapi-rs/doc/rfsapi/struct.FilesetData.html) in the following format:
```js
{
  "writes_supported": boolean,  // whether the server supports/allows write requests like PUT or DELETE
  "is_root": boolean,           // if this is a top-level directory
  "is_file": boolean,
  "files": Array<RawFileData>
}
```

The [`RawFileData`](https://rawcdn.githack.com/nabijaczleweli/rfsapi-rs/doc/rfsapi/struct.RawFileData.html) objects describe each individual file in a listing:
```js
{
  "mime_type": MIME type as string,
  "name": string,                           // filename, no path
  "last_modified": RFC3339 Date as string,
  "size": integer,
  "is_file": boolean
}
```

## Examples

Given the following tree at the root of the server (e.g. [`http`](https://crates.io/crates/https)), running on port 8000:
```plaintext
/
├── a.txt
├── ndis2-15.6.1.zip
└── works
    └── b.txt
```

Then the metadata of a `curl -v -H "X-Raw-Filesystem-API: 1" 127.0.0.1:8000` invocation might look something like this:
```plaintext
* Connected to 127.0.0.1 (127.0.0.1) port 8000 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:8000
> User-Agent: curl/7.64.0
> X-Raw-Filesystem-API: 1
>
< HTTP/1.1 200 OK
< Server: http/1.9.2
< X-Raw-Filesystem-API: 1
< Content-Type: application/json; charset=utf-8
< Content-Length: 843
< Date: Wed, 22 Apr 2020 17:46:51 GMT
<
* Connection #0 to host 127.0.0.1 left intact
```

Note:
  * The server returned `X-Raw-Filesystem-API: 1`
  * The server returned `Content-Type: application/json; charset=utf-8`

So we can be sure that the response body will be an RFSAPI `FilesetData`:
```json
{
  "writes_supported": false,
  "is_root": true,
  "is_file": false,
  "files": [
    {
      "mime_type": "application/zip",
      "name": "ndis2-15.6.1.zip",
      "last_modified": "2020-04-13T19:12:22.695457919Z",
      "size": 31387,
      "is_file": true
    },
    {
      "mime_type": "text/directory",
      "name": "works",
      "last_modified": "2020-04-22T13:03:33.898025702Z",
      "size": 0,
      "is_file": false
    },
    {
      "mime_type": "text/plain",
      "name": "a.txt",
      "last_modified": "2020-04-22T13:02:57.928406978Z",
      "size": 7,
      "is_file": true
    }
  ]
}
```

Say we append some data to `a.txt`, or just want to check if it was modified;
`curl -H "X-Raw-Filesystem-API: 1" 127.0.0.1:8000/a.txt` might give us:
```json
{
  "writes_supported": false,
  "is_root": false,
  "is_file": true,
  "files": [
    {
      "mime_type": "text/plain",
      "name": "a.txt",
      "last_modified": "2020-04-22T17:55:18.159945230Z",
      "size": 12,
      "is_file": true
    }
  ]
}
```

To see inside the `works` directory we can `curl -H "X-Raw-Filesystem-API: 1" 127.0.0.1:8000/works`:
```json
{
  "writes_supported": false,
  "is_root": false,
  "is_file": false,
  "files": [
    {
      "mime_type": "text/plain",
      "name": "b.txt",
      "last_modified": "2020-04-22T13:03:33.898026135Z",
      "size": 13,
      "is_file": true
    }
  ]
}
```
