## Chrome extension message passing made easy

This is a small library for sending messages across any extension parts (background, content script, popup or devtool).

It has a simple easy to use API, callback support and more.

### Why?

If you ever tried (like me) creating a fairly large extension which required communication between different parts, you might noticed that sending messages between the parts can get complicated and usually requires some relaying mechanism in the background page.
Adding callback functionality to these messages can make it even trickier.

Furthermore the chrome messaging API is not coherent or straight forward, sometimes requiring you to use _chrome.runtime.\*_ and sometimes _chrome.tabs.\*_ depending on which extension part you are currently in.

### Usage

#### 1) In any extension part -> Create a messenger instance.

    var Messenger = require('chrome-ext-messenger');
    var messenger = new Messenger();

NOTE: If you are not using npm/es6 you can add the library via script tag and use _window['chrome-ext-messenger']_.

#### 2) In the background page -> Init the background hub.
This is obligatory for the library to work and should be done as early as possible in your background page.

    messenger.initBackgroundHub({
        connectedHandler: function(extPart, name, tabId) { console.log('someone connected:', arguments); }, 
        disconnectedHandler: function(extPart, name, tabId) { console.log('someone disconnected:', arguments); }
    });

#### 3) Init connections using "initConnection(name, messageHandler)".

    var messageHandler = function(message, from, sender, sendResponse) {
        if (message.name === 'how are you?') {
            sendResponse('A-OK');
        }
    };

    var connection = messenger.initConnection('main', messageHandler);

#### 4) Start sending messages between connections using "sendMessage(to, message, responseCallback)"
    // Parameters:
    // to - formatted string indicating where to send the message to: 'part:name'.
            for messages from background to other parts, tabId is also required like so: 'part:name:tabId'.
            extension part values: 'background', 'content_script', 'popup', 'devtool'.
    // message - the message to send.
    // responseCallback - function that will be called with the response value of the reciever.

    // devtool -> content script
    connection.sendMessage('content_script:main', { name: 'how are you?' }, function(response) {
       console.log(response);
    });

    // popup -> background
    connection.sendMessage('background:some connection name', { name: 'how are you?' }, function(response) {
       console.log(response);
    });

    // background -> content script ("150" is a tab id example).
    connection.sendMessage('content_script:main:150', { name: 'how are you?' }, function(response) {
       console.log(response);
    });

    // ...

#### More:

    // Sending a message to multiple connections is supported using 'part:name1,name2,...'.
    connection.sendMessage('content_script:main,main2', { name: 'how are you?' });

    // Sending a message to all connections is supported using wildcard value '*'.
    connection.sendMessage('devtool:*', { name: 'how are you?' });

### Developing Locally

    // install webpack
    npm install webpack -g

    // run the dev script
    npm run dev

You can now use the built messenger from the _dist_ folder in a local test extension (or use [npm link](https://docs.npmjs.com/cli/link)).
I have created one (for internal testing purposes) that you can use: [chrome-ext-messenger-test](https://github.com/asimen1/chrome-ext-messenger-test).

### Notes
* Requires your extension to have ["tabs" permission](https://developer.chrome.com/extensions/declare_permissions).
* Uses only long lived port connections via chrome.runtime.* API.
* This library should satisfy all your message passing demands, however if you are still handling some port connections manually using _chrome.runtime.onConnect_, you will also receive messenger ports connections. In order to identify connections originating from this library you can use the static method *Messenger.isMessengerPort(port)* which will return true/false.
* The Messenger messageHandler and chrome.runtime.onMessage similarities and differences:

    Same:
    * "sender" object.
    * "sendResponse" - The argument should be any JSON-ifiable object.
    * "sendResponse" - With multiple message handler, the sendResponse() will work only for the first one to respond.
    
    
    Different:
    * "from" object indicating the senders formatted identifier e.g. 'devtool:some connection name'.
    * Async sendResponse is supported directly (no need to return "true" value like _chrome.runtime.onMessage_ usage).

### Todos
* Support cross tabs communication (e.g. content script from tab 1 to content script of tab 2).
* connection.sendMessage: support array (multiples) in "toExtPart".
* connection.sendMessage: support array (multiples) in "toTabIds" for background to non background.
* connection.sendMessage: support * in "toTabIds" for background to non background (don't forget the "fromTabId" assignment...).

License
----
MIT
