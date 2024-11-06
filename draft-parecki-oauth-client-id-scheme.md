---
title: "OAuth 2.0 Client ID Scheme"
abbrev: "Client ID Scheme"
category: info

docname: draft-parecki-oauth-client-id-scheme-latest
submissiontype: IETF
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
  RFC6749:
  RFC5280:
  RFC7515:
  DID-Core:
    title: "DID Core"
    target: https://www.w3.org/TR/did-core/
    date: 2022-07-19
  OpenID.Federation:
    title: "OpenID Federation 1.0"
    date: 2024-05-17
    target: https://openid.net/specs/openid-federation-1_0.html
    author:
      - name: R. Hedberg
        org: independent
      - name: M.B. Jones
        org: Self-Issued Consulting
      - name: A.Å. Solberg
        org: Sikt
      - name: J. Bradley
        org: Yubico
      - name: G. De Marco
        org: independent
      - name: V. Dzhuvinov
        org: Connect2id

informative:
  RFC7591:
  RFC9068:
  I-D.draft-parecki-oauth-client-id-metadata-document:
  OpenID:
    title: "OpenID Connect Core 1.0"
    date: 2023-12-15
    target: https://openid.net/specs/openid-connect-core-1_0.html
    author:
      - name: N. Sakimura
        org: NAT.Consulting
      - name: J. Bradley
        org: Yubico
      - name: M. Jones
        org: Self-Issued Consulting
      - name: B. de Medeiros
        org: Google
      - name: C. Mortimore
        org: Disney


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

## Syntax

In the `client_id` Authorization Request parameter and other places where the Client Identifier is used, the Client Identifier Schemes are prefixed to the usual Client Identifier, separated by a `:` (colon) character:

    <client_id_scheme>:<orig_client_id>

Here, `<client_id_scheme>` is the Client Identifier Scheme and `<orig_client_id>` is an identifier for the Client within the namespace of that scheme. See {{client_identifier_schemes}} for Client Identifier Schemes defined by this specification.

Authorization Servers MUST use the presence of a `:` (colon) character to determine whether a Client Identifier Scheme is used. If a `:` character is present, the Authorization Server MUST interpret the Client Identifier according to the Client Identifier Scheme, here defined as the string before the (first) `:` character. If the Authorization Server does not support the Client Identifier Scheme, the Authorization Server MUST refuse the request.

For example, an Authorization Request might contain `client_id=client_attestation:example-client` to indicate that the `client_attestation` Client Identifier Scheme is to be used and that within this scheme, the Client can be identified by the string `example-client`. The presentation would contain the full `client_attestation:example-client` string as the audience (intended receiver) and the same full string would be used as the Client Identifier anywhere in the OAuth flow.

Note that the Client needs to determine which Client Identifier Schemes the Authorization Server supports prior to sending the Authorization Request in order to choose a supported scheme.


## Pre-Registered Clients

If a `:` character is not present in the Client Identifier, the Authorization Server MUST treat the Client Identifier as referencing a pre-registered client. This is equivalent to the {{RFC6749}} default behavior, i.e., the Client Identifier needs to be known to the Authorization Server in advance of the Authorization Request. The Client metadata is pre-registered using {{RFC7591}} or through out-of-band mechanisms.

For example, if an Authorization Request contains `client_id=example-client`, the Authorization Server would interprete the Client Identifier as referring to a pre-registered client.

From this definition, it follows that pre-registered clients MUST NOT contain a `:` character in their Client Identifier.


## Pre-Existing Ecosystems using https

Existing deployments that use `https` URLs as client IDs and that have only one way to resolve client metadata from the URL, MAY use full https URL as the client ID. If there is only one way to resolve client metadata then there is no ambiguity.

For example, an authorization server using only the Client ID Metadata Document {{I-D.draft-parecki-oauth-client-id-metadata-document}} method to retrieve client metadata MAY accept client IDs such as:

    https://client.example.com/metadata.json

This results in this non-normative example of an authorization request:

    GET /authorize?
      response_type=code
      &client_id=https%3A%2F%2Fclient.example.org%2Fmetadata.json
      &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcallback
      &code_challenge=GdE4nqBrwRxQfN2Y8fq3rrYk_kkpwg6tQ74J94-2nHw
      &code_challenge_method=S256
      &scope=write


## Defined Client Identifier Schemes {#client_identifier_schemes}

This specification defines the following Client Identifier Schemes, followed by the examples where applicable:

* `redirect_uri`: This value indicates that the Client Identifier (without the prefix `redirect_uri:`) is the Client's Redirect URI (or Response URI when Response Mode `direct_post` is used). The Authorization Request MUST NOT be signed. The Client MAY omit the `redirect_uri` Authorization Request parameter. Example Client Identifier: `redirect_uri:https%3A%2F%2Fclient.example.org%2Fcb`.

* `federation`: This value indicates that the Client Identifier is an Entity Identifier defined in OpenID Federation {{OpenID.Federation}}. Since the Entity Identifier is already defined to start with `federation:`, this Client Identifier Scheme MUST NOT be prefixed additionally. Processing rules given in {{OpenID.Federation}} MUST be followed. Automatic Registration as defined in {{OpenID.Federation}} MUST be used. The Authorization Request MAY also contain a `trust_chain` parameter. The final Client metadata is obtained from the Trust Chain after applying the policies, according to {{OpenID.Federation}}. Example Client Identifier: `federation:https://federation-client.example.com`.

* `did`: This value indicates that the Client Identifier is a DID defined in {{DID-Core}}. Since the DID URI is already defined to start with `did:`, this Client Identifier Scheme MUST NOT be prefixed additionally. The request MUST be signed with a private key associated with the DID. A public key to verify the signature MUST be obtained from the `verificationMethod` property of a DID Document. Since DID Document may include multiple public keys, a particular public key used to sign the request in question MUST be identified by the `kid` in the JOSE Header. To obtain the DID Document, the Authorization Server MUST use DID Resolution defined by the DID method used by the Client. Example Client Identifier: `did:example:123#1`.

* `client_attestation`: This Client Identifier Scheme allows the Client to authenticate using a JWT that is bound to a certain public key as defined in (OpenID4VP: Client Attestation). When the Client Identifier Scheme is `client_attestation`, the Client Identifier MUST equal the `sub` claim value in the Client attestation JWT. The request MUST be signed with the private key corresponding to the public key in the `cnf` claim in the Client attestation JWT. This serves as proof of possesion of this key. The Client attestation JWT MUST be added to the `jwt` JOSE Header of the request object (see (OpenID4VP: Client Attestation)). The Authorization Server MUST validate the signature on the Client attestation JWT. The `iss` claim value of the Client Attestation JWT MUST identify a party the Authorization Server trusts for issuing Client Attestation JWTs. If the Authorization Server cannot establish trust, it MUST refuse the request. If the issuer of the Client Attestation JWT adds a `redirect_uris` claim to the attestation, the Authorization Server MUST ensure the `redirect_uri` request parameter value exactly matches one of the `redirect_uris` claim entries. Example Client Identifier: `client_attestation:client.example`.

* `x509_san_dns`: When the Client Identifier Scheme is `x509_san_dns`, the Client Identifier MUST be a DNS name and match a `dNSName` Subject Alternative Name (SAN) {{RFC5280}} entry in the leaf certificate passed with the request. The request MUST be signed with the private key corresponding to the public key in the leaf X.509 certificate of the certificate chain added to the request in the `x5c` JOSE header {{RFC7515}} of the signed request object. The Authorization Server MUST validate the signature and the trust chain of the X.509 certificate. If the Authorization Server can establish trust in the Client Identifier authenticated through the certificate, e.g. because the Client Identifier is contained in a list of trusted Client Identifiers, it may allow the client to freely choose the `redirect_uri` value. If not, the FQDN of the `redirect_uri` value MUST match the Client Identifier without the prefix `x509_san_dns:`. Example Client Identifier: `x509_san_dns:client.example.org`.

* `x509_san_uri`: When the Client Identifier Scheme is `x509_san_uri`, the Client Identifier MUST be a URI and match a `uniformResourceIdentifier` Subject Alternative Name (SAN) {{RFC5280}} entry in the leaf certificate passed with the request. The request MUST be signed with the private key corresponding to the public key in the leaf X.509 certificate of the certificate chain added to the request in the `x5c` JOSE header {{RFC7515}} of the signed request object. The Authorization Server MUST validate the signature and the trust chain of the X.509 certificate. If the Authorization Server can establish trust in the Client Identifier authenticated through the certificate, e.g., because the Client Identifier is contained in a list of trusted Client Identifiers, it may allow the client to freely choose the `redirect_uri` value. If not, the `redirect_uri` value MUST match the Client Identifier without the prefix `x509_san_uri:`. Example Client Identifier: `x509_san_uri:https://client.example.org/cb`.

* `https`: This Client Identifier Scheme MUST NOT be registered.


# Example

The following is a non-normative example of an authorization request with the `redirect_uri` Client ID Scheme:

    GET /authorize?
      response_type=code
      &client_id=redirect_uri:https%3A%2F%2Fclient.example.org%2Fcb
      &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
      &code_challenge=GdE4nqBrwRxQfN2Y8fq3rrYk_kkpwg6tQ74J94-2nHw
      &code_challenge_method=S256
      &scope=write

# Todo: Server Metadata

e.g., a `client_id_schemes_supported` parameter in the Server Metadata and a `client_id_scheme_default` parameter.

# Todo: Registry


# Security Considerations

## Client Identifier Mixups

Confusing Clients using a Client Identifier Scheme with those using none can lead to attacks. Therefore, Authorization Servers MUST always use the full Client Identifier, including the prefix if provided, within the context of the Authorization Server or its responses to identify the client. This refers in particular to places where the Client Identifier is used in {{RFC6749}} as well as in any artifacts such as the `aud` claim of JWT access tokens {{RFC9068}}.



# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
