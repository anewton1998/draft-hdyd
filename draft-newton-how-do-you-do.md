%%%
Title = "Howdy: The How Do You Do Protocol"
area = "Applications and Real-Time Area (ART)"
workgroup = "Network Working Group"
abbrev = "howdy"
ipr= "trust200902"

[seriesInfo]
name = "Internet-Draft"
value = "draft-newton-how-do-you-do-00"
stream = "IETF"
status = "experimental"
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

This document describes a system to discover the identifiers of natural
persons while preserving their privacy.

{mainmatter}

# Background

In a song written by children's author Shel Silverstein and made famous by
Johnny Cash, a young man seeks out and identifies his long, astranged father
using an old, worn photograph. Upon finding his father, the young man gives his name
and asks of his father, "how do you do?"

This document describes a system, called "Howdy" which is short for "how do you do",
to aid in the discovery of identifers for
natural persons while preserving their privacy. The system uses a publicly
visible [@!Bloom] filter along with an exchange of messages between agents acting
on behalf of users. When an agent suspects that another agent possesses the
identifier of a user, the agents may interrogate each other to either confirm
or deny the suspicion. Upon confirmation, the agents notify their respective
users so that the users may exchange other information, such as new or different
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

Bob's agent obtains the list of hash values from Alice's agent, and discovers
that the hash value 12345 exists in both Bob's set and Alice's set.

If Alice desires contact from all readers of her blog, even the ones whom
she may not know, Alice's agent let's this be known. Here, Bob's agent can seek 
one-way confirmation of Alice's identifier
through a series of further exchanges using different hash values and/or stronger
hash algorithms.

If Alice desires contact from only known commenters of her blog, then
Bob's agent must engange in two-way confirmation. Here, Bob has placed into
his agent the hash value for his email address, bob@example.org, which is 67890.
As Bob was a known commenter on Alice's blog, Alice has also placed
this hash value in her agent. The hash algorithm used to calculate this value
is the same as the one used to calculate the hash value of Alice's blog URL,
and therefore has a significant probability of generating the same value for
other, unrelated identifiers. When Bob's agent initiates confirmation, it
does so by offering the hash value 67890. As with one-way confirmation, the agent's
exchange a series of messages to seek further confirmation of the identifiers using
different hash values and/or stronger hash algorithms.

# Functional Components

The agents holding the hash values and conducting identifier confirmation
communications are known as Exchange Agents (EA) in Howdy. Users interact with
User Agents (UA). User Agents hold the identifiers of the user and the identifiers
being sought by the user, and User Agents calculate the hash values for
identifiers which are then placed into Exchange Agents.

For convenience and quicker exchanges, User Agents and Exchange Agents
may be combined into one operating entity.  Conversely, there is no explicit 
one-to-one match between User Agents and Exchange Agents. Exchange Agents 
may serve many User Agents, and User Agents may utilize more than one
Exchange Agent.  This document does not define the communications between User Agents
and Exchange Agents.

An Exchange Agent initiating a one-way (see (#one_way)) or two-way (see (#two_way)) 
confirmation communication flow are known as Requesting Exchange Agents (REA) and
the other agent is known as a Confirming Exchange Agent (CEA).

Action     | Type           | Name
-----------|----------------|-------
Requesting | User Agent     | RUA
Requesting | Exchange Agnet | REA
Confirming | User Agent     | CUA
Confirming | Exchange Agnet | CEA

The identifier of a user is called a UID. For each UID, the UA calculates a hash value
from the UID using a collision-prone hash algorithm such as FNV (see [@!I-D.eastlake-fnv]). This hash value is called
UH1.

A second hash value, called UH2 is also calculated for each UID. This hash value uses a
separate, different collision-prone hash algorithm such as Murmur (see [@!Murmur3]).

Finally, for each UID a third hash value is also calculated. This value is called UH3 and is
calculated with a hash algorithm that is collision-resistent such as SHA-256 ([@!RFC6234]).

All three hash values, UH1, UH2, and UH3, are provisioned into the CEA by the CUA. The UID is not.

Hash of | Type                | Name | Example
--------|---------------------|------|---------------
UID     | Collision Prone     | UH1  | fnv( UID )
UID     | Collision Prone     | UH2  | murmur3( UID )
UID     | Collision Resistent | UH3  | sha256( UID )

The identifiers of a user's acquaintances, perhaps held in a contact database,
have the same set of values generated by the UA and provisioned into the EA. Each
acquaintance identifier is called an AID, and the hash values
are AH1, AH2, and AH3.

Hash of | Type                | Name | Example
--------|---------------------|------|---------------
AID     | Collision Prone     | AH1  | fnv( AID )
AID     | Collision Prone     | AH2  | murmur3( AID )
AID     | Collision Resistent | AH3  | sha256( AID )

# Flows

Preceding confirmation flows, the CEA should make UH1 values known through a publication
mechanism. Confirmation may then take place where an REA has a suspicion that a CEA
contains the same UID. Publication of UH1 MUST include a signal determining the type
of confirmation to be used with that value.

~~~ ascii-art
SEQUENCE: 1. Provision Hashses

+-----+       +-----+           +-----+       +-----+
| CUA |       | CEA |           | RUA |       | REA |
+-----+       +-----+           +-----+       +-----+
   |             |                 |             |
   |~~~hashes~~~>|                 |             |
   |             |                 |             |
   |<------------|                 |             |
   |             |                 |             |
   |             |                 |~~~hashes~~~>|
   |             |                 |             |
   |             |                 |<------------|
   |             |                 |             |
   |          ------------------   |             |
   |          | Publishes UH1s |   |             |
   |          ------------------   |             |
   |             |                 |             |
+-----+       +-----+           +-----+       +-----+
| CUA |       | CEA |           | RUA |       | REA |
+-----+       +-----+           +-----+       +-----+


SEQUENCE: 2. Confirmation

+-----+  +-----+
| REA |  | CEA |
+-----+  +-----+
   |        |
----------------
| confirmation |
----------------
   |        |
   |~~~~~~~>|
   |        |
   |<-------|
   |        |
+-----+  +-----+
| REA |  | CEA |
+-----+  +-----+
~~~
Figure: High Level Flow

Identifier confirmation occurs in two types of communication flows: one-way and two-way.
Each flow type has a set of steps specific to that flow. One-way confirmation is a flow
in which a UID is confirmed. Two-way confirmation is a flow in which both a UID and an AID
are confirmed.

## One Way Confirmation {#one_way}

These are the steps for one-way confirmation:

*Step 1:* The REA sends a message containing a UH1 and UH2.
Preceding this step, the REA could have learned of the UH1 value via publication by the CEA.
UH1 and UH2 MUST be hashed with different algorithms, both being collision prone. By providing
a message with UH1 and UH2, the REA is suggesting it may possess the value of UID.

*Step 2:* If both the UH1 and UH2 are associated with the same UID, the CEA confirms this assertion
by responding with the UH3 of that UID. As UH3 is more collision resistent, the value of UID is confirmed.
This step also includes a message to the user associated with the AID.

*Step 3:* Having established confirmation, subsequent messages are possible.
Optionally, the REA may followup with messages for the user using UH1 and UH3 as credentials.
These messages would then be passed by the CEA to the CUA.

~~~ ascii-art
SEQUENCE: One-Way Confirmation

+-----+                                +-----+
| REA |                                | CEA |
+-----+                                +-----+
   |                                      |
   |~~~step 1: UH1 + UH2~~~~~~~~~~~~~~~~~>|
   |                                      |
   |<--step 2: UH3 + msg_for_AID----------|
   |                                      |
   |~~~step 3: UH1 + UH3 + msg_for_UID~~~>|
   |                                      |
   |<--OK---------------------------------|
   |                                      |
+-----+                                +-----+
| REA |                                | CEA |
+-----+                                +-----+


SEQUENCE: Message to Users

+-----+            +-----+ +-----+            +-----+
| REA |            | RUA | | CEA |            | CUA |
+-----+            +-----+ +-----+            +-----+
   |                  |       |                  |
   |~~~msg_for_AID~~~>|       |                  |
   |                  |       |                  |
   |<-----------------|       |                  |
   |                  |       |                  |
   |                  |       |~~~msg_for_UID~~~>|
   |                  |       |                  |
   |                  |       |<-----------------|
   |                  |       |                  |
+-----+            +-----+ +-----+            +-----+
| REA |            | RUA | | CEA |            | CUA |
+-----+            +-----+ +-----+            +-----+
~~~
Figure: One Way Confirmation

## Two Way Confirmation {#two_way}

These are the steps for two-way confirmation:

*Step 1:* The REA sends a message containing a UH1 and AH1. As this is two-way confirmation,
the AH1 is associated with an AID believed to be known by the user identified by the UID.

*Step 2:* If the CEA can associate the AH1 with the UH1 
(that is, the AID is associated with the UID), it responds with a simple positive acknowledgement of the match.

*Step 3:* Next, the REA sends the UH2 and AH2 corresponding to the UH1 and AH1 in the first message. UH1 and UH2
MUST be hashsed with different algorithms, and AH1 and AH2 MUST be hashed with different algorithms.

*Step 4:* If the CEA can associate the UH2 with the UH1 of the first message and the AH2 with 
the AH1 of the first message, the CEA confirms by sending the UH3 and AH3 thus confirming the CEAs
knowledge of both association with the UID and AID. This step includes a message for the user associated with the AID.

*Step 5:* Finally, the REA confirms the AID by sending a message to the CEA containing the AH3.
This is similar to step 3 in (#one_way).

~~~ ascii-art
SEQUENCE: Two-Way Confirmation

+-----+                                +-----+
| REA |                                | CEA |
+-----+                                +-----+
   |                                      |
   |~~~step 1: UH1 + AH1~~~~~~~~~~~~~~~~~>|
   |                                      |
   |<--step 2: OK-------------------------|
   |                                      |
   |~~~step 3: UH2 + UH2~~~~~~~~~~~~~~~~~>|
   |                                      |
   |<--step 4: UH3 + AH3 + msg_for_AID----|
   |                                      |
   |~~~step 5: AH3 + msg_for_UID~~~~~~~~~>|
   |                                      |
   |<--OK---------------------------------|
   |                                      |
+-----+                                +-----+
| REA |                                | CEA |
+-----+                                +-----+


SEQUENCE: Message to Users

+-----+            +-----+ +-----+            +-----+
| REA |            | RUA | | CEA |            | CUA |
+-----+            +-----+ +-----+            +-----+
   |                  |       |                  |
   |~~~msg_for_AID~~~>|       |                  |
   |                  |       |                  |
   |<-----------------|       |                  |
   |                  |       |                  |
   |                  |       |~~~msg_for_UID~~~>|
   |                  |       |                  |
   |                  |       |<-----------------|
   |                  |       |                  |
+-----+            +-----+ +-----+            +-----+
| REA |            | RUA | | CEA |            | CUA |
+-----+            +-----+ +-----+            +-----+
~~~
Figure: Two Way Confirmation

# Identifier Canonicalization

Some identifiers have extraneous characters that are insignificant to the usage of those
identifiers. For such identifiers, User Agents MUST remove insignificant characters from those
identifiers before creating hashes.

For phone numbers, User Agents MUST remove all non-digit characters. For email addresses,
the display name and any characters used to distinguish the display name from the email address
must be removed (e.g. Bob \<bob@example.com\> should be bob@example.com).

# Internet HTTP Binding

The following sections describe a binding of Howdy to Internet
HTTP ([@!RFC9110]) where Exchange Agents are accessible as publicly available
resources. (#other_environments) discusses other environments and
deployment scenarios for Howdy.

Exchange Agents communicate by issuing HTTP requests using the paths and
query parameters defined in this document. They are configured to communicate
with each other using a "base URL" upon which the request componets defined herein
are then appended.

## Use of HTTP Signatures

Exchange Agents use HTTPS to communicate, and use
HTTP Signatures [@!I-D.ietf-httpbis-message-signatures] to sign requests, with an exception being the HTTP GET request to
fetch the HTTP signing key of a requester.

The keyId used in the HTTP signature MUST be an HTTPS URL, and the URL MUST
resolve to a resource that is a PEM-encoded public key. Exchange Agents
MAY cache keys according to HTTP caching headers.

Both the `host` and `date` HTTP headers MUST be signed, and the value of the `host` 
header SHOULD be equivalent to the host of the authority component of the URL that
is the value of the keyId (e.g. `host: foo.example` matches `https://foo.example/key.pem`).

A> TODO: There is probably a need to specify the key types and signature algorithms
A> to use.

## JSON Vocabulary {#json_vocabulary}

The messages described in the following section contain JSON. The meaning of these JSON values
are:

* `version` - this is a simple integer and MUST be 1.
* paging values:
  * `next` - a string value which maybe used in subsequent requests as the `next` query parameter.
  * `prev` - a string value which maybe used in subsequent requests as the `prev` query parameter.
* `hash_values` - an array of JSON objects, each containing hashes.
* `two_way` - indicates that confirmation of an identifier requires the two-way process.
* `created` - an RFC 3339 data and time in UTC with no more than seconds resolution indicating when an associated item was placed in the Exchange Agent.
* UID related values:
  * `uh1_u32` - an unsigned 32-bit integer that is a UH1.
  * `uh1_alg` - the name of the hash algorithm used to generate UH1.
  * `uh2_u32` - an unsigned 32-bit integer that is a UH2.
  * `uh2_alg` - the name of the hash algorithm used to generate UH2.
  * `uh3_base64` - a Base64 string containing a UH3.
  * `uh3_alg` - the name of the hash algorithm used to generate UH3.
* AID related values:
  * `ah1_u32` - an unsigned 32-bit integer that is a AH1.
  * `ah1_alg` - the name of the hash algorithm used to generate AH1.
  * `ah2_u32` - an unsigned 32-bit integer that is a AH2.
  * `ah2_alg` - the name of the hash algorithm used to generate AH2.
  * `ah3_base64` - a Base64 string containing a AH3.
  * `ah3_alg` - the name of the hash algorithm used to generate AH3.
* message values:
  * `msgs` - an array containing message objects.
  * `msg_type` - a string signifying the type of the message content.
  * `msg_content` - the JSON type of this value is dependent on the accompanying `msg_type` value.
* `exchange_agents` - an array containg exchange agent location objects.
* `agent_url` - a string containing a URL of an agent.
* `notifications_accepted` - a boolean indicating if an Exchange Agent accepts notifications.

A> TODO: specify an IANA registry of algorithms to use.

## Publication {#values_request}

Publication is accomplished using a mechanism to allow REAs to collect pages of UH1s from CEAs.
A push based mechanism is also defined in (#notifications).

An Exchange Agent may request UH1 values from another agent using the `/hashes` path
(i.e. <base URL>/hashes). This request may optionally have the `next` and `prev`
query parameters, the value of each being a string. These parameters are used
to paginate the values in the request, where `next` is an indicator that only
hash values created before its value are to be in the returned result, and `prev`
is an indicator that only hash value created after its value are to be in the
returned result. The format for neither value is defined, and agents should omit
the use of both parameter to request the latest values.

The values are returned as a JSON object with the following form (see (#json_vocabulary)):

    {
      "version": 1,
      "next" : "abcdefg",
      "prev" : "hijklmn",
      "notifications_accepted": true
      "hash_values": [
        {
          "uh1_u32": 11112,
          "alg": "FNV",
          "two_way": false,
          "created": "20230102T12:59:00Z"
        },
        {
          "uh1_u32": 22221,
          "alg": "FNV",
          "two-way": true,
          "created": "20230101T12:59:00Z"
        },
      ]
    }

If the `next` string is not given, this indicates there are no more hash values available using the `next` query parameter.
If the `prev` string is not given, this indicates there are no more hash values available using the `prev` query parameter.
All other values MUST be given, but the `hash_values` array MAY be empty. However, if it is not empty each JSON object
MUST be in reverse chronological order according to the value in `created`.

If the requesting Exchange Agent finds a UH1 value matching one of its AH1 values, it may begin an identifier confirmation based on the
`two_way` value for that hash.

## One-Way Confirmation {#http_one_way}

One-way confirmation begins with the REA sending an HTTP POST to the CEA at the path `/one_way` (i.e. <base URL>/one_way).
The data posted is a JSON object of the form (see (#json_vocabulary)):

    {
      "version": 1,
      "uh1_u32": 11112,
      "uh1_alg": "FNV",
      "created": "20230102T12:59:00Z",
      "uh2_u32": 44443,
      "uh2_alg": "Murmur3"
    }

The CEA uses the UH1 value and the `created` value to reference a set of user identifier hashes (UH1, UH2, and UH3).
If the UH2 value given matches the UH2 value in that set of hashes, the CEA responds with the UH3 in this form (see (#json_vocabulary)):

    {
      "version": 1,
      "uh3_base64": "b6fb35773416f37e51eb893a0b0682e23b4f758d5004542450be61607253f899",
      "uh3_alg": "SHA-256",
      "msgs" : [
        {
          "msg_type": "json_text",
          "msg_content": "I moved my blog, it is at https://example.com/alices/diner"
        }
      ]
    }

If the REA can match this UH3 value to its corresponding value, then the REA may send a
subsequent HTTP POST to the path `/one_way_msg` (i.e. <base URL>/one_way_msg) with data of
the form (see (#json_vocabulary)):

    {
      "version": 1,
      "uh1_u32": 11112,
      "uh1_alg": "FNV",
      "created": "20230102T12:59:00Z",
      "uh3_base64": "b6fb35773416f37e51eb893a0b0682e23b4f758d5004542450be61607253f899",
      "uh3_alg": "SHA-256",
      "msgs" : [
        {
          "msg_type": "json_text",
          "msg_content": "hi Alice, it's Bob."
        }
      ]
    }

## Two-Way Confirmation {#http_two_way}

Two-way confirmation begins with the REA sending an HTTP POST to the CEA at the path `two_way_step1`
(i.e. <base URL>/two_way_step1). The data posted is a JSON object of the form (see (#json_vocabulary)):

    {
      "version": 1,
      "uh1_u32": 22221,
      "uh1_alg": "FNV",
      "created": "20230102T12:59:00Z",
      "ah1_u32": 33331,
      "ah1_alg": "FNV",
    }

If CEA allows two-way confirmation for the given UH1 and it has an AH1 associated with the UH1 value, 
it replys with a simple HTTP OK.

If the REA receives an OK, it may continue by sending an HTTP POST to the CEA at the path
`two_way_step3` (i.e. <base URL>/two_way_step3) (see step 3 in (#two_way)). The data posted is a JSON object of the form (see (#json_vocabulary)):

    {
      "version": 1,
      "uh2_u32": 44443,
      "uh2_alg": "Murmur3",
      "created": "20230102T12:59:00Z",
      "ah1_u32": 33331,
      "ah1_alg": "FNV",
      "ah2_u32": 55556,
      "ah2_alg": "Murmur3",
    }

If the CEA matches the given UH2 with its UH2 and `created` values, and the AH1 and AH2 values match
to a set of AH1 and AH2 values associated with the user of UH2, then the CEA responds
with a JSON object of the form (see (#json_vocabulary)):

    {
      "version": 1,
      "uh3_base64": "b6fb35773416f37e51eb893a0b0682e23b4f758d5004542450be61607253f899",
      "uh3_alg": "SHA-256",
      "ah3_base64": "b6fb35773416f37e51eb893a0b0682e23b4f758d5004542450be61607253f899",
      "ah3_alg": "SHA-256",
      "msgs" : [
        {
          "msg_type": "json_text",
          "msg_content": "Hi Bob. I moved my blog, it is at https://example.com/alices_diner"
        }
      ]
    }

If the REA can match this UH3 value to its corresponding value, then the REA may send a
subsequent HTTP POST to the path `/two_way_msg` (i.e. <base URL>/two_way_msg) with data of
the form (see (#json_vocabulary)):

    {
      "version": 1,
      "uh1_u32": 11112,
      "uh1_alg": "FNV",
      "created": "20230102T12:59:00Z",
      "ah3_base64": "b6fb35773416f37e51eb893a0b0682e23b4f758d5004542450be61607253f899",
      "ah3_alg": "SHA-256",
      "msgs" : [
        {
          "msg_type": "json_text",
          "msg_content": "hi Alice, it's Bob. My new email is bobthebob@example.net"
        }
      ]
    }

## Message Types

In the previous sections, the examples used the message type `json_text` to convey simple, JSON compatible strings.
However, the protocol supports multiple messages of various types. Each type is to be listed in an IANA registry.

### Simple Message Types

This document defines the following "simple" message types.

* `json_text` - the `msg_content` is a JSON compatible string.
* `email` - the `msg_content` is a JSON string containing an [@!RFC6530] compliant email address.
* `web` - the `msg_content` is a JSON string containing an [@!RFC3982] URL of a resource intended to be used with a web browser.
* `profile` - the `msg_content` is a JSON string containing an [@!RFC3982] URL that resolves to a message set (see (#message_sets)).

### Cryptographic Message Types

If the message type is `pem_pubkey`, the `msg_content` is an array of JSON strings containing an [@!RFC1422] PEM encoded [@!RFC5280] public key.
Each string of the array represents a "line" of text of the PEM structure.

    "msg_type": "pem_pubkey",
    "msg_content": [
      "-----BEGIN PUBLIC KEY-----",
      "MIIBCgKCAQEA1e8xKwbqeUNyCKMjsiGhIAgQ8KG6dHBmt10QcDszctb64Fb5lju",
      "rNwzOJ4ue4cbXRfD66ZvWzBHXsJmJCk5qkLEcdbZZ4zGz2N4wf7GxIiJXSfviH+",
      "3tt2Fd+/YcqGsyTZtjYyvcE6b1eighG8JKl15c7tq9lSFxz0PvshNEWXXEhML8n",
      "wDqIRnPqKIw4v3dDFd4rqzVNGKhMQ0DaKplmbRRLavdrsgBOZhhyanZEQKBL3/8",
      "+3rQ7vMSc7/3FBUIncu5rvFgoT10Pv4KDDjv3UMXeC669RNmOjgJQQ0Y2o1k1h2",
      "56meaojisqZ59Fr2YQYqg24F3Tzu5yKLgIDAQAB",
      ""-----END RSA PUBLIC KEY-----"
    ]

If the message type is `pem_cms`, the `msg_content` is an array of JSON strings containing an [@!RFC1422] PEM encoded CMS ([@!RFC5280]) object
in the form described above.

If the message type is `self_pem_cms`, the `msg_content` is an array of JSON strings of the same type as `pem_cms`, however the key material
used for the CMS is either the UID or AID, or known to be associated with the UID or AID (such as key id).

A> TODO: This needs some more work, obviously.

## Message Sets {#message_sets}

A set of messages can be grouped together for access using the following form:

    {
      "version": 1,
      "msgs" : [
        {
          "msg_type": "email",
          "msg_content": "bobthebob@example.net"
        },
        {
          "msg_type": "web",
          "msg_content": "https://example.com/blog/bob"
        }
      ]
    }

Howdy defines no URL pattern for access to message sets, however Exchange Agents MAY
make them publicly accessible based on permissions from a User Agent (e.g. https://example.com/profile/bob).

## Distribution {#distribution}

An Exchange Agent may request the locations of other agents from a known agent
using the `/exchange_agents` path (i.e. <base URL>/exchange_agents). 
This request may optionally have the `next` and `prev`
query parameters, the value of each being a string. These parameters are used
to paginate the values in the request, where `next` is an indicator that only
agents created before its value are to be in the returned result, and `prev`
is an indicator that only agents created after its value are to be in the
returned result. The format for neither value is defined, and agents should omit
the use of both parameter to request the latest values.

The values are returned as a JSON object with the following form (see (#json_vocabulary)):

    {
      "version": 1,
      "next" : "abcdefg",
      "prev" : "hijklmn",
      "notifications_accepted": true
      "exchange_agents" : [
        {
          "agent_url": "https://agent1.example",
          "created": "20230102T12:59:00Z"
        },
        {
          "agent_url": "https://agent2.example",
          "created": "20230101T12:59:00Z"
        }
      ]
    }

## Notifications {#notifications}

Exchange Agents may provide each other with unsolicited notifications. An Exchange Agent
indicates if it is willing to receive notifications using the `notifications_accepted` value
found in the JSON messages of (#distribution) and (#values_request).

Notifications are received using the `/notifications` path (i.e. <base URL/notifications) and
have the following form (see (#json_vocabulary)):

    {
      "version": 1,
      "hash_values": [
        {
          "uh1_u32": 11112,
          "alg": "FNV",
          "two_way": false,
          "created": "20230102T12:59:00Z"
        },
        {
          "uh1_u32": 22221,
          "alg": "FNV",
          "two-way": true,
          "created": "20230101T12:59:00Z"
        },
      ],
      "exchange_agents" : [
        {
          "agent_url": "https://agent1.example",
          "created": "20230102T12:59:00Z"
        },
        {
          "agent_url": "https://agent2.example",
          "created": "20230101T12:59:00Z"
        }
      ]
    }

The `hash_values` array is the same as is used (#values_request) and the `exchange_agents` array
is the same is used in (#distribution).

# Security Considerations

Identifiers of users are not generally secrets and are sometimes very well known.
This invites a type of attack where an Exchange Agent may purposefully be populated
with hashes of well-known identifiers for the purposes of attracting victims. For
Howdy, this is especially easy to accomplish with one-way confirmation. User Agents
MUST inform users to take additional measures of confirmation using out-of-band
communications if possible.


{backmatter}

# Other Environments {#other_environments}

## Fediverse

This document describes how Howdy works over the Internet by layering it over
HTTP. However, there may be other environments where parts of Howdy can be used
to achieve similar purposes. For example, the Activity Pub protocol and the
conventions established around the Fediverse provide enough properties to
to layer Howdy in Activity Pub itself, making the integration of Howdy more
seemless with that ecosystem.

## NOSTR

NOSTR is another environment similar to the Fediverse but with different goals.
NOSTR uses cryptographic public keys as user identifiers. Howdy could be used
in NOSTR to map NOSTR public keys to email addresses and other user identifiers,
including other NOSTR public keys.
Additionally, a binding of Howdy to the NOSTR protocol could make exchange of
user identifiers more seamless in that environment.

## LANS

Other environments, such as local area networks, may take parts of Howdy and
use them with mDNS or Bluetooth to facilitate the discovery of Exchange Agents.
This scenario being that a user with a smart phone containing both a User Agent 
and Exchange Agent may be notified that an acquaintance is physically nearby using Howdy.

## MIMI

The IETF's More Instant Messaging Interoperability (MIMI) working group is
defining protocols for the interchange of instant messages across protocol boundaries.
Howdy could be used for the discovery of identifiers and mapping of identifiers
between the varous instant messaging systems.

# Acknowledgements

A conversation had with libations and Eric Osterweil was the inspiration for Howdy.

<reference anchor="Murmur3" target="https://github.com/aappleby/smhasher/blob/master/src/MurmurHash3.cpp">
  <front>
    <title>Murmur Hash 3</title>
    <author fullname="Austin Appleby">
    </author>
  </front>
</reference>

<reference anchor="Bloom">
  <front>
    <title>Space/time trade-offs in hash coding with allowable errors.</title>
    <author fullname="Burton H. Bloom">
    </author>
    <date year="1970"/>
  </front>
  <seriesInfo name="Communications of the ACM" value="ACM 13"/>
</reference>


