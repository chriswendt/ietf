
### Persona Assertion Token (PASSporT)

**draft-ietf-stir-passport-00**

**Abstract**

This document defines a token format for verifying with non-repudiation information associated with the originator of personal communications.  In the process of originating personal communications and in order to protect the integrity of the information used to identify the originator, a cryptographic signature is defined.  The cryptographic signature is defined with the intention that the terminating party can verify the originator identity related information, that the originator is an authorized entity to assert the information, and that the originator information wasn't altered or modified in transit. Additionally, the token should be extensible to incorporate new types and profiles of information related to the originator identity or perhaps other information regarding the delivery of the personal communications.  In particular, verification of this information in the VoIP world is important for validating originating telephone numbers to avoid illegitimate "spoofing" of the identity for fraudulent or deceptive purposes.  PASSporT is particularly useful for many personal communications applications over IP networks and other multi-hop interconnection scenerios where there is an originating and terminating parties may not have a direct trusted relationship.

**1. Introduction**

This document will define a method for the creation and verification of an extensible canonical token that is intended for cryptographically verifying minimally an originating identity or more generally a URI representing the originator of personal communications and through extended profiles other information associated with the originating party or the transport of the personal communications.  The primary goal of PASSporT is to provide a common framework for signing persona related information in an extensible way.  A secondary goal is to provide this functionality independent of any specific personal communications signaling call logic, so creation and verification of information can be implemented in a flexible way and used in many personal communications implementations or even between different signaling protocol interworking.  It is anticipated that there will be signaling protocol specific guidance on how to use and transport PASSporT tokens, however this is intentionally out of scope for this document.  Note: As of the authoring of this document, draft-ietf-stir-rfc4474bis provides details of how to use PASSporT within SIP signaling for the signing and verification of telephone numbers.

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

For PASSporT, the "alg" should be defined as RS256 as the recommended algorithm.  Note that JWA [RFC7518] defines other algorithms that may be utilized or updated in the future depending on cryptographic strength requirements guided by current security best practice.
 
**3.1.3 "x5u" (X.509 URL) Header Parameter**

As defined in JWS, the "x5u" header parameter is used to provide a URI [RFC3986] referring to the resource for the X.509 public key certificate or certificate chain [RFC5280] corresponding to the key used to digitally sign the JWS.  Note: The definition of what the URI represents in terms of the actor serving the X.509 public key is out of scope of this document.  However, generally this would correspond to an HTTPS or DNSSEC resource with the guidance that it MUST be a TLS protected, per JWS spec.

**3.2. PASSporT Token Claims**

The token claim should consist of the information which needs to be verified at the terminating party.  This claim should correspond to a JWT claim [RFC7519] and be encoded as defined by the JWS Payload [RFC7519]. 

The PASSporT should use a number of standard JWT defined headers as well as some addition custom headers specifically required for two party communications with an originator and terminator.  These headers or key value pairs will be explained here, but some of the security implications will be explained further in the security considerations section below.

The JSON claim MUST include the following registered JWT defined claims unless noted optional:

* "jti" - required - unique identifier of the JWT, useful for both tracking and avoiding replay of JWT
* "iat" - required - issued at, time the JWT was issued, used for expiration

Verified Token specific claims MUST be included unless noted optional:

* "orig" - required - the originating identity claimed.  (e.g. for SIP, the FROM or PAI associated e.164 telephone number, TEL or SIP URI)  This MAY be in URI format as defined in [RFC3986] or an application specific identity string.
* "term" - required - the terminating identity claimed as the intended destination by the originating party. (e.g. for SIP, the TO associated e.164 telephone number, TEL or SIP URI)  This MAY be in URI format as defined in [RFC3986] or an application specific identity string.

An example claim is as follows,

	{ 
    "iat": 1443208345, 
    "jti": "FAhNaPk0onffyJvykJZC2A==",
    "orig":"+12155551212",
	  "term":"sip:+12155551213@example.com"
  }
 
**3.3 Verified Token Signature**

The signature of the PASSporT is created as specified by JWS using the private key corresponding to the X.509 public key certificate referenced by the "x5u" header parameter. 

**4. Extending PASSporT**

PASSporT represents the bare minimum set of claims needed to assert the originating identity, however there will certainly be new and extended applications and usage of PASSPorT that will have the need of extending claims to represent other information specific to the origination identities beyond the identity itself.

The suggested mechanism would be to create a new media subtype, inline with MIME media type definitions.  This subtype SHOULD follow the convention that if passport is the base subtype, the extended application subtype should use the prefix "passport+" and append the name.  For example, the base application type is "application" and subtype is "passport".  If a new specification would like to define a new usage and set of claims and call it "foo", the new subtype would be defined as "passport+foo". 

**5. Deterministic JSON Serialization**

In order to provide a deterministic representation of the PASSporT Header and Claims, particularly if PASSporT is used across multiple signalling environments, the JSON header object and JSON Claim object should be computed as follows. 

The JSON object should follow similar rules to the construction of the thumbprint of a JSON Web Key (JWK) [RFC7638] Section 3.  Each JSON object MUST contain no whitespace or line breaks before or after any syntactic elements and with the required members ordered lexicographically by the Unicode [UNICODE] code points of the member names.

In addition, the JSON header and claim members MUST follow the lexicographical ordering and character and string rules defined in [RFC7638] Section 3.3.

**6. Human Readability**

JWT is defined to apply Base64 encoding to the Header and Claims sections.  Many protocols, like SIP and XMPP, generally utilize a "human readable" format to allow for ease of use and ease of operational debugging and monitoring.  For these protocols, the specifications defining the usage of PASSporT SHOULD provide guidance on whether Base64 encoding can be removed from the construction of the JWT.

**7. Security Considerations**

There are a number of security considerations required for preventing various attacks on the validity or impersonation of the signature.

**7.1 Validation of the Issuer and Certificate Signature**

Use of X.509 based signatures for the JWT implies normal validation of the certificate ownership based on the binding of the public key certificate to the distinguished name representing the authorized originator.  The iss field of the signed claim should also match this distinguished name of the certificate used for signing the verified token.

**7.2 Avoidance of replay and cut and paste attacks**

There are a number of security considerations for use of the token for avoidance of replay and cut and paste attacks.
Verified tokens must be sent along with other application level protocol information (e.g. for SIP an INVITE as defined in [RFC3261]).  There should be a link between various information provided in the token and information provided by the application level protocol information.
These would include:

* "iat" claim should closely correspond to a date/time the message was originated.  It should also be within a relative delta time that is reasonable for clock drift and transmission time characteristics associated with the application using the verified token.
* "jti" claim could be used to exactly correspond to a unique identifier generated by originator of PASSporT
* "term" claim is included to prevent the ability to use a previously originated message to send to another terminating party

**8. IANA Considerations**

**8.1 Media Type Registration**

**8.1.1  Media Type Registry Contents Additions Requested**

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

**8.2 JSON Web Token Claims Registration**

**8.2.1 Registry Contents Additions Requested**

* Claim Name: "orig"
* Claim Description: Originating URI
* Change Controller: IESG
* Specification Document(s): Section 3.2 of draft-wendt-verified-token-00

* Claim Name: "term"
* Claim Description: Terminating URI
* Change Controller: IESG
* Specification Document(s): Section 3.2 of draft-wendt-verified-token-00

**9. Acknowledgements**

Particular thanks to members of the ATIS and SIP Forum NNI Task Group including Martin Dolly, Richard Shockey, Jim McEchern, John Barnhill, Christer Holmberg, Victor Pascual Avila, Mary Barnes, Eric Burger for their review, ideas, and contributions also thanks to Henning Schulzrinne, Russ Housley, Alan Johnston, Richard Barnes for valuable feedback on the technical and security aspects of the document.

**10. References**

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
