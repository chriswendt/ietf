
### Persona Assertion Token (PASSporT)

**draft-ietf-stir-passport-00**

**Abstract**

This document defines a token format for verifying with non-repudiation information associated with the originator of personal communications. A cryptographic signature is defined to protect the integrity of the information used to identify the originator of a personal communications session toward a terminating entity. The cryptographic signature is defined with the intention that it can confidentially verify the originating persona even when the signature is sent to the terminating party over a potentially unsecure channel.  This persona information minimally includes the originator identity, but could also, through an extensible mechanism, provide other identity related information or details of the origination of the communications. Using the digital signature can also confirm that the originator is authorized to assert the persona information. In particular, verification of the persona information in the VoIP world is important for validating originating telephone numbers to avoid illegitimate "spoofing" of the identity for fraudulent or deceptive purposes.  PASSporT is particularly useful for many personal communications applications over IP networks and other multi-hop interconnection scenarios where the originating and terminating parties may not have a direct trusted relationship.

**1. Introduction**

This document defines a method for creating and validating a token that cryptographically verifies an originating identity, or more generally a URI representing the originator of personal communications. Through extended profiles other information associated with the originating party or the transport of the personal communications can be attached to the token.  The primary goal of PASSporT is to provide a common framework for signing persona related information in an extensible way.  A secondary goal is to provide this functionality independent of any specific personal communications signaling call logic, so that creation and verification of persona information can be implemented in a flexible way and can be used in many personal communications applications including end-to-end applications that require different signaling protocol interworking.  It is anticipated that signaling protocol specific guidance will be provided in other related documents and specifications to specify how to use and transport PASSporT tokens, however this is intentionally out of scope for this document.  

Note: As of the authoring of this document, ietf-stir-rfc4474bis provides details of how to use PASSporT within SIP signaling for the signing and verification of telephone numbers.

**2. Token Overview**

Tokens are a convenient way of encapsulating information with associated digital signatures.  They are used in many applications that require authentication, authorization, encryption and other use cases.  JWT [RFC7519] and JWS [RFC7515] are designed to provide a compact form for many of these purposes and define a specific method and syntax for signing a specific set of information or "claims" within the token and therefore providing an extensible set of claims. Additionally, JWS provides extensible mechanisms for specifying the method and cryptographic algorithms used for the associated digital signatures. 

**3. PASSporT Definition**

The PASSporT is constructed based on JWT [RFC7519] and JWS [RFC7515] specifications.  JWS defines the use of JSON data structures in a specified canonical format for signing data corresponding to JOSE header, JWS Payload, and JWS Signature.  JWT defines specific set of claims that are represented by specified key value pairs which can be extended with custom keys for specific applications. 

**3.1. PASSporT Header**

The JWS token header is a JOSE header that defines the type and encryption algorithm used in the token.  

An example of the header for the case of a RSASSA-PKCS1-v1_5 SHA-256 digital signature would be the following,

	  { 
      "typ":"passport",
      "alg":"RS256",
      "x5u":"https://tel.example.org/passport.crt" 
    }
      
**3.1.1 "typ" (Type) Header Parameter**
      
JWS defines the "typ" (Type) Header Parameter to declare the media type [IANA.MediaTypes] of the JWS. 

This represents that the encoded token is a JWT, and the JWT is a JWS using the RSASSA-PKCS1-v1_5 SHA-256 algorithm.  

For PASSporT Token the "typ" header MUST minimally include and begin with "passport".

**3.1.2 "alg" (Algorithm) Header Parameter**

For PASSporT, the "alg" should be defined as RS256 as the recommended algorithm.  Note that JWA [RFC7518] defines other algorithms that may be utilized or updated in the future depending on cryptographic strength requirements guided by current security best practice.
 
**3.1.3 "x5u" (X.509 URL) Header Parameter**

As defined in JWS, the "x5u" header parameter is used to provide a URI [RFC3986] referring to the resource for the X.509 public key certificate or certificate chain [RFC5280] corresponding to the key used to digitally sign the JWS.  Note: The definition of what the URI represents in terms of the actor serving the X.509 public key is out of scope of this document.  However, generally this would correspond to an HTTPS or DNSSEC resource with the guidance that it MUST be a TLS protected, per JWS spec.

**3.2. PASSporT Token Claims**

The token claim should consist of the information which needs to be verified at the terminating party.  This claim should correspond to a JWT claim [RFC7519] and be encoded as defined by the JWS Payload [RFC7519]. 

The PASSporT should use a number of standard JWT defined headers as well as some addition custom headers specifically required for two party communications with an originator and terminator.  These headers or key value pairs will be explained here, but some of the security implications will be explained further in the security considerations section below.

The JSON claim MUST include the following registered JWT defined claims:

* "iat" - issued at, time the JWT was issued, used for expiration

Verified Token specific claims MUST that be included:

* "orig" - the originating identity claimed.  (e.g. for SIP, the FROM or P-AssertedID associated e.164 telephone number, TEL or SIP URI)  This SHOULD be in URI format as defined in [RFC3986] but could also be an application specific identity string.
* "term" - the terminating identity claimed as the intended destination by the originating party. (e.g. for SIP, the TO associated e.164 telephone number, TEL or SIP URI)  This SHOULD be in URI format as defined in [RFC3986] but could also be an application specific identity string.

An example claim is as follows,

	{ 
    "iat": 1443208345, 
    "orig":"+12155551212",
	  "term":"sip:+12155551213@example.com"
  }
 
**3.3 Verified Token Signature**

The signature of the PASSporT is created as specified by JWS using the private key corresponding to the X.509 public key certificate referenced by the "x5u" header parameter. 

**4. Extending PASSporT**

PASSporT represents the bare minimum set of claims needed to assert the originating identity, however there will certainly be new and extended applications and usage of PASSPorT that will need to extending claims to represent other information specific to the origination identities beyond the identity itself.

**4.1 "ppt" (PASSporT) header parameter**

For these extended profiles of PASSporT, the JWS header parameter "ppt" SHOULD be used with a string that uniquely identifies the profile specification that defines any new claims that would extend the base set of claims of PASSporT.

An example header with an extended PASSporT profile of "foo" is as follows:

	{ 
      "typ":"passport",
      "ppt":"foo",
      "alg":"RS256",
      "x5u":"https://tel.example.org/passport.crt"
  }  

**4.2 Extended PASSporT Claims**

Future specifications that define such extensions to the PASSporT mechanism MUST explicitly designate what claims they include, the order in which they will appear, and any further information necessary to implement the extension. All extensions MUST incorporate the baseline JWT elements specified in Section 3; claims may only be appended to the
claims object specified in there, they can never be subtracted re-ordered. Specifying new claims follows the baseline JWT procedures ([RFC7519] Section 10.1 <https://tools.ietf.org/html/rfc7519#section-10.1>). Note that understanding an extension as a verifier is always optional for compliance with this specification (though future specifications or
profiles for deployment environments may make other "ppt" values mandatory). The creator of a PASSporT object cannot assume that verifiers will understand any given extension. Verifiers that do support an extension may then trigger appropriate application-level behavior in the presence of an extension; authors of extensions should provide appropriate extension-specific guidance to application developers on this point.

**4.3 Alternate PASSporT Extension**

Some applications may want to use the mechanism of the PASSporT digital signature that is not a superset of the base set of claims of the PASSporT token as defined in Section 3.  Rather, a specification may use PASSporT with it's own defined set of claims.

In this case, the specification should define it's own MIME media type [RFC2046] in the "Media Types" registry [IANA.MediaTypes].  It is recommended that the MIME subtype start with the string "passport-" to signify that it is related to the PASSporT token.  For example, for the "foo" application the MIME type/sub-type could be defined as "application/passport-foo".

**4.4 Registering PASSporT Extensions**

In order for interoperability and maintaining uniqueness of the extended PASSporT profile header parameter string, there SHOULD be an industry registry that tracks the definition of the profile strings.

**5. Deterministic JSON Serialization**

In order to provide a deterministic representation of the PASSporT Header and Claims, particularly if PASSporT is used across multiple signalling environments, the JSON header object and JSON Claim object should be computed as follows. 

The JSON object should follow the rules for the construction of the thumbprint of a JSON Web Key (JWK) as defined in [RFC7638] Section 3.  Each JSON object MUST contain no whitespace or line breaks before or after any syntactic elements and with the required members ordered lexicographically by the Unicode [UNICODE] code points of the member names.

In addition, the JSON header and claim members MUST follow the lexicographical ordering and character and string rules defined in [RFC7638] Section 3.3.

**6. Human Readability**

JWT [RFC7519] and JWS [RFC7515] are defined to apply Base64 encoding to the Header and Claims sections.  Many personal communications protocols, such as SIP and XMPP, generally utilize a "human readable" format to allow for ease of use and ease of operational debugging and monitoring.  For these protocols, flexibility in the usage of Base64 encoding that obfuscates the readable JSON objects is a consideration.  Therefore, specifications defining the usage of PASSporT may provide guidance on whether Base64 encoding can be eliminated from the construction of the PASSporT Header and Claim sections.

**7. Security Considerations**

**7.1  Avoidance of replay and cut and paste attacks**

There are a number of security considerations for use of the token for avoidance of replay and cut and paste attacks.
Verified tokens must be sent along with other application level protocol information (e.g. for SIP an INVITE as defined in [RFC3261]).  There should be a link between various information provided in the token and information provided by the application level protocol information.
These would include:

* "iat" claim should closely correspond to a date/time the message was originated.  It should also be within a relative delta time that is reasonable for clock drift and transmission time characteristics associated with the application using the verified token.
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
* Published specification: draft-ietf-stir-passport-00
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
* Specification Document(s): Section 3.2 of draft-ietf-stir-passport-00

* Claim Name: "term"
* Claim Description: Terminating URI
* Change Controller: IESG
* Specification Document(s): Section 3.2 of draft-ietf-stir-passport-00

**9. Acknowledgements**

Particular thanks to members of the ATIS and SIP Forum NNI Task Group including Jim McEchern, Martin Dolly, Richard Shockey, John Barnhill, Christer Holmberg, Victor Pascual Avila, Mary Barnes, Eric Burger for their review, ideas, and contributions also thanks to Henning Schulzrinne, Russ Housley, Alan Johnston, Richard Barnes for valuable feedback on the technical and security aspects of the document.

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
