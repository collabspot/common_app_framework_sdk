## Zendesk App Framework v2 Beta
The Zendesk App Framework (ZAF) v2 introduces a new way of building Zendesk apps using iframes.

The v2 Framework improves upon v1 by:
- improving on security by taking advantage of the browsers own cross-domain security features.
- making it easy to use whatever technologies you're familiar with.
- providing consistent APIs across all supported Zendesk Products

### How does it work?
ZAF v2 apps include a manifest.json file and an assets folder. The manifest file includes a slightly modified locations property, which specifies URL paths for each location. These URL paths can be absolute, pointing to an external page, or relative, pointing to a html file inside the assets folder.

e.g.
```json
"location": {
  "zendesk": {
    "ticket_sidebar":       "assets/iframe.html?location=ticket_sidebar",
    "new_ticket_sidebar":   "assets/iframe.html?location=new_ticket_sidebar",
    "user_sidebar":         "assets/user_sidebar.html",
    "organization_sidebar": "assets/organization_sidebar.html",
    "nav_bar":              "https://dashboard.myapp.com/nav_bar",
    "top_bar":              "https://dashboard.myapp.com/top_bar"
  }
}
```

The HTML files referenced on the manifest file will need to include the ZAF SDK v2 library in a script tag. This will allow your iframe to interact with the new ZAF SDK APIs documented below.

i.e.
```html
<script src="http://assets.zendesk.com/apps/sdk/2.0.0-beta1/zaf_sdk.js"></script>
```

### Feature parity
While ZAF v2 aims to support all existing framework features, in some cases due to the nature of the new architecture some features may be missing or incomplete. For more information check out our feature parity status page [here]().

### Migration
Existing Zendesk Apps can migrate to ZAF v2 using the Iframe App Template. Check out our [Migration Guide]() for more information.

### Creating a new ZAF v2 app
The Zendesk App Tools (ZAT) provides a new option to create ZAF v2 apps, also called Iframe Only apps.

```
$ zat new --iframe-only
```

After following the usual prompts ZAT will ask you if you want to enter an iframe URI. If you leave the URI blank, ZAT will create a basic HTML page for you, using the [template here](https://github.com/zendesk/zendesk_apps_tools/blob/master/app_template_iframe/assets/iframe.html).

## ZAF SDK v2
ZAF SDK v2 allows you to interact with the framework directly from your iframe. If you haven't used ZAF SDK before check out [Iframes In Apps: ZAFClient API](./iframes_in_apps#zafclient-api), just keep in mind the `postMessage` API described there has been deprecated in v2, since ZAF v2 apps interact directly with the framework, rather than through a legacy Zendesk app.

## ZAF SDK v2 API Reference

#### client.metadata()

Request metadata for the app, such as the app ID and installation ID.

##### Returns

A [Promises/A+](https://promisesaplus.com) conformant `promise` object.

```javascript
var client = ZAFClient.init();

client.metadata().then(function(metadata) {
  console.log(metadata); // { appId: 1234, installationId: 1234 }
});
```

#### client.context()

Request context for the app, such as the host and location.
Depending on the location, the context will provide additional identifiers
which can be used with the REST API to request additional data.

##### Returns

A [Promises/A+](https://promisesaplus.com) conformant `promise` object.

```javascript
var client = ZAFClient.init();

client.context().then(function(metadata) {
  console.log(metadata); // { host: 'zendesk', hostAccountId: 'mysubdomain', location: 'ticket_sidebar', ticketId: 1234 }
});
```

#### client.get(paths)

Get data from the UI asynchronously. For a complete list of all paths supported check out our [Feature parity status]() page.

##### Arguments

  * `paths` an array of path strings or a single path string

##### Returns

A [Promises/A+](https://promisesaplus.com) conformant `promise` object.

```javascript
var client = ZAFClient.init();

client.get(['ticket.subject', 'ticket.assignee.name']).then(function(data) {
  console.log(data); // { 'ticket.subject': 'Help, my printer is on fire', 'ticket.assignee.name': 'Mr Smith' }
});
```

#### client.set(key, val)
#### client.set(obj)

Set data in the UI asynchronously. For a complete list of all paths supported check out our [Feature parity status]() page.

##### Arguments

  * `key` the path to which to set the value `val`
  * `obj` a json object containing the keys and values to update

##### Returns

A [Promises/A+](https://promisesaplus.com) conformant `promise` object returning the updated data.

```javascript
var client = ZAFClient.init();

client.set('ticket.subject', 'Printer Overheating Incident').then(function(ticket) {
  console.log(ticket);
});

// or

client.set({'ticket.subject': 'Printer Overheating Incident', 'ticket.type': 'incident' }).then(function(ticket) {
  console.log(ticket);
});
```

#### client.invoke(name, [data])

Remotely invoke an API from the host framework. For a complete list of all paths supported check out our [Feature parity status]() page.

##### Returns

A [Promises/A+](https://promisesaplus.com) conformant `promise` object.

```javascript
var client = ZAFClient.init();

client.invoke('ticket.comment.appendText', 'Help!').then(function(data) {
  console.log('text has been appended');
});
```