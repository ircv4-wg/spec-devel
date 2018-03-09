---
Title: JSON Framing for IRCv4
Author: Elizabeth Myers <elizabeth@interlinked.me>
Date: 9 March 2018
Status: Draft
---

# Preface
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

# Background
RFC1459 framing is known to be deficient in many ways. It's difficult to parse correctly (quick: what does foo in `:foo@bar CMD` mean?), ambiguous (:nick is not the same thing as :nick.user.host), and unflexible (only one parameter is permitted to have spaces, and that is the last one). Most IRC daemons support a maximum of 8 parameters as well, which creates massive headaches for those wanting to create new commands; look no further than `WHO` to see the mess.

There are also problems created by the now anemic 512 character length for messages, which often results in splitting messages.

It's a format that dates from 1988, and has changed none since it's inception (although IRCv3 attempted to add tags, the adoption of tags has been far from widespread, and complicates an already annoying format further). Almost no IRC clients truly parse RFC1459 frames correctly, only bothering with the format FreeNode uses.

It's time to retire it and replace it with a more modern format.

# Alternatives
Some solutions that have been suggested are:
- Leaving the framing format alone; compatible, but not ideal for reasons shown above, and achieves nothing
- Using RFC1459 framing with an increased line length; solves one problem, but not the others
- A key:value based net string format; easy to parse, but no off-the-shelf parsers. Magic characters are needed to prevent ambiguity in most proposals.
- Extending RFC1459 framing; tags were suggested by IRCv3, but they have very little adoption and just put lipstick on a pig
- A binary protocol such as BSON, protobufs, or a custom binary protocol; good in theory, but often gives little advantage in the real world in terms of parsing speed, makes debugging harder, and makes presentation harder; there is also no schema system defined for such protocols (except protobufs)
- XML; error-prone, slow to parse, and many XML parsers [still contain vulnerabilities](https://docs.python.org/3/library/xml.html#xml-vulnerabilities); experience with XMPP shows little enthusiasm for XML-based protocols
- JSON; had some ambiguities years back with parsers behaving badly, but the situation has improved; there is also a schema system known as [JSON Schema](http://json-schema.org/).

# Proposal
It is proposed that the framing format from IRC SHALL be changed from RFC1459 to JSON.

[JSON Schema](http://json-schema.org/) SHALL be used to define frames and command layouts.

This proposal means the probable abandonment of *de facto* ports 6667, 6697, and 9999, as the protocol is no longer wire compatible. Implementers MAY choose to use JSON framing on ports 6667, 6697, and 9999, but it is NOT RECOMMENDED. Another port will be chosen in a later proposal, with probable abandonment of plaintext. In the interim, existing de facto ports SHALL remain used.

# Details
The schemata proposed for usage can be found in `schemata/` in the root directory of this project. This draft defers to those schemata, for conciseness and to allow further changes. Implementers MUST conform to these schemata upon adopting this proposal.

# Security issues
None known.

# Backwards compatibility
This change is not backwards compatible with the old protocol. Compatibility will be handled on a command-by-command basis. A future draft will define mappings between old commands and new ones, to ensure interoperability.
