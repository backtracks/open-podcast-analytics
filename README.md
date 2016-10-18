# Open Audio Analytics (OAA) Protocol

Open audio analytics protocol specification. A unified spec for providing a common interface for sending audio analytics related events.

## How does it work?

The simple protocol defines a common data interchange format and behaviors to allow a variety of mobile and desktop applications to emit/publish audio analytics related events over the internet. The data sent in the protocol is not sensitive data (it is still recommended to send data over a `HTTPS` connection), yet is a sufficient amount of data for analytics services conforming/consuming to the protocol to provide analytics services based on that data.

## Example Data
```json
[
   {
      "name":"media.play",
      "props":{
         "author":"Jonathan Gill",
         "title":"Example Title",
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

## Design Considerations

The protocol/specification is designed to have the ability to be efficiently utilized in both clientside and serverside scenarios and leverage pervasive and known technologies. Some of the efficiency comes from aggregate network traffic at scale and limited code footprint.

At a large scale shortening terms like `properties` to `props` in the protocol results in less network data transfer as well as fewer characters in source code, logs, etc.

Casing of property names is also something that was taken into account. Properties like `currentTime` are in [`lower camel case`](https://en.wikipedia.org/wiki/Camel_case) since their origin is likely from a language or variable that is already in `lower camel case` such as HTML5 Media/Audio resulting in less "variable name/value translation overhead." For custom variables [`snake case`](https://en.wikipedia.org/wiki/Snake_case) may be more appropriate. The structure of the data payload for core scenarios is also minimally nested by design. The protocol uses existing platform agnostic standards like [`ISO 8601`](https://en.wikipedia.org/wiki/ISO_8601) formatted dates vs. ['Unix/Epoch Time`](https://en.wikipedia.org/wiki/Unix_time) when appropriate and property names and values like `playbackRate` mirror `HTML5 Audio/Media` properties.

 - [`JSON`](https://en.wikipedia.org/wiki/JSON) is the data interchange format utilized by the protocol (ideally without `whitespace` in `production`) as there is common support across numerous programming languages and environments
 - `HTTP` headers are also leveraged due to their inherent support in web requests. Headers like [`X-Forwarded-For`](https://en.wikipedia.org/wiki/X-Forwarded-For) can be utilized in serverside requests to replace and/or append the `IP address` of the original requesting client instead of the server IP address.

# Protocol Event Object Properties
## Top Level Properties
| Property Name | Type | Required | Description |
| ------------- | ------------- | ------------- | ------------- |
| name | string(255) | Yes | Name of the event and generally takes the form of `major_topic.sub_topic` |
| nonce | string(255) | No | Check value on uniqueness of an event. When specified the value shall be an arbitrary `string` and functions as as a [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce). The property is also useful in retry scenarios. |
| properties | object | Yes | A `dictionary` of arbitrary `key-value pairs` of data where the values may be simple values (e.g. strings, integers, doubles/decimals/floats) or other nested `JSON` objects |
| timestamp | date/string | Yes | An [`ISO 8601`](https://en.wikipedia.org/wiki/ISO_8601) compatible date/time stamp in UTC of the time the event occurred |
| token | string(255) | No | An `access key`, `public API key`, `project key`, etc. that may be utilized by analytics providers to know how to route the data being passed. This field may also serve as a method of authentication in particular scenarios. |

## Second Level Properties
Second level properties are children of the property `props`. Custom properties are allowed, however *all custom* properties shall be children or descendants of the `props` property similar to the known and expected properties. In the table below the `Mirrors HTML5` column indicates if the property name and value mirrors an HTML5 Media and/or Web Audio API property of the same name.

| Property Name | Type | Required | Description | Mirrors HTML5 |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| author | string(255) | No | Name of the author or artist of the media/work | No |
| client | string(255) | No | Unique identifier or name of the client or media player performing the action related to the request | No |
| currentTime | number/double | Yes | Current time of the client/user in media playbace in seconds. | Yes |
| loop | boolean | Yes | Indication of if the media is set to loop on the end of playback. | Yes |
| media_ids | array(`Media Id Type`) | No | Array of `Media Id Type`. See `Media Id Type` for a type definition | No |
| muted | boolean | Yes | Indication of if the media is muted. For example the usual volume setting of the media may be at 1 (100%), however the client has the media muted. | Yes |
| networkState | integer/short | Yes | An integer in the set of 0-3 that indicates the current network state. | Yes |
| paused | boolean | Yes | Indication of if the media is paused. | Yes |
| playbackRate | number/double | Yes | A number like 1 or 1.5 that indicates the relative speed of playback of the media where 1 = normal speed and values above or below 1 indicate a speed/rate change. Zero (0) is not a valid value. | Yes |
| publisher | string(255) | No | Name of the publisher of the media/work related to the request (this may be different than the author) | No |
| readyState | integer/short | Yes | An integer in the set of 0-4 that indicates the current media readiness state for playback. | Yes |
| title | string(255) | No | Title of the media/work | No |
| user_id | string(255) | No | Unique identifier for the user performing the action related to the request. This identifier is typically unique to an application or organization and is not an IP address. | No |
| volume | number/double | Yes | A number between 0 and 1 (where 0 = 0% and 1 = 100%) that indicates the volume setting of the media. Example: .75 = 75% volume | Yes |


### Media Id Type
| Property Name | Type | Required | Description |
| ------------- | ------------- | ------------- | ------------- |
| id | string(255) | Yes | Unique id of the media |
| type | string(255) | Yes | Type of media id, see `Known Media Id Types` below for known values. This is an open-ended property and not an restricted [`enum`](https://en.wikipedia.org/wiki/Enumerated_type) so the type value may be any valid string.  |


### Known Media Id Types
| Type Name | Description |
| ------------- | ------------- |
| barcode | Unique, generally commercially registered, idenitider where the value can be a `upc`, `ean`, etc. (UPC and EAN are really just both versions of the same underlying barcode structure) |
| uuid | "Arbitrary" universally unique identifier. This is sometimes called a [`guid`]() or "Globally Unique Identifier". |
| isbn | [International Standard Book Number](https://en.wikipedia.org/wiki/International_Standard_Book_Number) |
| issn | [International Standard Serial Number](https://en.wikipedia.org/wiki/International_Standard_Serial_Number) |
| isrc | [International Standard Recording Code](https://en.wikipedia.org/wiki/International_Standard_Recording_Code) |
| iswc | [International Standard Musical Work Code](https://en.wikipedia.org/wiki/International_Standard_Musical_Work_Code) |
| org_uuid | Organization, application, publisher identifier such as a release number or an internal id that is unique to an organization |


### Event Submission Workflow Example
1. Construct the `JSON` formatted data payload representing one or more events
2. Convert the data payload to an array of bytes
3. Convert the array of bytes to a [`Base 64`](https://en.wikipedia.org/wiki/Base64) encoded string
4. Transmit the HTTP request as a `GET` or `POST` request
5. Receive the HTTP response

When sending a HTTP request, standard HTTP headers such as [`User-Agent`](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields) should be included. `User-Agent` shall also be populated for requests originating from with mobile applications.

When sending a HTTP `GET` request, populate the querystring parameter `d` with the `base64` encoded data. When sending a HTTP `POST ` request, populate the `body` of the request with the `base64` encoded data and set the `Content-Type` header of the HTTP request to ``application/x-www-form-urlencoded`.

If/when receiving a HTTP response, typical [`HTTP status codes`](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) apply where a static code in the 200 range indicates success and 400 and 500 series status codes indicate a failure.

## Example Event Submission
Here is an example of what an event submission using the protocol represented in HTTP request headers.

```http
GET /?d=ICBbDQogICAgew0KICAgICAgIm5hbWUiOiJtZWRpYS5wbGF5IiwNCiAgICAgICJwcm9wcyI6ew0KICAgICAgICAgImF1dGhvciI6IkpvbmF0aGFuIEdpbGwiLA0KICAgICAgICAgInRpdGxlIjoiRXhhbXBsZSBUaXRsZSIsDQogICAgICAgICAicGxheWJhY2tSYXRlIjoxLA0KICAgICAgICAgInZvbHVtZSI6MSwNCiAgICAgICAgICJuZXR3b3JrU3RhdGUiOjEsDQogICAgICAgICAicmVhZHlTdGF0ZSI6NCwNCiAgICAgICAgICJtdXRlZCI6ZmFsc2UsDQogICAgICAgICAibG9vcCI6ZmFsc2UsDQogICAgICAgICAicGF1c2VkIjpmYWxzZSwNCiAgICAgICAgICJjdXJyZW50VGltZSI6Mi45OTc3MjkNCiAgICAgIH0sDQogICAgICAidGltZXN0YW1wIjoiMjAxNi0xMC0xNlQwMDoyMzoxMy40MTFaIg0KICAgIH0sDQogICAgew0KICAgICAgIm5hbWUiOiJtZWRpYS50aW1ldXBkYXRlIiwNCiAgICAgICJwcm9wcyI6ew0KICAgICAgICAgImF1dGhvciI6IkpvbmF0aGFuIEdpbGwiLA0KICAgICAgICAgInRpdGxlIjoiRXhhbXBsZSBUaXRsZSIsDQogICAgICAgICAicGxheWJhY2tSYXRlIjoxLA0KICAgICAgICAgInZvbHVtZSI6MSwNCiAgICAgICAgICJuZXR3b3JrU3RhdGUiOjEsDQogICAgICAgICAicmVhZHlTdGF0ZSI6NCwNCiAgICAgICAgICJtdXRlZCI6ZmFsc2UsDQogICAgICAgICAibG9vcCI6ZmFsc2UsDQogICAgICAgICAicGF1c2VkIjpmYWxzZSwNCiAgICAgICAgICJjdXJyZW50VGltZSI6My4xNTQ0MzQNCiAgICAgIH0sDQogICAgICAidGltZXN0YW1wIjoiMjAxNi0xMC0xNlQwMDoyMzoxMy40MTFaIg0KICAgIH0sDQogIF0= HTTP/1.1
Host: example.com
Connection: keep-alive
User-Agent: Mozilla/5.0 (NeXTStep 3.3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36
Accept-Language: en-US,en;q=0.8
```

### Unencoded Raw Data
```json
  [
    {
      "name":"media.play",
      "props":{
         "author":"Jonathan Gill",
         "title":"Example Title",
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
    {
      "name":"media.timeupdate",
      "props":{
         "author":"Jonathan Gill",
         "title":"Example Title",
         "playbackRate":1,
         "volume":1,
         "networkState":1,
         "readyState":4,
         "muted":false,
         "loop":false,
         "paused":false,
         "currentTime":3.154434
      },
      "timestamp":"2016-10-16T00:23:13.411Z"
    },
  ]
```

## Querystring Parameters
The following querystring parameters shall be supported on both `GET` and `POST` requests. The `d` querystring parameter is ignored on `POST` requests since that content will be in the body of the request.

| Parameter Name | Type | Required | Description |
| ------------- | ------------- | ------------- | ------------- |
| d | string | On `GET` | `Base64` encoded version of the data payload |
| callback | string | No | JavaScript function name to call on the return of the request's response. When utilized the `Content-Type` of the response will be `application/javascript`. |

### Examples
```
https://example.com/?d=<data>&callback=<callback>
```
```
https://example.com/?d=ICBbDQogICAgew0KICAgICAgIm5hbWUiOiJtZWRpYS5wbGF5IiwNCiAgICAgICJwcm9wcyI6ew0KICAgICAgICAgImF1dGhvciI6IkpvbmF0aGFuIEdpbGwiLA0KICAgICAgICAgInRpdGxlIjoiRXhhbXBsZSBUaXRsZSIsDQogICAgICAgICAicGxheWJhY2tSYXRlIjoxLA0KICAgICAgICAgInZvbHVtZSI6MSwNCiAgICAgICAgICJuZXR3b3JrU3RhdGUiOjEsDQogICAgICAgICAicmVhZHlTdGF0ZSI6NCwNCiAgICAgICAgICJtdXRlZCI6ZmFsc2UsDQogICAgICAgICAibG9vcCI6ZmFsc2UsDQogICAgICAgICAicGF1c2VkIjpmYWxzZSwNCiAgICAgICAgICJjdXJyZW50VGltZSI6Mi45OTc3MjkNCiAgICAgIH0sDQogICAgICAidGltZXN0YW1wIjoiMjAxNi0xMC0xNlQwMDoyMzoxMy40MTFaIg0KICAgIH0sDQogICAgew0KICAgICAgIm5hbWUiOiJtZWRpYS50aW1ldXBkYXRlIiwNCiAgICAgICJwcm9wcyI6ew0KICAgICAgICAgImF1dGhvciI6IkpvbmF0aGFuIEdpbGwiLA0KICAgICAgICAgInRpdGxlIjoiRXhhbXBsZSBUaXRsZSIsDQogICAgICAgICAicGxheWJhY2tSYXRlIjoxLA0KICAgICAgICAgInZvbHVtZSI6MSwNCiAgICAgICAgICJuZXR3b3JrU3RhdGUiOjEsDQogICAgICAgICAicmVhZHlTdGF0ZSI6NCwNCiAgICAgICAgICJtdXRlZCI6ZmFsc2UsDQogICAgICAgICAibG9vcCI6ZmFsc2UsDQogICAgICAgICAicGF1c2VkIjpmYWxzZSwNCiAgICAgICAgICJjdXJyZW50VGltZSI6My4xNTQ0MzQNCiAgICAgIH0sDQogICAgICAidGltZXN0YW1wIjoiMjAxNi0xMC0xNlQwMDoyMzoxMy40MTFaIg0KICAgIH0sDQogIF0=&callback=customJavaScriptFunction
```

### Event Examples
Samples events in their unencoded format will be linked here.


### Tips and FAQs
- *In a serverside request, how do I specify the IP address of the user and not the IP address of the server?*
  - Create or append to the [`X-Forwarded-For`](https://en.wikipedia.org/wiki/X-Forwarded-For) header (append if it already exists) in the request that is sent the analytics service.
- *How can I add custom properties to events?*
  - Add any custom properties as descendants of the `props` property. The use of descendants and not children is to indicate that nested structures can be supported.
- *Why is the data sent via the protocol [`Base 64`](https://en.wikipedia.org/wiki/Base64) encoded?*
  - While it may seem that `JSON` is primarily a string based data interchange format, the payload of data (which is essentially a structured string) is converted to bytes (e.g. an array of bytes) then `base64` encoded with multiple purposes in mind.
    - All strings regardless of text encoding can be converted to byte arrays and all bytes can be encoded as `base64`
    - [`UTF-8`](https://en.wikipedia.org/wiki/UTF-8) strings, such as 音频, can be encoded in `base64` if converted to bytes first, which opens up the protocol for use with values from different languages.
    - Encoding potential querystring parameter values as `base64` eliminates the need to also [`url encode`](https://en.wikipedia.org/wiki/Percent-encoding) or escape the value(s)
    - A light amount of obfuscation of the data
- *Is there a recommended querystring parameter to use for data in a `GET` request?*
  - Yes, it is recommended to use `d` as the querystring parameter due to brevity (`d` = `data`).
- *How do I send multiple events in one request?*
  - Multiple events should be represented as a `JSON` array where each event is a `JSON` object. Here is an unencoded representation of the data before it is `base64` encoded:
  ```json
  [
    {
      "name":"media.play",
      "props":{
         "author":"Jonathan Gill",
         "title":"Example Title",
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
    {
      "name":"media.timeupdate",
      "props":{
         "author":"Jonathan Gill",
         "title":"Example Title",
         "playbackRate":1,
         "volume":1,
         "networkState":1,
         "readyState":4,
         "muted":false,
         "loop":false,
         "paused":false,
         "currentTime":3.154434
      },
      "timestamp":"2016-10-16T00:23:13.411Z"
    },
  ]
  ```
- *Should `GET` and `POST` requests both support receiving multiple events in one request?*
 -  Yes
- *What should the `Content-Type` HTTP header be of `POST` requests in the protocol?*
  - The `Content-Type` of a `POST` request shall be `application/x-www-form-urlencoded`.
- *Is whitespace important?*
  - We recommend that `JSON` not have whitespace outside of the whitespace within a string value in production usage. Whitespace outside of this use is not significant in this protocol and simply increases the data payload size.
- *Why do you use the term `uuid` instead of `guid`?*
  - Because `guid` aka Globally Unique Identifier does not quite fit for an interplanetary species.


The specification described on this page or document is available under the Apache 2.0 License. This has been another Backtracks, Inc. & Friends joint.
