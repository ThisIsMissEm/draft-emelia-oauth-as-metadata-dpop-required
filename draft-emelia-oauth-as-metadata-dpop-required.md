---
title: "OAuth Authorization Server Metadata Parameter for Requiring DPoP"
abbrev: "AS Metadata for requiring DPoP"
category: std

docname: draft-emelia-oauth-as-metadata-dpop-required-latest
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
  github: "ThisIsMissEm/draft-emelia-oauth-as-metadata-dpop-required"
  latest: "https://drafts.thisismissem.social/draft-emelia-oauth-as-metadata-dpop-required/draft-emelia-oauth-as-metadata-dpop-required.html"

author:
  - fullname: Emelia Smith
    email: emelia@brandedcode.com
    uri: https://thisismissem.social

normative:
  RFC8414:
  RFC9126:
  RFC9449:

informative:
  RFC6749:
  RFC9728:

entity:
  SELF: "This document"

--- abstract

This specification defines a `dpop_bound_access_tokens_required` metadata parameter for the OAuth Authorization Server Metadata ({{RFC8414}}), which allows an authorization server to indicate that it always issues DPoP-bound access tokens and requires DPoP ({{RFC9449}}) when interacting with the authorization server, such as at the pushed authorization request endpoint ({{RFC9126}}).

This matches the metadata parameter of the same name defined for protected resources in OAuth 2.0 Protected Resource Metadata ({{RFC9728}}).

--- middle

# Introduction

An OAuth 2.0 ({{RFC6749}}) client can determine from the `dpop_signing_alg_values_supported` authorization server metadata parameter (Section 5.1 of {{RFC9449}}) whether an authorization server supports DPoP, but not whether the authorization server requires it.

Without this information, a client can only discover that DPoP is required by sending a request without DPoP, receiving an error, and retrying the request with DPoP. {{RFC9449}} does not define a specific error for a request that is missing a required DPoP proof, so the client cannot reliably distinguish this case from other failures. Additionally, a client that does not support DPoP has no way to determine in advance that it cannot complete an authorization flow with the authorization server, and may only discover this after the user has completed the authorization request.

OAuth 2.0 Protected Resource Metadata ({{RFC9728}}) registered a `dpop_bound_access_tokens_required` parameter in the IANA "OAuth Protected Resource Metadata" registry established by that specification; however, no similar parameter was added to the "OAuth Authorization Server Metadata" registry established by OAuth 2.0 Authorization Server Metadata ({{RFC8414}}). This specification defines that metadata parameter for authorization servers.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Authorization Server Metadata

The metadata parameter `dpop_bound_access_tokens_required` is an optional Boolean value specifying whether the authorization server always issues DPoP-bound access tokens ({{RFC9449}}), and requires DPoP for all requests made to it to obtain access tokens. The default value is `false`.

If `true`:

- the authorization server MUST NOT issue access tokens that are not DPoP-bound, and MUST reject token requests that do not include a DPoP proof (Section 5 of {{RFC9449}}).
- the authorization server MUST also publish the `dpop_signing_alg_values_supported` metadata parameter (Section 5.1 of {{RFC9449}}).
- the client MUST include a DPoP proof in token requests (Section 5 of {{RFC9449}}).
- if the client uses pushed authorization requests ({{RFC9126}}), the client MUST bind them to a DPoP key (Section 10.1 of {{RFC9449}}), and the authorization server MUST reject pushed authorization requests that are not bound to a DPoP key.

A client that does not support DPoP SHOULD NOT initiate an authorization request with an authorization server that advertises a value of `true`.

This metadata parameter applies to all clients of the authorization server. A requirement that a specific client always uses DPoP is instead expressed using the `dpop_bound_access_tokens` client metadata parameter (Section 5.2 of {{RFC9449}}).

Unlike the metadata parameter of the same name for protected resources ({{RFC9728}}), which indicates that a protected resource only accepts DPoP-bound access tokens, this metadata parameter also covers the requests a client makes to the authorization server to obtain access tokens.

Since access tokens issued by such an authorization server are DPoP-bound, clients present them using the DPoP authentication scheme (Section 7 of {{RFC9449}}). As a result, these access tokens can only be used with resource servers that support DPoP; a resource server that does not support DPoP will reject such requests.

## Example

The following is a non-normative example of authorization server metadata that includes the `dpop_bound_access_tokens_required` metadata parameter:

~~~ json
{
  "issuer": "https://as.example.com",
  "authorization_endpoint": "https://as.example.com/authorize",
  "token_endpoint": "https://as.example.com/token",
  "pushed_authorization_request_endpoint": "https://as.example.com/par",
  "dpop_signing_alg_values_supported": ["ES256", "EdDSA"],
  "dpop_bound_access_tokens_required": true
}
~~~

# Security Considerations

The security considerations of {{RFC9449}} apply to this specification.

The `dpop_bound_access_tokens_required` parameter informs clients of the authorization server's requirements; it does not enforce them. Requests from clients that ignore or do not retrieve the authorization server metadata are rejected by the authorization server, as described in {{authorization-server-metadata}}.

A DPoP-bound access token only provides protection against token replay if the resource server verifies the binding. An attacker that obtains a DPoP-bound access token may attempt to present it using the Bearer authentication scheme to a resource server that does not verify the `cnf` claim of the access token. Resource servers that support both the Bearer and DPoP authentication schemes need to reject DPoP-bound access tokens presented using the Bearer authentication scheme, as described in Section 7.2 of {{RFC9449}}. This specification does not change those requirements.

# IANA Considerations

## OAuth Authorization Server Metadata Registry

The following authorization server metadata value is defined by this specification and registered in the IANA "OAuth Authorization Server Metadata" registry established in OAuth 2.0 Authorization Server Metadata ({{RFC8414}}).

- Metadata Name: dpop_bound_access_tokens_required
- Metadata Description: Boolean value specifying whether the authorization server always issues DPoP-bound access tokens and requires DPoP for all requests to its pushed authorization request endpoint and token endpoint
- Change Controller: IETF
- Specification Document: {{&SELF}}

--- back

# Acknowledgments
{:numbered="false"}

The author would like to thank Max Gerber (Twilio) for identifying the gap in OAuth Authorization Server Metadata that this document addresses.

# Document History
{:numbered="false"}

(This appendix to be deleted by the RFC editor in the final specification.)

## -00
{:numbered="false"}

Initial draft
