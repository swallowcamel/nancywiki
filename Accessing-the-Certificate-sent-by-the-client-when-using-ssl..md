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

Nothing is ever easy with `WCF` configuration, this is no exception. 

Lets start with the basic host:

```c#
var host = new WebServiceHost(
    new NancyWcfGenericService(new DefaultNancyBootstrapper()),
    BaseUri);
```

We need to tell the binding we want to use `Transport Security` and we need to tell it to expect a certificate from the client. We also need to tell `WCF` not to worry about whether the certificate is valid. Or at least determine ourselves what valid is.

###Binding

```C#
var binding = new WebHttpBinding();
binding.Security.Mode = WebHttpSecurityMode.Transport;
binding.Security.Transport.ClientCredentialType = HttpClientCredentialType.Certificate;
```

###Custom validation of the certificate.

```C#
public class Auth : X509CertificateValidator
{
    public override void Validate(System.Security.Cryptography.X509Certificates.X509Certificate2 certificate)
    {
        return;
    }
}
```

Tell the host where to find the `Validator`

```C#
host.Credentials.ClientCertificate.Authentication.CertificateValidationMode = System.ServiceModel.Security.X509CertificateValidationMode.Custom;
host.Credentials.ClientCertificate.Authentication.CustomCertificateValidator = new Auth();
```

###Endpoint

```C#
host.AddServiceEndpoint(typeof(NancyWcfGenericService),binding,"");
```
Tell the host where to find the server certificate:
```C#
host.Credentials.ServiceCertificate.SetCertificate(StoreLocation.LocalMachine, StoreName.My, X509FindType.FindByThumbprint, "30 3b 4a db 5a eb 17 ee ac 00 d8 57 66 93 a9 08 c0 1e 0b 71");
```


