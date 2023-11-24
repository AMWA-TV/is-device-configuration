# API messaging

All message results MUST return a response which inherits from the base [NcMethodResult](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodresult) that contains a status of type [NcMethodStatus](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodstatus). If the method call encountered an error then the response result returned MUST inherit from [NcMethodResultError](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodresulterror) and include an errorMessage of type [NcString](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#primitives).

## Control session

Concurrency control is left to specific device implementations, however devices MUST always return relevant error response messages and statuses when there are conflicts, errors or other noteworthy states (see [Error response messages](#error-response-messages)).

## URL and query parameters usage

The URL provided in the [IS-04 device](IS-04%20interactions.md) is used as the base URL for all subsequent requests.

The device model of the device can be navigated by appending [NcObject roles](https://specs.amwa.tv/ms-05-02/latest/docs/NcObject.html) on top of the base URL starting from the `root block`.

This means for a given base URL the `root` block can be targeted by using `{baseUrl}/root` as the URL.

Query parameters MUST be used to target properties or methods for a particular object located by URL.

This means using the base URL the [userLabel](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncobject) of the `root` block can be targeted by using `{baseUrl}/root?level=1&index=6` as the URL.

All HTTP requests map to invoking methods on the resources located by the URL.

ALL HTTP responses MUST set the HTTP status code to the same code as the status property returned as part of [NcMethodResult](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodresult).

## GET

### Getting a property

URL format: `{baseUrl}/{rolePath}?level={propertyLevel}&index={propertyIndex}`

The URL MUST target a specific property of an object by locating the object using its role path and adding `level` and `index` query parameters. The response MUST be of type [NcMethodResultPropertyValue](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodresultpropertyvalue) with the contents of that property.

This is equivalent to invoking the generic [Get method](https://specs.amwa.tv/ms-05-02/latest/docs/NcObject.html#generic-getter-and-setter) on the specific object for the required property.

### Getting the members of a block

URL format: `{baseUrl}/{rolePath}`

The URL MUST target a specific block object in the device model and MUST NOT contain any query parameters. Devices MUST treat this as a request to retrieve the [members](https://specs.amwa.tv/ms-05-02/latest/docs/Blocks.html#device-model-discovery) property of that block. The response MUST be of type [NcMethodResultPropertyValue](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodresultpropertyvalue) with the contents of the `members` property. The response MUST be the same to the one received when appending the `level` and `index` query parameters and targeting the `members` property as described in [Getting a property](#getting-a-property).

This is equivalent to invoking the generic [Get method](https://specs.amwa.tv/ms-05-02/latest/docs/NcObject.html#generic-getter-and-setter) on the specific object for the `members` property.

`TBD`: What to return when using that URL format but the located object is not a block? Do we say devices MUST then return the class descriptor instead?

### Getting the class descriptor of an object

URL format: `{baseUrl}/{rolePath}?describe=true`

The URL MUST target a specific object in the device model and include the `describe=true` query parameter. Devices MUST treat this as a request to retrieve the [class descriptor](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncclassdescriptor) of that object.

This is equivalent to invoking the [GetControlClass method](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncclassmanager) on the [Class Manager object](https://specs.amwa.tv/ms-05-02/latest/docs/Managers.html#class-manager) and including all inherited elements.

### Getting the datatype descriptor of a property

URL format: `{baseUrl}/{rolePath}?level={propertyLevel}&index={propertyIndex}&describe=true`

The URL MUST target a specific property of an object by locating the object using its role path and adding `level`, `index` and `describe=true` query parameters. Devices MUST treat this as a request to retrieve the [datatype descriptor](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncdatatypedescriptor) of that property.

This is equivalent to invoking the [GetDatatype method](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncclassmanager) on the [Class Manager object](https://specs.amwa.tv/ms-05-02/latest/docs/Managers.html#class-manager) and including all inherited elements.

## PUT

URL format: `{baseUrl}/{rolePath}?level={propertyLevel}&index={propertyIndex}`

The PUT verb MUST only be used for setting individual object properties.

The URL MUST target a specific property of an object by locating the object using its role path and adding `level` and `index` query parameters. The response MUST be of type [NcMethodResult](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodresult).

This is equivalent to invoking the generic [Set method](https://specs.amwa.tv/ms-05-02/latest/docs/NcObject.html#generic-getter-and-setter) on the specific object for the required property.

## PATCH

URL format: `{baseUrl}/{rolePath}`

The PATCH verb MUST only be used for invoking object methods.

The URL MUST target a specific object by locating the object using its role path.
The body of the request MUST include an object which includes a [methodId](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodid) with `level` and `index` followed by any arguments required by the method.

```json
{
    "methodId": {
        "level": ...,
        "index": ...
    },
    "arguments": {
        ...
    }
}
```

`TODO`: Link to schema?

The response MUST be of type [NcMethodResult](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodresult) or a derived type.

This is equivalent to invoking the specified method.

## Error response messages

When any request encounters an error, the response MUST be [NcMethodResultError](https://specs.amwa.tv/ms-05-02/latest/docs/Framework.html#ncmethodresulterror) or a derived datatype.
