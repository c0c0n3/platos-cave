IAM Intro Readings
==================
> Broad overviews of the field.

I jotted down below some notes about an intro article on Wikipedia.
Another similar intro is:

* [Identity and Access Management (IAM)][techtarget.iam] on TechTarget.

Also, here's some more background reading from Wikipedia:

* [Authentication][wiki.authentication]
* [Authentication Protocol][wiki.authentication-protocol]
* [Authorisation][wiki.authorisation]


### Wikipedia Article

*Identity management (IdM), also known as identity and access
management (IAM), is a framework of policies and technologies
for ensuring that the proper people in an enterprise have the
appropriate access to technology resources*.
[Says Wikipedia][wiki.iam]...

Below are my notes from the article.

###### Identity management (IdM)
The task of controlling information about users on computers.
Such information typically includes:

* How to authenticate users
* Roles & permissions
* User identities & additional user info

###### Functionality
IdM can involve four basic functions:

* *Pure identity*: create/manage/delete identities
* *User access*, i.e. log-on/authentication
* *Service*: personalised, role-based, online, on-demand, multimedia
(content), presence-based services to users and their devices.
* *Identity federation*: share identities and authenticate users without
passwords.

With identity federation, to access a service at a service provider (SP),
users first have to authenticate with an SP-trusted identity provider (IdP)
that, on successful authentication, sends trusted information about the
user to the SP.

Read more about federated identity on Wikipedia:

* [Federated Identity][wiki.fed-id]
* [Single Sign-on][wiki.sso]
* [Identity Provider][wiki.idp]

Besides the basics, an IdM can also do:

* *Authorisation*: Managing authorisation info that defines what operations
an entity can perform in the context of a specific application.
* *Delegation*: One user can allow another to perform actions on their
behalf.
* *Interchange*: Exchange identity information between identity domains
using e.g. SAML or OpenID Connect.




[techtarget.iam]: https://searchsecurity.techtarget.com/definition/identity-access-management-IAM-system
    "Identity and Access Management (IAM)"
[wiki.authentication]: https://en.wikipedia.org/wiki/Authentication
    "Authentication"
[wiki.authentication-protocol]: https://en.wikipedia.org/wiki/Authentication_protocol
    "Authentication Protocol"
[wiki.authorisation]: https://en.wikipedia.org/wiki/Authorization
    "Authorisation"
[wiki.fed-id]: https://en.wikipedia.org/wiki/Federated_identity
    "Federated Identity"
[wiki.iam]: https://en.wikipedia.org/wiki/Identity_management
    "Identity Management"
[wiki.idp]: https://en.wikipedia.org/wiki/Identity_provider
    "Identity Provider"
[wiki.sso]: https://en.wikipedia.org/wiki/Single_sign-on
    "Single Sign-on"
