---
title: "Setting \"quoted-printable\" In A commons-email Body Part"
date: "2010-09-02"
categories: 
  - "development"
---

I'm revamping the inner workings of a mail sending tool to use [commons-email](http://commons.apache.org/email).  I wanted to make sure I set the text body parts to the appropriate transfer encoding to allow for non-ASCII languages to be used. My failed attempts and final success are documented below.

### 1\. \[bad\] Set Message 'Content-transfer-encoding'

The problem with this is that everything in the message is processed as "quoted-printable" which will turn your multipart boundaries into useless wastes of text.

```java
// this chews up your multipart boundary
HtmlEmail email = new HtmlEmail();
email.addHeader("Content-transfer-encoding", "quoted-printable");
```

So rather than having this in your raw email _(note reported and actual boundary match)_:

`Content-Type: multipart/alternative; boundary="----=_Part_7_1164835397.1283397465477"`

`------=_Part_7_1164835397.1283397465477 Content-Type: text/plain; charset=UTF-8`

You end up with this in your raw email _(note: actual boundary has = replaced by =3D)_:

`Content-Type: multipart/alternative; boundary="----=_Part_7_1164835397.1283397465477"`

`----=3D_Part_7_1164835397.1283397465477 Content-Type: text/plain; charset=UTF-8`

### 2\. \[useless\] Setting Each Body Part's transfer encoding

I'll save you the pain. This begins to turn commons-email back into javamail. It saves you nothing, adds lots of code, doesn't really work and, given the next solution, is the wrong approach.

### 3\. \[Best!\] Let commons-email do it's job

The solution I ended up with is to just let commons-email figure it out. My test text is "You get that thing I sent you?\\n3x6=18\\nåæÆÐ". This has several non-ASCII characters that are just ripe for conversion. The raw bit of email I find is: `------=_Part_4_644193719.1283397465436 Content-Type: text/plain; charset=UTF-8 Content-Transfer-Encoding: quoted-printable`

`You get that thing I sent you? 3x6=3D18 =C3=A5=C3=A6=C3=86=C3=90`

The content-transfer-encoding is set as I expected and the content has been properly escaped. Kudus to the commons-email team for making this just happen.
