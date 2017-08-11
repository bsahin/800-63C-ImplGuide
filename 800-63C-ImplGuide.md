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

Selecting and conforming to an FAL should be part of a larger risk management process and program. Conforming to FAL3 does not make your organization's security infallible. The risks of implementing a system at FAL1 or below may be negligible depending on relevant use cases and attack vectors.

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

Different flows are appropriate for different kinds of applications at different FALs.

##### Kerberos

### Guidance for Identity Providers

The security of all users rests on the security of their identity provider. The guidance below will indicate how to implement strong security.

#### Purpose

IdPs have the difficult task of keeping track of many RPs and deciding how much trust to bestow on each one. This guidance is intended to highlight best practices to accomplish that objective.

#### General Guidance

Support discovery and easy registration.

#### Guidance by Product Family

There are three main product families that enable federated identity transactions - SAML, OAuth/OpenID Connect, and Kerberos

##### SAML

Publish metadata in a well-known location. Apply best practices to protect user information. All pii must conform to FAL 2.

##### OAuth and OpenID Connect

IdPs at FAL 1 must never pass pii in the authentication assertion, but it's fine to make it available at an API endpoint.

##### Kerberos

### Example Scenarios

Shibboleth and a SAML RP

Vanilla OIDC and an OAuth RP

Parallel Auth

### Educational Resources

OAuth working group  
OIDC working group  
SAML spec

### Communicating with Stakeholders

While it's tempting for stakeholders to request the highest level of security, that is not always in the best interest of the organization. These standards can be cumbersome and expensive to implement. Stakeholders may benefit from awareness of the fact that a majority of large companies are only conformant with FAL1.

### Conclusion

There are many ways to secure federated identity transactions, and many products do a very adequate job of it today, but there are ways to improve your security posture with regard to federation if doing so makes sense in your overall risk management framework.