OAuth 2.0 Distilled
===================
> RESTful framework for delegated authorisation.

What is OAuth 2? Straight from the [horse's mouth][oauth2-spec]:

> The OAuth 2.0 authorization framework enables a third-party application
> to obtain limited access to an HTTP service, either on behalf of a resource
> owner by orchestrating an approval interaction between the resource owner
> and the HTTP service, or by allowing the third-party application to obtain
> access on its own behalf.

The spec refers to the third-party application as the "client" whereas
the HTTP service is called the "resource server". The resource owner is
often the end user or in general an "entity capable of granting access
to a protected resource". The orchestra director hinted at above is
the "authorisation server".


### The basic idea

A resource server hosts some resources (data and services) that belong
to a resource owner. An app, the client, wants to access some of those
resources---what the client wants to do on which resources is referred
to as the access scope. OAuth 2 is a protocol that specifies ways in
which the resource owner can grant the client the requested access scope
for a limited period of time. This happens through an intermediary, the
authorisation server, that can authenticate the resource owner and knows
which clients can be granted which access scopes. If the resource owner
grants the client the requested access scope, the authorisation server
issues the client with a token the client can then use to access only
the specified resources on the resource server and for a set period of
time. In fact, on receiving a request from the client, the resource
server fulfils it only if it contains a valid token---i.e. the server
checks scope, expiry, etc. Here's a diagram to show the *conceptual*
protocol flow for a common scenario where the client asks the resource
owner for some access scope which the resource owner grants through
the authorisation server.

![Conceptual protocol flow][conceptual-flow.dia]

For example, think of a scenario where a program (website, mobile app,
etc.), the client, wants to read your calendar events which a cloud
service provider like Google or Apple, the resource server, maintains
for you. The client asks you, the resource owner, to go get permission
from an authorisation sever both the client and the service provider
trust---often, the authorisation server is managed by the same cloud
provider. So you log in with the authorisation server (e.g. Google
login) and give your consent which then the authorisation server turns
into a token the client can then use to read your calendar events.

This is one way in which the resource owner can grant the client the
requested access scope and the flow sketched out in the diagram is a
high-level, *conceptual* explanation of the actual protocol flow that
OAuth 2 calls *Authorisation Code Grant*. There are a few variations on
the theme though. In fact, OAuth 2 defines another three ways in which
the resource owner can grant access to the client: *Implicit Grant*,
*Resource Owner Password Credentials Grant* and *Client Credentials Grant*.
The *Implicit Grant* differs from the *Authorisation Code Grant* in
the mechanism through which the client gets the token, but *conceptually*
the high-level flow is the same as that in the diagram above---yep,
same same but different! The *Client Credentials Grant* is also easily
explained in terms of the above diagram: here the client is also the
resource owner, so step 1 and 2 get collapsed into one. On the other
hand, the *Resource Owner Password Credentials Grant* is a different
way to do the delegation steps 1 and 2 in the diagram: here the resource
owner gives their credentials to the client directly and the client uses
them to get an access token from the authorisation server with the desired
scope---steps 3 to 5 are the same as in the diagram.

While the story so far may be good enough to develop an intuition of
what the heck is OAuth 2, we should try putting some flesh on the bones
of that. So here goes.


### The problem OAuth tackles

As the [spec intro][oauth2-spec.sec-1] points out, customary client-server
authentication doesn't mesh well with third-party authorisation since:

* to access protected resources on the server, you have to authenticate
  with the server using the resource owner's credentials;
* so any third-party app needs the resource owner's credentials to access
  those resources.

This comes with a bag of drawbacks and security threats:

* sharing of credentials with third parties
* resource servers have to support authentication
* owner can't restrict duration or access to a subset of resources 
* owner can't revoke access to an individual third party without
  revoking access to all
* credentials leak in any third party leaves unprotected all data
  accessible through those credentials


### OAuth's solution

OAuth 2 factors out authentication and authorisation from customary
client-server interaction and makes a separate component responsible
for that as we saw earlier in the conceptual protocol flow. This way,
the resource owner can keep their credentials private and decide what
access scope to grant and for how long to each third-party individually.
Since tokens are bound to a scope, hijacking of a token only gives
the attacker access to those resources within the token scope which
is hopefully less than the whole of the resource owner's data. Also,
tokens can be revoked individually in the case of a security breach
in a client, leaving all the other clients unaffected. Finally, resource
servers don't have to support authentication and keep credentials
around which is notoriously an endless source of headaches---imagine
having to do that for dozens of micro-services.


### How it actually happens

The [spec defines][oauth2-spec.sec-4] several ways in which the resource
owner can grant the client scoped, time-limited access to owned resources.
Basically it all boils down to two conceptual, consecutive phases where
the resource owner delegates resource access to the client:

1. the client gets an authorisation grant; and then
2. exchanges the grant for an access token with the authorisation server.

#### Getting an access token
An [authorisation grant][oauth2-spec.sec-1.3] is a piece of information
(credentials) saying: the resource owner agreed to give the client access
to resources within a specified scope. The spec defines several grant
types:

* [Authorisation Code][oauth2-spec.sec-1.3.1]: grant capturing resource
  owner's approval of client access. The client asks the resource owner
  for approval, the resource owner goes to the authorisation server to
  get a grant certifying approval and then passes that grant on to the
  client.
* [Implicit][oauth2-spec.sec-1.3.2]: same concept as above except the
  authorisation server gives the token directly to the resource owner
  instead of the grant: the issuing of a token implies the grant.
* [Resource Owner Password Credentials][oauth2-spec.sec-1.3.3]: grant
  made up by the resource owner's username and password which the client
  gets directly from the resource owner.
* [Client Credentials][oauth2-spec.sec-1.3.4]: grant made up by the
  client's own credentials---typically, the client is also the resource
  owner in this case.

Each grant type determines how (1) and (2) are actually carried out
at the protocol level---i.e. the flow of messages exchanged by the
various parties. We dive into the protocol flows in the section below,
but here's a handy diagram to sum up access delegation at a conceptual
level:

![Conceptual access delegation][access-delegation.dia]

Regardless of the actual message flow, the goal here is for the
client to acquire, a piece of information (credentials), the [access
token][oauth2-spec.sec-1.4], that says: the client may access resources
within the specified scope until the given expiry date. On top of that,
the authorisation server may also bless the client with [refresh
tokens][oauth2-spec.sec-1.5] the client can use to renew an access
token before or after it's expired. Oh my!

#### Using the access token
With a freshly minted access token in hand, the client can happily
make requests to the resource server to get the goods, which the server
should deliver only if the token the client passed along is valid.
But OAuth 2 doesn't say how the authorisation server should send
the token to the client nor does it say how the client should pass
it along to the resource server. Fortunately, there's a companion
spec, [Bearer Token Usage][oauth2-bearer-spec], which is often used
in practice to fill this gap. Here's what a typical message exchange
could look like. First, the authorisation server returns an access
token of type `Bearer`:

    HTTP/1.1 200 OK
    Content-Type: application/json;charset=UTF-8
    Cache-Control: no-store
    Pragma: no-cache

    {
        "access_token":"Im_4.B34r3r.t0k3n",
        "token_type":"Bearer",
        "expires_in":3600,
        "refresh_token":"r3fr3sh-t0k3n"
    }

Then the client extracts it from the response and puts it in the request
it makes to the resource server to e.g. get some data:

    GET /some/data HTTP/1.1
    Host: resource.server
    Authorization: Bearer Im_4.B34r3r.t0k3n

The bearer authorisation scheme you see above is actually defined by
the [Bearer Token Usage][oauth2-bearer-spec] spec itself, which also
caters for another two methods the client can use to make a request
to the resource server with a bearer token: `POST` it as a form-encoded
body parameter or add it to the request URI as a query parameter---the
horror!

Whether you use bearer token or another mechanism to shuttle access
tokens to the resource server, once received, you may be wondering,
how the heck is the server supposed to extract information from it
and validate it in order to enforce access control?! Again, OAuth 2
says all this isn't in scope, but using the methods detailed in the
below specs are popular choices to get the job done:

* [JSON Web Token][jwt] (JWT, pronounced "jot"): encode token information
  in a digitally signed JSON object with well-known fields.
* [OAuth 2.0 Token Introspection][oauth2-introspection-spec]: query the
  authorisation server to find out all you need to know about the token.


### Diving into the protocol flows

So we saw there are four authorisation grant types. Correspondingly,
there are four protocol flows to specify how the resource owner can
grant the client scoped, time-limited access to owned resources. To
rehash, this boils down to two conceptual, consecutive phases where
the resource owner delegates resource access to the client:

1. the client gets an authorisation grant; and then
2. exchanges the grant for an access token with the authorisation server.

To get the job done, HTTP messages flow through two endpoints at the
authorisation server and one at the client. The server endpoints are:

* **Authorisation**. Caters to the process of getting an authorisation
  grant from the resource owner.
* **Token**. Where the client exchanges an authorisation grant for an
  access token, typically with client authentication.

And the client endpoint:

* **Redirection**. Where the authorisation server can return authorisation
  grants to the client.

It's time we looked at each flow in detail, so take a deep breath before
diving in!

#### Authorisation code grant
In [this flow][oauth2-spec.sec-4.1], the client asks the resource owner
for access scope approval, the resource owner goes to the authorisation
server to get a grant certifying approval and then passes that grant on
to the client which finally gives it back to the authorisation server in
exchange for an access token. The diagram below spells it out, showing a
typical interaction in terms of HTTP messages over the wire.

![Authorisation code flow][auth-code-flow.dia]

The flow begins with step 1 where the client sends the resource owner's
user agent to the authorisation endpoint URI. The client first builds
the URI from the base authorisation endpoint URI (e.g. `https://AS/auth`)
by appending two mandatory query parameters: `response_type=code` and
`client_id` set to the client ID known to the authorisation server---e.g.
`https://AS/auth?response_type=code&client_id=C`. Typically, the client
also adds other, non-mandatory, parameters: `scope`, `redirect_uri`
(i.e. the redirection endpoint URI) and a `state` value the client
can use to maintain state during the whole process. With the complete
URI in hand, the client can issue an HTTP redirect or perhaps return
a web page where it asks the resource owner to get authorised for the
desired scope---the page could have a button or link to the authorisation
URI as shown in the diagram. How exactly this happens is up to the client.
Also OAuth 2 doesn't say how the heck the server is even supposed to
know about client ID, redirection URI and scopes---it only says that
stuff must be agreed on during an upfront registration process, see
the section below about it.

Anyway, the user agent eventually makes a GET request using the
authorisation URI specified by the client---step 2 in the diagram.
The server checks the URI parameters, authenticates the resource
owner and gets approval. Again, OAuth 2 doesn't really say how this
is supposed to happen, but it does say that at the end of the process,
the server issues an HTTP redirect to the client redirection endpoint
URI to which the server appends a `code` query parameter containing
the authorisation code grant. Also, if the client specified a `state`,
that too gets added, as is, to the URI. So we've come to the end of
step 3 in the diagram.

Note the HTTP `303` status code in the server response in step 3.
[According to the HTTP 1.1 spec][http1.1-303-code], on seeing a `303`
the user agent is supposed to do a `GET` on the URI specified in the
response's `Location` header which you can see happen in step 4. Also
with a `303`, the user agent is supposed to drop the body of the original
request when following the redirect to the client redirection endpoint
which avoids leaking resource owner's credentials to the client. In fact,
if the resource owner authenticated with a POST request containing their
credentials in the body (as in step 3), that data may be passed on to
the client if the authorisation sever uses a different redirection code,
e.g. `307`. (You can read more about how redirection codes can jeopardise
OAuth 2 security in [this paper][sec-analysis-of-oauth2].)

Now with the authorisation code grant in hand, the client is ready to
request the coveted access token so it hits the authorisation server's
token endpoint with a POST. Step 5 in the diagram shows how that happens
in the case the client is supposed to authenticate with the server---a
so called *confidential* client. In this case, the client has both an
ID and a secret that were set up on the authorisation server during an
upfront registration process---see the section below about registration.
The client authenticates with the server using the HTTP Basic scheme
so it encodes ID and secret in base 64 and stashes them away in the
`Authorisation` header. (Some variations on the theme are possible here,
e.g. the client could add credentials to the request body instead or
could have arranged with the server to authenticate by some other means.
Read the spec for the details.) The request body has to contain these
two parameters: `grant_type=authorization_code` and `code` set to the
authorisation code grant. If everything checks, the server returns an
access token and possibly a refresh token.

It could be that the client isn't required to authenticate at all
(what OAuth 2 calls a *public* client) in which case there would be
no credentials in the header or request body, but the request body
will still have to have a `client_id` parameter in it set to the ID
known to the server. Also, if the client used a `redirect_uri` parameter
when sending the user agent to the authorisation sever (step 1 in the
diagram), it has to add the exact same parameter again in the request
body.

There are a number of small details I left out, like error messages,
but at this point you should be able to pick up easily from the spec
whatever I left out...

#### Implicit grant
[This flow][oauth2-spec.sec-4.2] is quite similar to the authorisation
code flow, except the authorisation server gives the token directly to
the resource owner instead of the grant: the issuing of a token implies
the grant. If you got the hang of the authorisation code flow, you should
be able to just read the spec to figure out how this flow works.
But keep in mind the implicit flow isn't recommended anymore, you should
rather use the authorisation code flow with PKCE---see
[here][oauth2-implicit-dead] and [here][oauth2-4-mobile-n-native].

#### Resource owner password credentials grant
You could say [this flow][oauth2-spec.sec-4.3] is a well-trodden path
to token bliss: the resource owner gives directly to the client their
username and password (think client login form) and the client uses
them in a straight request to the authorisation server's token endpoint
to get the token. No redirection madness here, this is the kind of
flow most of us are used to---think token = session key. If you'd like
to use this flow, you can easily pick up the details from the spec,
but keep in mind this kind of flow is only suitable when the resource
owner trusts the client...with their life!

#### Client credentials grant
[This flow][oauth2-spec.sec-4.4] is even simpler than the previous one.
Here the client comes with its own credentials which it uses to make a
direct request to the authorisation server's token endpoint to get the
token. Typically in this scenario, the client is also the resource owner.
Easy peasy lemon squeezy! 


### Extending OAuth 2 and OpenID Connect

Authorisation grants come in four flavours but if none suits your taste,
there's an extension mechanism to define [new grant types][oauth2-spec.sec-4.5]
and [protocol flows][oauth2-spec.sec-8].

[OpenID Connect][oidc] extends OAuth 2 with authentication and identity
management but takes a different route: it tweaks the existing OAuth 2
authorisation flows. This is basically [done][oidc1-spec] by specifying
new values for the `response_type` (=`id_token`) and `scope` (=`openid`)
parameters and how an authorisation server should handle requests where
these parameters contain those OpenID Connect values, so that it can
issue identity tokens over and above access tokens.
On a side note, OpenID Connect also defines the `offline_access` scope
to let clients access user info even when the user isn't logged in but,
in [the wild][keycloak-admin-offline-access], this feature is also used
(abused, rather?) by authorisation servers to issue tokens the client
can use to carry out arbitrary "offline** actions on behalf of the user
at the resource server.


### Client registration

As you can imagine, to raise bar for attackers, the authorisation
server has to be able to trust the client. But how does it get to
trust the client? Well, that's supposed to happen through an upfront
registration process which, wait for it, is not covered by the spec!
Oh dear. Anyhoo, come what may, at the end of the registration process
the authorisation server should have:

* A **client identifier**. The server assigns a unique (to the server)
  string to the client, the client ID. This ID is *not* a secret and
  can't be used alone for client authentication.
* **Client credentials**, if the client is to be authenticated. The server
  and the client have to agree on which authentication method to use
  at registration time.
* Client **redirection URIs**.

Also at registration time the authorisation server should determine
what kind of client it's dealing with. OAuth 2 defines two client
types:

* **Confidential**. The client can keep its credentials secret or is
  able to authenticate securely by other means. Typically the client
  runs on a secure server with restricted access to credentials.
* **Public**. The client isn't able to keep its credentials secret and
  can't authenticate securely by any other means. Typically the client
  runs on the resource owner's device---e.g. mobile or browser-based app.

In practice which type the client gets labelled with for registration
purposes pretty much depends on whether or not it's acceptable to
expose client credentials.

Also note that a client may well be a whole distributed system. In that
case you actually may want to have more than one client: each component
gets registered as a separate client with a different client type and
security context---e.g. confidential server web app and public JavaScript
front end.


### But which flow to use when?!

I've heard some devs say you should only ever use the authorisation
code grant to which others promptly replied: [Yeah, well, you know,
that's just like uh, your opinion man][the-dude]. In practice which
flow you decide to go with really depends on the situation at hand
and trade-offs you can afford to make, like security vs simplicity.
But here's a [handy table][apigee-oauth2-usecases] from the Apigee
docs to get you started thinking about use cases and trade-offs.
Also, I should mention I really enjoyed watching [OAuth 2.0 and OpenID
Connect (in plain English)][oidc-in-plain-en]---slides over
[here][oidc-in-plain-en.slides]. The guy makes an awesome job at
explaining in simple terms what the heck OAuth 2.0 and OpenID Connect
are and gives some advice on what flow to use when---if you don't have
time to watch the vid, perhaps just look at slides 33 through 35.


### Is OAuth 2 secure?

I don't think there's a straight answer to that question. The spec is
quite vague about so many things and leaves so much out that at the end
of the day security really depends on the quality of the software you
deploy and how wisely you configure it to operate in production. I reckon
OAuth 2's lead author and editor [puts it quite well][road-to-hell]:

> To be clear, OAuth 2.0 at the hand of a developer with deep understanding 
> of web security will likely result is a secure implementation. However,
> at the hands of most developers — as has been the experience from the
> past two years — 2.0 is likely to produce insecure implementations.

To be fair though, the spec does provide some basic guidance such as

* Use TLS to secure interactions.
* Ignore empty/unrecognised parameters.
* Never duplicate parameters in requests/responses.
* Avoid URIs with a fragment component.
* Use absolute redirection URIs.
* Make sure all clients register their redirection endpoints. Also, ideally
  clients should register URI scheme/authority/path so that they can only
  change the query component dynamically when requesting authorisation.

You can find more advice in the [Security Considerations][oauth2-spec.sec-10]
section. Also of note, under some assumptions the protocol can actually
be made bomb proof: [A Comprehensive Formal Security Analysis of OAuth
2.0][sec-analysis-of-oauth2] explains how and shows how OAuth 2 with
security recommendations and best practices in place can provide strong
authorisation, authentication, and session integrity guarantees.


### Conclusion

* TODO: further reading
* TODO: play w/ it! -> keycloak




[access-delegation.dia]: ./access-delegation.png
[apigee-oauth2-usecases]: https://docs.apigee.com/api-platform/security/oauth/oauth-introduction#summaryofoauth20usecases
    "Apigee - Summary of OAuth 2.0 use cases"
[auth-code-flow.dia]: ./authorisation-code-flow.png
[conceptual-flow.dia]: ./oauth2-conceptual-flow.png
[http1.1-303-code]: https://tools.ietf.org/html/rfc7231#section-6.4.4
    "HTTP/1.1 - 6.4.4.  303 See Other"
[jwt]: https://tools.ietf.org/html/rfc7519
    "JSON Web Token (JWT)"
[keycloak-admin-offline-access]: https://www.keycloak.org/docs/latest/server_admin/index.html#_offline-access
    "Keycloak Server Admin - 13.4. Offline Access"
[oauth2-4-mobile-n-native]: https://developer.okta.com/blog/2018/12/13/oauth-2-for-native-and-mobile-apps
    "OAuth 2.0 for Native and Mobile Apps"
[oauth2-bearer-spec]: https://tools.ietf.org/html/rfc6750
    "The OAuth 2.0 Authorization Framework: Bearer Token Usage"
[oauth2-implicit-dead]: https://developer.okta.com/blog/2019/05/01/is-the-oauth-implicit-flow-dead
    "Is the OAuth 2.0 Implicit Flow Dead?"
[oauth2-introspection-spec]: https://tools.ietf.org/html/rfc7662
    "OAuth 2.0 Token Introspection"
[oauth2-spec]: https://tools.ietf.org/html/rfc6749
    "The OAuth 2.0 Authorization Framework"
[oauth2-spec.sec-1]: https://tools.ietf.org/html/rfc6749#page-4
    "The OAuth 2.0 Authorization Framework - Introduction"
[oauth2-spec.sec-1.3]: https://tools.ietf.org/html/rfc6749#section-1.3
    "The OAuth 2.0 Authorization Framework - Authorization Grant"
[oauth2-spec.sec-1.3.1]: https://tools.ietf.org/html/rfc6749#section-1.3.1
    "The OAuth 2.0 Authorization Framework - Authorization Code"
[oauth2-spec.sec-1.3.2]: https://tools.ietf.org/html/rfc6749#section-1.3.2
    "The OAuth 2.0 Authorization Framework - Implicit"
[oauth2-spec.sec-1.3.3]: https://tools.ietf.org/html/rfc6749#section-1.3.3
    "The OAuth 2.0 Authorization Framework - Resource Owner Password Credentials"
[oauth2-spec.sec-1.3.4]: https://tools.ietf.org/html/rfc6749#section-1.3.4
    "The OAuth 2.0 Authorization Framework - Client Credentials"
[oauth2-spec.sec-1.4]: https://tools.ietf.org/html/rfc6749#section-1.4
    "The OAuth 2.0 Authorization Framework - Access Token"
[oauth2-spec.sec-1.5]: https://tools.ietf.org/html/rfc6749#section-1.5
    "The OAuth 2.0 Authorization Framework - Refresh Token"
[oauth2-spec.sec-4]: https://tools.ietf.org/html/rfc6749#section-4
    "The OAuth 2.0 Authorization Framework - Obtaining Authorization"
[oauth2-spec.sec-4.1]: https://tools.ietf.org/html/rfc6749#section-4.1
    "The OAuth 2.0 Authorization Framework - Authorization Code Grant"
[oauth2-spec.sec-4.2]: https://tools.ietf.org/html/rfc6749#section-4.2
    "The OAuth 2.0 Authorization Framework - Implicit Grant"
[oauth2-spec.sec-4.3]: https://tools.ietf.org/html/rfc6749#section-4.3
    "The OAuth 2.0 Authorization Framework - Resource Owner Password Credentials Grant"
[oauth2-spec.sec-4.4]: https://tools.ietf.org/html/rfc6749#section-4.4
    "The OAuth 2.0 Authorization Framework - Client Credentials Grant"
[oauth2-spec.sec-4.5]: https://tools.ietf.org/html/rfc6749#section-4.5
    "The OAuth 2.0 Authorization Framework - Extension Grants"
[oauth2-spec.sec-8]: https://tools.ietf.org/html/rfc6749#section-8
    "The OAuth 2.0 Authorization Framework - Extensibility"
[oauth2-spec.sec-10]: https://tools.ietf.org/html/rfc6749#section-10
    "The OAuth 2.0 Authorization Framework - Security Considerations"
[oidc]: https://openid.net/connect/
    "OpenID Connect Home"
[oidc1-spec]: https://openid.net/specs/openid-connect-core-1_0.html
    "OpenID Connect Core 1.0"
[oidc-in-plain-en]: https://www.youtube.com/watch?v=996OiexHze0
    "OAuth 2.0 and OpenID Connect (in plain English)"
[oidc-in-plain-en.slides]: https://speakerdeck.com/nbarbettini/oauth-and-openid-connect-in-plain-english
    "OAuth 2.0 and OpenID Connect (in plain English) - slides"
[road-to-hell]: https://hueniverse.com/oauth-2-0-and-the-road-to-hell-8eec45921529
    "OAuth 2.0 and the Road to Hell"
[sec-analysis-of-oauth2]: https://arxiv.org/pdf/1601.01229.pdf
    "A Comprehensive Formal Security Analysis of OAuth 2.0"
[the-dude]: https://www.youtube.com/watch?v=pWdd6_ZxX8c
