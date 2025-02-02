
# APIs

## API Specifications

The Configuration API is specified using:

* The following sub-sections describing common NMOS API properties.
* [RAML](http://raml.org/) documents and [JSON schemas](http://tools.ietf.org/html/draft-zyp-json-schema-04) in the [APIs](../APIs/) folder.
* Normative text in files in the [docs](../docs) directory.

Examples of JSON format output are provided in the [examples](../examples/) folder.

## API Validation

JSON schemas are included with the RAML API definitions.
These schemas validate the communication with the API.
It is RECOMMENDED that implementers of a Configuration API use these JSON schemas as part of a validation stage when receiving interactions from clients.
Note that where schemas reference MS-05-02 datatypes defined in the [NMOS Control Framework specification](https://specs.amwa.tv/ms-05-02/branches/v1.0.x/models/datatypes/) and [NMOS Feature Sets Register](https://specs.amwa.tv/nmos-control-feature-sets/branches/publish-device-configuration/device-configuration/), these datatypes are not validated by the JSON schemas.

### Content Types

All APIs MUST provide a JSON representation signalled via 'Content-Type: application/json' headers.
This SHOULD be the default content type in the absence of any requested alternative by clients.
Other content types (such as HTML) are permitted if they are explicitly requested via Accept headers.

### API Paths

All NMOS APIs MUST use a path in the following format.
Other HTTP resources MAY be presented on the same port by the Node, provided all NMOS resources are available from the /x-nmos/ path as follows:

```
http(s)://<ip address or hostname>:<port>/x-nmos/<api type>/<api version>/
```

At each level of the API from the base resource onwards, the response SHOULD include a JSON format array of the child resources available from that point.

Nodes MAY extend the path for the Configuration API with a suffix.
For example, a Node implementation might choose to extend the path so that Devices on the Node have separate Configuration API instances without having to use a different port for each Device's API.
Such a path MUST be formatted as follows:

```
http(s)://<ip address or hostname>:<port>/x-nmos/<api type>/<api version>/<api selector>/
```

The `<api selector>` identifier MUST conform to the following regex pattern:

```
^[a-zA-Z0-9\-_]+$
```

This permits the use of a UUID, such as an associated IS-04 Device `id`, in the usual format or succinct local identifiers such as `slot2B`.
Human-readable information can be provided, for example, via the IS-04 Device `label` and `description`.

### Versioning

All public APIs are versioned as follows:

* Requesting the API base resource (such as http(s)://&lt;ip address or hostname&gt;:&lt;port&gt;/x-nmos/configuration/) will provide a list containing the versions of the API present on the Node.
* A versioned API response must include only resources which match the schema for that API version.
* Data which is held for mismatched minor API versions may be returned if it can be conformed to the correct schema (see example below). Data must never be conformed between major API versions.

#### Versioning Example

A v1.1 API response may include:

* v1.1 data without modification.
* Data conforming to schemas less than v2.0 but greater than v1.1, with any non-v1.1 keys removed.

### Common API Base Resource

```json
[
  "v1.0/",
  "v2.0/",
  "v3.0/"
]
```

* Appending /v1.0/ to the API base resource will request version 1.0 of the API if available.
* The versioning format is v&lt;#MAJOR&gt;.&lt;#MINOR&gt;
* MINOR increments SHOULD be performed for non-breaking changes (such as the addition of attributes in a response)
* MAJOR increments MUST be performed for breaking changes (such as the renaming of a resource or attribute)
* Versions MUST be represented as complete strings. Parsing MUST proceed as follows: separate into two strings, using the point (.) as a delimiter. Compare integer representations of MAJOR, MINOR version (such that v1.12 is greater than v1.5).
* Clients are responsible for identifying the correct API version they require.

### URLs: Approach to Trailing Slashes

For consistency and in order to adhere to how these APIs are specified in RAML, the 'primary' path for every resource has the trailing slash omitted.

In order to overcome shortcomings in some common libraries, the following requirements are imposed for the support of URL paths with or without trailing slashes.

#### GET and HEAD Requests

* Clients performing requests using these methods SHOULD correctly handle a 301 response (moved permanently).
* When a 301 is supported, the client MUST follow the redirect in order to retrieve the required response payload.
* Servers implementing the APIs MUST support requests using these methods to both the trailing slash and non-trailing slash path to each resource. One of these MAY produce a 301 response causing a redirect to the other if preferred.

#### All Other Requests (PUT, POST, DELETE, OPTIONS etc)

* Clients performing requests using these methods MUST use URLs with no trailing slash present.
* Servers implementing the APIs MUST correctly interpret requests using these methods to paths without trailing slashes present.
* Servers implementing the APIs MAY correctly interpret requests using these methods to paths with trailing slashes present for backward compatibility.
* Servers SHOULD NOT respond with 3xx codes for these request types.

### Error Codes & Responses

The NMOS APIs use HTTP status codes to indicate success, failure and other cases to clients as per [RFC 7231](https://tools.ietf.org/html/rfc7231) and related standards.
Where the RAML specification of an API specifies explicit response codes it is expected that a client will handle these cases in a particular way.
As explicit handling of every possible HTTP response code is not expected, clients must instead implement more generic handling for ranges of response codes (1xx, 2xx, 3xx, 4xx and 5xx).
