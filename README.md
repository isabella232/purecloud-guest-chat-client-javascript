---
title: PureCloud Guest Chat Client - JavaScript
---

## Resources

[![GitHub release](https://img.shields.io/github/release/mypurecloud/purecloud-guest-chat-client-javascript.svg)]()
[![npm](https://img.shields.io/npm/v/purecloud-guest-chat-client.svg)](https://www.npmjs.com/package/purecloud-guest-chat-client)

* **Documentation** https://developer.mypurecloud.com/api/rest/client-libraries/javascript-guest/
* **Source** https://github.com/MyPureCloud/purecloud-guest-chat-client-javascript
* **Guest chat documentation** https://developerpreview.inindca.com/api/webchat/guestchat.html (preview documentation)

## CommonJS

For node.js via [NPM](https://www.npmjs.com/package/purecloud-guest-chat-client):

```{"language":"sh"}
npm install purecloud-guest-chat-client
```

```{"language":"javascript"}
// Obtain a reference to the platformClient object
const platformClient = require('purecloud-guest-chat-client');
```

For direct use in a browser script:

```{"language":"html"}
<!-- Include the CJS SDK -->
<script src="https://sdk-cdn.mypurecloud.com/javascript-guest/5.2.0/purecloud-guest-chat-client.min.js"></script>

<script type="text/javascript">
  // Obtain a reference to the platformClient object
  const platformClient = require('platformClient');
</script>
```


## AMD

```{"language":"html"}
<!-- Include requirejs -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.3.5/require.min.js"></script>

<script type="text/javascript">
  // Obtain a reference to the platformClient object
  requirejs(['https://sdk-cdn.mypurecloud.com/javascript-guest/amd/5.2.0/purecloud-guest-chat-client.min.js'], (platformClient) => {
    console.log(platformClient);
  });
</script>
```

## ES6 Classes and Other Entry Points

The node package's [package.json](https://github.com/MyPureCloud/purecloud-guest-chat-client-javascript/blob/master/build/package.json) file contains the following entry points for use with various packaging systems:

* **jsnext:main**
  * Entry point: src/purecloud-guest-chat-client/index.js
  * The main ES6 class in the source code
* **main**
  * Entry point: dist/node/purecloud-guest-chat-client.js
  * The CJS module for node apps
* **browser**
  * Entry point: dist/web-cjs/purecloud-guest-chat-client.min.js
  * The [Browserify](http://browserify.org/)ed CJS module for standalone use in a browser


## Using the SDK

### Creating a chat

The guest chat APIs do not require standard PureCloud authentication, but do require the JWT to be set for all API calls other than creating a new chat.  

```{"language":"javascript"}
const client = platformClient.ApiClient.instance;
let chatInfo, socket;

// Create API instance
const webChatApi = new platformClient.WebChatApi();

const createChatBody = {
  organizationId: '12b1a3fe-7a80-4b50-45fs-df88c0f9efad',
  deploymentId: 'a3e316a7-ec8b-4fe9-5a49-dded9dcc097e',
  routingTarget: {
    targetType: 'QUEUE',
    targetAddress: 'Chat Queue',
  },
  memberInfo: {
    displayName: 'JavaScript Guest',
    profileImageUrl: 'http://yoursite.com/path/to/guest/image.png',
    customFields: {
      firstName: 'John', 
      lastName: 'Doe'
    }
  }
};

// Create chat
webChatApi.postWebchatGuestConversations(createChatBody)
  .then((createChatResponse) => {
    // Store chat info
    chatInfo = createChatResponse;

    // Set JWT
    client.setJwt(chatInfo.jwt);

    // Connect to notifications
    socket = new WebSocket(chatInfo.eventStreamUri);

    // Connection opened
    socket.addEventListener('open', function (event) {
      console.log('WebSocket connected');
    });

    // Listen for messages
    socket.addEventListener('message', function (event) {
      const message = JSON.parse(event.data);

      // Chat message
      if (message.metadata) {
        switch(message.metadata.type) {
          case 'message': {
            // Handle message from member
            break;
          }
          case 'member-change': {
            // Handle member state change
            break;
          }
          default: {
            console.log('Unknown message type: ' + message.metadata.type);
          }
        }
      }

    });
  })
  .catch(console.error);
```


## Environments

If connecting to a PureCloud environment other than mypurecloud.com (e.g. mypurecloud.ie), set the environment on the `ApiClient` instance.

```{"language":"javascript"}
const client = platformClient.ApiClient.instance;
client.setEnvironment(platformClient.PureCloudRegionHosts.eu_west_1);
```


## Making Requests

All API requests return a Promise which resolves to the response body, otherwise it rejects with an error. After setting the JWT, the following code will make an authenticated request:

```{"language":"javascript"}
// Create API instance
const webChatApi = new platformClient.WebChatApi();

// Authenticate
webChatApi.postWebchatGuestConversations(createChatBody)
  .then(() => { 
    // Set JWT
    platformClient.ApiClient.instance.setJwt(chatInfo.jwt);

    // Send a message
    return webChatApi.postWebchatGuestConversationMemberMessages(chatInfo.id, chatInfo.member.id, { 
      body: 'Message from chat guest' 
    });
  })
  .then(() => {
    // Message sent
  })
  .catch((err) => {
    // Handle failure response
    console.log(err);
  });
```


### Extended Responses

By default, the SDK will return only the response body as the result of an API function call. To retrieve additional information about the response, enable extended responses. This will return the extended response for all API function calls:

```{"language":"javascript"}
const client = platformClient.ApiClient.instance;
client.setReturnExtendedResponses(true);
```

Extended response object example (`body` and `text` have been truncated):

```{"language":"json"}
{
  "status": 200,
  "statusText": "",
  "headers": {
    "inin-ratelimit-allowed": "180",
    "inin-ratelimit-count": "3",
    "inin-ratelimit-reset": "3",
    "pragma": "no-cache",
    "inin-correlation-id": "ec35f2a8-289b-42d4-8893-c50eaf81a3c1",
    "content-type": "application/json",
    "cache-control": "no-cache, no-store, must-revalidate",
    "expires": "0"
  },
  "body": {},
  "text": "",
  "error": null
}
```


### Using a Proxy (Node.js only)

Using a proxy is accomplished in two steps:

1. Apply the `superagent-proxy` package to the `client.superagent` object
2. Set proxy settings on the `client` object

After both steps have been completed, the configured proxy server will be used for all requests.

NOTE: SDK proxy configuration is only available in the node.js package due to `superagent-proxy`'s incompatibility with browserify. Additionally, `superagent-proxy` is not included a dependency of the SDK and must be provided by your node application's dependencies.

```{"language":"javascript"}
const client = platformClient.ApiClient.instance;
require('superagent-proxy')(client.superagent);
// Configure settings for your proxy here
// Documentation: https://www.npmjs.com/package/proxy-agent
client.proxy = {
  host: '172.1.1.100',
  port: 443,
  protocol: 'https',
};
```


### Error Responses

Error responses will always be thrown as an extended response object. Note that the `error` property will contain a JavaScript [Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) object.

Example error response object:

```{"language":"json"}
{
  "status": 404,
  "statusText": "",
  "headers": {
    "inin-ratelimit-allowed": "300",
    "inin-ratelimit-count": "6",
    "inin-ratelimit-reset": "38",
    "pragma": "no-cache",
    "inin-correlation-id": "d11bd3b3-ab7e-4fd4-9687-d04af9f30a63",
    "content-type": "application/json",
    "cache-control": "no-cache, no-store, must-revalidate",
    "expires": "0"
  },
  "body": {
    "status": 404,
    "code": "not.found",
    "message": "The requested operation failed with status 404",
    "contextId": "d11bd3b3-ab7e-4fd4-9687-d04af9f30a63",
    "details": [],
    "errors": []
  },
  "text": "{\"status\":404,\"code\":\"not.found\",\"message\":\"The requested operation failed with status 404\",\"contextId\":\"d11bd3b3-ab7e-4fd4-9687-d04af9f30a63\",\"details\":[],\"errors\":[]}",
  "error": {
    "original": null
  }
}
```


## Debug Logging

There are hooks to trace requests and responses.  To enable debug tracing, provide a log object. Optionally, specify a maximum number of lines. If specified, the response body trace will be truncated. If not specified, the entire response body will be traced out.

```{"language":"javascript"}
const client = platformClient.ApiClient.instance;
client.setDebugLog(console.log, 25);
```


## Versioning

The SDK's version is incremented according to the [Semantic Versioning Specification](https://semver.org/). The decision to increment version numbers is determined by [diffing the Platform API's swagger](https://github.com/purecloudlabs/platform-client-sdk-common/blob/master/modules/swaggerDiff.js) for automated builds, and optionally forcing a version bump when a build is triggered manually (e.g. releasing a bugfix).


## Support

This package is intended to be forwards compatible with v2 of PureCloud's Platform API. While the general policy for the API is not to introduce breaking changes, there are certain additions and changes to the API that cause breaking changes for the SDK, often due to the way the API is expressed in its swagger definition. Because of this, the SDK can have a major version bump while the API remains at major version 2. While the SDK is intended to be forward compatible, patches will only be released to the latest version. For these reasons, it is strongly recommended that all applications using this SDK are kept up to date and use the latest version of the SDK.

For any issues, questions, or suggestions for the SDK, visit the [PureCloud Developer Forum](https://developer.mypurecloud.com/forum/).
