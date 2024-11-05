---
title: "OAuth 2.0 Client ID Scheme"
abbrev: "Client ID Scheme"
category: info

docname: draft-parecki-oauth-client-id-scheme-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Web Authorization Protocol"
keyword:
 - oauth
venue:
  group: "Web Authorization Protocol"
  type: "Working Group"
  mail: "oauth@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/oauth/"
  github: "aaronpk/oauth-client-id-scheme"
  latest: "https://drafts.aaronpk.com/oauth-client-id-scheme/draft-parecki-oauth-client-id-scheme.html"

author:
  -
    fullname: "Aaron Parecki"
    organization: Okta
    email: "aaron@parecki.com"
  -
    fullname: "Daniel Fett"
    organization: Authlete
    email: "mail@danielfett.de"
  -
    fullname: "Joseph Heenan"
    organization: Authlete
    email: "joseph@heenan.me.uk"

normative:
  - RFC6749:

informative:
  - RFC7591:


--- abstract

This specification defines the concept of a Client Identifier Scheme to enable Authorization Servers and Clients to use more than one mechanism to obtain and validate Client metadata.

--- middle

# Introduction

A Client Identifier is used by an OAuth 2.0 Client to identify itself to an Authorization Server. The Client Identifier is used in the Authorization Request and various other places throughout OAuth flows. In ecosystems where more than one method of obtaining and validating Client metadata is used, it is necessary to indicate unambiguously which method is used. This specification defines a structure for Client Identifiers that includes a prefix indicating the Client Identifier Scheme.

# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Client Identifier Scheme

This specification defines the concept of a Client Identifier Scheme that indicates how an Authorization Server is supposed to interpret the Client Identifier and associated data in the process of Client identification, authentication, and authorization. The Client Identifier Scheme enables deployments of this specification to use different mechanisms to obtain and validate metadata of the Client beyond the scope of {{RFC6749}}.

The Client Identifier Scheme is a string that MAY be communicated by the Client in a prefix within the `client_id` parameter in the Authorization Request. A fallback to pre-registered Clients as in {{RFC6749}} remains in place as a default mechanism in case no Client Identifier Scheme was provided. A certain Client Identifier Scheme may require the Client to sign the Authorization Request as means of authentication and/or pass additional parameters and require the Authorization Server to process them.

### Syntax

In the `client_id` Authorization Request parameter and other places where the Client Identifier is used, the Client Identifier Schemes are prefixed to the usual Client Identifier, separated by a `:` (colon) character:

```
<client_id_scheme>:<orig_client_id>
```

Here, `<client_id_scheme>` is the Client Identifier Scheme and `<orig_client_id>` is an identifier for the Client within the namespace of that scheme. See (#client_identifier_schemes) for Client Identifier Schemes defined by this specification.

Authorization Servers MUST use the presence of a `:` (colon) character to determine whether a Client Identifier Scheme is used. If a `:` character is present, the Authorization Server MUST interpret the Client Identifier according to the Client Identifier Scheme, here defined as the string before the (first) `:` character. If the Authorization Server does not support the Client Identifier Scheme, the Authorization Server MUST refuse the request.

For example, an Authorization Request might contain `client_id=Client_attestation:example-client` to indicate that the `Client_attestation` Client Identifier Scheme is to be used and that within this scheme, the Client can be identified by the string `example-client`. The presentation would contain the full `Client_attestation:example-client` string as the audience (intended receiver) and the same full string would be used as the Client Identifier anywhere in the OAuth flow.

Note that the Client needs to determine which Client Identifier Schemes the Authorization Server supports prior to sending the Authorization Request in order to choose a supported scheme.

Depending on the Client Identifier Scheme, the Client can communicate a JSON object with its metadata using the `client_metadata` parameter which contains name/value pairs.

### Fallback

If a `:` character is not present in the Client Identifier, the Authorization Server MUST treat the Client Identifier as referencing a pre-registered client. This is equivalent to the {{RFC6749}} default behavior, i.e., the Client Identifier needs to be known to the Authorization Server in advance of the Authorization Request. The Client metadata is obtained using {{RFC7591}} or through out-of-band mechanisms.

For example, if an Authorization Request contains `client_id=example-client`, the Authorization Server would interprete the Client Identifier as referring to a pre-registered client.

From this definition, it follows that pre-registered clients MUST NOT contain a `:` character in their Client Identifier.

### Todo: Exception for Federation and Client ID Metadata by Aaron

### Security Considerations

Confusing Clients using a Client Identifier Scheme with those using none can lead to attacks. Therefore, Authorization Servers MUST always use the full Client Identifier, including the prefix if provided, within the context of the Authorization Server or its responses to identify the client. This refers in particular to places where the Client Identifier is used in {{RFC6749}} and in the presentation returned to the Client.

### Defined Client Identifier Schemes {#client_identifier_schemes}

This specification defines the following Client Identifier Schemes, followed by the examples where applicable:

* `redirect_uri`: This value indicates that the Client Identifier (without the prefix `redirect_uri:`) is the Client's Redirect URI (or Response URI when Response Mode `direct_post` is used). The Authorization Request MUST NOT be signed. The Client MAY omit the `redirect_uri` Authorization Request parameter. Example Client Identifier: `redirect_uri:https%3A%2F%2Fclient.example.org%2Fcb`.

The following is a non-normative example of a request with this Client Identifier Scheme:

```
HTTP/1.1 302 Found
Location: https://client.example.org/universal-link?
  response_type=vp_token
  &client_id=redirect_uri:https%3A%2F%2Fclient.example.org%2Fcb
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
  &presentation_definition=...
  &nonce=n-0S6_WzA2Mj
  &client_metadata=%7B%22vp_formats%22:%7B%22jose_vp%22:%
  7B%22alg%22:%5B%22EdDSA%22,%22ES256K%22%5D%7D,%22di
  _vc%22:%7B%22proof_type%22:%5B%22DataIntegrityProof%22%5D,%22
  cryptosuite%22:%5B%22ecdsa-sd-2023%22%5D%7D%7D%7D
```

* `federation`: This value indicates that the Client Identifier is an Entity Identifier defined in OpenID Federation [@!OpenID.Federation]. Since the Entity Identifier is already defined to start with `federation:`, this Client Identifier Scheme MUST NOT be prefixed additionally. Processing rules given in [@!OpenID.Federation] MUST be followed. Automatic Registration as defined in [@!OpenID.Federation] MUST be used. The Authorization Request MAY also contain a `trust_chain` parameter. The final Client metadata is obtained from the Trust Chain after applying the policies, according to [@!OpenID.Federation]. The `client_metadata` parameter, if present in the Authorization Request, MUST be ignored when this Client Identifier scheme is used. Example Client Identifier: `federation:https://federation-Client.example.com`.

* `did`: This value indicates that the Client Identifier is a DID defined in [@!DID-Core]. Since the DID URI is already defined to start with `did:`, this Client Identifier Scheme MUST NOT be prefixed additionally. The request MUST be signed with a private key associated with the DID. A public key to verify the signature MUST be obtained from the `verificationMethod` property of a DID Document. Since DID Document may include multiple public keys, a particular public key used to sign the request in question MUST be identified by the `kid` in the JOSE Header. To obtain the DID Document, the Authorization Server MUST use DID Resolution defined by the DID method used by the Client. All Client metadata other than the public key MUST be obtained from the `client_metadata` parameter as defined in (#new_parameters). Example Client Identifier: `did:example:123#1`.

* `Client_attestation`: This Client Identifier Scheme allows the Client to authenticate using a JWT that is bound to a certain public key as defined in (OpenID4VP: Client Attestation). When the Client Identifier Scheme is `Client_attestation`, the Client Identifier MUST equal the `sub` claim value in the Client attestation JWT. The request MUST be signed with the private key corresponding to the public key in the `cnf` claim in the Client attestation JWT. This serves as proof of possesion of this key. The Client attestation JWT MUST be added to the `jwt` JOSE Header of the request object (see (OpenID4VP: Client Attestation)). The Authorization Server MUST validate the signature on the Client attestation JWT. The `iss` claim value of the Client Attestation JWT MUST identify a party the Authorization Server trusts for issuing Client Attestation JWTs. If the Authorization Server cannot establish trust, it MUST refuse the request. If the issuer of the Client Attestation JWT adds a `redirect_uris` claim to the attestation, the Authorization Server MUST ensure the `redirect_uri` request parameter value exactly matches one of the `redirect_uris` claim entries. All Client metadata other than the public key MUST be obtained from the `client_metadata` parameter. Example Client Identifier: `Client_attestation:Client.example`.

* `x509_san_dns`: When the Client Identifier Scheme is `x509_san_dns`, the Client Identifier MUST be a DNS name and match a `dNSName` Subject Alternative Name (SAN) [@!RFC5280] entry in the leaf certificate passed with the request. The request MUST be signed with the private key corresponding to the public key in the leaf X.509 certificate of the certificate chain added to the request in the `x5c` JOSE header [@!RFC7515] of the signed request object. The Authorization Server MUST validate the signature and the trust chain of the X.509 certificate. All Client metadata other than the public key MUST be obtained from the `client_metadata` parameter. If the Authorization Server can establish trust in the Client Identifier authenticated through the certificate, e.g. because the Client Identifier is contained in a list of trusted Client Identifiers, it may allow the client to freely choose the `redirect_uri` value. If not, the FQDN of the `redirect_uri` value MUST match the Client Identifier without the prefix `x509_san_dns:`. Example Client Identifier: `x509_san_dns:client.example.org`.

* `x509_san_uri`: When the Client Identifier Scheme is `x509_san_uri`, the Client Identifier MUST be a URI and match a `uniformResourceIdentifier` Subject Alternative Name (SAN) [@!RFC5280] entry in the leaf certificate passed with the request. The request MUST be signed with the private key corresponding to the public key in the leaf X.509 certificate of the certificate chain added to the request in the `x5c` JOSE header [@!RFC7515] of the signed request object. The Authorization Server MUST validate the signature and the trust chain of the X.509 certificate. All Client metadata other than the public key MUST be obtained from the `client_metadata` parameter. If the Authorization Server can establish trust in the Client Identifier authenticated through the certificate, e.g., because the Client Identifier is contained in a list of trusted Client Identifiers, it may allow the client to freely choose the `redirect_uri` value. If not, the `redirect_uri` value MUST match the Client Identifier without the prefix `x509_san_uri:`. Example Client Identifier: `x509_san_uri:https://client.example.org/cb`.



# Example


The following is a non-normative example of an Authorization Request:

```
GET /authorize?
  response_type=vp_token
  &client_id=redirect_uri:https%3A%2F%2Fclient.example.org%2Fcb
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
  &presentation_definition=...
  &transaction_data=...
  &nonce=n-0S6_WzA2Mj HTTP/1.1
```

# Todo: Server Metadata

e.g., a `client_id_schemes_supported` parameter in the Server Metadata and a `client_id_scheme_default` parameter.

# Todo: Registry


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
