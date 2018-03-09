# IRCv4 - discussion with others and own thoughts

## Why it's needed
- IRC poorly standardised
- Very outdated
- Inflexible in a lot of ways
- Lots of outdated philosophy baked in
- Poor usability
- Outdated framing format
- Evolution of existing standard improves adoption chances
- Keeping things intuitive for existing and new users alike

## Alternative protocols

### The good
- Matrix: scrollback, federation (!), account management, avatars, customisable, renaming channels, some VoIP
- Slack: scrollback, good usability, reasonably decent UI, very scriptable
- Discord: scrollback, VoIP, very usable, file transfers

### The bad
- Matrix: not so good VoIP game, janky, still not user-friendly, bad clients, synapse blows, annoying, power levels reinvent IRCServices in a bad way
- Slack: usable, but reinvents services, proprietary
- Discord: proprietary, you don't really get your own server, trolls

### The ugly
- Matrix: uses HTTP based protocol
- Slack: proprietary protocol, rate-limited
- Discord: proprietary protocol, bad interoperability, very rate-limited API

## Goals for overhaul of IRC
- Improve usability
- Make competitive with alternative offerings
- Ease implementation
- Remove cruft from s2s and c2s
- Increase extensibility
- Simplify, simplify, simplify
- Increase standardisation --foxiepaws
- Make modification of existing clients simple whilst fixing actual bugs in the protocol
- Existing servers are a secondary consideration; most are crufty and overengineered and need replacement anyway

## Suggested improvements
- REFERENCE LIBRARY/IMPLEMENTATIONS for major languages
- JSON based formatting with netstring preamble
  - no, no, no, gross.  nested netstrings alone are very good --[Proceeding unsigned comment by awilfox]
    - Motivation for JSON is to get rid of half-baked parsers (no one tries to parse JSON themselves since they know they'll fuck it up) and have a flexible, easy to use, reasonably fast, and already standard framing format; most people asked would prefer JSON over alternatives for ease of use from a programming perspective --Elizafox
      - LMAO "no one tries to parse JSON themselves".  I have at least 17 packages to show you if you want this point refuted.  it is not easy to program for, it is not standardised (there are three "JSON"s, the original one being the only one spec'd however no one uses it because it is limited and has poor wording for some types) --awilfox
        - There are 3 standards yes: ECMA-404, RFC8259, and the newer RFC7493. The one everyone *effectively* uses now is RFC8259 (sometimes RFC7493, but strictly speaking that's just a profile of JSON and not a separate standard), although ECMA-404 is in some use as an official reference (the two might as well be the same). The standards are pretty much interoperable and only differ in minor ways. Ultimately the differences are very minor details that will never affect us unless we plan on representing numbers larger than (2^53)-1 :p (there's also some theoretical issues with escaping unicode, but in practise these problems never manifest themselves and any good parser will handle it for you). I think if we have numbers that big, we have done something terribly wrong. Some people do try to parse JSON themselves, but it's a pretty standard format that's hard to fuck up. Fuck-ups tend to present themselves as "just doesn't work" rather than the subtle failure modes of RFC1459. It's also good to use something that has an existing parser, and people asked preferred JSON (even older folks). Simplicity and decent parsers already existing wins, every time. It's not our fault if a user uses a bad parser. That's on them. --Elizafox
          - ooh, unicode problems that any "good" parser will handle for you.  trust me, you can get the subtle failure modes of RFC1459 with JSON parsers, especially in languages like erlang or such that people think are "cool" but don't have a large enough ecosystem and therefore have a dearth of parsers.  also, I can name at least six C parsers of JSON right now (Jansson, libjson, JSONPP (yes, preprocessor), mowgli2, json-c, and rtljson), and none of them are compatible with *each other*.  and you're saying we're going to have a client lib for "major languages".  yeah.  good luck with that. --awilfox
            - De facto standard for C at this point as far as I can tell is jansson. At least, it's getting that way. They're not API-compatible, but as long as they generate interoperable JSON it doesn't really matter (which I certainly hope they do if they are communicating on the internet!). --Elizafox
              - they don't.  they can't even agree on how to represent null, and whether starting a stanza with an array instead of an object is permissible.  and if someone fucks up their client lib and sends a [] instead of a {}, they're going to get a server error -- or worse, if there's more than one server, or forks of servers, they might even get invalid or incorrect data back!  --awilfox
                - It doesn't matter how they represent null as long as it is emitted as null in the JSON (no quotes). "null" exists in all standards. If they don't use it right, they are breaking the standard. Period. It's a shitty parser, and if it's used with the server, well that's on them, we're not going to sop; most /web app servers/ don't sop anyway (as they shouldn't). The only two interoperability issues documented in the real world besides shitty JavaScript types being shoved into JSON (lol) have been Unicode escapes (but in both ECMA-404 and RFC8259, the big ones, you have to escape using UTF-16 or not at all... the answer there is "don't escape", the always permissible option) and numbers larger than the JavaScript float type (because lol). The former is easily remedied and I imagine any parser worth anything would do it, and if they don't, they're breaking the standard (*both* standards). The latter will never be an issue. In fact, if it is, we've just broken comapt with JavaScript, in terms of the protocol, bad because we *need* that. Additionally, in reality, we will be using object for every stanza, because everyting is a key-value, so that doesn't affect us. --Elizafox
                - Make JSON an Optional format then, we are simplifying CAPs so they will still there, there are tons of reasons to at the very least support it. --foxiepaws
                  - JSON implies a key:value storage way of doing things; any alternate format would likely need to support it (or there would be a LOT of sopping) and all I'm seeing here is mostly FUD against JSON and experience with bad JSON libs. The server can't do anything about a bad JSON parser any more than an IRC server can handle a bad RFC1459 frame parser. --Elizafox
                    - mostly agree, Point here was, no matter what it should be an option. if not the main thing. That said, if we can point to a better thing that isn't JSON i think it should be at least investigated if it can be something simpler/nicer.  --foxiepaws
- Unique generated ID's for everything everywhere (actions, etc. for ease of verification the action took place)
- Simplified CAP negotiation (server sends caps, client replies with supported ones, done)
- Channel and user metadata (this is a BIG one!)
- Drop a shitload of old features that are redundant
- UTF-8 EVERYWHERE
- Line limit raised considerably (16k with 8k max keys? advertised by server?) 
- Use verbs in place of hard-to-use numerics (e.g. 001 becomes WELCOME)
- Federation similar to Matrix or ActivityPub with whitelisting or blacklisting options
  - activitypub is a terrible standard to compare to. --awilfox
    - It was an example of federation known to work and be easy to grasp, but the final form federation should take is TBD --Elizafox
    - I originally when i described my thoughts, used "awoo.space style (whitelisted) Activitypub-esq federation. Mostly as an example to kind of describe what i thought would be an ~alright~ idea to get something on the ground. This style of federation of course, has heaploads of problems in a chat protocol, and it was just to give an ~idea~ of what maybe people would like, since this is where Elizafox and us are are using federation on a day to day basis --foxiepaws
- Optional WebSockets to aid implementation from browsers
- Optional OAuth2 as a sop to webshits
- Registered names are *mandatory*
- Channels are registered to a user upon joining (TBD: when a channel is dropped?)
- User/channel settings with friendly names instead of modes
- Settings configurable from the IRC server; less is more
- Integrated services via central or distributed database (either at admin option)
- App service port similar to (but prolly not HTTP-based like) Matrix to allow deep integration of other services like legacy IRC gateways
  - I don't understand why the protocol needs to suck so much that "services" need a separate one. --awilfox
    - Remember u:lined servers in IRC? this replaces it; there ARE advantages, like introduction of pseudoclients from pseudoservers, and privileges you wouldn't grant a normal server --Elizafox
- Mandatory and standardised token bucket rate limiting for users
- Persistent clients for multiple presence
  - hopefully something like XMPP's --awilfox
    - More like existing protocols, that have you stay in the room after, with presence notifications; think server-side bouncer --Elizafox
- Scrollback
  - like XMPP MAM? --awilfox
    - Ish, for PM's and channels, bursted to clients when they join up from the last message they sent --Elizafox
      - main concern here is bursting something like #ubuntu from freenode; gigs and gigs of backlog you probably won't ever need.  this should be chunked or something with the client setting its desired chunk size. --awilfox
        - Compression is prolly a must for that, and clients can set backlog limits I suppose with a signon message --Elizafox
          - woe to the person who has a 3G connection and needs help with Ubuntu.  compression only does so much, and at the size and scale we're talking about, now you're going in to impacting battery life decompressing it all.  there should be a default chunk size that is conservative (maybe 64K?) and the client can use more if it's on a solid, reliable, fast, unmetered connection. --awilfox
            - My proposal for that is to simply tell the server with a setting of some kind or with the handshake in some way, "I can't handle more than X amount of scrollback, don't send any more than that". So say you set 1K or something of backlog (this is obviously unworkable, just an example), it would only send lines up to 1K --Elizafox
              - still needs a default, otherwise you're requiring every client author to know what connection type is being used (this won't even be possible for the 'poor' webshits, who don't have an NSConnection or WSA type API to query). --awilfox
- Use of oper power is noisy and tells everyone, "hey this person is using oper power!"
- Reduce oper powers to the bare essentials needed to operate a network
- Server-side contact lists for great justice
  - so this would be on the user's server, not distributed through the networks, right? --awilfox
    - It's local-only... why would this have to be federated? --Elizafox
- Nickname is not tied to account name; differentiation can be via user ID
- Identity verification through optional PGP integration (works great with metadata)
- Media integration (using central HTTP server or S3 or w/e the oper configures, upload done out of band via HTTP)
  - it seems dumb to use HTTP, though I can see the argument.  it should probably default to using a built-in simple static HTTP server like atheme's --awilfox
    - There is good reason to do this: HTTP is good at file transfers, probably one of the few protocols built to do it in 2018 that does it well; also, I was thinking small embedded HTTP server, or you can configure S3 or such --Elizafox
- Avatars via metadataa
- PRIVMSG key "action" for replacing CTCP action (other possibilities open up) and "ping" for a ping message (is this really necessary?)
- User to user file transfers... perhaps some other out of band negotiaton method? How to deal with NAT traversal crap?
- Presence notifications like MONITOR but better
- More data bursted to clients when a user joins a channel (when a user joins a channel you're in and when you first join a channel)
- User to user E2E key exchange (probably not feasible for channels)
- VoIP negotiation (defers to other protocol for voice/video)
- PRIVMSG/NOTICE formatting keys to replace colour codes (HTML-esque formatting, others possible)
  - why not something more intuitive like md, rST, etc? --awilfox
    - Actually, this was discussed with rachel, and that's a great idea; there are multiple ways to format messages, and a "formatting" key and value in the PRIVMSG frame would allow support for whatever someone thought best (markdown and HTML-like was considered) --Elizafox
      - I endorse this behaviour.  there needs to be a specified default if the formatting key is not present, though, just to remove ambiguity. --awilfox
        - Default is "nothing" and transmitted without being molested. --Elizafox
- Presence and information via metadata
- Subscriptions to metadata updates
- ACL's replace +ov etc and opers
- Overhauled server-level ban system; perhaps done via ACL's?
- For the love of god, something better than SNOMASKs
- The \& channel type becomes local-only, non-federated
- Improved S2S auth game using modern techniques (tokens, etc.)

### Dropped features
- AWAY; use metadata
- WHOIS; replaced with user metadata lookup
- WHO/WHOX/NAMES/LIST; data should be automatically synchronised properly, manual queries are noise and these functions are redundant
- USER/NICK; use metadata
- SUMMON; officially killed off for good, once and for all
- \*LINE commands; IP bans, account bans, and perhaps metadata filtering settings replace it
- RESV (aka QLINE); no longer makes sense
- TOPIC; use metadata
- STATS; use metadata and server information
- ISON/MONITOR; use presence notifications and metadata
- OPER; replaced with user ACL's in server scope
- MODE (list, e.g. +ovb); use ACL's
- MODE (non-list); use settings, many modes obsolete
- DIE/REHASH/RESTART; more of a liability than an asset
  - something like REHASH may make sense if a server setting was not distributed properly. --awilfox
- WALLOPS/OPERWALL/LOCOPS; better ways to do that nowadays, services integration means you get GLOBAL
- JOIN 0; just leave all channels yourself
  - more specifically, client-side automation could be done ("Leave all rooms" in a client could just issue all the PART commands itself) --awilfox
- i:lines; settings achieve the same thing essentially with mandatory registration
- RPL\_ISUPPORT aka numeric 005; use CAPs
- CTCP; most of CTCP replaced via metadata, ACTION replaced via a tag in PRIVMSG/NOTICE
  - PING?  this could still be useful, and I think XMPP has something like it --awilfox
    - Expand the PING verb instead. ping already can take a server target, why not make it be able to take a client target as well? -- foxiepaws
- DCC; replacement TBD, more NAT friendly mechanism
- Extbans; outlived their usefulness
  - more specifically, I think that while what Extbans do is useful STILL, I think that they should just... be part of normal ACL stuff. 
- Banmasks; not useful anymore, abused way too much, banning entire ISP's/countries makes no sense with multiple presence
- Exempts; *almost* meaningless, use ACL's to allow someone to join without invite if channel is invite-only
  - that's not joining without invite, that's +I in ACL form --awilfox
- Multi-targeting for PRIVMSG and NOTICE; no replacement, only useful for spam
- User GECOS; use metadata
- All channel types except # and \&; not useful anymore, barely supported as it is
- Numerics; use verbs instead 
    - some places where numerics might still make sense to be kept, is for error content, similar to how they are used in HTTP still, e.g. 400 and 500 errors are pretty understandable to tell where something has gone wrong and in an irc context does still make sense ... for everything else though, kill em. -- foxiepws
- RFC1459 casemapping; UTF-8 everywhere
- Services and u:lines; functionality integrated with the server
- Client-initiated CAP negotiation; server sends CAPs, client replies with what it supports, negative acknowledgement is useless and passive-aggressive
- RFC1459 framing; use JSON
  - more discussion on this, for sure. preferably somewhere we can really just suss it out unlike we can here [so IRC?] --foxiepaws
- Ident; no replacement, this is 2018
- Hostnames; doesn't make sense with multiple presence, metadata replaces many use cases
- TS and ND/CD rules; registered names and channels render them obsolete, greatly simplifying S2S
- Fake lag; it's lame, most servers do it but it's dumb, just drop messages (with action ID's, clients know what messages didn't make it)
- Most oper powers; obsolete, don't make sense anymore, or way too prone to abuse (How do you do KILL with multiple presence? Useless, just ban the account!)
    - A sngle argument for KILL, and some of those sorts of oper powers are often used as a "knock it off" kind of action. we'd say keep it in under first version of spec as a deprecated command and see how it plays out? -- foxiepaws
- Local operators; not useful anymore, ACL's can achieve most of the same thing
- Unregistered nicknames; guest accounts are too open to abuse
  - I partially disagree with this idea, however, The way it should be handled should be fairly limited in what you can do. People being able to drive by and preview the content of whats in a channel is a good thing. also in some places, they make even more sense, such as places wehre you actually do want a bot that only sometimes joins [such as a commit bot for a project]. Interestingly, Discord actually allows unregistered accounts, but they have to be invited to a channel by someone who knows who they are. so the idea isn't complately considered gone even in the modern chat hell. Instead: Ratelimit Guests extremely heavily. Disallow certain actions [configurable] such as channel creation, ACL addition, most metadata actions.  --foxiepaws
- Unregistered channels; impossible
- Jupes; I think I've used this like once... just use configuration settings to achieve the same thing
