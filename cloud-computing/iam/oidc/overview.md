The Lunar-Orbit Overview
========================
> RESTful frameworks for identity, authentication and delegated authorisation.


### The big picture

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

Cool, but what is OpenID Connect? Again, let the [horse speak][oidc1-spec]:

> OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0
> protocol. It enables Clients to verify the identity of the End-User
> based on the authentication performed by an Authorization Server, as
> well as to obtain basic profile information about the End-User...

And here's a diagram to sum up the most important bits and pieces at
a conceptual level:

![Overview diagram][overview.dia]


### Dive in!

Have you ever heard that the devil is in the details? The picture above
is only good to develop an intuition, but fortunately there's plenty of
resources on the interwebs to learn about this stuff. Here's a hitchhiker
guide to OAuth 2 and OpenID Connect.

For starters:

1. If you like my writing, try: [OAuth 2.0 Distilled][oauth2-distilled]
   (how's that for shameless self promotion?!)
2. Watch [OAuth 2.0 and OpenID Connect (in plain English)][oidc-in-plain-en],
   slides [here][oidc-in-plain-en.slides]
3. Same as above but in a slightly different sauce, if you'd rather read:
   [What the Heck is OAuth?][wot-d-heck-is-oauth]---covers OpenID Connect
   too.
4. This one article only covers OAuth 2 from a conceptual standpoint, but
   is quite nice with lots of diagrams:
   [The Simplest Guide To OAuth 2.0][simplest-oauth2].
5. Also there's a (official?) [getting-started guide][oauth2-simplified].

Main course:

1. Read the specs. No really, I'm not joking!
2. Here are some nice diagrams to visualise the
   [OAuth 2 flows][oauth2-flows-diagrams] and the
   [OpenID Connect ones][oidc-flows-diagrams].
3. The implicit flow isn't recommended anymore, rather use the authorisation
   code flow with PKCE---see [here][oauth2-implicit-dead] and
   [here][oauth2-4-mobile-n-native].

For some bullet-point advice on use cases and which flows to use when,
look at [slides][oidc-in-plain-en.slides] 33 through 35 from the video
above.

Some other resources worth checking out:

* Under which assumptions is OAuth 2 secure? [A Comprehensive Formal Security
  Analysis of OAuth 2.0][oauth2-formal-sec] is a paper worth reading...
* [Full-Scratch Implementor of OAuth and OpenID Connect Talks About
  Findings][full-scratch]. This guy implemented the specs from scratch and
  has lots of insights to share.
* [Deployment and Hosting Patterns in OAuth][deploy-n-host-oauth] sums up
  some options to roll out the beast whereas [New Architecture of OAuth 2.0
  and OpenID Connect Implementation][semi-hosted-pattern] dives into the
  semi-hosted service pattern.




[deploy-n-host-oauth]: https://medium.com/@justinsecurity/deployment-and-hosting-patterns-in-oauth-a1666dc0d966
    "Deployment and Hosting Patterns in OAuth"
[full-scratch]: https://medium.com/@darutk/full-scratch-implementor-of-oauth-and-openid-connect-talks-about-findings-55015f36d1c3
    "Full-Scratch Implementor of OAuth and OpenID Connect Talks About Findings"
[oauth2-4-mobile-n-native]: https://developer.okta.com/blog/2018/12/13/oauth-2-for-native-and-mobile-apps
    "OAuth 2.0 for Native and Mobile Apps"
[oauth2-distilled]: ./oauth2-distilled.md
    "OAuth 2.0 Distilled"
[oauth2-flows-diagrams]: https://medium.com/@darutk/diagrams-and-movies-of-all-the-oauth-2-0-flows-194f3c3ade85
    "Diagrams And Movies Of All The OAuth 2.0 Flows"
[oauth2-formal-sec]: https://arxiv.org/pdf/1601.01229.pdf
    "A Comprehensive Formal Security Analysis of OAuth 2.0"
[oauth2-implicit-dead]: https://developer.okta.com/blog/2019/05/01/is-the-oauth-implicit-flow-dead
    "Is the OAuth 2.0 Implicit Flow Dead?"
[oauth2-simplified]:  https://aaronparecki.com/oauth-2-simplified/
    "OAuth 2 Simplified"
[oauth2-spec]: https://tools.ietf.org/html/rfc6749
    "The OAuth 2.0 Authorization Framework"
[oidc1-spec]: https://openid.net/specs/openid-connect-core-1_0.html
    "OpenID Connect Core 1.0"
[oidc-flows-diagrams]: https://medium.com/@darutk/diagrams-of-all-the-openid-connect-flows-6968e3990660 
    "Diagrams of All The OpenID Connect Flows"
[oidc-in-plain-en]: https://www.youtube.com/watch?v=996OiexHze0
    "OAuth 2.0 and OpenID Connect (in plain English)"
[oidc-in-plain-en.slides]: https://speakerdeck.com/nbarbettini/oauth-and-openid-connect-in-plain-english
    "OAuth 2.0 and OpenID Connect (in plain English) - slides"
[overview.dia]: ./overview.png
[semi-hosted-pattern]: https://medium.com/@darutk/new-architecture-of-oauth-2-0-and-openid-connect-implementation-18f408f9338d
    "New Architecture of OAuth 2.0 and OpenID Connect Implementation"
[simplest-oauth2]: https://medium.com/@darutk/the-simplest-guide-to-oauth-2-0-8c71bd9a15bb
    "The Simplest Guide To OAuth 2.0"
[wot-d-heck-is-oauth]: https://developer.okta.com/blog/2017/06/21/what-the-heck-is-oauth
    "What the Heck is OAuth?"
