# Additional configuration

When using SSO, look at the documentation to view how to set it up with SAML. After that, download the provider metadata file and save it as `joplin-idp.xml`. After that, create a file called `joplin-sp.xml` with the following contents, replacing `joplin.company` with your joplin URL and `<application_slug>` with your application slug / entity ID:

```xml
<?xml version="1.0"?>
<md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" entityID="<application_slug>">
  <md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <md:NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</md:NameIDFormat>
    <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
      Location="https://joplin.company/api/saml"
      index="1" />
  </md:SPSSODescriptor>
</md:EntityDescriptor>
```

Put these files at the root of the `joplin-config` volume, which is usually located at `/var/lib/containers/storage/volumes/joplin-config/_data/`. Make sure `SAML_ENABLED` is set to true and that the APP and API url are the same.

# Sources

<https://github.com/laurent22/joplin/blob/dev/.env-sample>

<https://hub.docker.com/_/postgres>

<https://discourse.joplinapp.org/t/breaking-change-on-mailer-configuration-in-joplin-server-2-7/23464/13>

<https://integrations.goauthentik.io/chat-communication-collaboration/joplin/>
