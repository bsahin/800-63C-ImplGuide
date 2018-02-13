# Federation Operations Guide

## Table of Contents

1. [Introduction](#1-introduction)  
1. [Risk Management](#2-risk-management)
1. [Selecting an FAL](#3-selecting-an-fal)  
1. [Guidance for Relying Parties](#4-guidance-for-relying-parties)  
 4.1 [Purpose](#41-purpose)  
 4.2 [General Guidance](#42-general-guidance)  
     * 4.2.1 [Validating IdP Signatures](#421-validating-idp-signatures)  
     * 4.2.2 [Checking Assertion Expirations](#422-checking-assertion-expirations) 
     * 4.2.3 [Checking Audience Parameters](#423-checking-audience-parameters)  
4.3 [Guidance by Product Family](#43-guidance-by-product-family)  
     * 4.3.1 [SAML](#431-saml)  
     * 4.3.2 [OAuth and OpenID Connect](#432-oauth-and-openid-connect) 
1. [Guidance for Identity Providers](#5-guidance-for-identity-providers)  
 5.1 [Purpose](#51-purpose)  
 5.2 [General Guidance](#52-general-guidance)  
 5.3 [Guidance by Product Family](#53-guidance-by-product-family)  
     * 5.3.1 [SAML](#531-saml)  
     * 5.3.2 [OAuth and OpenID Connect](#532-oauth-and-openid-connect) 
1. [Example Scenarios](#6-example-scenarios)  
 6.1 [Shibboleth and SAML](#61-shibboleth-and-saml)  
 6.2 [OpenID Connect and OAuth](#62-openid-connect-and-oauth)  
 6.3 [Personal Identity Verification (PIV) card](#63-personal-identity-verification-piv-card)  
 6.4 [Privacy-enhancing Federated Identity](#64-privacy-enhancing-federated-identity)  
 6.5 [Parallel Authentication](#65-parallel-authentication)  
1. [Brokered Identity Management](#7-brokered-identity-management)
1. [Educational Resources](#8-educational-resources)
1. [Communicating with Stakeholders](#9-communicating-with-stakeholders)
1. [Conclusion](#10-conclusion)

### 1. Introduction

Federated identity transactions allow for a more secure and more usable internet by allowing subscribers to have a smaller number of accounts that can be used across many sites and applications, without using the same authenticator at multiple sites or applications. There are several major protocols that enable federation transactions, and a multitude of software packages and libraries that implement them. This document outlines what to look for in software that enables federation and how to apply that.

This document is intended to provide more direct guidance than SP 800-63C, which was written to be intentionally technology-agnostic. While this choice makes the document applicable across a wide array of technologies and circumstances, the abstract nature can make it difficult for implementers to understand what was intended by the document with regard to specific protocols or products. This guide is intended to provide more concrete guidance. 

### 2. Choosing Security Parameters

Different federation protocols and implementations of those protocols have many options that lead to different outcomes in the security of a system. All of these options have trade-offs in terms of complexity, robustness, and other characteristics. Choosing the right set of options for a given situation helps ensure that transactions will be as secure, functional, and efficient as possible.

### 2.1. Selecting a Federation Assurance Level (FAL)

The Federation Assurance Level (FAL) defined in SP 800-63C [[section 4]] provides a set of requirements for federation transactions. These requirements are grouped into an ascending scale of three levels: FAL1, FAL2, and FAL3. Each successive level includes all the features of lower levels and adds additional requirements on top of them. 

FAL1 provides a solid basis for federation and is appropriate for the vast majority of use cases. Most off-the-shelf products operate at FAL1 today without additional features. Most common deployments of SAML and OpenID Connect fulfill all the requirements of FAL1 today. FAL1 requires that all assertions be signed, time limited, and audience-restricted to prevent an assertion intended for one relying party to be replayed at another. 

FAL2 provides an extra layer of security by requiring that the assertion be encrypted so that the RP is the only party that can decrypt it. This protection is required if personally identifiable information will be sent in an identity assertion, as personally identifiable information must always be encrypted in transit and an assertion is often borne by intermediate parties in a transaction. This level requires that the IdP manage an encryption key for the RP, which necessitates additional complexity for both parties. The keys could be managed with a traditional PKI infrastructure that relies on a trusted certificate authority, but with many protocols the keys can be instead registered directly between parties. The RP also needs to manage and protect its decryption keys in order to read the information in the assertion. If the client's private decryption keys are leaked to another party, the additional protections provided by FAL2 no longer apply. 

FAL3 is intended to be forward-looking and is not yet readily available in off-the-shelf standards and products. FAL3 provides an additional layer in the form of a cryptographic key that must be presented by the subscriber in addition to the signed and encrypted assertion itself. This level requires that the IdP manage references to keys representing the subscriber at each RP in addition to managing the keys for the RPs themselves. The IdP needs to correctly associate the subscriber's key to the correct RP in the assertion, and the RP needs to be able to process and validate the presentation of the key by the subscriber. This key could be the same key that's presented by the subscriber at the IdP, such as an X.509 certificate, or it could be a separate key with no identity information associated with it that's not used at the IdP, such as a FIDO token. In both cases, the assertion needs to reference the key and the RP needs to ensure the correct key is presented by the subscriber.

### 2.2. Risk Management

Selecting and conforming to an FAL should be part of a larger risk management process and program. Conforming to FAL3 does not make an organization's security infallible, but instead provides protection against particular attacks while incurring certain costs to both the applications and the subscribers. Rather than attempting to make your federation infrastructure conform to the highest standards available, you should analyze the risks that are inherent in your organization and choose how strongly to protect against them given their severity and likelihood of occurrence.

The additional information management and implementation complexity of higher FALs cannot be ignored, and the costs to all involved must be weighed against perceived benefits. Unless there is a compelling reason to use the features of higher FALs, FAL1 should be the default for most use cases. FAL1 is the industry standard for authentication at this point in time. The risks of implementing a system at FAL1, when compared to higher FALs, may be negligible depending on relevant use cases and attack vectors. When PII needs to be passed, it should be passed in a secondary channel where possible instead of in the assertion itself, regardless of the FAL.

Because it is the front door to many critical systems, authentication is a key piece of risk management strategy. Strong federation can protect against many potential subscriber impersonation and man-in-the-middle attacks. Instead of each RP needing to manage subscriber accounts and authenticators separately, creating many vulnerable surfaces, federation concentrates the key security practices in a dedicated component, the IdP. Upgrades to authenticators, software, and practices at the IdP automatically benefit the downstream RPs and the overall network. 

### 2.3. Personally Identifiable Information (PII)

Personally Identifiable Information (PII) needs to be limited to only what's needed to perform a transaction [[section 7.3]]. For many login transactions, the RP will need to know only an identifier for the current subscriber. After an initial login, this identifier is used by the RP to tie the subscriber to a record in the RP application, and this record often contains attributes collected from various sources including the IdP and direct interaction with the subscriber. 

Assertions at all levels can include additional attributes about the subscriber [[section 6]], potentially including PII. The RP might require some of this information to perform its function, such as an email address to contact a subscriber. However, the RP will not know if it needs information for a given subscriber before that subscriber has been logged in, since the required information could be available locally to the RP without requesting it from the IdP. This presents a dillema for systems trying to practice data minimization.

Some protocols, such as SAML, require all available information be present in the assertion itself. In such cases, the RP needs to be very judicious about which attributes it requests. In other protocols, such as OpenID Connect, attributes can be sent through both the assertion and a secondary channel, the UserInfo endpoint. In OpenID Connect the ID Token serves as the assertion, and by default it contains a non-PII subject identifier for the subscriber. Additional information about the subscriber can be obtained through the UserInfo Endpoint by using an OAuth access token. Since this information is communicated in the back channel from the IdP to the RP over a an authenticated protected channel, it need not be separately encrypted as it is not handled or presented by an intermediary party (though of course it can be encrypted as well).

### 2.4. Selecting a Presentation Mechanism

Assertions can be sent either over the back channel between the IdP and RP [[section 7.1]] or over the front channel using the subscriber and their browser as an intermediary [[section 7.2]]. While both methods are allowed at all FALs, back channel presentation has a number of advantages and is preferred where possible.

Since the assertion is transmitted directly from the IdP to the RP over an authenticated protected channel, the RP has a high level of assurance that the assertion is from the IdP in question. The RP can also be sure that the assertion is generated in response to a specific authentication request since the RP needs to present an assertion reference to retrieve it. 

Front channel presentation systems are simpler for RPs to implement, as they don't have to handle assertion references or trading them for assertions. As a consequence, front channel presentation systems can be more susceptible to assertion injection, whereby an attacker can present a valid assertion to an unsuspecting RP in order to force a login at that RP, potentially taking over a session. A back channel presentation system is subject only to injection of an assertion reference, which can be more strongly tied to an authentication request. 

To enforce least-privilege and least-knowledge security principles, it is preferable to have each RP request its own assertion instead of re-using one assertion for multiple RPs. With front-channel presentation, it is tempting for a system to create a single assertion to be presented to multiple RPs by the browser. The browser then needs to determine which sites to present the assertion to, and which to request a new assertion for. With a back channel presentation mechanism, only the assertion reference is passed to the RP from the browser. Since the assertion reference is one-time use and limited to a single RP [[section 7.1]], it cannot be used accidentally multiple times at multiple RPs.

Additionally, assertions passed in the front channel are visible to an intermediary party, the browser. The assertion's payload, which intended for consumption by the RP, can include PII attributes, internal security information, or other sensitive data. To avoid untended data leakage, the IdP can employ several techniques:

1. Encrypt the payload to the RP's key, as is required at FAL2 and FAL3.
1. Limit the information contained in the assertion payload to non-sensitive information, and use a secondary mechanism to convey sensitive details such as in {{section 2.3}}.
1. Use a back-channel presentation mechanism to prevent the assertion itself from being seen by the intermediary. 

### 3. Guidance for Relying Parties

While it is the responsibility of the IdP to provide strong and trustworthy federation assertions, relying parties need to validate the elements of an assertion during a federated transaction [[section 6.2]]. 

#### 3.1. Purpose

Relying parties can be a valuable target for attackers to impersonate valid subscribers or gain valuable information about them. If relying parties do not check the validity of the information they receive, attackers can gain access to the various services that subscribers are logging in to. 

If all of these checks are performed properly, the compromise of a single relying party does not threaten the rest of the network. This is in contrast to systems with individual authenticators at each site, where the theft of a subscriber's password from one site often leads to the compromise of other sites in the network due to password reuse. 

#### 3.2. General Guidance

Relying parties need to validate IdP signatures, assertion expirations, and audience parameters. Additionally, RPs need to test that these validation checks are working at all times because there will be no outward indication that something is wrong with the system until an attack occurs. Relying parties must also verify the origin of the information that they receive, as an attacker might try to inject a valid assertion from another subscriber in order to take over an account. 

##### 3.2.1. Validating IdP Signatures

At all FALs, an identity assertion is signed by an IdP so that it cannot be forged by an attacker [[section 6.2.2]]. The IdP is the only entity with access to its private key [[section 4.1]], so a valid signature indicates that the assertion is from the IdP itself and not an attacker. If an RP does not check the validity of the IdP signature, attackers will be able to forge identity assertions and gain access to protected systems without authorization. Additionally, if the relying party does not check the signature, an attacker could modify an otherwise valid assertion in transit, associating attributes and access rights to the current subscriber that were not asserted by the IdP. 

Some protocols cover the entire assertion with a signature, while others cover only portions of it. The relying party must take care to not accept unsigned portions of the assertion as validated even when presented alongside a signed assertion.

The RP needs to make sure it is using the correct key for the claimed IdP, especially if the RP accepts assertions from multiple IdPs. In OpenID Connect, for example, the IdP is identified by the "iss" field of the ID Token's payload, and the signing key is identified by the "kid" field in the ID Token's header. The RP should accept the token if and only if the signature validates using the identified key from the identified issuer, and then only if the issuer is trusted to provide identities to this RP.

Testing whether RPs will reject unsigned assertions or assertions with invalid signatures is critical, though not an obvious test to do. Properly authorized transactions will still work even if an RP isn't checking assertion signatures, since the RP will accept the (valid) assertion whether or not it has a valid signature. Therefore, in such cases there is no outward indication of a problem in the system and there will be no error messages or login failures to indicate that something is wrong. Only a failure from a negative test -- that is to say, the explicit rejection of an unsigned assertion or an assertion with an invalid signature -- will indicate that a relying party is properly checking keys and signatures.

##### 3.2.2. Checking Assertion Expirations

Federated identity assertions are intended to be short-lived, since they are used to establish a session at the RP and not to manage a full session at the RP [[section 5.2]]. While details vary per protocol family, an assertion lasting a small number of minutes should in most cases give the system ample time to process the assertion and create a session for the subscriber. Since federation assertions are passed between different systems on the network, it is reasonable to allow a small amount of padding to the time checks to account for clock skew. This skew should be very short, such as a few seconds, so as to not inadvertently open the attacker's window for using expired assertions. A time synchronization protocol should be used on all systems on the network if possible to ensure the system clocks are as accurate as possible. 

An identity assertion which expires quickly makes it difficult for attackers to misuse the assertion and also ensures that any identity or authorization information included in the assertion is not out-of-date. RPs need to be tested to ensure they do not accept expired assertions, which can be done by presenting the RP with an expired but otherwise valid assertion and seeing if the RP accepts or rejects it.

Some assertions also contain a timestamp indicating when the assertion was issued, and an RP should not accept any assertion that claims to have been issued in the future. Some assertions will also have a timestamp indicating when the assertion is not to be used before, which an RP should process to ensure it is not accepting an assertion too early. The use of the "not-before" processing mechanism is relatively rare in modern federation protocols, as the assertions are created in response to specific login requests. 

##### 3.2.3. Checking Audience Parameters

When an IdP creates an assertion, it includes an audience field indicating which RP requested the assertion [[section 6.2.4]]. By checking the audience field, an RP can detect when an attacker is presenting an assertion intended for a different RP.

If an RP does not check for a matching audience parameter, it is possible for an attacker to get a valid assertion from any RP registered with the IdP and replay it at the target RP to gain unauthorized access.

An RP that isn't checking audience parameters will still accept a valid authorization with no outward indication of a problem. Therefore, it is important to test the RP with an assertion containing an errant or missing audience field.

##### 3.2.4. Checking Assertion Uniqueness

An attacker that gains possession of a bearer assertion could try to replay that assertion at an RP in order to take over a subscriber's session. To prevent this, an IdP is required to make each assertion unique [[section 6.2.1]]. The RP consequently needs to check the assertion for uniqueness within the assertion's expiry window by checking any unique identifiers within the assertion and accepting each unique assertion identifier once and only once to establish a session with a single subscriber. If an assertion is seen multiple times by an RP, especially from multiple connections, the RP can consider this assertion stolen. 

The RP should remember the identifiers of assertions as long as those identifiers are valid. Since assertions should have a relatively short lifespan, this can be accomplished 

##### 3.2.5. Retrieving IdP Keys

The RP can trust the assertion's signature only as much as it can trust that the keys used to verify the signature are associated with the IdP [[section 6.2.2]]. The keys must be retrieved in a secure fashion, such as over an authenticated protected channel or pre-placed by a systems administrator. Only the keys identified in the transaction should be used to evaluate the signature of an assertion. 

#### 3.3. Guidance by Product Family

This document covers two main product families that enable federated identity transactions - SAML and OpenID Connect, the latter of which is built on top of OAuth. Other protocols and approaches are possible to use while fulfilling the requirements of the guidelines.

##### 3.3.1. SAML

All parties need to be careful about passing and validating metadata. Incorrectly communicated or configured metadata could leak information about a subscriber that was not approved for distribution. Metadata that is not validated could have been tampered with by an attacker to gain access to valuable personal information.

Always check certificates before accepting identity assertions. Attackers can forge certificates and phish subscribers in an attempt to impersonate them. 

##### 3.3.2. OpenID Connect

Different OAuth grant types or "flows" are appropriate for different kinds of applications at different FALs.

The authorization code flow is a back-channel presentation mechanism and should be used whenever possible, particularly for web server, native, or mobile applications. It is the most common and most secure way to implement OAuth, the underlying protocol of OpenID Connect. It can accomodate all three FALs depending on the exact configuration of the application. The authorization code flow makes use of back channel assertion presentation, which reduces the attack surface of the RP significantly by sending the assertion directly from the IdP to the RP without an intermediary party touching it. The RP should authenticate itself when presenting the authorization code to the IdP. If the RP is a native or mobile application, it should use the PKCE extension or dynamic client registration to ensure that different copies of the client software can't impersonate each other at the IdP. 

The implicit grant type is a front-channel presentation mechanism and is appropriate for applications which are implemented entirely in front-end code and have to capability to store secrets outside of the subscriber's web browser. The lack of ability to store secrets means that these sorts of applications can only function at FAL1 because they have no method of private key management which would enable encryption of identity assertions.

The hybrid grant types are allowable only if all appropriate checks are made by the RP as defined in the standard. The `client credentials` and `resource owner credentials` grant types are not allowed at any FAL.

### 4. Guidance for Identity Providers

The nature of federation protocols allows the IdP to specialize in security in a way that benefits all RPs that connect to it. With traditional application security methods, such as having a password on each RP, every RP is fully responsible for its security. However, due to common practices like password reuse, compromise of a single RP can lead to the compromise of many other RPs for a given account. In a federation network, the identity provider (IdP) is the only party that can assert the presence and validity of subscribers and their attributes. The compromise of a single RP does not cascade through the network. Instead, the security of all subscribers rests on the security of their IdP. Consequently, a compromise of the IdP will affect all downstream parties. As such, it is vitally important that the IdP be held to the highest of security standards in implementation and deployment. 

#### 4.1. Purpose

As the lynchpin of security in a federation network, IdPs have the difficult task of keeping track of both subscribers and RPs, and connecting them in a secure fashion with a federation protocol. However, unlike RPs that are trying to provide a service or application, the IdP's primary purpose is to act as a security component for the rest of the federation. As a specialty service, it makes sense to invest heavily in good security practices.

#### 4.2. General Guidance

IdPs manage the primary authenticators and authentication processes for subscribers in a federation. Guidance for managing such authentication can be found in [[SP 800-63B]], all of which needs to be applied at the IdP. Additionally, the attributes and identities asserted by the IdP are subject to whatever verification practices the IdP uses. Guidelines for such identity proofing and verification are found in [[SP 800-63A]]. IdPs should implement phishing-resistant technologies in subscriber-facing pages and may want to consider the use of heuristic risk-based security for all connections, including APIs.

Much of the technical friction in setting up a federation stems from IdPs which are built and configured in such a way that onboarding new RPs requires a significant amount of manual human intervention [[section 5.1.1]]. Any time there is unwarranted friction in a security process, the consumers of that process (in this case RP implementors) will often find creative and usually-insecure workarounds to that process. Much of this friction is removed when IdPs support automated discovery mechanisms and simple automated registration [[section 5.2.2]]. In order to ease RP onboarding, IdPs should make their configurations discoverable in a machine-readable format over a secure protected channel, as appropriate to the protocol in use. 

IdPs must use approved cryptographic systems to generate all key material [[section 4.1]]. IdPs must securely store all private key material. Public keys need to be made available to RPs over authenticated protected channels or via trusted out of band processes, such as hand configuration by a systems administrator. An IdP can use a single key pair across different RPs on the network, and the keys can be rotated on a regular basis.

Private keys need to be protected from subscribers, RPs, and other unintended parties. If the IdP's private keys are compromised, an attacker could generate arbitrary assertions and impersonate any subscriber on the network at any RP. If an RP's keys are compromised, an attacker could impersonate a request from that RP but not attack any other RPs or the IdP itself. 

IdPs must securely store any symmetric secrets used by RPs in a fashion that reduces the likelihood of their capture, such as by storing a hash of the secret instead of the secret itself. All symmetric secrets need to be generated using approved cryptography, and a different secret needs to be generated for every RP that the IdP associates with. Similarly, if an RP talks to multiple IdPs, it should have a separate secret for each IdP. 

#### 4.3. Guidance by Product Family

This document covers two main product families that enable federated identity transactions - SAML and OpenID Connect, the latter of which is built on top of OAuth. Other protocols and approaches are possible to use while fulfilling the requirements of the guidelines.

##### 4.3.1. SAML

Both IdPs and RPs should publish metadata in a well-known location. While there is no widely accepted standard for SAML metadata exchange, it is advisable to use a well-documented metadata endpoint to serve the IdPs metadata in the form of a single XML file to any RP who wishes to consume it.

The IdP needs to always check signatures on metadata, and only accept metadata that has been signed by the presenting RP.

Identity federations like InCommon share the metadata of hundreds of IdPs and RPs in a structured manner [[section 5.1.3]]. Adding an IdP's metadata to such federations will help RPs to find it easily.

Apply best practices to protect subscriber information. All SAML assertions containing personally identifiable information should be encrypted to the relying party to protect the PII from being leaked to the browser. Assertions containing only authentication information and no personally identifiable information can relax this encryption requirement.

##### 4.3.2. OpenID Connect

IdPs should use OpenID Connect's discovery mechanism, published in JSON format at an HTTPS location ending in `/.well-known/openid-configuration` as specified in the OpenID Connect discovery specification. The discovery document contains all of the information that an RP would need to interact with the server. This document should be available in a location based on the IdP's unique issuer URL.

If personally identifiable information is bundled with authentication information in a token, it should be protected through encryption of the ID Token or use of a back-channel presentation mechanism. If personally identifiable information is made available at the UserInfo Endpoint, such need not be encrypted but must be passed over an authenticated protected channel.

It is recommended that OpenID Connect IdPs support a dynamic client registration to make it easy for RPs to register without manual intervention. Note that dynamic registration does not release data to the RP, it merely allows the RP to ask for login to be authorized at runtime [[section  4.2]].

There is a test suite for OpenID Connect providers to verify whether their instance of OpenID Connect is compliant with the standard, available from [http://openid.net/certification/testing/](http://openid.net/certification/testing/). 

The OpenID Connect community has reviewed libraries in several different languages to search for bugs and non-compliant processes, available at [http://openid.net/developers/libraries/](http://openid.net/developers/libraries/). Whenever possible, leverage these libraries in development.
 
OpenID Connect relies heavily on the [JOSE standard](https://datatracker.ietf.org/wg/jose/about/), particularly JWT and JWK. All tokens an keys in an implementation should conform to those standards. It is recommended that IdPs use an established JOSE and JWT library to ensure all appropriate checks have been made during implementation. 

Additional security standards like [token binding](https://datatracker.ietf.org/wg/tokbind/documents/) and [proof key for code exchange](https://tools.ietf.org/html/rfc7636). While those standards are not factors in determining the FAL of an OpenID Connect IdP, they considered to be best practice in the industry.

### 5. Example Scenarios

#### 5.1. Shibboleth and SAML

SAML Federations like InCommon can operate at FAL1 or FAL2. Most InCommon IdPs are running on a Shibboleth identity provider. They pass assertions through a response to an authentication event. Most often, those assertions are not encrypted to the RP and therefore conform to FAL1. For a Shibboleth IdP, either encrypt all assertions to the RP or refrain from sending personally identifiable information such as `eduPersonPrincipalName` (or `eppn`) over the wire as an unencrypted SAML assertion.

SAML could reach FAL3 by providing an attribute within the SAML assertion that references a cryptographic key to be presented by the subscriber at the RP.

#### 5.2 OpenID Connect

Typically, OpenID Connect Providers interact with OpenID Connect relying parties by providing a signed authentication assertion (the ID Token) which is separate from the transfer of personally identifiable information (from the UserInfo Endpoint). As such, these providers can safely operate at FAL1 because they are not bundling identity assertions with authentication information. This characterization is true for both the authorization code and implicit client types. 

If the ID Token contains personally identifiable information, it must be encrypted to the RP at FAL2. This is accomplished by using the JSON Web Encryption (JWE) specification and a key registered to the client. 

OpenID Connect could reach FAL3 by providing a claim within the ID Token that references a cryptographic key to be presented by the subscriber at the RP.

#### 5.3 Personal Identity Verification (PIV) Card 

PIV cards are considered an authentication technology by the SP 800-63 guidelines, not a federation technology. However, PIV cards can be used to authenticate to an IdP and start a federation transaction. This approach allows the complex processing and validation of the PIV certificate chain to be handled by a specialized security component, the IdP, and identity information be sent to the downstream RPs by the federation protocol. The attributes contained in the PIV certificates can be transmitted by the IdP to the RP, assuming all consent and privacy considerations around attribute release have been followed as usual.

FALs are independent of AALs, and any authentication device may be used to start a federation transaction at any FAL. Therefore, the use of a PIV does not guarantee a high FAL, but it can help.

There are many benefits to using federated identity management as opposed to requiring independent registration of PIV cards for each subscriber. Federated identity management protocols such as OpenID Connect allow subscribers to authenticate at a relying party regardless of which authenticator they use at their IdP. This allows relying parties to securely accept registered subscribers whose IdPs do not use PIV cards.

#### 5.4 Privacy-enhancing Federated Identity

In many cases, relying parties do not need to know the full identity of a subscriber. Relying parties should only request as much information as they need to complete the transaction requested by the subscriber, and IdPs should limit what information relying parties have access to within a transaction [[section 5.2]]. Furthermore, with protocols like OpenID Connect, the attributes of the subscriber can be sent separately from the assertion itself, limiting leakage of this information. 

Pairwise identifiers should be used in place of persistent or correlatable identifiers whenever possible [[section 6.3]]. This limits relying parties in attempts of tracking or identifying individual subscribers across different systems. 

When possible, claim references should be used to communicate identity information rather than raw data [[section 7.3]]. For example, if a relying party needs to know whether a subscriber is over eighteen years old, the IdP should respond that the subscriber is over eighteen without sharing the subscriber's age or birthdate.

#### 5.5 Parallel Authentication

In some cases a relying party may wish to confirm certain aspects of a subscriber's identity above and beyond what the IdP provides. For example, a relying party could log in a subscriber using an IdP, receive a picture of the subscriber from the IdP, and require that an in-person agent verify that the picture matches the identity of the person authenticating. This use case is known as "parallel authentication" because two authentication events are happening next to each other: the assertion, and the verification of the biometric (photo) by a trusted agent. The focus of FAL is primary on the assertions being passed from the IdP to the RP, so most authentication events occurring at the RP would not affect the FAL of the transaction. 

At FAL3, holder-of-key transactions occur by verifying both the assertion from the IdP as well as the subscriber's presentation of proof of their personal key attested to in the assertion. 

### 6. Brokered Identity Management

Some federated identity architectures are based on brokered identity management [[section 5.1.4]], where a single broker intermediates transactions between registered IdPs and RPs. In this architecture, each entity in the system only has to register with one broker in order to interoperate with everyone else in the system. It also means that an IdP can authenticate a subscriber without knowledge of which RP requested the authentication event.

Recent advances in automated registration processes have made IdP/RP integrations much less onerous than they used to be. It is possible for an IdP and RP to register with each other in a very short amount of time without any manual processes. This has lessened the value of brokered identity architectures, since interoperability can be simple and fast even without a central broker.

While brokered identity management systems may appear to protect privacy by blinding an IdP from an RP, keep in mind that the broker itself is aware of all the parties involved in the transaction, and in some cases can see personally identifiable information about subscribers. Brokered identity management systems do not prevent subscriber tracking all together, they merely shift the ability to track subscribers from the IdPs and RPs to the broker.

Additionally, because brokers have access to active valid identity assertions, they are capable of impersonating subscribers. This increases the risk inherent in the entire architecture, since the broker represents a single point of failure which, if it is compromised, can in turn compromise every participant in the system.

NIST has been promoting privacy-enhancing technology in the brokered identity management space through the [Privacy-Enhanced Identity Federation project](https://nccoe.nist.gov/projects/building-blocks/privacy-enhanced-identity-brokers). This NIST building block outlines a set of goals which would constitute a new kind of brokered architecture. This architecture leverages a broker which cannot impersonate or track subscribers. This architecture is still theoretical, and may allow for a privacy-preserving and secure version of brokered identity management in the future.

### 7. Educational Resources

All specifications for identity federation standards are freely available online:

[OAuth 2 (RFC 6749)](https://tools.ietf.org/html/rfc6749])  
[OpenID Connect](http://openid.net/connect/)  
[Security Assertion Markup Language](http://saml.xml.org/saml-specifications)

These specifications outline multiple, sometimes mutually exclusive ways to implement federated identity, so be sure to read the specifications in their entirety before creating an implementation.

Federation standards communities actively track known vulnerabilities in existing standards. 

The IETF lists OAuth Security Concerns in in [RFC 6819](https://tools.ietf.org/html/rfc6819) and hosts a [working group](https://tools.ietf.org/wg/oauth/) to track OAuth standards and vulnerabilities.

The OpenID Foundation lists OpenID Connect security concerns [within the specification itself](http://openid.net/specs/openid-connect-core-1_0.html#Security) and hosts a [working group](http://openid.net/wg/connect/) which actively tracks vulnerabilities.

OASIS has published [SAML Privacy and Security Considerations](http://docs.oasis-open.org/security/saml/v2.0/saml-sec-consider-2.0-os.pdf) and hosts a [mailing list](https://lists.oasis-open.org/archives/security-services/) to track SAML vulnerabilities.

### 8. Communicating with Stakeholders

Stakeholders should be aware that selecting an FAL is part of a larger risk- and resource-management process. While it is tempting for stakeholders to request the highest level of security, that is not always in the best interest of the organization. Federated identity projects at higher FALs can be long and complicated, and such complications can take resources away from other work that a security team could be doing that would be of greater benefit to the organization.

Many organizations today operate at FAL1, which is sufficient for most use cases. FAL1 is the industry standard, and there are many libraries and off-the-shelf products that can help an organization implement an FAL1 conformant federated identity system.

Conformance to FAL2 or FAL3 is appropriate for some business cases where there is a risk of fraudulent activity which would be prevented by token encryption, or when the transactions protected by the login is of particularly high value to warrant the additional complexity.

### 9. Conclusion

There are many ways to secure federated identity transactions, and many products do a very adequate job of it today, but there are ways to improve your security posture with regard to federation if doing so makes sense in a overall risk management framework.