---
title: Post a reply (Minipub)
order: 4
---

# Post a federated reply comment using a self-hosted ActivityPub server (Minipub)

_A server-to-server (or s2s) ActivityPub interaction_

In this scenario, the podcast client app hosts a custom ActivityPub backend that ties into the same user system used by the app, in order to make the login process as easy as possible.

In this case, the custom backend ([Minipub](/get-started)) will make direct AP calls to the server specified in the RSS feed to handle federation.

## Overview from the user / podcast listener’s perspective

The user creates zero Fediverse accounts in this scenario, they just use podcast apps.

The user opens an episode in their native or web-based podcast player (called PodSqueeze in this example), reads the comments without leaving the podcast app (app pulls the comment data by [enumerating the thread](/info/activitypub-for-podcast-apps/display-comments) below the root post specified in the RSS feed for that episode).

The user, already logged in to their podcast app, wants to post a comment. The user hits reply to the root comment or a subcomment in the app's UI. When the user hits "send", the app ensures this user has an identity set up on the app's underlying Minipub server automatically, creates and saves the local comment on the Minipub server automatically, and then federates that reply over to the target comment server using another Minipub call.

When other apps see that federated comment, it will have an identity like `alice@comments.podsqueeze.com` (for the user "alice" on the Podsqueeze app) in the enumerated thread. No local users ever get created over on the podcaster's social network server other than the account posting the root post, yet it still has a robust thread contributed to from many podcast apps (and possibly other Fediverse servers)

## Overview from the app developer’s perspective

Simply [displaying a comment thread](/info/activitypub-for-podcast-apps/display-comments) can be done without Minipub.

However, when one of your users wants to post a comment, you'll need to ensure they have a corresponding user (ActivityPub Actor) on the Minipub server.

_The example calls here use the minipub cli for clarity, but you'd probably use the admin rpc json api in practice._

```sh
minipub create-user \
  --origin https://comments.podsqueeze.com \
  --pem /path/to/admin.private.pem \
  --username alice \
  --name "Alice Doe" \
  --icon /path/to/alice.150.jpg \
  --icon-size 150
```
This will return a json response with the user's `actorUuid`, which you'll save in your app's existing backend database for all future interactions.
You can rely on the actor uuid to remain immutable, regardless of any future username or other property changes to this user.  In this example, let's say the actor uuid is `81ce080ac25d4b3294b9eef08422964f`.

The url for this user's Actor will be `https://comments.podsqueeze.com/actors/81ce080ac25d4b3294b9eef08422964f`

Viewing this url in the browser will display the ActivityPub json other Fediverse servers will read automatically for authentication.  The actor url will also be linked from any WebFinger lookups for `@alice@comments.podsqueeze.com`.

When the user hits _Send_ on the reply from your app's UI, you'll need to create the new Note on your Minipub server, then federate it out to the recipient's server.

Specify the actor making the comment as the first argument to `create-note`:

```sh
minipub create-note 81ce080ac25d4b3294b9eef08422964f \
  --origin https://comments.podsqueeze.com \
  --pem /path/to/admin.private.pem \
  --content "this is my reply" \
  --in-reply-to "https://example.social/users/bob/statuses/123456123456123456" \
  --to "https://www.w3.org/ns/activitystreams#Public" \
  --cc "https://example.social/users/bob"
```
This first call returns the new object (Note) uuid, and the corresponding Create activity uuid. Let's say the object uuid is `284ed7ebf48949aea975453ae5ebc337`, 
and the activity uuid is`db8b86e411b04a899c1a0e1a288b7f75`.

The url for the new Note object will be `https://comments.podsqueeze.com/actors/81ce080ac25d4b3294b9eef08422964f/objects/284ed7ebf48949aea975453ae5ebc337`

The url for the new Create activity will be `https://comments.podsqueeze.com/actors/81ce080ac25d4b3294b9eef08422964f/activities/db8b86e411b04a899c1a0e1a288b7f75`

The activity is what you want to federate, so it's the first argument to `federate-activity`:

```sh
minipub federate-activity db8b86e411b04a899c1a0e1a288b7f75 \
  --origin https://comments.podsqueeze.com \
  --pem /path/to/admin.private.pem
```

`federate-activity` will identify the receipient of the comment, discover the remote inbox, and make the POST call to federate this comment to the remote server.
It's safe to call `federate-activity` multiple times, Minipub saves federation progress state, so it will only make a successful call once.  This is nice in case
something goes wrong and you need to retry failed federation again for an existing comment.
