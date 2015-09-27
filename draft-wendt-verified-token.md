
### Verified Token

**draft-wendt-verified-token-00**

**Abstract**

This memo defines a token format for securing verifiable information where an originator of information needs to cryptographically prove to a terminating party that the originator is both an authorized sender and the information wasn’t altered or modified in transit.  This verified information could include network identity, device identity, realm of origin, and other metadata that could cross both trusted and untrusted or unknown points in a network.  Verification of this information in the telephony world is important for securing telephone calls end-to-end and can be utilized as an important tool for combat spoofing of identity and other forms of impersonation.

**1. Introduction**

This document will define a method for the creation and verification of an extensible canonical token that cryptographically represents a particular set of information, originator the information, and can chain the originator to a trust anchor.  A goal of this approach is to be implementable in a straight-forward, extensible way.  A second goal is to be separable from any specific signaling call logic, so creation and verification of information can be implemented in a flexible way with minimal dependence on specific signaling constructs.  A third goal is to utilize as much as possible existing technologies and infrastructure dependency to allow flexible deployment strategies.  Deployment specifics will be out of scope for this document to allow industry specific solutions for managing PKI and other dependencies.

**1.1. Conventions used in this document**

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].

**2. Token Overview**

Tokens are a convenient way of encapsulating information with associated cryptographic signatures.  They are used in many applications that require authentication, authorization, encryption and other use cases that involve digital signatures.  JWT [RFC7519] and JWS [RFC7515] are designed to provide a compact form for many of these purposes and define a specific form for specifying information associated with the token and an extensible mechanisms for applying digital signatures and the cryptographic algorithms used.  JWT has the form "header.claim.signature" and JWS provides standard ways to form a digital signature.  Note: In this document, we will focus on digital signatures, but JWT and JWS also support HMAC symmetric key based algorithms as well.

**3. Verified Token (VT)**

The Verify Token (VT) is constructed based on JWT [RFC7519] and JWS [RFC7515] specifications.  JWS defines the use of JSON data structures in a specified canonical format for signing data coresponding to JOSE header, JWS Payload, and JWS Signature.  JWT defines specific set of claims that are represented by specified key value pairs which can be extended with custom keys for specific applications. 

**3.1. Verified Token Header**

The JWS token header is a JOSE header that defines the type and encryption algoirthm used in the token.  An example of the header for the case of a RSASSA-PKCS1-v1_5 SHA-256 digital signature would be the following,

	{ "typ":"JWT",
      "alg":"RS256"}

This represents that the encoded token is a JWT, and the JWT is a JWS using the RSASSA-PKCS1-v1_5 SHA-256 algorithm.  

For Verified Token the "typ" header MUST be "JWT".

Because Verified Token is defined for use with PKI based digital signatures, the "alg" header is recommended to be one of the following algorithms as defined in JWA [RFC7518]:
 
 * RS256
 * ES256

The "alg" header could optionally be one of the following:
 
 * RS384
 * RS512
 * ES384
 * ES512


**3.2. Verified Token Claim**

The token claim should consist of the information which needs to be verified at the terminating party.  This claim should correspond to a JWT claim [RFC7519] and be encoded as defined by the JWS Payload [RFC7519]. 

The Verified Token should use a number of standard defined token headers as well as some addition custom headers specifically required for two party communications with an originator and terminator.  These headers or key value pairs will be explained here, but some of the security implications will be explained further in the security considerations section below.

The JSON claim MUST include the following registered JWT defined claims unless noted optional:

* "iss" - required - principal that issued and signed the JWT.  This is an https URL with the domain of the authorized originator of the token (e.g. "https://pstn.example.com)
* "jti" - required - unique identifier of the JWT, useful for both tracking and avoiding replay of JWT
* "exp" - optional - expiration time, typically used with an application specific leeway on success of verification of token (e.g. 30 seconds)
* "iat" - required - issued at, time the JWT was issued, used for expiration

Verified Token specific claims MUST be included unless noted optional:

* "orig" - the identity claimed by the originating party.  (e.g. for SIP, the FROM or PAI associated e.164 telephone number, TEL or SIP URI)  This MAY be in URI format as defined in [RFC3986] or an application specific identity string.
* "term" - the terminating identity claimed by the originating party. (e.g. for SIP, the TO associated e.164 telephone number, TEL or SIP URI)  This MAY be in URI format as defined in [RFC3986] or an application specific identity string.

NOTE: for identities such as SIP URIs where there is a domain associated with a user part, i.e. user@domain, the domain in the claim, may or may not correspond to either the TO/FROM/PAI and/or the "iss".  The determination of the validity of the claimed identity, either user part alone or the full user@domain would be up to the application.

An example claim is as follows,

	{ "orig":"+12155551212",
	  "term":"sip:+12155551213@example.com",
      "iss":"https://pstn.example.com",
      "jti": "FAhNaPk0onffyJvykJZC2A==",
      "iat": 1443208345 }
 
**4. Verified Token Signature**



**4. Acknowledgements**

   Would like to thank members of the ATIS and SIP Forum NNI Task Group
   for feedback encouragement particularly ...

**5. IANA Considerations**

   This memo includes no request to IANA.

**6. Security Considerations**

   Security Considerations

**7. Normative References**

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119,
              DOI 10.17487/RFC2119, March 1997,
              <http://www.rfc-editor.org/info/rfc2119>.

   [RFC7515]  Jones, M., Bradley, J., and N. Sakimura, "JSON Web
              Signature (JWS)", RFC 7515, DOI 10.17487/RFC7515, May
              2015, <http://www.rfc-editor.org/info/rfc7515>.

   [RFC7519]  Jones, M., Bradley, J., and N. Sakimura, "JSON Web Token
              (JWT)", RFC 7519, DOI 10.17487/RFC7519, May 2015,
              <http://www.rfc-editor.org/info/rfc7519>.

**Appendix A.  Example Tokens**

   Provide some example tokens

**Author's Address**

   Chris Wendt (editor)
   Comcast
   One Comcast Center
   Philadelphia, PA
   US

   Phone: +1-215-286-7093
   Email: chris_wendt@cable.comcast.com
