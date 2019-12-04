---
title: "Fix API Doctor errors"
description: "Fix validation errors that API Doctor discovers."
localization_priority: Normal
author: "davidmu1"
ms.prod: "microsoft-identity-platform"
doc_type: resourcePageType
---

# Fix API Doctor errors

API Doctor validates an HTTP example against the related resources and checks for the following:

- Any properties in the example response body must be part of the @odata.type you specified for that response. All the properties in the response must be defined as part of the JSON in the **JSON representation** section of the resource type topic.
- The property values in the example response must match how you defined the corresponding property types in the **JSON representation** section of the resource type topic.

## Find the error message

1. API Doctor runs when you push a change to a pull request. If API Doctor discovers an issue, you see the following message in the GitHub web page. Click **Details** to proceed:

![API Doctor error](graph-api-doctor-error.png)

2. On the API Doctor Logs page, click the text that says **PowerShell exited with code \'1\'**:

![API Doctor logs](graph-api-doctor-logs.png)

3. Towards the bottom of the page, you will find the errors that occured:

![API Doctor list](graph-api-doctor-list.png)

In this case, the error was caused by having no blank line between the headers and the body of the request in the example that is provided in the topic.

The error message contains the `name` that is defined in the HTML comment as described in the [JSON-descriptions](#JSON-descriptions). The error message also contains the description of the error.

## Set up for API Doctor in a resource topic

When you run the stub generator, the HTML comments needed for running API Doctor is automatically added to the markdown files that are generated. For example, if you open a file for a resource that was created by the stub generator, you will see this comment before the JSON representation:

```html
<!-- {
  "blockType": "resource",
  "optionalProperties": [],
  "keyProperty": "id",
  "@odata.type": "microsoft.graph.group"
}-->
```

For a description of the JSON elements used in the HTML comment, see [JSON-descriptions](#JSON-descriptions).

## Set up for API Doctor in an API topic

In the context of example requests and responses, each HTML comment should include a JSON object that tells API Doctor that the following lines represent an example request or response, what to expect in the response, and if applicable, validate the properties and their types in the response.

The following comment is added before the request example:

```html
<!-- {
  "blockType": "request",
  "name": "get_group"
}-->
```

The following comment is added before the response example:

```html
<!-- {
  "blockType": "response",
  "truncated": true,
  "@odata.type": "microsoft.graph.group",
  "isCollection": true,
  "name": "get_groups"
} -->
```

API Doctor can validate an example only if you organize the request and response as a pair. Although not required, it’s a good practice to specify a unique name for the request and response in the pair. API Doctor references that name in error messages about that example.

To document cases where one example request can result in mulitple possible responses, it’s better to separately describe and form a request-response pair for each case. This way, you can enable API Doctor validation for each of the example responses.

## JSON descriptions

The most important property in the JSON object is `blockType`, which must be one of:

- `resource`: Identifies the lines that follow this blockType statement as a JSON blob, which represents the properties of an entity or complex type. Occurs in the “JSON representation” section of a resource type topic.
- `request`: an example HTTP request.
- `response`: an example HTTP response.
- `ignored`: a block that API Doctor should ignore validating. Use this if the code block isn’t part of an example, such as the “HTTP request” section of a method topic.

Besides `blockType` (which is required), there are several other properties.

- **@odata.type** or just @type: Represents the fully qualified type name in Microsoft Graph (e.g., microsoft.graph.user). You must specify this for resource definitions and example responses which include a response body. Always use the same case for the same resource in any occurrence of @odata.type, and the case should also match the definition in the Microsoft Graph metadata. In other words, API Doctor considers “micrsoft.graph.user” and “microsoft.graph.User” as two different types.
- **baseType**: Specify this in resource type topics, for resources that have a parent type. For many entities, this is microsoft.graph.entity. Exclude this property if there is no base type.
- **keyProperty**: Specify this in resource type topics. This is the property name that represents the key in the metadata. For entities which inherit from the microsoft.graph.entity type, this is the **id** property. Also make sure that the Properties section includes this property name (e.g., **id**) in the properties table. For complex types, exclude keyProperty, as complex types primarily only define data structures in Microsoft Graph.

    API Doctor complains if it sees a keyProperty on a resource that doesn’t have a property with that name. That’s a hint that either:
    - The property is really a complex type and hence has no key.
    - The property is an entity type that inherits from `microsoft.grpah.entity` but is missing the baseType attribute.
    - The property is supposed to be an entity and the keyProperty attribute is set correctly, but the resource definition and/or table are missing that key property.

- **openType**: Specify this in resource type topics for resources that are open types, meaning they support dynamic properties that aren’t in the schema. This property basically does two things in API Doctor:
    - When it produces an EDMX file, it’ll appropriately declare the type as open.
    - When API doctor validates responses, it’ll be more lenient about unknown properties in a type marked as open.
- **optionalProperties**: Specify this in resource type topics. List the properties that are not always returned in a GET operation here. By default, API Doctor assumes that a full response in an example in the docs contains all the properties defined in the JSON (specified in the **JSON representation** section of the resource type topic). Including navigation properties in the JSON of an entity is optional. However, if you do include navigation properties in the JSON of an entity, then you must include those navigation properties in optionalProperties as well. Otherwise API Doctor would expect those navigation properties in GET responses.
- **name**: Specify this for example HTTP requests and responses. The name is used in error messages and reporting. Specify a unique friendly name so that you can conveniently identify the location of an example error called out by API Doctor. It is optional on example HTTP response blocks, as API Doctor assumes an example response applies to the previous example request. In cases where that’s not true, including the name property in a response block tells API Doctor which request it matches with.
- **truncated**: Set this to true if the example response doesn’t include a response body. Where the example response does include a response body, setting truncated to true is optional, and doing so avoids API Doctor returning an error if your example leaves out some properties that you have defined in the JSON (under the **JSON representation** section of the resource type). Setting truncated to true has the disadvantage of not getting notified if your example response is out-of-date.