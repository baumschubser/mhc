# MHC (ME-Https-Client)

MHC (ME-Https-Client) is a J2ME library written in Java which offers an alternative implementation
for the javax.microedition.io.HttpsConnection interface.

You can use it to connect to secure (SSL/TLS) websites, specially when the default implementation
of a particular device does not support SSL/TLS certificates issued by non-trusted certificate
authorities nor does it allow the possibility to install these certs/keys into its certificate store.

It is released under the Apache License 2.0.

### This is a fork that

* patches (and bundles) Bouncy Castle to make it work with S40 Nokia phones
* Makes it possible to send HTTP bodies (for POST requests)
* comes with the Netbeans obfuscator plugin (and the correct ProGuard version) that is necessary
* updates the Readme to save you hours of research that I had to go through
* backports fixes for all known security issues.

# Setup

I use [JDK 1.7](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html) with [Netbeans 7.4](https://netbeans.org/downloads/7.4/), [ProGuard 5.0](https://sourceforge.net/projects/proguard/files/proguard/5.0/) and Sun's [Wireless Toolkit 2.5.2](https://www.oracle.com/technetwork/java/download-135801.html).

Install WTK, Netbeans and the obfuscator plugin (you find it in this repository). However, you have to download ProGuard 5 and replace the file `<netbeans-install-dir\>/mobility/external/proguard/proguard4.4.jar` with version 5 (but rename it to proguard4.4.jar) because of compatibility issues.

Create a J2ME project and copy the files in `net.wstech2.me.lib-j2me-https-client/src` from this repository into the `src` folder of your Netbeans project.

Add the folder `res` from this repository to your project ressources. In this folder come the certificates (a Let's Encrypt cert in the correct format and file name is already there). Careful: That's `res/res/certs/xxx.der` (two res folders).

In the properties, activate obfuscation and set it to 9 (Maximum).

Create a MIDlet to test the HTTPS connection. You can use a code block along these lines:

```Java

    String c = fetchUrl("letsencrypt.org", 443);

    public String fetchUrl(String url, int port) {
        HttpsConnection connection = new HttpsConnectionImpl(url, port, "");
        ((HttpsConnectionImpl)(connection)).setAllowUntrustedCertificates(true); // has an effect?

        // basic auth if you want
        connection.setRequestProperty("Authorization", "Basic "+ Base64.toBase64String("user:password".getBytes()));

        // POST data if you want
        String d = "{\"cmd\":\"version\"}";
        connection.setRequestProperty("Content-Length", String.valueOf(d.getBytes().length));
        connection.setRequestMethod("POST"); // default: GET
        ((HttpsConnectionImpl)connection).setRequestBody(d);

        System.out.println("Response Message: " + connection.getResponseMessage());
        String response = getResponse(connection.openInputStream());
        System.out.println("Response: " + response);

        connection.close();
    }


    private String getResponse(InputStream in) throws IOException {
        StringBuffer retval = new StringBuffer();
        byte[] content = new byte[5];

        int read = 0;
        while ((read = in.read(content)) != -1) {
        	// this is for testing purposes only
        	// an adequate solution should handle charsets here
        	retval.append(new String(content, 0, read));

        }
        return retval.toString();
    }
```


# Certificates
Finally, in order to validate a website certificate it is required to create a DER-formatted file in
the MidLet's JAR containing the C.A.'s certificate used to sign the website or the certificate
itself if it is a self-signed one.
IMPORTANT: The file name must follow exactly the content set for the "issuer" attribute from the
issued certificate or the "CN" segment, if available, from the same attribute. For example, at the
client-samples we are testing our client against the website https://www.google.ca:443/.
In this case, the root CA "C=US,O=Equifax,OU=Equifax Secure Certificate Authority" is at the
highest level of the signing hierarchy used by google, as indicated by the "issuer" attribute of its
immediate child. Therefore, we have to place its certificate in a file called
"C=US,O=Equifax,OU=Equifax Secure Certificate Authority.der" in the folder "/res/certs" of the Midlet JAR.

A sample certificate/key pair, with the respective C.A. certificate, is packaged along with
the sample codes.
For this certificate the issuer subject is identified as
"C=BR, ST=Parana, O=j2metest company, OU=j2metest unit name, CN=j2metest.local-ca".
Hence, its certificate can stored in a file named either
"C=BR, ST=Parana, O=j2metest company, OU=j2metest unit name, CN=j2metest.local-ca.der"
or "j2metest.local-ca.der".

<b>IMPORTANT: These files are for test purposes only, they should not be applied in production
environments.</b>

For more comprehensive details about the library usage and capabilities, please check the project
net.wstech2.me.lib-j2me-https-client-samples.

# Restrictions
I guess you will not officially sign your application to make it trusted (you can only rely on baked in certificates for this and I guess a DigiCert certificate is too much money for this kind of vintage software development).

However, unsigned apps have restrictions. My Nokia does not let me connect to ports < 1024. This custom HTTPSConnection implementation helps me to circumvent certificate problems - but really to connect to a TLS web server, I have to have it running on port 2000 or so.

# Security

This is more or less retro programming. [Bouncy Castle 1.51 has security issues](https://nvd.nist.gov/vuln/search/results?adv_search=true&cves=on&cpe_version=cpe%3A%2Fa%3Abouncycastle%3Alegion-of-the-bouncy-castle-java-crytography-api%3A1.51).
   Fixes are backported, but still use with caution.
