%%%
Title = "How Do You Do"
area = "Applications and Real-Time Area (ART)"
workgroup = "Dispatch"
abbrev = "hdyd"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-newton-how-do-you-do"
stream = "IETF"
status = "standard"
date = 2023-08-22T00:00:00Z

[[author]]
initials="A."
surname="Newton"
fullname="Andy Newton"
organization="ICANN"
[author.address]
email = "andy@hxr.us"

%%%

.# Abstract

This document describes a system to discover the identifies of natural
persons while preserving their privacy.

{mainmatter}

# Background

In a song written by children's author Shel Silverstein and made famous by
Johnny Cash, a young man seeks out and identifies his long, astranged father
using an old, worn photograph. Upon giving his name, the young man asks of
his father, "how do you do?"

This document describes a system, HDYD for "how do you do",
to aid in the discovery of identifers for
natural persons while preserving their privacy. The system uses a publicly
visible Bloom filter along with an exchange of messages between agents acting
on behalf of users. When an agent suspects that another agent possesses the
identify of a user, the agents may interrogate each other to either confirm
or deny the suspicion. Upon confirmation, the agents notify their respective
users so that the users my exchange other information, such as new or different
identifiers.

Confirmation of identifiers can either be one-way or two-way. Consider the
following scenario.

User Alice has an old acquaintance, User Bob. Many years ago, Bob was a frequent
reader and commenter on Alice's blog at https://example.com/alices-restaurant.

Alice has placed into her agent the hashes of several of her identifiers. The
hash value for her blog's URL is 12345. Bob has placed into his agent the hash
values for identifiers of many of his acquaintances, among them that of Alice's
blog URL. Both Alice and Bob are using a hash algorithm with high a collision rate, and
therefore the hash value 12345 may also be the hash value for another identifier
probably unrelated to either Alice or Bob.

Bob's agent obtains of list of hash values from Alice's agent, and discovers
that the hash value 12345 exists in both Bob's set and Alice's set.

If Alice desires contact from all readers of her blog, even the ones whom
she may not know, Bob's agent can seek one-way confirmation of Alice's identifier
through a series of further exchanges using different hash values and/or stronger
hash algorithms.

If Alice desires contact from only authenticated commenters of her blog, then
Bob's agent must engange in two-way confirmation. Here, Bob has placed into
his agent the hash value for his email address, bob@example.org, which is 67890.
As Bob was an authenticated commenter on Alice's blog, Alice has also placed
this hash value in her agent. The hash algorithm used to calculate this value
is the same as the one used to calculate the hash value of Alice's blog URL,
and therefore has a significant probability of generating the same number for
other, unrelated identifiers. When Bob's agent initiates confirmation, it
does so by offering the hash value 67890. As with one-way confirmation, the agent's
exchange a series of messages to seek further confirmation of the identifiers using
different hash values and/or stronger hash algorithms.

# Functional Components

The agents holding the hash values and conducting identity confirmation
communications are known as Exchange Agents in HDYD. Users interact with
User Agents. User Agents hold the identities of the user and the identities
being sought by the user, and User Agents calculate the hash values for
identities which are then placed into Exchange Agents.

As the generation of hash values is the responsibility of the User Agent,
but the Exchange Agent requires different or stronger hash values for
identity confirmation, the communications between Exchange Agents for the
purposes of identity confirmation is "asynchronous" and does not match 
typical request/response patterns. That is, Exchange Agents will be
required to retrieve new hash values to conduct identity confirmation using
User Agents which may or may not be immediately accessible.

For convenience and quicker exchanges, User Agents and Exchange Agents
may be combined into one operating entity.  Conversely, there is no explicit 
one-to-one match between User Agents and Exchange Agents. Exchange Agents 
may server many User Agents, and User Agents may utilize more than one
Exchange Agent.

Finally, this document does not define the communications between User Agents
and Exchange Agents.

# Internet HTTP Protocol

The following sections describe how HDYD is to work over the Internet
using HTTP where Exchange Agents are accessible as publicly available
resources.

Exchange Agents communicate by issuing HTTP requests using the paths and
query parameters defined in this document. They are configured to communicate
with each other using a "base URL" upon which the request componets defined herein
are then appended.

## Use of HTTP Signatures

Exchange Agents use HTTPS to communicate, and all but one HTTP exchange uses
HTTP Signatures to sign requests. The exception is the HTTP GET request of
a senders HTTP signing key.

The keyId used in the HTTP signature MUST be an HTTPS URL, and the URL MUST
resolve to a resource that is a PEM-encoded public key. Exchange Agents
MAY cache keys according to HTTP caching headers.

Both the Host and Date HTTP headers MUST be signed.

> There is probably a need to specify the key types and signature algorithms
> to use.

## Requesting Values

An Exchange Agent may request hash values from another agent using the `/hashes` path
(i.e. <base URL>/hashes). This request may optionally have the `before` and `after`
query parameters, the value of each being a string. These parameters are used
to paginate the values in the request, where `before` is an indicator that only
hash values created before its value are to be in the returned result, and `after`
is an indicator that only hash value created after its value are to be in the
returned result. The format for neither value is defined, and agents should use
neither parameter to request the latest hash values.

The hash values are returned as a JSON object with the following form:

    {
      "version": 1,
      "next" : "abcdefg",
      "prev" : "hijklmn",
      "hash_values": [
        {
          "u32": 11112,
          "alg": "FNV132",
          "two_way": false,
          "created": "20230102T12:59:00Z"
        },
        {
          "u32": 2221,
          "alg": "FNV132",
          "two-way": true,
          "created": "20230101T12:59:00Z"
        },
      ]
    }

The JSON members have the following meaning:

* `version` - this is a simple integer and MUST be 1.
* `next` - a string value which maybe used in subsequent requests as the `next` query parameter.
* `prev` - a string avlue which maybe used in subsequent requests as teh `prev` query parameter.
* `hash_values` - an array of JSON objects, each having:
  * `u32` - an unsigned 32-bit integer that is the value of the hash.
  * `alg` - the name of the hash algorithm.
  * `two_way` - indicates that confirmation of the identity derived from the hash value requires two-way confirmation.
  * `created` - an RFC 3339 data and time in UTC with no more than seconds resolution indicating when the hash value was placed in the Exchange Agent.

If the `next` string is not given, this indicates there are no more hash values available using the `next` query parameter.
If the `prev` string is not given, this indicates there are no more hash values available using the `prev` query parameter.
All other values MUST be given, but the `hash_values` array MAY be empty.

{backmatter}

# Other Environments

This document describes how HDYD works over the Internet by layering it over
HTTP. However, there may be other environments where parts of HDYD can be used
to achieve similar purposes. For example, the Activity Pub protocol and the
conventions established around the Fediverse provide enough properties to
to layer HDYD in Activity Pub itself, making the integration of HDYD more
seemless with that ecosystem.

Other environments, such as local area networks, may take parts of HDYD and
use them with mDNS or Bluetooth to facilitate the discovery of Exchange Agents.
The scenario being that a user with a smart phone containing both a User Agent 
and Exchange Agent may be notified that an acquaintance is nearby using HDYD.

# Acknowledgements

A conversation had with libations and Eric Osterweil was the inspiration for HDYD.
