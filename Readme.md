# Grist SAML SSO with KeyCloak

The Latest version of Grist Support SAML Authentication with Grist Core. SAML is one of the most mature SSO Protocol Out there. Its been an industry standard for years, and widely used across Large Organizations.
  
The Configuration for SAML is documented throughly here https://support.getgrist.com/install/saml/ 

But the configuration for Keycloak as Identity Provider is not well documented, which is exactly what we are going to discuss and walkthrough here.

## Keycloak Setup

For the tutorial we are going to run Keycloak with docker in dev mode.

```
docker run -d --name keycloak -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:20.0.1 start-dev
```

After this command keycloak will be up and running and availabe at http://localhost:8080 with admin username and password `admin/admin`

## Configure Keycloak

- Create a new Realm named `grist` in keycloak

![create-realm](create-realm.png)

- Add a User for Grist Realm with email `admin@example.com`

![create-user](create-user.png)

- Configure a User Password

- Configure User as Realm Admin

- Create SAML Client, use Grist SAML metadata url as client ID (i.e https://<grist-host>/saml/metadata.xml), for our demo it will be http://localhost:8484/saml/metadata.xml

## Configure SAML Client:

General Settings:

- Home URL: http://localhost:8484

- Valid redirect URIs: `http://localhost:8484/*`

- Master SAML Processing URL: `http://localhost:8484/saml/assert`

Protocol mappers are needed; the builtin mappers work, but their SAML attribute names should be changed to work with SAML2-js:

    givenName: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
    surname: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname
    email: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress

For keycloak its rather simple.

- Go to Client Scopes Tab in client configuration

- Select http://localhost:8484/saml/metadata.xml-dedicated

- Select add predefined mappers, and select `X500 email`, `X500 givenName`, `X500 surname`, and add.

Getting Server Certificate.

- Copy the server certificate content from SAML metadata url of the Realm at http://localhost:8080/realms/grist/protocol/saml/descriptor

- Paste it here https://www.samltool.com/format_x509cert.php, click on Format and copy the content of `X.509 cert with header` output



## Grist SAML Config

Following are the list of Environment Variables required for SAML Configuration.

- `GRIST_SAML_SP_HOST` - this is just the base URL of the Grist site, such as https://<grist-domain> (when SAML is active, there will be a /saml/assert endpoint available here for implementing the protocol).
- `GRIST_SAML_SP_KEY` - path to a file with our private key, in PEM format. This is the private key of the key pair created for Grist to use with the IdP.
- `GRIST_SAML_SP_CERT` - path to file with our public key, in PEM format. This is the public key of the key pair created for Grist to use with the IdP. It is not the public key/certificate of the IdP.
- `GRIST_SAML_IDP_LOGIN` - login url to redirect user to for log-in.
- `GRIST_SAML_IDP_LOGOUT` - logout URL to redirect user to for log-out.
- `GRIST_SAML_IDP_SKIP_SLO` - if set and non-empty, don’t attempt “Single Logout” SAML flow, but simply redirect to GRIST_SAML_IDP_LOGOUT after clearing session. Whether this flow is possible will depend on the IdP.
- `GRIST_SAML_IDP_CERTS` - comma-separated list of paths for certificates from the IdP, in PEM format. This is not the private or public key created for Grist.
- `GRIST_SAML_IDP_UNENCRYPTED` - if set and non-empty, allow unencrypted assertions, relying on https for privacy.



- Generate SP Certificate and Key
```
openssl req -newkey rsa:2048 -nodes  -keyout key.pem -x509 -days 3650 -out certificate.pem
```