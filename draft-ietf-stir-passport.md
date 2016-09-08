
### Persona Assertion Token (PASSporT)

**draft-ietf-stir-passport-05**

**Abstract**

This document defines a canonical string object or 'token' including a digitial signature for verifying the author of the token, their authority to author the token and the information asserted in the token, minimally, the  originating identity or 'persona' corresponding specifically to the originator of 'personal communications', or signalled communications between a set of parties with identities. The PASSporT token is cryptographically signed to protect the integrity of the identify the originator of a personal communications session (e.g. the telephone number or URI) and verify the assertion of the identity information at the destination. The cryptographic signature is defined with the intention that it can confidently verify the originating persona even when the signature is sent to the destination party over an insecure channel. PASSporT is particularly useful for many personal communications applications over IP networks and other multi-hop interconnection scenarios where the originating and destination parties may not have a direct trusted relationship.

**1. Introduction**

In today's IP-enabled telecommunications world, there is a growing concern about the ability to trust incoming invitations for communications sessions, including video, voice and messaging [RFC 7340]. As an example, modern telephone networks provide the ability to spoof the calling party telephone number for many legitimate purposes including providing network features and services on the behalf of a legitimate telephone number.  However, as we have seen, bad actors have taken advantage of this ability for illegitimate and fraudulent purposes meant to trick telephone users to believe they are someone they are not. This problem can be extended to many emerging forms of personal communications.

This document defines a method for creating and validating a token that cryptographically verifies an originating identity, or more generally a URI or telephone number representing the originator of personal communications. Through extensions defined in this document, in Section 5.2, other information relevant to the personal communications can also be added to the token.  The goal of PASSporT is to provide a common framework for signing originating identity related information in an extensible way.  Additionally, this functionality is independent of any specific personal communications signaling call logic, so that the assertion of originating identity related information can be implemented in a flexible way and can be used in applications including end-to-end applications that require different signaling protocols or gateways between different communications systems.  It is anticipated that signaling protocol specific guidance will be provided in other related documents and specifications to specify how to use and transport PASSporT tokens, however this is intentionally out of scope for this document.  

[I-D.ietf-stir-rfc4474bis] provides details of how to use PASSporT within SIP signaling for the signing and verification of telephone numbers.

**2. PASSporT Token Overview**

JSON Web Token (JWT) [RFC7519] and JSON Web Signature (JWS) [RFC7515] and related specifications define a standard token format that can be used as a way of encapsulating claimed or asserted information with an associated digital signature using X.509 based certificates. JWT provides a set of claims in JSON format that can conveniently accomidate asserted originating identity information and is easily extensible for extension mechanisms defined below. Additionally, JWS provides a path for updating methods and cryptographic algorithms used for the associated digital signatures. 

JWS defines the use of JSON data structures in a specified canonical format for signing data corresponding to JOSE header, JWS Payload, and JWS Signature. JWT defines a set of claims that are represented by specified key value pairs which can be extended with custom keys for specific applications. The next sections define the header and claims that MUST be minimally used with JWT and JWS for PASSporT.

**3. PASSporT Header**

The JWS token header is a JOSE header, [RFC7515] Section 4, that defines the type and encryption algorithm used in the token. 

PASSporT header should include, at a minimum, the following header parameters defined the the next three subsections.
      
**3.1. "typ" (Type) Header Parameter**
      
The "typ" (Type) Header Parameter is defined in JWS Section 4.1.9. to declare the media type of the complete JWS. 

For PASSporT Token the "typ" header MUST be the string "passport". This represents that the encoded token is a JWT of type passport. 


**3.2 "alg" (Algorithm) Header Parameter**

The "alg" (Algorithm) Header Parameter is defined in JWS Section 4.1.1. This definition includes the ability to specify the use of a cryptographic algorithm for the signature part of the JWS.  It also refers to a list of defined "alg" values as part of a registry established by JSON Web Algorithms (JWA) [RFC7518] and defined in Section 3.1.

For the creation and verification of PASSporT tokens and their digital signatures ES256 MUST be implemented as defined in JWA Section 3.4. 

Note that JWA defines other algorithms that may be utilized or updated in the future depending on cryptographic strength requirements guided by current security best practice.
 
**3.3 "x5u" (X.509 URL) Header Parameter**

As defined in JWS Section 4.1.5., the "x5u" header parameter defines is used to provide a URI [RFC3986] referring to the resource for the X.509 public key certificate or certificate chain [RFC5280] corresponding to the key used to digitally sign the JWS. Generally, as defined in JWS section 4.1.5, this would correspond to an HTTPS or DNSSEC resource using integrity protection.

**3.4 Example PASSporT header

An example of the header, would be the following, including the specified passport type, ES256 algorithm, and a URI referencing the network location of the certificate needed to validate the PASSporT signature.

    { 
      "typ":"passport",
      "alg":"ES256",
      "x5u":"https://cert.example.org/passport.cer" 
    }

**4. PASSporT Payload**

The token claims consist of the information which needs to be verified at the destination party.  These claims follow the definition of a JWT claim [RFC7519] Section 4 and be encoded as defined by the JWS Payload [RFC7515] Section 3. 

PASSporT defines the use of a standard JWT defined claim as well as custom claims corresponding to the two parties associated with personal communications, the originator and destination as detailed below. 

Any claim key values outside the US-ASCII range should be encoded using percent encoding as described in section 2.1 of RFC 3986, case normalized as described in 6.2.2.1 of RFC 3986.

**4.1. JWT defined claims**

**4.1.1 "iat" - Issued At claim**

The JSON claim MUST include the "iat" [RFC7519] Section 4.1.6 defined claim Issued At.  As defined this should be set to the date and time of the origination of the personal communications. The time value should be of the format defined in [RFC7519] Section 2 NumericDate.  This is included for securing the token against replay and cut and paste attacks, as explained further in the security considerations in section 6.

**4.2. PASSporT specific claims**

**4.2.1. Originating and Destination Identity Claims**

PASSporT defines claims that convey the identity of the origination and destination of personal communications.  Origination in the context of PASSporT and for a given application’s use of PASSporT is the point in the network that has the authority to assert the callers identity.  This authority is represented in PASSporT by the certificate holder and is signed at the applications choice of authoritative point(s) in the network, for example, at a device that has authenticated with a user, or at a network entity with an authenticated trust relationship with that device and it's user.  Destination represents the intended destination of the personal communications, i.e. the identity(s) being called by the caller, The destination point(s) determined by the application must have the capability to verify the PASSporT token and the digital signature. The PASSporT associated certificate is used to validate the authority of the originating signer, generally via a certificate chain to the trust anchor for that application.   

The origination and destination identities are represented by two claims that are required for PASSporT, the "orig" and "dest" claims. Both "orig" and "dest" MUST have claims where the key represents an identity type and the value is the identity string, both defined in subsecquent subsections.  Currently, these identities can be represented as either telephone numbers or Uniform Resource Indicators (URIs).

The "orig" JSON object MUST only have one key value pair representing the asserted identity of any type (currently either "tn" or "uri") of the originator of the personal communications signaling.

The "dest" JSON object MUST have at least have one key value pair, but could have multiple identity types (i.e. "tn" and/or "uri") but only one of each.  If both "tn" and "uri" are included, the JSON object should list the "tn" array first and the "uri" array second.  Within the "tn" and "uri" arrays, the identity strings should be put in lexiographical order including the scheme-specific portion of the URI characters.  Additionaly, in the case of "dest" only, the identity type key value MUST be an array signaled by standard JSON brackets, even when there is a single identity value in the identity type key value. 

**4.2.1.1. "tn" - Telephone Number identity**

If the originating or destination identity is a telephone number, the key representing the identity MUST be "tn". 

Telephone Number strings for "tn" MUST be canonicalized according to the procedures specified in [ietf-stir-rfc4474bis-12] Section 8.3.

**4.2.1.2. "uri" - URI identity**

If any of the originating or destination identities is of the form URI, as defined in [RFC3986], the key representing the identity MUST be "uri"  URI form of the identity.

**4.2.1.3. Future identity forms**

We recognize that in the future there may be other standard mechanisms for representing identities.  The "orig" and "dest" claims currently support "tn" and "uri" but could be extended in the future to allow for other identity types with new IANA registered unique types to represent these forms.

**4.2.1.4. Examples**

Single Originator, with telephone number identity +12155551212, to Single Destination, with URI identity 'sip:alice@example.com', example:

	{ 
    	"dest":{"uri":["sip:alice@example.com"]},    		
    	"iat":"1443208345",
    	"orig":{"tn":"12155551212"}
    }
    
Single Originator, with telephone number identity +12155551212, to Multiple Destination Identities, with telephone number identity +12125551212 and two URI identities, sip:alice@example.com and sip:bob@example.com, example:

	{ 
    	"dest":{
    			"tn":["12125551212"],
    		    "uri":["sip:alice@example.com",
    		        "sip:bob@example.net"]
    	},
    	"iat":"1443208345",
    	"orig":{"tn":"12155551212"}
    }

**4.2.2. "mky" - Media Key claim**

Some protocols that use PASSporT may also want to protect media security keys delivered within their signaling in order to bind those keys to the identities established in the signaling layers. The "mky" is an optional PASSporT claim defining the assertion of media key fingerprints carried in SDP [RFC4566] via the "a=fingerprint" attribute [RFC4572] Section 5. This claim can support either a single or multiple fingerprints appearing in a single SDP body corresponding to one or more media streams offered. 
The "mky" claim MUST be formated in a JSON form including the "alg" and "dig" keys with the corresponding algorithm and hexadecimal values. If there are more that one fingerprint values associated with different media streams in SDP, the fingerprint values MUST be constructed as a JSON array denoted by bracket characters.
For the 'dig' key value, the hash value MUST be the hexadecimal value without any colons. The "mky" array should order the JSON objects containing both "alg" and "dig" key values in lexiographic order of the "dig" string values and within each of those objects the JSON keys should have "alg" first and "dig" second.

An example claim with "mky" claim is as follows:

For an SDP offer that includes the following fingerprint values,

	a=fingerprint:sha-256 4A:AD:B9:B1:3F:82:18:3B:54:02:12:DF:3E:
	5D:49:6B:19:E5:7C:AB:3E:4B:65:2E:7D:46:3F:54:42:CD:54:F1
	a=fingerprint:sha-256 02:1A:CC:54:27:AB:EB:9C:53:3F:3E:4B:65
	:2E:7D:46:3F:54:42:CD:54:F1:7A:03:A2:7D:F9:B0:7F:46:19:B2


the PASSporT Payload object would be:

	{ 
	  	"dest":{"uri":["sip:alice@example.com"]},
	  	"iat":"1443208345", 
	  	"mky":[
        {
           "alg":"sha-256",
           "dig":"021ACC5427ABEB9C533F3E4B652E7D463F5442CD54
           	F17A03A27DF9B07F4619B2"
        },
        {
           "alg":"sha-256",
           "dig":"4AADB9B13F82183B540212DF3E5D496B19E57C
           	AB3E4B652E7D463F5442CD54F1"
        }
      ],
    	"orig":{"tn":"12155551212"}
    }
  
**4.3. PASSporT Signature**

The signature of the PASSporT is created as specified by JWS using the private key corresponding to the X.509 public key certificate referenced by the "x5u" header parameter. 

**5. Extending PASSporT**

PASSporT includes the bare minimum set of claims needed to securely assert the originating identity and support the secure propoerties discussed in various parts of this document. JWT supports a straight forward way to add additional claims by simply adding new claim key pairs. PASSporT can be extended beyond the defined base set of claims to represent other information requiring assertion or validation beyond the originating identity itself as needed.

**5.1 "ppt" (PASSporT) header parameter**

For extension of the base set of claims defined in this document, a new JWS header parameter "ppt" MUST be used with a unique string.  Any PASSporT extension should be defined in a specification describing the PASSporT extension and the string used in the "ppt" hedaer string that defines any new claims that would extend the base set of claims of PASSporT.

An example header with a PASSporT extension type of "foo" is as follows:

	{ 
      "alg":"ES256",
      "ppt":"foo",
      "typ":"passport",
      "x5u":"https://tel.example.org/passport.cer"
    }  

**5.2 Extended PASSporT Claims**

Specifications that define extensions to the PASSporT mechanism MUST explicitly specify what claims they include beyond the base set of claims from this document, the order in which they will appear, and any further information necessary to implement the extension. All extensions MUST include the baseline JWT elements specified in Section 3; claims may only be appended to the claims object specified; they can never be removed or re-ordered. Specifying new claims follows the baseline JWT procedures ([RFC7519] Section 10.1). Understanding an extension or new claims defined by the extension on the destination verification of the PASSporT token is optional. The creator of a PASSporT object cannot assume that destination systems will understand any given extension. Verification of PASSporT tokens by destination systems that do support an extension may then trigger appropriate application-level behavior in the presence of an extension; authors of extensions should provide appropriate extension-specific guidance to application developers on this point.

An example set of extended claims, extending the first example in Section 4.1.2.4. using "bar" as the newly defined claim would be as follows:

	{ 
        "bar":"beyond all recognition"
    	"dest":{"uri":["sip:alice@example.com"]},    		
    	"iat":"1443208345",
    	"orig":{"tn":"12155551212"}
    }


**6. Deterministic JSON Serialization**

JSON, as a canonical format, can include spaces, line breaks and key value pairs can occur in any order and therefore makes it, from a string format, non-deterministic. In order to make the digitial signature verification work deterministically, the JSON representation of the PASSporT Header and Claims, particularly if PASSporT is used across multiple signaling environments, specifically the JSON header object and JSON Claim object MUST be computed as follows. 

The JSON object MUST follow the rules for the construction of the thumbprint of a JSON Web Key (JWK) as defined in [RFC7638] Section 3 Step 1. 
The PASSporT header and claim members MUST only follow the lexicographical ordering rules.  Any members that themselves contain JSON objects or arrays, such as "dest" or "mky" MUST follow their own lexiographical ordering and whitespace and line break rules.  This includes any header or claims defined in future specifications using PASSporT.

**6.1 Example PASSport deterministic JSON form**

This section demonstrate the deterministic JSON serialization for the example PASSporT Payload shown in Section 4.2.2.

The initial JSON object is shown here:

    { 
        "dest":{"uri":["sip:alice@example.com"]},
        "orig":{"tn":"12155551212"}
        "iat":"1443208345", 
        "mky":[
        {
           "alg":"sha-256",
           "dig":"021ACC5427ABEB9C533F3E4B652E7D463F5442CD54
           	F17A03A27DF9B07F4619B2"
        },
        {
           "alg":"sha-256",
           "dig":"4AADB9B13F82183B540212DF3E5D496B19E57C
           	AB3E4B652E7D463F5442CD54F1"
        }
      ],

    }

The parent members of the JSON object are as follows:

* "dest"
* "orig"
* "iat"
* "mky"

Their lexicographic order is:

* "dest"
* "iat"
* "mky"
* "orig"
    
The final constructed deterministic JSON serialization representation, with whitespace and line breaks removed, (with line brakes used for display purposes only) is:

	{"dest":{"uri":["sip:alice@example.com"],"iat":1443208345,"mky":
	  [{"alg":"sha-256","dig":"021ACC5427ABEB9C533F3E4B652E7D463F5442CD5
	  4F17A03A27DF9B07F4619B2"},{"alg":"sha-256","dig":"4AADB9B13F82183B5
	  40212DF3E5D496B19E57CAB3E4B652E7D463F5442CD54F1"}],
	  "orig":{"tn":"12155551212"}}

**7. Security Considerations**

**7.1  Avoidance of replay and cut and paste attacks**

There are a number of security considerations for use of the token for avoidance of replay and cut and paste attacks. PASSporT tokens should be sent with other application level protocol information (e.g. for SIP an INVITE as defined in [RFC3261]). In order to make the token signature unique to a specific origination of personal communications there should be a link between various information provided in the token and information provided by the application level protocol information.  This uniqueness specified using the following two claims:

* 'iat' claim should correspond to a date/time the message was originated.  It should also be within a relative  time that is reasonable for clock drift and transmission time characteristics associated with the application using the PASSporT token.  Therefore, validation of the token should consider date and time correlation, which could be influenced by signaling protocol specific use and network time differences.
* 'dest' claim is included to prevent the valid re-use of a previously originated message to send to another destination party.

**7.2 Solution Considerations**

The use of PASSporT tokens based on the validation of the digital signature and the associated certificate requires consideration of the authentication and authority or reputation of the signer to attest to the identity being asserted.  It should be recognized that the use of this token should not, in it's own right, be considered a full solution for absolute non-repudiation of the identity being asserted. It can and often is the case that the end user that the identity represents and signer are not one in the same.  However, applications that use PASSporT should ensure the signer is in an authoritative position to represent the user and authenticate the user onto the communications network and should be the responsible party for protecting the destination party from potential identity spoofing in addition to other abuse of the communications network outside of PASSporT.  

**7.3 Privacy Considerations**

Because PASSporT explicitly includes claims of identifiers of parties involved in communications, date and times, and potentially other call detail, care should be taken outside of traditional protected or private telephony communications paths where there may be concerns about exposing information to either unintended or illegitimate actors. These identifiers are often exposed through many communications signaling protocols as of today, but appropriate precautions should be taken.

**8. IANA Considerations**

**8.1 Media Type Registration**

**8.1.1  Media Type Registry Contents Additions Requested**

This section registers the "application/passport" media type [RFC2046] in the "Media Types" registry in the manner described in RFC 6838 [RFC6838], which can be used to indicate that the content is a PASSporT defined JWT and JWS.

* Type name: application
* Subtype name: passport
* Required parameters: n/a
* Optional parameters: n/a
* Encoding considerations: 8bit; application/passport values outside the US-ASCII range are encoded using percent encoding as described in section 2.1 of RFC 3986 (some values may be the empty string), each separated from the next by a single period ('.') character.
* Security considerations: See the Security Considerations section of RFC 7515.
* Interoperability considerations: n/a
* Published specification: draft-ietf-stir-passport-05
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
* Claim Description: Originating Identity String
* Change Controller: IESG
* Specification Document(s): Section 3.2 of draft-ietf-stir-passport-05

* Claim Name: "dest"
* Claim Description: Destination Identity String
* Change Controller: IESG
* Specification Document(s): Section 3.2 of draft-ietf-stir-passport-05

* Claim Name: "mky"
* Claim Description: Media Key Fingerprint String
* Change Controller: IESG
* Specification Document(s): Section 3.2 of draft-ietf-stir-passport-05

**9. Acknowledgements**

Particular thanks to members of the ATIS and SIP Forum NNI Task Group including Jim McEchern, Martin Dolly, Richard Shockey, John Barnhill, Christer Holmberg, Victor Pascual Avila, Mary Barnes, Eric Burger for their review, ideas, and contributions also thanks to Henning Schulzrinne, Russ Housley, Alan Johnston, Richard Barnes, Mark Miller, Ted Hardie, Dave Crocker, Robert Sparks for valuable feedback on the technical and security aspects of the document. Additional thanks to Harsha Bellur for assistance in coding the example tokens.

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


**Appendix A. Example ES256 based PASSporT JWS Serialization and Signature**

For PASSporT, there will always be a JWS with the following members:

* "protected", with the value BASE64URL(UTF8(JWS Protected Header))
* "payload", with the value BASE64URL (JWS Payload)
* "signature", with the value BASE64URL(JWS Signature)

Note: there will never be a JWS Unprotected Header for PASSporT.

First, an example PASSporT Protected Header is as follows:

	{ 
        "alg":"ES256",
    	"typ":"passport",
        "x5u":"https://cert.example.org/passport.cer" 
    }
	
This would be serialized to the form (with line brake used for display purposes only):

	{"alg":"ES256","typ":"passport","x5u":"https://cert.example.org
		/passport.cer"}
	
Encoding this with UTF8 and BASE64 encoding produces this value:
	
	eyJhbGciOiJFUzI1NiIsInR5cCI6InBhc3Nwb3J0IiwieDV1IjoiaHR0cHM6Ly9j
	ZXJ0LmV4YW1wbGUub3JnL3Bhc3Nwb3J0LmNlciJ9
	
Second, an example PASSporT Payload is as follows:

	{ 
    	"dest":{"uri":["sip:alice@example.com"]}
    	"iat":"1443208345",
    	"orig":{"tn":"12155551212"}
    }
  
This would be serialized to the form (with line brake used for display purposes only):

	{"dest":{"uri":["sip:alice@example.com"]},"iat":"1443208345",
	    "orig":{"tn":"12155551212"}}
	
Encoding this with the UTF8 and BASE64 encoding produces this value:

	eyJkZXN0Ijp7InVyaSI6WyJzaXA6YWxpY2VAZXhhbXBsZS5jb20iXX0sImlhdCI
	6IjE0NDMyMDgzNDUiLCJvcmlnIjp7InRuIjoiMTIxNTU1NTEyMTIifX0
	
Computing the digital signature of the PASSporT Signing Input ASCII(BASE64URL(UTF8(JWS Protected Header)) || '.' || BASE64URL(JWS Payload))

	rq3pjT1hoRwakEGjHCnWSwUnshd0-zJ6F1VOgFWSjHBr8Qjpjlk-cpFYpFYsojN
	CpTzO3QfPOlckGaS6hEck7w
	
The final PASSporT token is produced by concatenating the values in the order Header.Payload.Signature with period ('.') characters.  For the above example values this would produce the following (with line brakes between period used for readability purposes only):
		
	eyJhbGciOiJFUzI1NiIsInR5cCI6InBhc3Nwb3J0IiwieDV1IjoiaHR0cHM6Ly9j
	ZXJ0LmV4YW1wbGUub3JnL3Bhc3Nwb3J0LmNlciJ9
	.
	eyJkZXN0Ijp7InVyaSI6WyJzaXA6YWxpY2VAZXhhbXBsZS5jb20iXX0sImlhdCI
	6IjE0NDMyMDgzNDUiLCJvcmlnIjp7InRuIjoiMTIxNTU1NTEyMTIifX0
	.
	rq3pjT1hoRwakEGjHCnWSwUnshd0-zJ6F1VOgFWSjHBr8Qjpjlk-cpFYpFYsojN
	CpTzO3QfPOlckGaS6hEck7w
	

**Appendix A.1. X.509 Private Key Certificate for ES256 Example**

	-----BEGIN EC PRIVATE KEY-----
	MHcCAQEEIFeZ1R208QCvcu5GuYyMfG4W7sH4m99/7eHSDLpdYllFoAoGCCqGSM49
	AwEHoUQDQgAE8HNbQd/TmvCKwPKHkMF9fScavGeH78YTU8qLS8I5HLHSSmlATLcs
	lQMhNC/OhlWBYC626nIlo7XeebYS7Sb37g==
	-----END EC PRIVATE KEY-----


**Appendix A.2. X.509 Public Key Certificate for ES256 Example**

	-----BEGIN PUBLIC KEY-----
	MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE8HNbQd/TmvCKwPKHkMF9fScavGeH
	78YTU8qLS8I5HLHSSmlATLcslQMhNC/OhlWBYC626nIlo7XeebYS7Sb37g==
	-----END PUBLIC KEY-----
	

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
