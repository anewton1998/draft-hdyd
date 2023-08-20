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

This document describes a system to aid in the discovery of identifers for
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

{backmatter}
