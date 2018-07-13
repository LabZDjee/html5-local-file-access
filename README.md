# Access to Local Text Files in HTML5

## Brief

Test in vanilla HTML5 to both read and store plain text files. As per browser constraints any file can be selected from reading and storage imposes no special filename but path is obligatory and implicitly the *download* area managed by the browser

## Reading

### Selecting File Name

Done through `input` of `type="file"` 

In order to style this area as a button, should match the input `label` through the input `id`

Then `label` bears the style and its contents the pseudo button text

At a minimum this style should include `text-decoration: none;` 

This input is clickable and triggers a user file dialog in order to select a file name

### Reading File Contents

Type of files tested: **text**

Method to read them: `FileReader` / `readAsText` (asynchronous)

Should listen to `"change"` event of the `input` element used for file name selection as described above

The handler to this event will get the `file` object  as `files[0]` property of the `input` element for file selection. This property is an array because attribute `multiple` of this `input` can allow selection of multiple files (not used in this example) 

In order to get contents of the selected file, an object of class `FileReader` is instantiated  and its method `readAsText` is called on the `file` object. Then the `onload(e)` function is called when file contents is correctly read and `e.target.result` has the contents

## Writing

A text file can be written to user storage through a special `<a href=` link with a `download` attribute

As already said writing can only be done in the download are the browser currently points to. any path in the filename will not be taken into account (but instead have its folder separators changed into acceptable characters for a file name such as underscore characters )

So, this `<a href="` element must have a `download` attribute with value the filename to store

Technically speaking what we can write is an object `data` of class `Blob`. This object is created with parameters `[text], {type: 'text/plain'}` This mime type could of course be refined such as text/xml... `text` is a string which contains file content

From this data blob, we have the `href` value called an *object URL* as `window.URL.createObjectURL(data)` . However, this value is stored in a variable in order to call `window.URL.revokeObjectURL(data)`  in case it has already being assigned, this to avoid memory leaks. This is the main reason why we have the `onStoreFileNameUpdate`  helper

When users click this special `<a` link, <u>they are always prompted</u> through a dialog box to confirm they are okay to save

In order to disable this clickable `<a` link it came as handy to create a CSS class disabled with a special style when disabled and `pointer-events: none;`  which make it disabled. Enabling this link takes calling `classList.remove("disabled")` on this element and disabling it `saveDomElement.classList.add("disabled")`

## Bonus: md5

Along the way, we wanted to display [md5](https://en.wikipedia.org/wiki/MD5) of files we read. This is simply done with method `CryptoJS.MD5(fileContent)` from [crypto-js](https://github.com/brix/crypto-js) library written and maintained by '*brix*' (Evan Vosberg ) where we only used `core.js` and `md5.js` modules taken from a CDN