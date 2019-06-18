# Access to Local Text Files in HTML5

## Brief

Reference test in vanilla [HTML5](https://en.wikipedia.org/wiki/HTML5) to both read and store plain text files. As per browser constraints any file can be selected from reading and storage imposes no special filename but path is obligatory and implicitly located the *download* area managed by most browsers

## Runtime

This test can be run from: https://labzdjee.github.io/html5-local-file-access

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

Getting contents of the selected file takes an object `reader` of class `FileReader`: after being instantiated, its method `readAsText` (text or possibly `readAsArrayBuffer`  to get contents as binary) is called on the `file` object. Then the `onload(e)` function is called when file contents is correctly read and `e.target.result` has the contents. In fact, and as no surprise, `e.target` equals `reader`

## Storing

A text file can be written to user storage through a special `<a href=` link with a `download` attribute

As already mentioned, writing can only be done in the download are the browser currently points to. any path in the filename will not be taken into account (but instead have its folder separators changed into acceptable characters for a file name such as underscore characters)

So, this `<a href="` element must have a `download` attribute with value the filename to store

Technically speaking what we can write is an object `data` of class `Blob`. This object is created with parameters `[text], {type: 'text/plain'}` This mime type could of course be refined such as text/xml... `text` is a string which contains file content

From this data [Blob](https://en.wikipedia.org/wiki/Binary_large_object), we have the `href` value called an *object URL* as `window.URL.createObjectURL(data)` . However, this value is stored in a variable in order to call `window.URL.revokeObjectURL(previousUrlObject)`  in case it has already being assigned, this to avoid memory leaks. This is the main reason why we have the `onStoreFileNameUpdate`  helper

When users click this special `<a` link, they are sometimes prompted through a dialog box to confirm they are okay to save. In fact, what happens on such click greatly depends on the browser implementation

In order to disable this clickable `<a` link it came as handy to create a CSS class disabled with a special style when disabled and `pointer-events: none;`  which make it disabled. Enabling this link takes calling `classList.remove("disabled")` on this element and disabling it `saveDomElement.classList.add("disabled")`

## File Format (UTF-8 vs Latin-1) and MD5

Along the way, we wanted to display [md5](https://en.wikipedia.org/wiki/MD5) hash of files we read. This is done with method `CryptoJS.MD5` from [crypto-js](https://github.com/brix/crypto-js) library written and maintained by '*brix*' ([Evan Vosberg](https://urban.to/evanvosberg/about)) where we only used `core.js` and `md5.js` modules taken from a [CDN](https://en.wikipedia.org/wiki/Content_delivery_network)

However one important difficulty we had to be overcome to manage MD5 over *binary* file not *text* file. The distinction is important when file is not encoded as multi-byte *[UTF-8](https://en.wikipedia.org/wiki/UTF-8)* 16-bit encoding because `CryptoJS.MD5` transforms *ECMAscript* Strings into binary assuming *UTF-8*. So when a *Latin-1* file is loaded with `readAsText` there is distortion and no direct way to tell `CryptoJS.MD5` not to consider a string as an *UTF-8* stream which it wasn't originally

The only way is to feed `CryptoJS.MD5` with raw data.  In order to do this, first we read file with `reader.readAsArrayBuffer(file)`. Then the result is stored into a `Uint8Array` and transformed into a `CryptoJS.lib.WordArray` which is a 32-bit array with the following internal format:

- byte stream **B0, B1, B2, B3, B4, B5, B6**, for example, is transformed into an array of two 32-bit words like this:

  ​	**[ (B0<<24)|(B1<<16)|(B2<<8)|B3, (B4<<24)|(B5<<16)|(B6<<8)|0 ]**

This `CryptoJS.lib.WordArray` is fine for calculation of *MD5*. Now, we need to store this array as a string. First, we try `toString(CryptoJS.enc.Utf8)` on it. If the file contains proper multi-byte *UTF-8* or only 7-bit characters then it succeeds. So, this makes *UTF-8*, the <u>default</u>. Now, if some [Latin-1 Supplement](https://en.wikipedia.org/wiki/Latin-1_Supplement) character is met along the way (e.g. Unicode *U+00D1* for *Ñ*), the *UTF-8* encoding will fail and we then use `toString(CryptoJS.enc.Latin1)`  on this word array

 An attempt is also made to store the file as *UTF-8* or *Latin-1*, matching how it has been read (stored in variable `utf8Sucess`). This is refined in function `makeTextFile` where we use different [Blob](https://en.wikipedia.org/wiki/Binary_large_object)'s in those two cases:

- *UTF-8*: we simply make `new Blob` with file contents as String
- *Latin-1*: we build a generic `ArrayBuffer`, called `genericBuffer` and associate it to an `Uint8Array` which we fill with first eight bits of each string character value (using `charCodeAt`). This array buffer is passed to new Blob as `[genericBuffer]`. Use of this generic buffer instead of simply an `Unit8Array` seems a little bit academic but honors `new Blob` API closely (see [here](https://developer.mozilla.org/en-US/docs/Web/API/Blob/Blob))









