# Federation Operations Guide

## Table of Contents

1. [Executive Summary](#executive-summary)
1. [Introduction](#introduction)  
1. [Threat Models](#threat-models)
1. [Trust Models](#trust-models)
1. [Risk Management](#risk-management)
1. [Selecting an FAL](#selecting-an-fal)  
1. [Guidance for Relying Parties](#guidance-for-relying-parties)  
 7.1 [Purpose](#purpose)  
 7.2 [General Guidance](#general-guidance)  
 7.3 [Guidance by Product Family](#guidance-by-product-family)  
     * 7.3.1 [SAML](#saml)  
     * 7.3.2 [OAuth](#oauth) 
     * 7.3.3 [OpenID Connect](#openid-connect)
     * 7.3.4 [Kerberos](#kerberos)
1. [Guidance for Identity Providers](#guidance-for-identity-providers)  
 8.1 [Purpose](#purpose-1)  
 8.2 [General Guidance](#general-guidance-1)  
 8.3 [Guidance by Product Family](#guidance-by-product-family-1)  
     * 8.3.1 [SAML](#saml-1)  
     * 8.3.2 [OAuth](#oauth-1) 
     * 8.3.3 [OpenID Connect](#openid-connect-1)
     * 8.3.4 [Kerberos](#kerberos-1)
1. [Unique Scenarios](#unique-scenarios)
1. [Training](#training)
1. [Communicating with Stakeholders](#communicating-with-stakeholders)
1. [Conclusion](#conclusion)

### Executive Summary

Always check signatures and certificates. Always be careful with pii. Follow principles of least priviledge and minimal disclosure.

### Introduction

Federated identity transactions allow for a more secure more usable internet. Many products and libraries exist which enable those transactions at various levels of security. We'll be talking about what to look for in software that enables federation.

### Trust Models

There are lots of existing SAML federations you could join like inCommon.

While SAML can only maintain white and blacklists, it is possible for IdPs using OAuth and OpenID connect to have greylists which enable them to make more nuanced security decisions.

### Risk Management

Selecting and conforming to an FAL should be part of a larger risk management process and program. Conforming to FAL3 does not make your organization's security infallible. Rather than attempting to make your federation infrastructure conform to the highest standards available, you should analyze the risks that are inherent in your organization and choose how strongly to protect against them given their severity and liklihood of occurance.

FAL 1 is the industry standard for authentication at this point in time. The risks of implementing a system at FAL1 or below may be negligible depending on relevant use cases and attack vectors.

Because it is the front door to many critical systems, authentication is a key piece of risk management strategy. Strong federation can protect against many potential user impersonation and man-in-the-middle attacks. If those threats are common to your organization, you should consider implementation of FAL2 or FAL3.

### Selecting an FAL

FAL1 is absolutely fine for a vast majority of use cases. FAL2 provides extra security. FAL3 is an extrememly secure forward-thinking standard that is not yet in production in off-the-shelf products.

### Guidance for Relying Parties

Relying parties need to validate a variety of elements during a federated transaction. The guidance below will indicate how to do that.

#### Purpose

Relying parties can be a valuable target for attackers to impersonate valid users or gain valuable information about them. The security of the entire ecosystem is dependent upon the security of relying parties.

#### General Guidance

Relying parties need to validate IdP signatures, assertion expirations, and audience parameters.

#### Guidance by Product Family

There are three main product families that enable federated identity transactions - SAML, OAuth/OpenID Connect, and Kerberos

##### SAML

Be careful about passing and validating metadata. Always check certificates.

##### OAuth and OpenID Connect

Different OAuth grant types or "flows" are appropriate for different kinds of applications at different FALs.

The authorization code flow should be used whenever possible. It is the most common and most secure way to implement OAuth. It can accomodate all three FALs depending on the exact configuration of the application.

The implicit grant type is appropriate for applications which are implemented entirely in front-end code and have to capability to store secrets outside of the user's web browser. The lack of ability to store secrets means that these sorts of applications can only function at FAL1 because they have no method of private key management which would enable encryption of identity assertions.

The client credentials grant type is not recommended, since it does not require user interaction. Because applications using this grant type are capable of securely storing keys, these applications can function at FAL1 and FAL2. However, since no user interaction takes place during the authentication event, a holder-of-key proof cannot occur, preventing these sorts of applications from functioning at FAL3.

The resource owner credentials grant type does not qualify for FAL1, FAL2, or FAL3, and should be avoided.

### Guidance for Identity Providers

The security of all users rests on the security of their identity provider. The guidance below will indicate how to implement strong security.

#### Purpose

IdPs have the difficult task of keeping track of many RPs and deciding how much trust to bestow on each one. This guidance is intended to highlight best practices to accomplish that objective.

#### General Guidance

Much of the technical friction in identity transactions stems from IdPs which are built and configured in such a way that onboarding new RPs requires a significant amount of manual human intervention.

Much of this friction is removed when IdPs support known discovery mechanisms and simple registration.

#### Guidance by Product Family

There are three main product families that enable federated identity transactions - SAML, OAuth/OpenID Connect, and Kerberos

##### SAML

Publish metadata in a well-known location. While there is no widely accepted standard for SAML metadata exchange, it is advisable to use a well-documented metadata endpoint to serve your IdPs metadata in the form of a single XML file to any RP who wishes to consume it.

Only accept metadata that has been signed by the presenting RP, and always check the signature on that metadata.

Identity federations like InCommon share the metadata of hundreds of IdPs and RPs in a structured manner. Adding your IdP's metadata to such federations will help RPs to find it easily.

Apply best practices to protect user information. All SAML assertions containing personally identifyable information must conform to FAL2. That means the assertion must be encrypted to the relying party.

Assertions containing only authentication information and no personally identifiable information do not need to encrypt their assertions. They may operate at FAL1.

##### OAuth and OpenID Connect

OpenID Connect has an established discovery mechanism. Your IdP's discovery documents should be published in JSON format at an HTTPS location ending in /.well-known/openid-configuration as specified in the OpenID Connect discovery specification.

If personally identifiable information is made available at the UserInfo endpoint, it need not be encrypted. However, if personally identifiable information is bundled with authentication information in a token, it must be encrypted to the RP and conformant with FAL2.

It is recommended that OpenID Connect IdPs support a registration endpoint to make it easy for RPs to register without manual intervention.

There is a test suite for OpenID Connect providers to verify whether their instance of OpenID Connect is compliant with the standard. That test is located at [http://openid.net/certification/testing/](http://openid.net/certification/testing/). If your provider passes these tests, you have the option of becoming a self-certified OpenID Connect provider. This will help other participants in the OpenID Connect ecosystem find you, and find out the exact configuration of your system.

The OpenID Connect community has reviewed libraries in several different languages to search for bugs and non-compliant processes.Whenever possible, you should leverage [these libraries](http://openid.net/developers/libraries/) in your development.
 
OpenID Connect relies heavily on the [JOSE standard](https://datatracker.ietf.org/wg/jose/about/), particularly JWT and JWK. All tokens an keys in your implementation should conform to those standards.

You may also consider implementation of additional security standards like [token binding](https://datatracker.ietf.org/wg/tokbind/documents/) and [proof key for code exchange](https://tools.ietf.org/html/rfc7636). While those standards are not factors in determining the FAL of your instance, they considered to be best practice in the industry.

### Example Scenarios

#### Shibboleth and SAML

SAML Federations like InCommon can operate at FAL1 or FAL2. Most InCommon IdPs are running on a Shibboleth identity provider. They pass assertions through a response to an authentication event. Most often, those assertions are not encrypted to the RP.

If you are running a Shibboleth IdP, you must either encrypt your assertions to the RP, or refrain from sending personally identifiable information such as eppn over the wire as an unecrypted SAML assertion.

#### OpenID Connect and OAuth

Typically, OpenID Connect Providers interact with OAuth relying parties by providing an authentication mechanism which is separate from the transfer of personally identifiable information. As such, these providers can safely operate at FAL1 because they are not bundling identity assertions with authentication information.

#### PIV

#### Parallel Authentication

In some cases a relying party may wish to confirm certain aspects of a user's identity above and beyond what the IdP provides. For example, a relying party could verify a user with an IdP, receive a picture of the user, and require that an in-person agent verify that the picture matches the identity of the person authenticating.

We call this use case "parallel authentication" because two authentication events are happening in parallel. The focus of FAL is primary on the assertions being passed from the IdP to the RP, so most authentication events occuring at the RP would not affect the FAL of the transaction. The exception to this rule is a holder-of-key authentication event. If the RP verifies in parallel that the user is in possession of a key that the IdP says they should have, that verification counts as a factor toward FAL3.

### Educational Resources

OAuth working group  
OIDC working group  
SAML spec

### Communicating with Stakeholders

While it's tempting for stakeholders to request the highest level of security, that is not always in the best interest of the organization. These standards can be cumbersome and expensive to implement. Stakeholders may benefit from awareness of the fact that a majority of large companies are only conformant with FAL1.

### Conclusion

There are many ways to secure federated identity transactions, and many products do a very adequate job of it today, but there are ways to improve your security posture with regard to federation if doing so makes sense in your overall risk management framework.