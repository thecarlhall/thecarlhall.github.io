---
title: "keytool is Strict"
date: "2010-04-05"
categories: 
  - "development"
tags: 
  - "java"
  - "keytool"
  - "ldap"
---

Let's say you are setting up a connection to an LDAP server and need to get your security bits in place.  To get the certificate chain from the server, you may do something like this:

> openssl s_client -showcerts -connect dir.example.com:636 > cert_chain.pem

Now with the output of that, you want to create or add to a keystore. That command looks like this:

> keytool -import -storetype jks -keystore dir_example.jks -storepass dir-example -file cert_chain.pem -alias ldap-ca -noprompt

But, oh no! Java spits back,

> keytool error: java.lang.Exception: Input not an X.509 certificate

Well, that sucks, but have no fear! The trick to clearing this up is quite easy. keytool is quite strict. The output from the openssl command above is far too verbose for keytool, so you'll need to clean up any text outside of the -----BEGIN CERTIFICATE----- and -----END CERTIFICATE----- markers in the file.
