+++
date = "2016-04-26T00:00:00+00:00"
description = "Experiences from CopenHacks 2016."
title = "Hacking privacy into Facebook's Messenger in 24 hours"

+++

**TL;DR**: My team's and my experiences from CopenHacks 2016. We built a userscript that seamlessly adds PGP to Messenger. This was originally posted by my friend [Stanko on Medium](https://medium.com/@stanko_k).

<!--more-->

Hackathons are great.
When a friend of mine asked me if I wanted to go with him to [CopenHacks](http://copenhacks.com/") I had no idea that we would spend 24 hours reverse engineering [Facebook's Messenger](https://www.messenger.com), let alone win first place.

## The journey to Copenhagen

We usually go to a lot of hackathons and coding competitions, but we never went to a hackathon outside Croatia.
These are usually either too expensive or at an inconvenient date for us.
When we heard of [CopenHacks](http://copenhacks.com/") we decided to go, as all of us were free at the time and wanted to see what hackathons looked like in other countries.
Hackathons are also a good opportunity to meet recruiters or get internships at one of the sponsors.
Finally, let's not forget all the swag that gets handed out!

[CopenHacks](http://copenhacks.com/") was a new thing, so we didn't know what to expect.
The idea for such a competition came around mid 2015 by a student at the local technical university in the beautiful city of Copenhagen, Denmark.
His idea was to create a competition where people could come and build amazing stuff, learn new things and meet fellow hackers from around the world.
That's quite different from the other hackathons I went to.
Usually the main sponsoring company gives out a specification for an app and then you have a set amount of time to implement it.
This was completely different.
We could do whatever we wanted.
This was also a problem because many questions arose, like 'what could we do?', 'what would impress the judges?', 'what did we have time to implement in 24 hours?</em>'.
We had several ideas but none of them felt right.
We knew we wanted to make something fun and open-source, so all commercial projects fell out of the picture.

Our flight to Copenhagen was getting closer and closer and we couldn't settle for an idea.
The last night before the flight all of us got into a call and started brainstorming.
As all of us are cryptography enthusiasts (and perhaps a bit paranoid) and all of us use Messenger all the time.
We decided to combine the two together just because it would be convenient and secure.
Thus the idea for [MessengerPG](http://messengerpg.tech/) was born!

MessengerPG should be a browser extension that unobtrusively adds client-side message signing, encryption and decryption with [PGP](https://www.wikiwand.com/en/Pretty_Good_Privacy).
The cryptography should be done using the local GPG installation.
Messenger would only be used as means to transfer the messages from client to client.
The extension would collect the public PGP keys for all chat participants from their Facebook profiles.
And, yes, [you can add your public PGP key to your Facebook profile](https://www.facebook.com/notes/protect-the-graph/securing-email-communications-from-facebook/1611941762379302/).
The sent messages can only be decrypted with the private keys of each individual participant of the conversation.
Everybody else sees only seemingly random strings of letters -- this includes Facebook.

More importantly, our extension would be a platform on top of which anybody could build their own extension to Messenger.

## The hackathon

My friend once said 'Hackathons are an interesting social experiment'.
That's perhaps the shortest way to say it.
Hackathons give quite a small amount of time to create something complicated, which causes a lot of stress and leads to sleep deprivation.
That's when people start getting into arguments and fights.
Then the whole experience turns into a delicate dance of not stepping on each other's toes.

Every time we started getting angry at each other we took a walk, grabbed a soda, played some table-tennis or went to somebody's desk and tried to help them with their problem.
This helped, at least me, a lot.
Not only did I get to meet people but I saw all the brilliant ideas that others came up with.

![Us playing table-tennis at 3 am](https://cdn-images-1.medium.com/max/1600/1*wCSUEbT71HObqp90DNlDHA.gif)

Another good piece of advice is to celebrate each victory no matter how small it is as it helps to keep your spirits up.
I can recall the moment when we finally managed to intercept incoming and outgoing messages.
We got up and started cheering.
Each time we managed to solve a problem, we gave each other a high-five.

But the key to getting the best possible product in the smallest amount of time is planning and organization.
An hour before the start of the competition we divided our responsibilities.
[Marko Božac](https://medium.com/u/58c4a981178) would be making the presentation website.
[Luka Strižić](https://luka.strizic.info/) would be making an interface between the local GPG installation and our extension.
[Petar Šegina](https://medium.com/u/34412de2f81) and I would reverse engineer Messenger and find a way to inject our code.
That plan worked out quite well for us, as all of us managed to get three hours of sleep, several meals and we even finished with half an hour to spare.

## Reverse engineering Messenger

Finding an entry vector in JS applications usually isn't all that hard because you can read their source and manipulate all objects as they live in your browser.
This was a bit harder.
Messenger's source is split into several minified files which makes them hard to understand and it's source is bundled with [React](https://facebook.github.io/react/) which adds a lot of code to sift through.

It was obvious that Messenger needs to send and receive data from a server so we opened up [Chrome's developer tools](https://developer.chrome.com/devtools#improving-network-performance) and started to monitor incoming and outgoing network traffic.
There we noticed that each time you sent a message a request to `send_messages.php` was being made.
This was our entry point.
We created an [XHR breakpoint](http://devtoolsecrets.com/secret/debugging-xhr-breakpoints.html) with the URL and sent a message.
This triggered the breakpoint and showed us the call-stack which we then traversed until we found the `getValue` function that read text entered into the input field.
This was the function we needed to monkey patch to call another function that would process the message before it was sent.

We needed to find a way to get hold of a reference to the object which made the call to `getValue` and monkey patch it.
The window object doesn't have a reference to Messenger's main process but it has a reference to the module loader `__d` which we then patched to give us a reference to the object we needed.

Receiving messages was a different ball game.
We couldn't simply listen for incoming messages and decrypt them before they were passed on to the rendering logic because then only newly received messages would get decrypted.
After a page refresh all messages would appear encrypted again.
We decided to monkey patch React's render method, as we were sure that it would get called for each and every message that needed to be displayed on the screen.
We used the same code injection method as before.
The code we added waits for a render call to create a object with a message's CSS class.
Then it tries to find a PGP encrypted and signed message and decrypts it.

This all sounds easy enough, but it was a really complicated and frustrating task.
We didn't know what Messenger's architecture looked like so we had to figure out how all the things work together.
We spent a lot of time trying different approaches and discussing possible solutions.
Perhaps the best metric to show how complicated the reverse engineering process was is the fact that two engineers wrote a total of 10 (usable) lines of code per hour.

Many compromises have been made during the development process, some due to time and some due to technical constraints.
In order to access the local GPG keychain we built a simple Node.js application that allows us to sign, encrypt and decrypt messages through a simple HTTP API.
This led to an unexpected problem with [CSP](https://developer.mozilla.org/en-US/docs/Web/Security/CSP) because Facebook prohibits any outgoing communication to non-Facebook approved domains.
We tried to use [unsafe XMLHTTPRequests](https://wiki.greasespot.net/GM_xmlhttpRequest) but ultimately failed, so a hack was in order.
We registered `pgp.messenger.com` in the local `hosts` file, and pointed it to `127.0.0.1`.
This meant that our local API was now located on a CSP trusted domain.
However, we still needed to access the API over SSL (as the CSP required `https://`), so we created a proxy server to our GPG API server, running with locally generated SSL certificates.
Getting the browser to trust the generated SSL certificate, being the last hurdle, was not much of a problem.

![Warning, hack in progress!](https://cdn-images-1.medium.com/max/800/1*w_5AG9SzEJ3--E-27k2yyA.jpeg)

## The anticipation

I think we all felt a great relief when we implemented the last feature with half an hour to spare.
But the relief was short-lived.
Soon the presentations started and to the stage came team after team of really talented people.
One particular team found a way to misuse Android's accessibility features to create a system wide chat-bot.
A different team created a chat-bot that understood natural language and helped sort out arguments between people.
Others made games that looked really polished.
And then came we, with our short presentation which showed a few messages being exchanged.
Plain text on the left, encrypted and signed on the right.
Packaged in with a short talk about how we managed to hook in to Messenger and that this can be used not only for encryption but as a platform for all kinds of extensions.

{{< youtube S-LVSCIzip0 >}}

After the presentations were over none of us believed that we would win anything.
But that didn't matter!
We created something really cool, traveled together, had an excellent weekend, drank a couple of beers and had fun in general.
For us, no matter if we won any prize or not, this was a success.
This was what a hackathon should be about!

After a while the room filled with people again, but this time for the prizes to be handed out.
One prize was handed out after the other, but our name was never mentioned.
Then came the prizes for the top three teams.
By this point most of us stopped listening and only waited for the ceremony to be over as we desperately needed sleep.
Before the first prize was given out there was a short inspiring speech by the organisers.
It explained that they picked first place based on the hacker spirit, problem complexity and general usefulness of the project.
Then they called our name.
We were in shock.
No one expected this to happen.
Everybody's faces were white.
When it finally hit us that we won, we rushed to the stage to pick up our prize.
And so the whole experience was over.

## Conclusion -- Privacy is hard

This hackathon was something I will remember, not just because we did something awesome, but because it was really fun.

The organisers did an outstanding job, the food was great, the venue was even better.
Thanks to all the sponsors for all the awesome lectures (and all the cool swag).
Most of all, thanks to my team which made Copenhagen an experience to remember.

If you are thinking about going to such an event then do it.
Don't hesitate.
It doesn't matter if you win a prize or not.
At the end of the day you will either create something awesome or learn a valuable lesson.
Similarly, if you are thinking of organising a hackathon, don't hesitate.
You will change somebody's life forever and meet really cool people in the process.

If you want to help us make MessengerPG a thing or to build your own extension on top of Messenger, take a look at [our Github page](https://github.com/Stankec/copenhacks-2016).
Fork us.
Look at the code.
Change something.
And feel free to open up a pull request.
We will gladly take all the help we can get.

Finally, a word of caution.
There is currently no way to guarantee that the code you audit will be the code you run next time you open Messenger.
This allows malicious servers to serve targeted users with different versions of the code.
So, while 99% of users may get clean, non-malicious code, the interesting 1% can get served with code endangering their privacy.
This is also a problem with closed-source messaging services which offer end-to-end encryption which cannot be properly audited.
Even if the messages being exchanged are encrypted end-to-end, what's stopping the proprietary code from collecting the messages you exchange and exposing them over a different channel?

Privacy is hard.
It is not something we should take for granted, but something we should actively try to achieve and protect.