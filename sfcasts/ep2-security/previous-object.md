# Previous Object

Coming soon...

It's the `access_control` on the `PUT` endpoint. We were pretty easily able to make sure
that only the owner of this cheese listing can edit it, otherwise you won't be in the
act and access denied. And this is something that we have actually assertive inside
of our test. So check this out. I'm going to try to trick our security system. So
right now we're logging in as `user2@example.com` but the `CheeseListing` we're trying
to update here is owned by `user1@example.com` which is why we're ultimately getting
the `403` status code.

But one of the fields that you can update at least currently is you can actually
update the `owner` field. So let's update the honor fields who 
`/api/users/` user2 right. We should not be allowed to do this just because,

At the moment we start this request, this `CheeseListing` is owned by user1 so this
should absolutely be rejected. But if you move over and around the test, 

```terminal-silent
php bin/phpunit --filter=testUpdateCheeseListing
```

it's allowed. Check this out. We expected a `403`, we've got a `200` this is a very
important Gotcha. With how API platform does their security. So the process of when a
request comes in, the process looks like this, the first and that API platform does,
is it deserializes the JSON into an object? So it actually goes to the process of
calling JSON Decode and actually following, follow the serializer rules to create the
`CheeseListing` object. Because of this, if you actually, because of this, if I'm, even
if you use those and have access to an endpoint, if they submit, if they submit
really invalidated, like invalid JSON or an owner, a field that I'm

that's malformed, they will actually get a `400` status code a instead of a `403`
So that's something that you need to watch out for.

So the first step of the process is the deserealization. Then security is applied and
then validation is applied. The validation rules. So what's happening in this case is
by the time that we call, our `access_control` is run here. This object is the updated
object, meaning the owner has already been changed. And so now the owner is equal to
the currently logged in user. So actually object is probably not something that you
should use most of the time. Instead you're going to use something called 
`previous_object`. Previous object is basically what I would use in pretty much all cases and
less. For some reason you actually wanted to do an access control decision based on
the posted data. So as soon as we change this back to `previous_object`
now when we were on tests, 

```terminal-silent
php bin/phpunit --filter=testUpdateCheeseListing
```

can I scroll up? Yes, it works. So use `previous_object`
All right, so now I have a really nice `access_control` for our um, `CheeseListing`
Um, so let's repeat this. Uh, same thing over on our `User` entity right now
you can see that we don't have actually any `access_control` stuff over here. So let's
start by saying `itemOperations=`

and we'll make the `get` item operation. I'm going to steal some stuff over here. I'm
going to go steal an `access_control` entry from my user. So if we want to fetch a
user's Day, let's say that you need to at least be logged into fetch another users
user data. So use the `ROLE_USER` access control. And then if you want to post a user,
the next one is `PUT`, if you want to updates a user, you're probably gonna need to be
logged in and you're only going to be able to update your own. So here we're going to
say `is_granted('ROLE_USER') and object == user`. So we're going to say only if this
object is actually equal to this currently authenticated user. In this case we don't
need to use `previous_object` cause there's no way to actually change the entire
object. Um, so either of those would work just fine in that case.

And then for the last one here, for item operations, `delete`. Let's say that you can
only delete if you are an admin. So we'll do the `access_control` for `ROLE_ADMIN`.
Cool. And above this I'm actually gonna. Now I'm going to do `collectionOperations`
and same thing `get`, let's say get, you need to be logged in. And then for `post`
posting a collection, that's actually how you create a user. So that's actually
registration. So for post, I am going to just leave nothing. Okay, cool. So this was
simple enough that I didn't have a test for it. Let's at least kind of make sure one
of them works. Oh, of course. Syntax Air [inaudible] with my `itemOperations`,
probably extra common there at the end. Refresh that again. Oh t closed parentheses,
super. Oh I have a syntax. Air, uh, expected t clause penalty has got item
operations. That is actually something missing, a common, they're actually missing a
common there. And I have an extra comma right there. You're fresh. That all right.
Cool. So if I right now I am actually anonymous at phone and get users execute and
perfect. I get the `401` status code on that

Now in addition to doing the access control, like on the operation level, you can
also do it at the resource level. Um, the one weird thing is that in this case it's
actually gonna call the `accessControl` with an upper case c instead of a lower case.
And you'd see a couple of these extra require a `ROLE_USER`. So if I wanted to, I could
actually say `accessControl=` `ROLE_USER` and then I don't have to have,

that `access_control` on get anymore. I can just say `get`, I don't need access
control down here on `get` either. But because we were not posed to be anonymous, we
want to do this, we would still need to actually now override post and say 
`IS_AUTHENTICATED_ANONYMOUSLY`. So the default is `ROLE_USER`. So get is going to require 
`ROLE_USER` on both collection and item and all the other ones completely replaced on that.
So the post endpoint is going to be allowed to be anonymous. So we can actually try
that over here by open up the post here and I'm just going to send an empty body just
to see if I have access and well I don't, I got a `500` air. Ah, another syntax error.
Boy, I'm terrible with that. Flip over, add the comma on. Get as soon as annotation
is even for me, they're tricky. I'm going to have to record this whole thing. Aren't
I? Such a fucking disaster? It's going great. Actually, I forgot that. Come on. The
end of both gets, so now let's go around alternative reexecute and there we got 400
this value should not be blank, so I proved it. I actually got through security. That
one is still anonymous.