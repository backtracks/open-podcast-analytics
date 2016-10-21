# Open Podcast Analytics (OPA)

The Open Podcast Analytics specification is a a unified spec for providing an open and common interface for sending podcast analytics related events. OPA is based on the [`Open Audio Analytics`](https://github.com/backtracks/open-audio-analytics) specification with extensions specifically for podcasting analytics. Apps and clients like podcast discovery and listening software send data in the format in this specification. Analytics providers with specific domain knowledge of audio and/or podcasting can provide insights that are not available with the data from a generic domain non-specific format.

## How does it work?

The simple protocol defines a common data interchange format and behaviors to allow a variety of mobile and desktop applications to emit/publish podcast analytics related events over the internet. Applications/clients may send one or more events in one call which enables queuing and batching. The data sent in the protocol is not sensitive data (it is still recommended to send data over a `HTTPS` connection), yet is a sufficient amount of data for analytics services conforming/consuming to the protocol to provide analytics services based on that data.

## Example Data
```json
[
  {
   "name":"media.download",
   "props":{
      "client":"Example App",
      "client_version":"1.0.0",
      "author":"Series Author or Series Title",
      "title":"Episode Title",
      "publisher": "Example Podcast Publisher",
      "publisher_url": "http://example.com",
      "playbackRate":1,
      "volume":1,
      "muted":false,
      "paused":false,
      "currentTime":0,
      "duration":3599.574513,
      "explicit":false,
      "user_id":"4300f781-0044-4d69-a8cc-4b0be0fcad0f",
      "src":"https://www.example.com/media/example.mp3",
      "media_ids": [
         {
            "id": "624fb990cdd94623b77f41fef0aa0e1d",
            "type": "uuid",
         },
         {
            "id": "Episode 25 of Example Podcast",
            "type": "guid",
         }
      ],
      "categories": [
         {
            "label":"Society &amp; Culture",
            "label_encoded":true,
            "categories": [
               {
                  "label":"History"
               }
            ]
         },
         {
            "label":"Music"
         },
      ],
      "tags": [
         "hot",
         "new",
         "usa"
      ],
      "series": {
            "label": "Example Podcast",
            "type": "series"
      },
      "season": {
            "label": "Season 1",
            "number": 1,
            "type": "season"
      },
      "episode": {
            "label": "Episode 25",
            "number": 25,
            "type": "episode"
      }
   },
    "time":"2016-10-16T00:23:13.411Z"
  },
  {
      "name":"media.play",
      "props":{
        "client":"Example App",
        "client_version":"1.0.0",
        "author":"Series Author or Series Title",
        "title":"Episode Title",
        "publisher": "Example Podcast Publisher",
        "publisher_url": "http://example.com",
        "playbackRate":1,
        "volume":1,
        "muted":false,
        "paused":false,
        "currentTime":0,
        "duration":3599.574513
      },
      "time":"2016-10-16T00:23:13.411Z"
   }
]
```

## Design Considerations

The protocol/specification is designed to have the ability to be efficiently utilized in both clientside and serverside scenarios and leverage pervasive and known technologies and standards. Some of the efficiency comes from aggregate network traffic at scale, limited code footprint, and the familiarity with concepts originating in specs like `HTML`, `HTML5`, etc. The protocol/specification's ability to allow queuing and batching of events allows systems to react to a variety of scenarios including intermitten internet connections.

Casing of property names is also something that was taken into account. Properties like `currentTime` are in [`lower camel case`](https://en.wikipedia.org/wiki/Camel_case) since their origin is likely from a language or variable that is already in `lower camel case` such as HTML5 Media/Audio resulting in less "variable name/value translation overhead." For custom variables [`snake case`](https://en.wikipedia.org/wiki/Snake_case) may be more appropriate. The structure of the data payload for core scenarios is also minimally nested by design. The protocol uses existing platform agnostic standards like [`ISO 8601`](https://en.wikipedia.org/wiki/ISO_8601) formatted dates vs. ['Unix/Epoch Time`](https://en.wikipedia.org/wiki/Unix_time) when appropriate and property names and values like `playbackRate` mirror `HTML5 Audio/Media` properties.

 - [`JSON`](https://en.wikipedia.org/wiki/JSON) is the data interchange format utilized by the protocol (ideally without `whitespace` in `production`) as there is common support across numerous programming languages and environments. There are many reasons `JSON` was chosen as the  data serialization format over formats like [`XML`](https://en.wikipedia.org/wiki/XML) (one of these reasons is that in `XML` attributes natively only support simple values and do not support complex values or custom types).
 - `HTTP` headers are also leveraged due to their inherent support in web requests. Headers like [`X-Forwarded-For`](https://en.wikipedia.org/wiki/X-Forwarded-For) can be utilized in serverside requests to replace and/or append the `IP address` of the original requesting client instead of the server IP address.

# Protocol Event Object Properties
## Top Level Properties
| Property Name | Type | Required | Description |
| ------------- | ------------- | ------------- | ------------- |
| name | string(255) | Yes | Name of the event and generally takes the form of `major_topic.sub_topic` |
| nonce | string(255) | No | Check value on uniqueness of an event. When specified the value shall be an arbitrary `string` and functions as as a [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce). The property is also useful in retry scenarios. |
| props | object | Yes | A `dictionary` of arbitrary `key-value pairs` of data where the values may be simple values (e.g. strings, integers, doubles/decimals/floats) or other nested `JSON` objects |
| time | date/string | Yes | An [`ISO 8601`](https://en.wikipedia.org/wiki/ISO_8601) compatible date/time stamp in UTC of the time the event occurred |
| id | string(255) | No | An `access key`, `public API key`, `project key`, `id`, etc. that may be utilized by analytics providers to know how to route the data being passed. This field may also serve as a method of authentication in particular scenarios. |

## Second Level Properties
Second level properties are children of the property `props`. Custom properties are allowed, however *all custom* properties shall be children or descendants of the `props` property similar to the known and expected properties. In the table below the `Mirrors HTML5` column indicates if the property name and value mirrors an HTML5 Media and/or Web Audio API property of the same name.

| Property Name | Type | Required | Description | Mirrors HTML5 |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| author | string(255) | No | Name of the author or artist of the media/work | No |
| client | string(255) | No | Unique identifier or name of the client or media player performing the action related to the request | No |
| currentTime | number/double | Yes | Current time of the client/user in media playback in seconds. | Yes |
| duration | number/double | Yes | Length of in media in seconds. | Yes |
| explicit | boolean | No | If true, the media contains explicit content. | Yes |
| loop | boolean | No | Indication of if the media is set to loop on the end of playback. | Yes |
| media_ids | array(`Media Id Type`) | No | Array of `Media Id Type`. See `Media Id Type` for a type definition. | No |
| muted | boolean | Yes | Indication of if the media is muted. For example the usual volume setting of the media may be at 1 (100%), however the client has the media muted. | Yes |
| networkState | integer/short | No | An integer in the set of 0-3 that indicates the current [`network state`](https://dev.w3.org/html5/spec-preview/media-elements.html#network-states). | Yes |
| paused | boolean | Yes | Indication of if the media is paused. | Yes |
| playbackRate | number/double | Yes | A number like 1 or 1.5 that indicates the relative speed of playback of the media where 1 = normal speed and values above or below 1 indicate a speed/rate change in relation to the normal value of 1. Zero (0) is not a valid value. | Yes |
| publisher | string(255) | No | Name of the publisher of the media/work related to the request (this may be different than the author) | No |
| readyState | integer/short | No | An integer in the set of 0-4 that indicates the current media [`readiness state`](https://dev.w3.org/html5/spec-preview/media-elements.html#ready-states) for playback. | Yes |
| title | string(255) | No | Title of the media/work | No |
| user_id | string(255) | No | Unique identifier for the user performing the action related to the request. This identifier is typically unique to an application or organization. | No |
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
3. Convert the array of bytes to a [`Base 64`](https://en.wikipedia.org/wiki/Base64) encoded string (with padding)
4. Transmit the HTTP request as a `GET` or `POST` request
5. Receive the HTTP response

When sending a HTTP request, standard HTTP headers such as [`User-Agent`](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields) should be included. `User-Agent` shall also be populated for requests originating from with mobile applications.

When sending a HTTP `GET` request, populate the querystring parameter `d` with the `base64` encoded data. When sending a HTTP `POST ` request, populate the `body` of the request with the `base64` encoded data and set the `Content-Type` header of the HTTP request to ``application/x-www-form-urlencoded`.

If/when receiving a HTTP response, typical [`HTTP status codes`](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) apply where a static code in the 200 range indicates success and 400 and 500 series status codes indicate a failure.

## Example Event Submission
Here is an example of what an event submission using the protocol represented in HTTP request headers.

### Actual Request
```http
GET /?d=W3sibmFtZSI6Im1lZGlhLnBsYXkiLCJwcm9wcyI6eyJhdXRob3IiOiJKb25hdGhhbiBHaWxsIiwidGl0bGUiOiJFeGFtcGxlIFRpdGxlIiwicGxheWJhY2tSYXRlIjoxLCJ2b2x1bWUiOjEsIm11dGVkIjpmYWxzZSwicGF1c2VkIjpmYWxzZSwiY3VycmVudFRpbWUiOjIuOTk3NzI5LCJkdXJhdGlvbiI6MzU5OS41NzQ1MTMsImN1c3RvbV9wcm9wZXJ0eTEiOiJBbnl0aGluZyB5b3Ugd2FudCIsImN1c3RvbV9wcm9wZXJ0eTIiOjMzLjMzfSwidGltZSI6IjIwMTYtMTAtMTZUMDA6MjM6MTMuNDExWiJ9LHsibmFtZSI6Im1lZGlhLnRpbWV1cGRhdGUiLCJwcm9wcyI6eyJhdXRob3IiOiJKb25hdGhhbiBHaWxsIiwidGl0bGUiOiJFeGFtcGxlIFRpdGxlIiwicGxheWJhY2tSYXRlIjoxLCJ2b2x1bWUiOjEsIm5ldHdvcmtTdGF0ZSI6MSwicmVhZHlTdGF0ZSI6NCwibXV0ZWQiOmZhbHNlLCJwYXVzZWQiOmZhbHNlLCJjdXJyZW50VGltZSI6My4xNTQ0MzQsImR1cmF0aW9uIjozNTk5LjU3NDUxM30sInRpbWUiOiIyMDE2LTEwLTE2VDAwOjIzOjEzLjQxMVoifSxd HTTP/1.1
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
         "muted":false,
         "paused":false,
         "currentTime":2.997729,
         "duration":3599.574513,
         "custom_property1": "Anything you want",
         "custom_property2": 33.33
      },
      "time":"2016-10-16T00:23:13.411Z"
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
         "paused":false,
         "currentTime":3.154434,
         "duration":3599.574513
      },
      "time":"2016-10-16T00:23:13.411Z"
    },
  ]
```

## Querystring Parameters
The following querystring parameters shall be supported on both `GET` and `POST` requests. The `d` querystring parameter is ignored on `POST` requests since that content will be in the `body` of the request.

| Parameter Name | Type | Required | Description |
| ------------- | ------------- | ------------- | ------------- |
| d | string | `GET` Only | `Base64` encoded (with padding) version of the data payload |
| callback | string | No | JavaScript function name to call on the return of the request's response. When utilized the `Content-Type` of the response will be `application/javascript`. |

### Examples
#### Placeholders
```
https://example.com/?d=<data>&callback=<callback>
```

#### Subsituted
```
https://example.com/?d=ICBbDQogICAgew0KICAgICAgIm5hbWUiOiJtZWRpYS5wbGF5IiwNCiAgICAgICJwcm9wcyI6ew0KICAgICAgICAgImF1dGhvciI6IkpvbmF0aGFuIEdpbGwiLA0KICAgICAgICAgInRpdGxlIjoiRXhhbXBsZSBUaXRsZSIsDQogICAgICAgICAicGxheWJhY2tSYXRlIjoxLA0KICAgICAgICAgInZvbHVtZSI6MSwNCiAgICAgICAgICJuZXR3b3JrU3RhdGUiOjEsDQogICAgICAgICAicmVhZHlTdGF0ZSI6NCwNCiAgICAgICAgICJtdXRlZCI6ZmFsc2UsDQogICAgICAgICAibG9vcCI6ZmFsc2UsDQogICAgICAgICAicGF1c2VkIjpmYWxzZSwNCiAgICAgICAgICJjdXJyZW50VGltZSI6Mi45OTc3MjksDQogICAgICAgICAiZHVyYXRpb24iOjM1OTkuNTc0NTEzLA0KICAgICAgICAgImN1c3RvbV9wcm9wZXJ0eTEiOiAiQW55dGhpbmcgeW91IHdhbnQiLA0KICAgICAgICAgImN1c3RvbV9wcm9wZXJ0eTIiOiAzMy4zMywNCiAgICAgIH0sDQogICAgICAidGltZSI6IjIwMTYtMTAtMTZUMDA6MjM6MTMuNDExWiINCiAgICB9LA0KICAgIHsNCiAgICAgICJuYW1lIjoibWVkaWEudGltZXVwZGF0ZSIsDQogICAgICAicHJvcHMiOnsNCiAgICAgICAgICJhdXRob3IiOiJKb25hdGhhbiBHaWxsIiwNCiAgICAgICAgICJ0aXRsZSI6IkV4YW1wbGUgVGl0bGUiLA0KICAgICAgICAgInBsYXliYWNrUmF0ZSI6MSwNCiAgICAgICAgICJ2b2x1bWUiOjEsDQogICAgICAgICAibmV0d29ya1N0YXRlIjoxLA0KICAgICAgICAgInJlYWR5U3RhdGUiOjQsDQogICAgICAgICAibXV0ZWQiOmZhbHNlLA0KICAgICAgICAgImxvb3AiOmZhbHNlLA0KICAgICAgICAgInBhdXNlZCI6ZmFsc2UsDQogICAgICAgICAiY3VycmVudFRpbWUiOjMuMTU0NDM0LA0KICAgICAgICAgImR1cmF0aW9uIjozNTk5LjU3NDUxMw0KICAgICAgfSwNCiAgICAgICJ0aW1lIjoiMjAxNi0xMC0xNlQwMDoyMzoxMy40MTFaIg0KICAgIH0sDQogIF0=&callback=customJavaScriptFunction
```

### Event Examples
Samples events in their unencoded format are below:

| Event Name |
| ------------- |
| [media.play](/examples/events/media.play.json) (Basic/Sample) |
| [media.play](/examples/events/media.play.extended.json) (Actual/Detailed) |
| [media.pause](/examples/events/media.pause.json) |
| [media.download](/examples/events/media.download.json) |
| [media.subscribe](/examples/events/media.subscribe.json) |
| [media.unsubscribe](/examples/events/media.unsubscribe.json) |
| [media.ended](/examples/events/media.ended.json) |
| [media.ratechange](/examples/events/media.ratechange.json) |
| [media.seeked](/examples/events/media.seeked.json) |
| [media.timeupdate](/examples/events/media.timeupdate.json) |
| [media.volumechange](/examples/events/media.volumechange.json) |
| [ui.action](/examples/events/ui.action.json) |


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
- *How do I know `base64` encode data?*
  - Most modern programming languages have built-in support for `base64` encoding and for languages and environments like `JavaScript/Node`, there are open source modules and libraries that can perform the `base64` encoding and decoding.
- *How do I know if my `base64` encoded data is padded correctly?*
  - The [`mod`](https://en.wikipedia.org/wiki/Modulo_operation) 4 of a base64 encoded string should be 0 and if the result is not 0 then there's some missing padding in the encoding algorithm. This may seem crazy, but often this just means your encoding algorithm just needs to add equal signs `=` at the end of the string until the `mod 4` of the encoded string equals 0.
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
         "muted":false,
         "paused":false,
         "currentTime":2.997729,
         "duration":3599.574513
      },
      "time":"2016-10-16T00:23:13.411Z"
    },
    {
      "name":"media.timeupdate",
      "props":{
         "author":"Jonathan Gill",
         "title":"Example Title",
         "playbackRate":1,
         "volume":1,
         "muted":false,
         "paused":false,
         "currentTime":3.154434,
         "duration":3599.574513
      },
      "time":"2016-10-16T00:23:13.411Z"
    },
  ]
  ```
- *What should the maximum length be of a custom `string` property?*
 -  The maximum length for any string is 255 characters. While your requirements may be that the string is shorter (which it can be), the maximum length for data storage of that string in any implementing analytics system will be 255 characters.
- *Are there any reserved characters in property names?*
 -  Yes, the characters that may not be used in property names are: `/`, `.`, and any character or complete phrase (e.g. `true` is not a valid property name) that is disallowed in a JavaScript property or variable name.
- *Are there any reserved property names?*
 -  Yes.
  - `time` shall always represent an `ISO 8601` compatible date/time format when used regardless of its location in a hierarchy.
- *Should `GET` and `POST` requests both support receiving multiple events in one request?*
 -  Yes
- *What should the `Content-Type` HTTP header be of `POST` requests in the protocol?*
  - The `Content-Type` of a `POST` request shall be `application/x-www-form-urlencoded`.
- *Is whitespace important?*
  - We recommend that `JSON` not have whitespace outside of the whitespace within a string value in production usage. Whitespace outside of this use is not significant in this protocol and simply increases the data payload size.
- *Why do you use the term `uuid` instead of `guid`?*
  - Because `guid` aka Globally Unique Identifier does not quite fit for an interplanetary species.


The specification described on this page or document is available under the Apache 2.0 License. This is a dervivative work of [`Open Audio Analytics`](https://github.com/backtracks/open-audio-analytics). This has been another Backtracks, Inc. & Friends joint.
