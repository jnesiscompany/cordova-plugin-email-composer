Cordova EmailComposer Plugin
====================

<img width="260px" align="right" hspace="7" vspace="5" src="http://flashsimulations.com/wp-content/uploads/2011/12/air-ios-in-app-mail-app.png">

The plugin provides access to the standard interface that manages the editing and sending an email message. You can use this view controller to display a standard email view inside your application and populate the fields of that view with initial values, such as the subject, email recipients, body text, and attachments. The user can edit the initial contents you specify and choose to send the email or cancel the operation.

Using this interface does not guarantee immediate delivery of the corresponding email message. The user may cancel the creation of the message, and if the user does choose to send the message, the message is only queued in the Mail application outbox. This allows you to generate emails even in situations where the user does not have network access, such as in airplane mode. This interface does not provide a way for you to verify whether emails were actually sent.


### Plugin's Purpose
The purpose of the plugin is to create an platform independent javascript interface for [Cordova][cordova] based mobile applications to access the specific email composition API on each platform.


## Overview
1. [Supported Platforms](#supported-platforms)
2. [Installation](#installation)
3. [ChangeLog](#changelog)
4. [Using the plugin](#using-the-plugin)
5. [Examples](#examples)
6. [Quirks](#quirks)


## Supported Platforms
- __iOS__ _(up to iOS8)_
- __Android__ _(up to KitKat and L)_
- __WP8__ _(up to WP8.1)_


## Installation
The plugin can either be installed from git repository, from local file system through the [Command-line Interface][CLI]. Or cloud based through [PhoneGap Build][PGB].

### Local development environment
From master:
```bash
# ~~ from master branch ~~
cordova plugin add https://github.com/katzer/cordova-plugin-email-composer.git
```
from a local folder:
```bash
# ~~ local folder ~~
cordova plugin add de.appplant.cordova.plugin.email-composer --searchpath path/to/plugin
```
or to use the last stable version:
```bash
# ~~ stable version ~~
cordova plugin add de.appplant.cordova.plugin.email-composer@0.8.2
```

### PhoneGap Build
Add the following xml to your config.xml to always use the latest version of this plugin:
```xml
<gap:plugin name="de.appplant.cordova.plugin.email-composer" version="0.8.2" />
```
More informations can be found [here][PGB_plugin].


## ChangeLog
#### Version 0.8.2 (not yet released)
- Added new namespace `cordova.plugins.email`<br>
  **Note:** The former `plugin.email` namespace is now deprecated and will be removed with the next major release.
- [___change:___] Unified `absolute:` and `relative:` to `file:`
- [___change:___] Renamed `isServiceAvailable` to `isAvailable`
- [feature:] `res:` prefix for native ressource attachments
- [enhancement:] `open` supports callbacks
- [enhancement:] `isHTML` can be used next `isHtml`
- [bugfix:] Defaults were ignored

#### Further informations
- See [CHANGELOG.md][changelog] to get the full changelog for the plugin.


## Using the plugin
The plugin creates the object ```cordova.plugins.email``` with following methods:

### Plugin initialization
The plugin and its methods are not available before the *deviceready* event has been fired.

```javascript
document.addEventListener('deviceready', function () {
    // cordova.plugins.email is now available
}, false);
```

### Determine if the device is capable to send emails
The ability to send emails can be revised through the `email.isAvailable` interface. The method takes a callback function, passed to which is a boolean property. Optionally the callback scope can be assigned as a second parameter.

The Email service is only available on devices capable which are able to send emails. E.g. which have configured an email account and have installed an email app. You can use this function to hide email functionality from users who will be unable to use it.

```javascript
cordova.plugins.email.isAvailable(
    function (isAvailable) {
        // alert('Service is not available') unless isAvailable;
    }
);
```

### Open a pre-filled email draft
A pre-filled email draft can be opened through the `email.open` or `email.openDraft` interface. The method takes a hash as an argument to specify the email's properties. All properties are optional. Further more it accepts an callback function to be called after the email view has been dismissed.

After opening the draft the user may have the possibilities to edit, delete or send the email.

#### Further informations
- An [configured email account][is_service_available] is required to send emails.
- Attachments can be either base64 encoded datas, files from the the device storage or assets from within the *www* folder.
- The default value for *isHTML* is *true*.
- See the [examples][examples] for how to create and show an email draft.

```javascript
cordova.plugins.email.open({
    to:          Array, // email addresses for TO field
    cc:          Array, // email addresses for CC field
    bcc:         Array, // email addresses for BCC field
    attachments: Array, // paths to the files you want to attach or base64 encoded data streams
    subject:    String, // subject of the email
    body:       String, // email body (could be HTML code, in this case set isHtml to true)
    isHtml:    Boolean, // indicats if the body is HTML or plain text
}, callback, scope);
```


## Examples

### Open an email draft
The following example shows how to create and show an email draft pre-filled with different kind of properties.

```javascript
cordova.plugins.email.open({
    to:      'max@mustermann.de',
    cc:      'erika@mustermann.de',
    bcc:     ['john@doe.com', 'jane@doe.com'],
    subject: 'Greetings',
    body:    'How are you? Nice greetings from Leipzig'
});
```

Of course its also possible to open a blank draft.
```javascript
cordova.plugins.email.open();
```

### Send HTML encoded body
Its possible to send the email body either as text or HTML. In the case of HTML the `isHTML` properties needs to be set.

```javascript
cordova.plugins.email.open({
    to:      'max@mustermann.de',
    subject: 'Greetings',
    body:    '<h1>Nice greetings from Leipzig</h1>',
    isHtml:  true
});
```

### Get informed when the view has been dismissed
The `open` method supports additional callback to get informed when the view has been dismissed.

```javascript
cordova.plugins.email.open(properties, function () {
    console.log('email view dismissed');
}, this);
```

### Adding attachments
Attachments can be either base64 encoded datas, files from the the device storage or assets from within the *www* folder.

#### Attach Base64 encoded content
The code below shows how to attach an base64 encoded image which will be added as a image with the name *icon.png*.

```javascript
cordova.plugins.email.open({
    subject:     'Cordova Icon',
    attachments: 'base64:icon.png//iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/...'
});
```

#### Attach files from the device storage
The path to the files must be defined absolute from the root of the file system.

```javascript
cordova.plugins.email.open({
    attachments: [
        'file:///storage/sdcard/icon.jpg', //=> Android
        'file:///Users/.../Library/Application Support/iPhone Simulator/7.0.3/Applications/E7981856-.../HelloCordova.app/icon.jpg' //=> iOS
    ]
});
```

#### Attach app's resources
Each app has a resource folder, e.g. the _res_ folder for Android apps or the _Resource_ folder for iOS apps.<br>
The following example shows how to attach the app icon from within the app's resource folder.

```javascript
cordova.plugins.email.open({
    attachments: 'res://icon.png'
});
```

#### Attach assets from within the www folder
The path to the files must be defined relative from the root of the mobile web app folder, which is located under the _www_ folder.

```javascript
cordova.plugins.email.open({
    attachments: [
        'file://img/logo.png' //=> e.g. assets/www/img/logo.png (Android)
    ]
});
```


## Quirks

### Email composer under Android and Windows Phone
An configured email account is required to send emails.

### Limited support for Windows Phone 8
Attachments and HTML formatted body are not supported through the native API.

### Compile error on iOS
The error indicates, that the `MessageUI.framework` is not linked to your project. The framework is linked automatically when the plugin was installed, but may removed later.

```
Undefined symbols for architecture i386:
  "_OBJC_CLASS_$_MFMailComposeViewController", referenced from:
      objc-class-ref in APPEmailComposer.o
ld: symbol(s) not found for architecture i386
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```


## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request


## License

This software is released under the [Apache 2.0 License][apache2_license].

© 2013-2014 appPlant UG, Inc. All rights reserved


[cordova]: https://cordova.apache.org
[ios_guide]: http://developer.apple.com/library/ios/documentation/MessageUI/Reference/MFMailComposeViewController_class/Reference/Reference.html
[wp8_guide]: http://msdn.microsoft.com/en-us/library/windowsphone/develop/hh394003.aspx
[CLI]: http://cordova.apache.org/docs/en/edge/guide_cli_index.md.html#The%20Command-line%20Interface
[PGB]: http://docs.build.phonegap.com/en_US/index.html
[PGB_plugin]: https://build.phonegap.com/plugins/705
[messageui_framework]: #compile-error-on-ios
[changelog]: https://github.com/katzer/cordova-plugin-email-composer/blob/master/CHANGELOG.md
[is_service_available]: #determine-if-the-device-is-able-to-send-emails
[examples]: #examples
[apache2_license]: http://opensource.org/licenses/Apache-2.0
