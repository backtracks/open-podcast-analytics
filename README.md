# Open Podcast Analytics Protocol 

Open audio and podcast analytics specification. A unified spec for providing a common interface for podcast publishing/pushing analytics.

## Top Level Properties
| Property Name | Type | Required | Description | 
| ------------- | ------------- | ------------- | ------------- |
| name | string | Yes | Name of the event and generally takes the form of `major_topic.sub_topic` |
| nonce | string | No | Optional top-level event property (a peer of `name`) that is a check value on uniqueness of an event. The value shall be an arbitrary `string` and functions as as a [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce). |
| properties | object | Yes | A `dictionary` of arbitrary `key-value pairs` of data where the values may be simple values (e.g. strings, integers, floats/decimals) or other nested `JSON` objects |
| timestamp | date/string | Yes | An [`ISO 8601`](https://en.wikipedia.org/wiki/ISO_8601) compatible date/time stamp in UTC of the time the event occurred `dictionary` of arbitrary `key-value pairs` of data where the values may be simple values (e.g. strings, integers, floats/decimals) or other nested `JSON` objects |

## Example
```json
[
   {
      "name":"media.timeupdate",
      "properties":{
         "author":"Jonathan Gill",
         "title":"Example Title",
         "name":"media.play",
         "playbackRate":1,
         "volume":1,
         "networkState":1,
         "readyState":4,
         "muted":false,
         "loop":false,
         "paused":false,
         "currentTime":2.997729
      },
      "timestamp":"2016-10-16T00:23:13.411Z"
   },
]
```

**URL:** `https://example.com`
- Accepts both `GET `and `POST` requests & accepts `single events` and `batched events`.
  - `GET` requires a `base64` encoded querystring parameter with the name `d` where the unencoded value of the data parameter is `JSON`
  - `POST` requires that the `body` of the request is `JSON`. `POST` is generally used for `BATCH` events to overcome limitations on maximum URL length.
  - When sending `BATCH` events send them as an array of JSON objects that is then `base64` encoded.

The specification described on this page or document is available under the [Open Web Foundation Agreement, Version 0.9](http://www.openwebfoundation.org/legal/the-0-9-agreements---necessary-claims).
