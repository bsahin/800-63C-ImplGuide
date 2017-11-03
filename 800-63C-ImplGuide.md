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

Federated identity transactions allow for a more secure and more usable internet by allowing users to have a smaller number of accounts that can be used across many sites and applications, without sharing credentials. There are several major protocols that enable federation transactions, and a multitude of software packages and libraries that implement them. This document outlines what to look for in software that enables federation and how to apply that.

This document is intended to provide more direct guidance than SP 800-63C, which was written to be intentionally technology-agnostic. While this choice is makes the document applicable across a wide array of technologies and circumstances, the abstract nature can make it difficult for implementers to understand what was intended by the document with regard to specific protocols or products. This guide is therefore intended to provide more concrete guidance. 

### 2. Choosing Security Parameters

Different federation protocols and implementations of those protocols have many options that lead to different outcomes in the security of a system. All of these options have trade-offs in terms of complexity, robustness, and other characteristics. Choosing the right set of options for a given situation helps ensure that transactions will be as secure, functional, and efficient as possible.

### 2.1. Selecting a Federation Assurance Level (FAL)

The Federation Assurance Level (FAL) defined in SP 800-63C provides a set of requirements for federation transactions. These requirements are grouped into an ascending scale of three levels: FAL1, FAL2, and FAL3. Each sucessive level includes all the features of lower levels and adds additional requirements on top of them. 

FAL1 provides a solid basis for federation and is appropriate for the vast majority of use cases. Most off-the-shelf products operate at FAL1 today without additional features. Most common deployments of SAML and OpenID Connect fulfill all the requirements of FAL1 today. FAL1 requires that all assertions be signed, time limited, and audience-restricted to prevent an assertion from being replayed between relying parties. While assertions at FAL1 can't contain personally identifiable information, such information can still be transmitted to the RP via a secondary channel and still be compliant with FAL1. For example, in OpenID Connect the ID Token serves as the assertion, and by default it contains a non-PII subject identifier for the user. Additional information about the user can be obtained through the UserInfo Endpoint by using an OAuth access token. Since this information is communicated directly from the IdP to the RP over a an encrypted channel, it need not be separately encrypted. 

FAL2 provides an extra layer of security by requiring that the assertion be encrypted so that the RP is the only party that can decrypt it. This protection is required if personally identifiable information will be sent in an identity assertion, as personally identifiable information must always be encrypted in transit and an assertion is often borne by intermediate parties in a transaction. This level reqires that the IdP manage an encryption key for the RP, which necessitates additional complexity for both parties. The keys could be managed with a traditional PKI infrstructure that relies on a trusted certificate authority, but with many protocols the keys can be instead registered directly between parties. The RP also needs to manage and protect its decryption keys in order to read the information in the assertion. If these keys are leaked to another party, the additional protections provided by FAL2 no longer apply. PII can still be sent in a secondary channel. 

FAL3 is intended to be forward-looking and is not yet available in off-the-shelf standards and products. FAL3 provides an additional layer in the form of a cryptographic key that must be presented by the user in addition to the signed and encrypted assertion itself. This level requires that the IdP manage references to keys representing the user at each RP in addition to managing the keys for the RPs themselves. The IdP needs to correctly associate the user's key to the correct RP in the assertion, and the RP needs to be able to process and validate the presentation of the key by the user.

### 2.2. Risk Management

Selecting and conforming to an FAL should be part of a larger risk management process and program. Conforming to FAL3 does not make your organization's security infallible, but instead provides protection against particular attacks while incurring certain costs to both the applications and the users. Rather than attempting to make your federation infrastructure conform to the highest standards available, you should analyze the risks that are inherent in your organization and choose how strongly to protect against them given their severity and liklihood of occurance.

The additional information management and implementation complexity of higher FALs cannot be ignored, and the costs to all involved must be weighed against perceived benefits. Unless there is a compelling reason to use the features of higher FALs, FAL1 should be the default for most use cases. FAL1 is the industry standard for authentication at this point in time. The risks of implementing a system at FAL1, when compared to higher FALs, may be negligible depending on relevant use cases and attack vectors. When PII needs to be passed, it should be passed in a secondary channel where possible instead of in the assertion itself, regardless of the FAL.

Because it is the front door to many critical systems, authentication is a key piece of risk management strategy. Strong federation can protect against many potential user impersonation and man-in-the-middle attacks. Instead of each RP needing to manage user accounts and credentials separately, creating many vulnerable surfaces, federation concentrates the key security practices in a dedicated component, the IdP. Upgrades to credentials, software, and practices at the IdP automatically benefit the downstream RPs and the overall network. 

### 4. Guidance for Relying Parties

Relying parties need to validate a variety of elements during a federated transaction. Because relying parties depend on endpoints that are open to the internet, it is essential that they verify the origin and validity of the information that they recieve.

#### 4.1 Purpose

Relying parties can be a valuable target for attackers to impersonate valid users or gain valuable information about them. If relying parties do not check the validity of the information they receive, attackers can gain valuable access. The security of the entire ecosystem is dependent upon the security of relying parties.

#### 4.2 General Guidance

Relying parties need to validate IdP signatures, assertion expirations, and audience parameters. Additionally, RPs need to test that these validation checks are working at all times because there will be no outward indication that something is wrong with the system until an attack occurs.

##### 4.2.1 Validating IdP Signatures

An identity assertion is signed by an IdP so that it cannot be forged by an attacker. The IdP is the only entity with access to its private key, so a valid signature indicates that the assertion is from the IdP itself and not an attacker. If an RP does not check the validity of the IdP signature, attackers will be able to forge identity assertions, and gain access to protected systems without authorization. 

It is critically important to check whether your RP will accept unsigned assertions or assertions signed with an invalid key. Properly authorized transactions will still work even if an RP isn't checking assertion signatures, so there is no outward indication of a problem in the system. There will be no error messages or login failures to indicate that something is wrong.

##### 4.2.2 Checking Assertion Expirations

Federated identity assertions are intended to be short-lived. An identity assertion which expires quickly makes it difficult for attackers to misuse the assertion. It also ensures that any identity or authorization information included in the assertion is not out-of-date. Authorized transactions can occur at an RP that is not checking assertion expirations, but that RP will be significantly less secure. Always test your system with an expired assertion to make sure that it is not accepted as valid by the RP.

##### 4.2.3 Checking Audience Parameters

When an IdP creates an assertion, it includes an audience parameter indicating which RP requested the assertion. This parameter is intended to reduce fraud by making it obvious when an attacker is replaying an assertion at a different RP. 

If an RP does not check for a matching audience parameter, it is possible for an attacker to get a valid assertion from any RP registered with the IdP, and replay it to gain unauthorized access to a different RP.

It is critically important to check whether your RP will accept assertions with a missing or incorrect audience parameter. Properly authorized transactions will still work even if an RP isn't checking audience parameters, so there is no outward indication of a problem in the system. There will be no error messages or login failures to indicate that something is wrong.

#### 4.3 Guidance by Product Family

There are two main product families that enable federated identity transactions - SAML and OAuth/OpenID Connect.

##### 4.3.1 SAML

Be careful about passing and validating metadata. Incorrectly communicated or configured metadata could leak information about a user that was not approved for distribution. Metadata that is not validated could have been tampered with by an attacker to gain access to valuable personal information.

Always check certificates before accepting identity assertions. Attackers can forge certificates and phish users in an attempt to impersonate them. 

##### 4.3.2 OAuth and OpenID Connect

Different OAuth grant types or "flows" are appropriate for different kinds of applications at different FALs.

The authorization code flow should be used whenever possible. It is the most common and most secure way to implement OAuth. It can accomodate all three FALs depending on the exact configuration of the application.

The implicit grant type is appropriate for applications which are implemented entirely in front-end code and have to capability to store secrets outside of the user's web browser. The lack of ability to store secrets means that these sorts of applications can only function at FAL1 because they have no method of private key management which would enable encryption of identity assertions.

The client credentials grant type is not recommended, since it does not require user interaction. Because applications using this grant type are capable of securely storing keys, these applications can function at FAL1 and FAL2. However, since no user interaction takes place during the authentication event, a holder-of-key proof cannot occur, preventing these sorts of applications from functioning at FAL3.

The resource owner credentials grant type does not qualify for FAL1, FAL2, or FAL3, and should be avoided.

### 5. Guidance for Identity Providers

The security of all users rests on the security of their identity provider. The guidance below will indicate how to implement strong security.

#### 5.1 Purpose

IdPs have the difficult task of keeping track of many RPs and deciding how much trust to bestow on each one. This guidance is intended to highlight best practices to accomplish that objective.

#### 5.2 General Guidance

Much of the technical friction in identity transactions stems from IdPs which are built and configured in such a way that onboarding new RPs requires a significant amount of manual human intervention.

Much of this friction is removed when IdPs support known discovery mechanisms and simple registration.

#### 5.3 Guidance by Product Family

There are three main product families that enable federated identity transactions - SAML, OAuth/OpenID Connect, and Kerberos

##### 5.3.1 SAML

Publish metadata in a well-known location. While there is no widely accepted standard for SAML metadata exchange, it is advisable to use a well-documented metadata endpoint to serve your IdPs metadata in the form of a single XML file to any RP who wishes to consume it.

Only accept metadata that has been signed by the presenting RP, and always check the signature on that metadata.

Identity federations like InCommon share the metadata of hundreds of IdPs and RPs in a structured manner. Adding your IdP's metadata to such federations will help RPs to find it easily.

Apply best practices to protect user information. All SAML assertions containing personally identifyable information must conform to FAL2. That means the assertion must be encrypted to the relying party.

Assertions containing only authentication information and no personally identifiable information do not need to encrypt their assertions. They may operate at FAL1.

##### 5.3.2 OAuth and OpenID Connect

OpenID Connect has an established discovery mechanism. Your IdP's discovery documents should be published in JSON format at an HTTPS location ending in /.well-known/openid-configuration as specified in the OpenID Connect discovery specification.

If personally identifiable information is made available at the UserInfo endpoint, it need not be encrypted. However, if personally identifiable information is bundled with authentication information in a token, it must be encrypted to the RP and conformant with FAL2.

It is recommended that OpenID Connect IdPs support a registration endpoint to make it easy for RPs to register without manual intervention.

There is a test suite for OpenID Connect providers to verify whether their instance of OpenID Connect is compliant with the standard. That test is located at [http://openid.net/certification/testing/](http://openid.net/certification/testing/). If your provider passes these tests, you have the option of becoming a self-certified OpenID Connect provider. This will help other participants in the OpenID Connect ecosystem find you, and find out the exact configuration of your system.

The OpenID Connect community has reviewed libraries in several different languages to search for bugs and non-compliant processes.Whenever possible, you should leverage [these libraries](http://openid.net/developers/libraries/) in your development.
 
OpenID Connect relies heavily on the [JOSE standard](https://datatracker.ietf.org/wg/jose/about/), particularly JWT and JWK. All tokens an keys in your implementation should conform to those standards.

You may also consider implementation of additional security standards like [token binding](https://datatracker.ietf.org/wg/tokbind/documents/) and [proof key for code exchange](https://tools.ietf.org/html/rfc7636). While those standards are not factors in determining the FAL of your instance, they considered to be best practice in the industry.

### 6. Example Scenarios

#### 6.1 Shibboleth and SAML

SAML Federations like InCommon can operate at FAL1 or FAL2. Most InCommon IdPs are running on a Shibboleth identity provider. They pass assertions through a response to an authentication event. Most often, those assertions are not encrypted to the RP.

If you are running a Shibboleth IdP, you must either encrypt your assertions to the RP, or refrain from sending personally identifiable information such as eppn over the wire as an unecrypted SAML assertion.

#### 6.2 OpenID Connect and OAuth

Typically, OpenID Connect Providers interact with OAuth relying parties by providing an authentication mechanism which is separate from the transfer of personally identifiable information. As such, these providers can safely operate at FAL1 because they are not bundling identity assertions with authentication information.

#### 6.3 Personal Identity Verification (PIV) card 

Federation Assurance Levels are independent of Authenticator Assurance Levels. Any authentication device may be used to obtain any FAL, including but not limited to a Personal Identity Verification (PIV) card.

There are many benefits to using federated identity management as opposed to requiring independent registration of PIV cards for each user. Federated identity management protocols such as OpenID Connect allow users to authenticate at a relying party regardless of which authenticator they use at their IdP. This allows relying parties to securely accept registered users whose IdPs do not use PIV cards.


#### 6.4 Privacy-enhancing Federated Identity

In many cases, relying parties do not need to know the full identity of a user. Relying parties should only request as much information as they need to complete the transaction requested by the user.

Pairwise identifiers should be used in place of perisistent or correlatable identifiiers whenever possible. This prevents relying parties from tracking or identifying individual users.

When possible, claims should be used to communicate identity information rather than raw data. For example, if a relying party needs to know whether a user is over eighteen years old, the IdP should respond that the user is over eighteen without sharing the user's age or birthdate.

#### 6.5 Parallel Authentication

In some cases a relying party may wish to confirm certain aspects of a user's identity above and beyond what the IdP provides. For example, a relying party could verify a user with an IdP, receive a picture of the user, and require that an in-person agent verify that the picture matches the identity of the person authenticating.

We call this use case "parallel authentication" because two authentication events are happening in parallel. The focus of FAL is primary on the assertions being passed from the IdP to the RP, so most authentication events occuring at the RP would not affect the FAL of the transaction. The exception to this rule is a holder-of-key authentication event. If the RP verifies in parallel that the user is in possession of a key that the IdP says they should have, that verification counts as a factor toward FAL3.

### 7. Brokered Identity Management

Some federated identity architectures are based on brokered identity management. In these architectures, a single broker intermediates transactions between registered IdPs and RPs. This means that each entity in the system only has to register with one broker in order to interoerate with everyone else in the system. It also means that an IdP can authenticate a user without knowledge of which RP requested the authentication event.

Recent advances in automated registration processes have made IdP/RP integrations much less onerous than they used to be. It is possible for an IdP and RP to register with each other in a very short amount of time without any manual processes. This has lessened the value of brokered identity architectures, since interoperability can be simple and fast even without a central broker.

While brokered identity management systems may appear to protect privacy by blinding an IdP from an RP, keep in mind that the broker itself is aware of all the parties involved in the transaction, and in some cases can see personally identifiable information about users. Brokered identity management systems do not prevent user tracking all together, they merely shift the ability to track users from the IdPs and RPs to the broker.

Additionally, because brokers have access to active valid identity assertions, they are capable of impersonating users. This increases the risk inherent in the entire architecture, since the broker represents a single point of failure which, if it is compromised, can in turn compromise every participant in the system.

NIST has been promoting privacy-enhancing technology in the brokered identity management space through the [Privacy-Enhanced Identity Federation project](https://nccoe.nist.gov/projects/building-blocks/privacy-enhanced-identity-brokers). This NIST building block outlines a set of goals which would constitute a new kind of brokered architecture. This architecture leverages a broker which cannot impersonate or track users. This architecture is still theoretical, and may allow for a privacy-preserving and secure version of brokered identity management in the future.

### 8. Educational Resources

All specifications for identity federation standards are freely available online:

[OAuth 2 (RFC 6749)](https://tools.ietf.org/html/rfc6749])  
[OpenID Connect](http://openid.net/connect/)  
[Security Assertion Markup Language](http://saml.xml.org/saml-specifications)

These specifications outline multiple, sometimes mutually exlusive ways to implement federated identity, so be sure to read the specifications in their entirety before creating an implementation.

Federation standards communities actively track known vulnerabilities in existing standards. 

The IETF lists OAuth Security Concerns in in [RFC 6819](https://tools.ietf.org/html/rfc6819) and hosts a [working group](https://tools.ietf.org/wg/oauth/) to track OAuth standards and vulnerabilities.

The OpenID Foundation lists OpenID Connect security concerns [within the specification itself](http://openid.net/specs/openid-connect-core-1_0.html#Security) and hosts a [working group](http://openid.net/wg/connect/) which actively tracks vulnerabilities.

OASIS has published [SAML Privacy and Security Considerations](http://docs.oasis-open.org/security/saml/v2.0/saml-sec-consider-2.0-os.pdf) and hosts a [mailing list](https://lists.oasis-open.org/archives/security-services/) to track SAML vulnerabilities.

### 9. Communicating with Stakeholders

While it is tempting for stakeholders to request the highest level of security, that is not always in the best interest of the organization. Federated identity projects can be long and complicated. They can take resouces away from other work that a security team could be doing.

Many organizations today operate at FAL1. FAL1 is the industry standard, and there are many libraries and off-the-shelf products that can help an organization implement an FAL1 conformant federated identity system.

Conformance to FAL2 or FAL3 is appropriate for some business cases where there is a risk of fraudulent activity which would be prevented by token encryption.

Stakeholders should be aware that selecting an FAL is part of a larger risk- and resource-management process.

### 10. Conclusion

There are many ways to secure federated identity transactions, and many products do a very adequate job of it today, but there are ways to improve your security posture with regard to federation if doing so makes sense in your overall risk management framework.