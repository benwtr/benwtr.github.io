---
layout: default
title: A Different Approach for Authenticating ChatOps Bots on External APIs
---
# A Different Approach for Authenticating ChatOps Bots with External APIs

tl;dr put _user_ tokens in hubot's brain so hubot can make API calls with the
user's credentials instead of a bot user's credentials.

I wrote one or two Hubot scripts some time ago that handle credentials for
external APIs a bit differently than what I've seen in the wild. I still haven't
seen anyone else use similar patterns in the Hubot community so my hope in
writing this is to share this idea, maybe someone will find it useful.

It's actually very simple- instead of having your chat bot access external APIs
with credentials configured for the bot globally, you can have the chat bot ask
users to register their own credentials with the bot (eg pasting an API key in a
private message to the bot, PIN/web OAuth, supplying an MFA code to a bot that
already has credentials stored, etc.) Once that's done, the bot can talk to the
external API **as** the user, instead of as a bot user. Effectively allowing users
to use the bot as their own personal client to the remote service.

If there is any "trick" to this, it's only that you need to key the data under
the user ID as returned by the hubot adapter. So you need to trust that your
chat service is correctly identifying users to the bot.

Some advantages of this approach:

-   no need to worry about authorization in the bot.

-   no need to manage another source of truth for authorization. eg:
    [hubot-auth](https://github.com/hubot-scripts/hubot-auth)

-   authentication and authorization on external services Just Works™.

-   no need to create a 'bot user' on the external services.

-   logs/history outside bot/chat show correct user information - This in
    your logs: "**alice** ran job _deploy to prod_ 2 hours ago" instead of this:
    "**hubot-user** ran job _deploy to prod_ 2 hours ago".

I mentioned before that there's more than one way to have users set up
credentials with the bot, one way is OAuth. Here's an example of a simple
twitter client implemented as a hubot script that exposes the PIN based OAuth
flow to users in the chat:

{% gist benwtr/830e9b099887ea09d75f %}

It works like this:

```
<alice> @hubot t setupauth
<hubot> https://api.twitter.com/oauth/authorize?oauth_token=DbdrwwAAAAAAiRcdAAABUKsq4I4
```

_Alice clicks on the URL, twitter asks for authorization then presents a PIN code:
748256_

```
<alice> @hubot t auth 748256
<hubot> You have authenticated
```

_Now Alice can search.._

```
<alice> @hubot t search foo
<hubot> Tue Oct 27 21:20:59 +0000 2015 @Foo<sub>Foo</sub><sub>Fati</sub>: RT @Bonnaroo: "Happiness does not depend on what you have or who you are. It solely relies on what you think." - Buddha #radiatepositivity
<hubot> Tue Oct 27 21:20:52 +0000 2015 @trappinbri69: @ArielleAguilar why you mad foo
<hubot> Tue Oct 27 21:20:22 +0000 2015 @Foo<sub>Foo</sub><sub>Fati</sub>: RT @LukeAdams95: "Tonight, I'm making what my girlfriend has always said she's wanted for dinner." <https://t.co/sle9j8T9Gw>
<hubot> Tue Oct 27 21:20:05 +0000 2015 @SortMusic: @Foofighters is on tour, check the #concerts map here: <https://t.co/8ssPOcc0dh> Netherlands, Germany, Poland, Austria, Italy, France, Spain
```

_Or Tweet using his own twitter account.._

```
<alice> @hubot t tweet I am tweeting through a hubot script yay
```

It should be fairly trivial to change this script to use a web based OAuth flow.
You would need to use Hubot's HTTP listener and set up the twitter app to
redirect the user to the listener instead of the step where the user sends the
PIN to the bot. It might also be worthwhile to have it run `setupRequestToken()`
if an API call fails with an unauthenticated status, so the bot just spits out a
URL and tells the user to enter a PIN if it needs to.

Another example:
[hubot-jenkins-userauth.coffee](https://github.com/benwtr/hubot-jenkins-userauth/blob/master/src/hubot-jenkins-userauth.coffee)
In this fork of the community hubot-jenkins script, users can store their
Jenkins API key in the Hubot brain and have Hubot perform actions as their _user
instead of as a bot user_. In case the contents of the brain are exposed, the
keys are encrypted using a secret stored in an environment variable.



