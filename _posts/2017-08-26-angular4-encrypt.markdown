---
layout: post
title: How to | Encrypt Files Client Side Using Angular 4
date: 2017-08-26T10:48:14.000Z
categories: Angular4
---

<a href="https://github.com/Gmentsik/gly-ng2-filenecrypt"><img src="/images/checkout-copy.PNG" style="float: right;margin-left:10px;width:200px"/></a>

We will use **CryptoJS** to encrypt,  and **FileReader** to read a file.
Furthermore,  we will split our files in **chunks** in order to manage memory usage.

In this how-to we will collect the chunks in an array, which is silly of course, because the (encrypted) file ends up in the memory anyway.

In a real world application we would upload the encrypted chunks for instance.

Content

+ Reading a File
+ Encrypting a String
+ Chunks
+ Putting It All Together
+ Decrypt the file



# Reading a file
I like to start with UI in my projects because it is a good way to describe what we want to achieve.

So let's create a field which can recieve a file as input. We want to wire this field up with a function that will encrypt the given file.

```
> app.component.html
<div>
  Select file to encrypt:
  <input type="file" (change)="readFile($event)">
</div>
```

If we now select a file,  `(change)="readFile($event)` will fire and call a function with the name `readFile`.

We will now head over to our `app.component.ts` and write our business logic as described in the beginning.

Reading a file with the `FileReader` is easy and works this way:

<pre><code data-language="javascript"> > app.component.ts
const reader = new FileReader();
reader.readAsText($event.target.files[0]);
</code></pre>

This code snippet will read the first file. In order to get access of the content of the file we have to provide the reader a **callback function** which will be triggered after the read has finished.

This callback function will be provided to `reader.onloadend` which then has access to `reader.result` which in turn will hold the file's content.

```
> app.component.ts
readFile($event){
    const reader = new FileReader();
    reader.onloadend = function (e) {
        console.log(reader.result);
    }
    reader.readAsText($event.target.files[0]);
 }
```

# Encrypting a String

> **CryptoJS** is a growing collection of standard and secure cryptographic algorithms implemented in JavaScript using best practices and patterns. They are fast, and they have a consistent and simple interface.


First of all we will need to import CryptoJS

```
 > app.component.ts
    import * as CryptoJS from 'crypto-js';
```

Using CryptoJS is straight forward.
First we have to specify a key ("password") and a initialization vector.

```
> app.component.ts
    let password = 'disPasswordIsSoSec§ur#!';
    const key = CryptoJS.enc.Base64.parse(password);
    const iv  = CryptoJS.enc.Base64.parse("#base64IV#");
```

This is all we need! The next step is to just encrypt that string!


```
> app.component.ts
  let password = 'disPasswordIsSoSec§ur#!';
  const key = CryptoJS.enc.Base64.parse(password);
  const iv  = CryptoJS.enc.Base64.parse("#base64IV#");
  let encrypted = CryptoJS.AES.encrypt("Secret Text", key, {iv: iv}).toString();
  console.log(encrypted);
```


This is it!

# Chunks

We could just combine the things above, which would work well for small files. In order to be able to cope with bigger files too, we should slice them up in chunks. This will make the whole process more reliable because if the encryption process fails, you can just retry it for that chunk only. This is also true if we want to upload the file to an endpoint.

There is a disadvantage however: since the encryption process adds some overhead to each chunk, the resulting filesize will be bigger. If a chunk of 20 bytes is encrypted, the resulting encrypted chunksize will be 44 bytes.

We will use the `Slice()` Method in combination with an old-fashioned 'for' -loop.

```
> app.component.ts
let chunksize = 20;
for (let i = 0; i < file.size; i=i+chunksize)
  this.readChunk(file,i,i+chunksize);

readChunk(file,start,end) {
    const reader = new FileReader();
    const blob = file.slice(start, end);
    reader.readAsText(blob);
  }
```


# Putting It All Together

Now we have all we need! I will just throw the source code at your face, which you will understand if you have read the chapters above :)

```
> app.component.ts
export class AppComponent {
  private chunkedFile = Array<String>();
  private password = 'disPasswordIsSoSec§ur#!';
  private chunkEncrypt = 20;
  private filename: string = "";

  encryptFile($event) : void {
    this.readAndCrypt($event.target.files[0],this.chunkEncrypt);
  }

  readAndCrypt(file: File, chunksize) : void {
    this.filename = file.name;

    for (let i = 0; i < file.size; i=i+chunksize)
      this.readChunk(file,i,i+chunksize,this.chunkedFile,this.password);
  }

  readChunk(file,start,end, ar: Array<String>,password) {
    const reader = new FileReader();
    const blob = file.slice(start, end);

    const key = CryptoJS.enc.Base64.parse(password);
    const iv  = CryptoJS.enc.Base64.parse("#base64IV#");

    reader.onloadend = function (e) {
        let encrypted = CryptoJS.AES.encrypt(reader.result, key, {iv: iv}).toString();
        ar.push(encrypted);
      };
    reader.readAsText(blob);
  }
}
```

This component will slice up the file in chunks, read the chunks as text which we call blobs, encrypt the chunks and then add the chunk to an array.

In order to further process the chunk, replace `ar.push(encrypted);` with any function of your choice eg. `uploadEncryptedChunk(encrypted)`.

**But I want to download the encrypted file!**
Here you go:
```
> app.component.ts
  downloadFile() {
    let content = this.chunkedFile.join('');
    let blob = new Blob([content], { type: 'application/octet-binary' });
    let url = window.URL.createObjectURL(blob);

    if(navigator.msSaveOrOpenBlob) {
      navigator.msSaveBlob(blob, this.filename);
    } else {
      let a = document.createElement('a');
      a.href = url;
      a.download = this.filename;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
    }
    window.URL.revokeObjectURL(url);
  }
```

  # Decrypt the file
  If we want to decrypt the file, we just have to slice up the file properly (if we joined the chunks) and decrypt the chunks. Like I mentioned above, the encrypted chunks are bigger eg. 20 bytes unencrypted and 44 bytes encrypted. This information has to be stored somewhere.

  We can decrypt the chunks with

```
> app.component.ts
CryptoJS.AES.decrypt(reader.result,key, {iv: iv}).toString(CryptoJS.enc.Utf8);
```

  # Checkout this project on GitHub
  https://github.com/Gmentsik/gly-ng2-filenecrypt
