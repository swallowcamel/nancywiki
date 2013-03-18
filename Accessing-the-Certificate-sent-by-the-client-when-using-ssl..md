To authenticate the client the client can send a certificate. To do this in Nancy you need one of three hosting solutions: `Aspnet`, `WCF` or `OWIN`. Here is shown howto configure all three to work with ssl and client certificates.

## Configuration of `Aspnet`.

If the `web.config` file we need to specify we want to be able to receive a ClientCertificate. Like this:

```xml
<security>
  <access sslFlags="SslNegotiateCert"/>
  <authentication>
    <clientCertificateMappingAuthentication enabled="true"/>
  </authentication>
</security>
```

You may get an error telling you this:

> This configuration section cannot be used at this path. This happens when the section is locked at a parent level. Locking is either by default (overrideModeDefault="Deny"), or set explicitly by a location tag with overrideMode="Deny" or the legacy allowOverride="false".

This is solved by editing your applicationhost.config and setting the `overrideModeDefault` to `Allow` for the following elements.

```xml
<section name="access" overrideModeDefault="Deny" />
<section name="clientCertificateMappingAuthentication" overrideModeDefault="Allow" />
```

See [here](http://www.microsoft.com/web/post/securing-web-communications-certificates-ssl-and-https) how to enable ssl for IISexpress.

See [here](http://www.iis.net/learn/manage/configuring-security/how-to-set-up-ssl-on-iis) how to do it on IIS.

##Configuration of `WCF`