
### Persona Assertion Token (PASSporT)

**draft-wendt-stir-passport-00**

**Abstract**

This memo defines a token format for verifying with non-repudiation the originator of a set of information.  This set of information, minimally the persona, in the context of this document represents the actor that either directly or indirectly represents the identity and associated data related to the entity that is originating the transmission of that information to another entity in a network.  For protecting the integrity of the transmission of this information, the originator uses a cryptographic signature generally with a key that embeds authorization from a trust anchor to prove to a terminating party that the originator is both authorized to assert the information including the persona of the sender and the information wasn't altered or modified in transit. Additionally, the token is extensible to incorporate other profiles of information related to the persona of the originator, other actors that may be part of the delivery of the token and other metadata.  In particular, verification of this information in the VoIP world is important for validating telephone calls and the telephone numbers they are presenting and can be utilized as an important tool for combat spoofing of identity and other forms of impersonation, but it is imagined that the PASSporT will be useful for many applications where there is an originating and terminating party that may not have a direct trust relationship.

**1. Introduction**

This document will define a method for the creation and verification of an extensible canonical token that is intended for cryptographically verifying both an originator, the validity of the originator, information associated with a specific transaction and an application specific a set of information through digital signatures and the associated chain of trust. The primary goal of PASSporT is to provide a framework for signing persona related information using a common, extensible approach. A second goal is for this token approach is to abstract from any specific signaling call logic, so creation and verification of information can be implemented in a flexible way with minimal dependence on specific signaling constructs.  There will be specifications that will provide signaling specific guidance on how to use and transport PASSporT tokens, but this is out of scope for this document.  Note: As of the authoring of this document, draft-ietf-stir-rfc4474bis provides details of how to use PASSporT within SIP signaling for the signing and verification of telephone numbers.

**2. Token Overview**

Tokens are a convenient way of encapsulating information with associated cryptographic signatures.  They are used in many applications that require authentication, authorization, encryption and other use cases that involve digital signatures.  JWT [RFC7519] and JWS [RFC7515] are designed to provide a compact form for many of these purposes and define a specific method and syntax for signing a specific set of information or "claims" within the token and therefore providing an extensible set of claims.  Additionally, JWS provides extensible mechanisms for specifying the method and cryptographic algorithms used for the associated digital signatures. 

**3. PASSporT Definition**

The PASSporT is constructed based on JWT [RFC7519] and JWS [RFC7515] specifications.  JWS defines the use of JSON data structures in a specified canonical format for signing data corresponding to JOSE header, JWS Payload, and JWS Signature.  JWT defines specific set of claims that are represented by specified key value pairs which can be extended with custom keys for specific applications. 

**3.1. PASSporT Header**

The JWS token header is a JOSE header that defines the type and encryption algorithm used in the token.  

An example of the header for the case of a RSASSA-PKCS1-v1_5 SHA-256 digital signature would be the following,

	{ "typ":"passport",
      "alg":"RS256",
      "x5u":"https://tel.example.org/passport.crt" }
      
**3.1.1 "typ" (Type) Header Parameter**
      
JWS defines the "typ" (Type) Header Parameter to declare the media type [IANA.MediaTypes] of the JWS. 

This represents that the encoded token is a JWT, and the JWT is a JWS using the RSASSA-PKCS1-v1_5 SHA-256 algorithm.  

For PASSporT Token the "typ" header MUST minimally be "passport".

**3.1.2 "alg" (Algorithm) Header Parameter**

Because PASSporT is defined for use with PKI based digital signatures, the "alg" header is recommended to be one of the following algorithms as defined in JWA [RFC7518]:
 
 * RS256
 * ES256

The "alg" header could optionally be one of the following:
 
 * RS384
 * RS512
 * ES384
 * ES512
 
For PASSporT, the recommended algorithm is RS256, but this may be updated in the future depending on cryptographic strength requirements guided by general accepted best practice.
 
**3.1.3 "x5u" (X.509 URL) Header Parameter**

As defined in JWS, the "x5u" header parameter is used to provide a URI [RFC3986] referring to the resource for the X.509 public key certificate or certificate chain [RFC5280] corresponding to the key used to digitally sign the JWS.  Note: The definition of what the URI represents in terms of the actor serving the X.509 public key is out of scope of this document.  However, generally this would correspond to an HTTPS or DNSSEC resource with the guidance that it MUST be a TLS protected, per JWS spec.

**3.2. PASSporT Token Claims**

The token claim should consist of the information which needs to be verified at the terminating party.  This claim should correspond to a JWT claim [RFC7519] and be encoded as defined by the JWS Payload [RFC7519]. 

The PASSporT should use a number of standard JWT defined headers as well as some addition custom headers specifically required for two party communications with an originator and terminator.  These headers or key value pairs will be explained here, but some of the security implications will be explained further in the security considerations section below.

The JSON claim MUST include the following registered JWT defined claims unless noted optional:

* "jti" - required - unique identifier of the JWT, useful for both tracking and avoiding replay of JWT
* "iat" - required - issued at, time the JWT was issued, used for expiration

Verified Token specific claims MUST be included unless noted optional:

* "oid" - required - the originating identity claimed.  (e.g. for SIP, the FROM or PAI associated e.164 telephone number, TEL or SIP URI)  This MAY be in URI format as defined in [RFC3986] or an application specific identity string.
* "tid" - required - the terminating identity claimed as the intended destination by the originating party. (e.g. for SIP, the TO associated e.164 telephone number, TEL or SIP URI)  This MAY be in URI format as defined in [RFC3986] or an application specific identity string.

An example claim is as follows,

	{ "oid":"+12155551212",
	  "tid":"sip:+12155551213@example.com",
      "jti": "FAhNaPk0onffyJvykJZC2A==",
      "iat": 1443208345 }
 
**3.3 Verified Token Signature**

The signature of the PASSporT is created as specified by JWS using the private key corresponding to the X.509 public key certificate referenced by the "x5u" header parameter. 


**4. Security Considerations**

There are a number of security considerations required for preventing various attacks on the validity or impersonation of the signature.

**4.1 Validation of the Issuer and Certificate Signature**

Use of X.509 based signatures for the JWT implies normal validation of the certificate ownership based on the binding of the public key certificate to the distinguished name representing the authorized originator.  The iss field of the signed claim should also match this distinguished name of the certificate used for signing the verified token.

**4.2 Avoidance of replay and cut and paste attacks**

There are a number of security considerations for use of the token for avoidance of replay and cut and paste attacks.
Verified tokens must be sent along with other application level protocol information (e.g. for SIP an INVITE as defined in [RFC3261]).  There should be a link between various information provided in the token and information provided by the application level protocol information.
These would include:

* "iat" claim should closely correspond to a date/time the message was originated.  It should also be within a relative delta time that is reasonable for clock drift and transmission time characteristics associated with the application using the verified token.
* "jti" claim could be used to exactly correspond to a unique identifier generated by originator of PASSporT
* "term" claim is included to prevent the ability to use a previously originated message to send to another terminating party

**5. IANA Considerations**

**5.1 Media Type Registration**

**5.1.1  Media Type Registry Contents Additions Requested**

This section registers the "application/passport" media type [RFC2046] in the "Media Types" registry [IANA.MediaTypes] in the manner described in RFC 6838 [RFC6838], which can be used to indicate that the content is a PASSporT defined JWT and JWS.

* Type name: application
* Subtype name: passport
* Required parameters: n/a
* Optional parameters: n/a
* Encoding considerations: 8bit; application/passport values are encoded as a series of base64url-encoded values (some of which may be the empty string), each separated from the next by a single period ('.') character.
* Security considerations: See the Security Considerations section of RFC 7515.
* Interoperability considerations: n/a
* Published specification: draft-wendt-stir-passport-00
* Applications that use this media type: STIR and other applications that require identity related assertion
* Fragment identifier considerations: n/a
* Additional information:

     Magic number(s): n/a
     File extension(s): n/a
     Macintosh file type code(s): n/a

* Person & email address to contact for further information: Chris Wendt, chris-ietf@chriswendt.net
* Intended usage: COMMON
* Restrictions on usage: none
* Author: Chris Wendt, chris-ietf@chriswendt.net
* Change Controller: IESG
* Provisional registration?  No

**5.2 JSON Web Token Claims Registration**

**5.2.1 Registry Contents Additions Requested**

* Claim Name: "oid"
* Claim Description: Originating Identity
* Change Controller: IESG
* Specification Document(s): Section 3.2 of draft-wendt-verified-token-00

* Claim Name: "tid"
* Claim Description: Terminating Identity
* Change Controller: IESG
* Specification Document(s): Section 3.2 of draft-wendt-verified-token-00

**6. Acknowledgements**

Particular thanks to members of the ATIS and SIP Forum NNI Task Group including Martin Dolly, Richard Shockey, Jim McEchern, John Barnhill, Christer Holmberg, Victor Pascual Avila, Mary Barnes, Eric Burger for their review, ideas, and contributions also thanks to Henning Schulzrinne, Russ Housley, Alan Johnston, Richard Barnes for valuable feedback on the technical and security aspects of the document.

**7. References**

   [RFC3261]  Rosenberg, J., Schulzrinne, H., Camarillo, G., Johnston,
              A., Peterson, J., Sparks, R., Handley, M. and E. Schooler,
              "SIP: Session Initiation Protocol", RFC 3261, DOI 10.17487
              /RFC3261, June 2002, <http://www.rfc-editor.org/info/
              rfc3261>.

   [RFC3986]  Berners-Lee, T., Fielding, R. and L. Masinter, "Uniform
              Resource Identifier (URI): Generic Syntax", STD 66, RFC
              3986, DOI 10.17487/RFC3986, January 2005, <http://www.rfc-
              editor.org/info/rfc3986>.

   [RFC7515]  Jones, M., Bradley, J. and N. Sakimura, "JSON Web
              Signature (JWS)", RFC 7515, DOI 10.17487/RFC7515, May
              2015, <http://www.rfc-editor.org/info/rfc7515>.

   [RFC7518]  Jones, M., "JSON Web Algorithms (JWA)", RFC 7518, DOI
              10.17487/RFC7518, May 2015, <http://www.rfc-editor.org/
              info/rfc7518>.

   [RFC7519]  Jones, M., Bradley, J. and N. Sakimura, "JSON Web Token
              (JWT)", RFC 7519, DOI 10.17487/RFC7519, May 2015, <http://
              www.rfc-editor.org/info/rfc7519>.
              

**Author's Address**

   Chris Wendt (editor)
   
   Comcast
   
   One Comcast Center
   
   Philadelphia, PA 19103
   
   US

   Email: chris-ietf@chriswendt.net
   
   
   Jon Peterson
   
   Neustar, Inc.
   
   1800 Sutter St Suite 570
   
   Concord, CA  94520
   
   US

   Email: jon.peterson@neustar.biz
