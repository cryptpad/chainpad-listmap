# chainpad-listmap
collaborative lists and maps in the browser.

## Usage

Best used in conjunction with [Chainpad-server](https://github.com/xwiki-labs/chainpad-server).

```
define([
    '/api/config',
    '/bower_components/chainpad-listmap/chainpad-listmap.js',
    '/bower_components/chainpad-crypto/crypto.js'
], function (Config, Listmap, Crypto) {
    var rt = Listmap.create({
        websocketURL: Config.websocketURL,
        channel: "b87dff2e9a465f0e0ae36453d19b087c",
        cryptKey: "sksyaHv+OOlQumRmZrQU4f5N",
        data: {},
        crypto: Crypto
    });

    var proxy = rt.proxy.on('ready', function () {
        window.alert("realtime proxy object is ready to use!");

        // listen for changes anywhere within the nested structure
        proxy.on('change', [], function (oldValue, newValue, path) {
            console.log("Realtime object was changed at path [%s]: %s => %s",
                path.join(','), oldvalue, newvalue);
        })
        // or listen for changes at specific paths (or children thereof)
        .on('change', ['one', 2, 'three'], function (o, n, p) {
            console.log("Realtime object was changed at path [%s]: %s => %s",
                path.join(','), oldvalue, newvalue);

            // return `false` to prevent this event from 'bubbling up'
            return false;
        });
    }).on('disconnect', function () {
        window.alert("network connection lost!");
    });
});
```

## API

### Listmap.create

`Listmap.create` is a function which takes a configuration object consisting of the following attributes:

* websocketURL
* channel
* cryptKey
* crypto
* data

With the exception of `data`, all configuration paramaters match those documented in [chainpad-netflux](https://github.com/xwiki-labs/chainpad-netflux#configuration-parameters), on which chainpad-listmap depends.

`data` is used to generate `initialState`, which is supplied to chainpad-netflux.
Chainpad expects that all users in a session will supply the same initialState, so unless you can guarantee that all users will provide an identical argument here, it is best to initialize data with an empty array or object.
chainpad-listmap supports both, but has been tested more extensively using objects.

`Listmap.create` will return an object with the following properties:

* realtime
  - the chainpad realtime object, in case you ever want to interact with chainpad directly
* proxy
  - the realtime list or map that was created based on your supplied `data` attribute
* network
  - the [Netflux](https://github.com/xwiki-labs/netflux-spec2) network object created and used by `chainpad-netflux` as a transport layer for your realtime object's messages.

#### Listmap.create().proxy

For most purposes, this is the object you'll be interacting with most frequently.

##### Arrays

If you created your proxy using using an array (aka a list (`[]`)) for a data attribute, you will be able to use the usual in-place array methods (push, pop, shift, unshift, sort).

Methods which return new arrays (map, slice, filter) will not trigger events.

##### Objects

If you created your proxy using an object (aka a map (`{}`)) for a data attribute, it will behave as expected.

##### proxy.on

The List/Map api provides a means of listening to a variety of events related to your realtime proxy:

###### `on('create', cb)`

Corresponds to [chainpad-netflux#onInit](https://github.com/xwiki-labs/chainpad-netflux#oninit).

###### `on('ready', cb)`

Corresponds to [chainpad-netflux#onReady](https://github.com/xwiki-labs/chainpad-netflux#onready)

###### `on('disconnect', cb)`

Corresponds to [chainpad-netflux#onAbort](https://github.com/xwiki-labs/chainpad-netflux#onabort)

###### `on('change', path, cb)`

Listen for changes which match a given path, where `cb` is a callback, and path is an array of property names which would select the targeted property given the root object.
For example:

```
var A = {
    b: [
        0,
        1,
        {
            c: 5
        }
    ]
};
```

`c` could be selected given the path `['b', 2, 'c']`.

Changes _bubble up_, so if you were to specify a path `['b', 2]`, changes to `['b', 2, 'c']` would trigger that listener.

Callbacks take the form:

```
function (oldvalue, newvalue, path, rootObject) {
    // do things
};
```

To prevent a change from bubbling up, return `false` from your callback, as you would with DOM event listeners.

Deletions are considered a change for the directly affected value.
If that value is an array or object, all of its children will be the subject of a `remove` event.

###### `on('remove', path, cb)`

You can listen for removals and you would listen for changes, except removals _cascade down_.
That is to say, if you remove an object, its children, and their children (etc) will be the subject of remove events.

`remove` callbacks take the form:

```
function (oldvalue, path, rootObject) {
    // do things
};
```

### DeepProxy

The list/map api exposes **DeepProy**, a submodule which is used internally for creating recursive proxies which execute a provided callback when modified.

For typical usage, users will not need to interact with DeepProxy, however, it is exposed in case you _do_ want access to its features.

If you are particularly interested in DeepProxy, please file an issue and I'll try to address your questions.

## Installation

`bower install chainpad-listmap`

## License

This software is and will always be available under the GNU Affero General Public License as
published by the Free Software Foundation, either version 3 of the License, or (at your option)
any later version. If you wish to use this technology in a proprietary product, please contact
sales@xwiki.com

