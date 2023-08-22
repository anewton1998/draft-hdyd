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
identifier of a user, the agents may interrogate each other to either confirm
or deny the suspicion. Upon confirmation, the agents notify their respective
users so that the users my exchange other information, such as new or different
identifiers.

Confirmation of identifiers can either be one-way or two-way. Consider the
following scenario.

User Alice has an old acquaintance, User Bob. Many years ago, Bob was a frequent
reader and commenter on Alice's blog at https://example.com/alices_restaurant.

Alice has placed into her agent the hashes of several of her identifiers. The
hash value for her blog's URL is 12345. Bob has placed into his agent the hash
values for identifiers of many of his acquaintances, among them that of Alice's
blog URL. Both Alice and Bob are using a hash algorithm with high a collision rate, and
therefore the hash value 12345 may also be the hash value for another identifier
probably unrelated to either Alice or Bob.

Bob's agent obtains of list of hash values from Alice's agent, and discovers
that the hash value 12345 exists in both Bob's set and Alice's set.

If Alice desires contact from all readers of her blog, even the ones whom
she may not know, Alice's agent let's this be known. Here, Bob's agent can seek 
one-way confirmation of Alice's identifier
through a series of further exchanges using different hash values and/or stronger
hash algorithms.

If Alice desires contact from only authenticated commenters of her blog, then
Bob's agent must engange in two-way confirmation. Here, Bob has placed into
his agent the hash value for his email address, bob@example.org, which is 67890.
As Bob was an authenticated commenter on Alice's blog, Alice has also placed
this hash value in her agent. The hash algorithm used to calculate this value
is the same as the one used to calculate the hash value of Alice's blog URL,
and therefore has a significant probability of generating the same value for
other, unrelated identifiers. When Bob's agent initiates confirmation, it
does so by offering the hash value 67890. As with one-way confirmation, the agent's
exchange a series of messages to seek further confirmation of the identifiers using
different hash values and/or stronger hash algorithms.

# Functional Components

The agents holding the hash values and conducting identifier confirmation
communications are known as Exchange Agents (EA) in HDYD. Users interact with
User Agents (UA). User Agents hold the identifiers of the user and the identifiers
being sought by the user, and User Agents calculate the hash values for
identifiers which are then placed into Exchange Agents.

For convenience and quicker exchanges, User Agents and Exchange Agents
may be combined into one operating entity.  Conversely, there is no explicit 
one-to-one match between User Agents and Exchange Agents. Exchange Agents 
may serve many User Agents, and User Agents may utilize more than one
Exchange Agent.  This document does not define the communications between User Agents
and Exchange Agents.

An Exchange Agent initiating a onw-way (see (#one_way)) or two-way (see #(two_way)) 
confirmation communication flow are known as Requesting Exchange Agents (REA) and
the other agent is known as a Confirming Exchange Agent (CEA).

The identifier of a user is called a UID. For each UID, the UA calculates a hash value
from the UID using a collision-prone hash algorithm such as FNV. This hash value is called
UH1.

A second hash value, called UH2 is also calculated for each UID. This hash value uses a
separate, different collision-prone hash algorithm such as Murmur.

Finally, for each UID a third hash value is also calculated. This value is called UH3 and is
calculated with a hash algorithm that is collision-resistent such as SHA-256.

All three hash valudes, UH1, UH2, and UH3, are provisioned into the EA by the UA. The UID is not.

The identifiers of a user's acquaintances, perhaps held in a contact database,
have the same set of values generated by the UA and provisioned into the EA. Each
acquaintance identifier is called an AID, and the hash values
are AH1, AH2, and AH3.

In the confirmation flows described in this document, the UH1, UH2, and UH3 values
are associated with identifiers of users associated with a CEA. The AH1, AH2, and AH3
values are associated with identifiers of users associated with an REA.

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

## Requesting Values {#values_request}

An Exchange Agent may request UH1 values from another agent using the `/hashes` path
(i.e. <base URL>/hashes). This request may optionally have the `before` and `after`
query parameters, the value of each being a string. These parameters are used
to paginate the values in the request, where `before` is an indicator that only
hash values created before its value are to be in the returned result, and `after`
is an indicator that only hash value created after its value are to be in the
returned result. The format for neither value is defined, and agents should omit
the use of both parameter to request the latest values.

The values are returned as a JSON object with the following form:

    {
      "version": 1,
      "next" : "abcdefg",
      "prev" : "hijklmn",
      "hash_values": [
        {
          "uh1_u32": 11112,
          "alg": "FNV132",
          "two_way": false,
          "created": "20230102T12:59:00Z"
        },
        {
          "uh1_u32": 22221,
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
  * `uh1_u32` - an unsigned 32-bit integer that is a UH1.
  * `uh1_alg` - the name of the hash algorithm used to generate UH1.
  * `two_way` - indicates that confirmation of the identifier derived from UH1 requires two-way confirmation.
  * `created` - an RFC 3339 data and time in UTC with no more than seconds resolution indicating when the UH1 was placed in the Exchange Agent.

If the `next` string is not given, this indicates there are no more hash values available using the `next` query parameter.
If the `prev` string is not given, this indicates there are no more hash values available using the `prev` query parameter.
All other values MUST be given, but the `hash_values` array MAY be empty. However, if it is not empty each JSON object
MUST be in reverse chronological order according to the value in `created`.

If the requesting Exchange Agent finds a UH1 value matching one of its AH1 values, it may begin an identifier confirmation based on the
value of the `two_way` value for that hash.

## One-Way Confirmation {#one_way}

One-way confirmation begins with the REA sending an HTTP POST to the CEA at the path `/one_way` (i.e. <base URL>/one_way).
The data posted is a JSON object of the form:

    {
      "uh1_u32": 11112,
      "uh1_alg": "FNV132",
      "created": "20230102T12:59:00Z",
      "uh2_u32": 44443,
      "uh2_alg": "Murmur3"
    }

These JSON members have the following meaning:

* `uh1_u32` - the unsigned 32-bit integer that is a UH1.
* `uh1_alg` - the hash algorithm used to calculate UH1.
* `created` - the date and time value taken from (#values_request).
* `uh2_u32` - the unsigned 32-bit integer that is a UH2.
* `uh2_alg` - the hash algorithm used to calculate UH2.

The CEA uses the UH1 value and the `created` value to reference a set of user identifier hashes (UH1, UH2, and UH3).
If the UH2 value given matches the UH2 value in that set of hashes, the CEA responds with the UH3 in this form:

    {
      "uh3_base64": "b6fb35773416f37e51eb893a0b0682e23b4f758d5004542450be61607253f899",
      "uh3_alg": "SHA-256",
      "message": "I moved my blog, it is at https://example.com/alices_diner"
    }

If the REA can match this UH3 value to its corresponding value, then the REA may send a
subsequent HTTP POST to the path `/one_way_msg` (i.e. <base URL>/one_way_msg) with data of
the form:

    {
      "uh1_u32": 11112,
      "uh1_alg": "FNV132",
      "created": "20230102T12:59:00Z",
      "uh3_base64": "b6fb35773416f37e51eb893a0b0682e23b4f758d5004542450be61607253f899",
      "uh3_alg": "SHA-256",
      "message": "hi Alice, it's Bob."
    }

## Two-Way Confirmation {#two_way}

Two-way confirmation begins with the REA sending an HTTP POST to the CEA at the path `two_way_step1`
(i.e. <base URL>/two_way_step1). The data posted is a JSON object of the form:

    {
      "uh1_u32": 22221,
      "uh1_alg": "FNV132",
      "created": "20230102T12:59:00Z",
      "ah1_u32": 33331,
      "ah1_alg": "FNV132",
    }

If CEA allows two-way confirmation for the given UH1 and it has an AH1 associated with the UH1 value, 
it replys with a simple HTTP OK.

If the REA receives an OK, it may continue with step 2 by sending an HTTP POST to the CEA at the path
`two_way_step2` (i.e. <base URL>/two_way_step2). The data posted is a JSON object of the form:

    {
      "uh2_u32": 44443,
      "uh2_alg": "Murmur3",
      "created": "20230102T12:59:00Z",
      "ah1_u32": 33331,
      "ah1_alg": "FNV132",
      "ah2_u32": 55556,
      "ah2_alg": "Murmur3",
    }

If the CEA matches the given UH2 with its UH2 and `created` values, and the AH1 and AH2 values match
to a set of AH1 and AH2 values associated with the user of UH2, then the CEA responds
with a JSON object of the form:

    {
      "uh3_base64": "b6fb35773416f37e51eb893a0b0682e23b4f758d5004542450be61607253f899",
      "uh3_alg": "SHA-256",
      "message": "Hi Bob. I moved my blog, it is at https://example.com/alices_diner"
    }

If the REA can match this UH3 value to its corresponding value, then the REA may send a
subsequent HTTP POST to the path `/two_way_msg` (i.e. <base URL>/one_way_msg) with data of
the form:

    {
      "uh1_u32": 11112,
      "uh1_alg": "FNV132",
      "created": "20230102T12:59:00Z",
      "ah3_base64": "b6fb35773416f37e51eb893a0b0682e23b4f758d5004542450be61607253f899",
      "ah3_alg": "SHA-256",
      "message": "hi Alice, it's Bob. My new email is bobthebob@example.net"
    }

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
