---
Title: Elimination Of Plaintext in IRC
Author: Elizabeth Myers <elizabeth@interlinked.me>
Date: 9 March 2018
Status: Draft
---

# Preface
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

# Background
It has long been considered undesirable to use plaintext on the open Internet. This is especially important in the modern day, with an ever-increasing number of hostile actors.

# Alternatives
Some solutions that have been suggested are:
- Do nothing; continue with the status quo; no security improvements would be effected by this, however
- IRCv3 has suggested HSTS, a system similar to STS in HTTP; ultimately, HSTS suffers from man-in-the-middle injection problems; although they could be remedied (with signatures and the like), there seems to be little purpose in doing so, as a client already supporting SSL could just try SSL first
- Try SSL first; would improve security, but leaves the possibility of plaintext open
- STARTTLS; used by many protocols, but open to MITM stripping of STARTTLS headers, and has caused headaches in the past
- Eliminate plaintext; the hazard is removed, with no complication

# Proposal
It is proposed that plaintext be prohibited. Clients MUST NOT use plaintext ports and use of SSL ports is REQUIRED.

Service MUST be discontinued on port 6667 and any other plaintext port.

# Security issues
Nothing in this proposal addresses any other issues, such as self-signed certificates. Such issues are outside the scope of this document.

# Backwards compatibility
Most clients will handle this fine, as virtually every modern client supports SSL; users can be instructed in the use of SSL.
