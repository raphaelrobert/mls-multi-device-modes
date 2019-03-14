---
title: The Messaging Layer Security (MLS) multi-device modes
abbrev: MLS multi-device modes
docname: draft-robert-mls-multi-device-modes-00
category: info

ipr: trust200902
area: Security
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: R. Robert
    name: Raphael Robert
    organization: Wire
    email: raphael@wire.com

informative:

  MLSARCH:
       title: "Messaging Layer Security Architecture"
       date: 2018
       author:
         -  ins: E. Omara
            name: Emad Omara
            organization: Google
            email: emadomara@google.com
         -  
            ins: R. Barnes
            name: Richard Barnes
            organization: Cisco
            email: rlb@ipv.sx
         -
	    ins: E. Rescorla 
            name: Eric Rescorla
            organization: Mozilla 
            email: ekr@rtfm.com
         -
            ins: S. Inguva 
            name: Srinivas Inguva 
            organization: Twitter 
            email: singuva@twitter.com
         -
            ins: A. Kwon 
            name: Albert Kwon
            organization: MIT 
            email: kwonal@mit.edu
         -
            ins: A. Duric 
            name: Alan Duric
            organization: Wire 
            email: alan@wire.com 


  MLSPROTO:
       title: "Messaging Layer Security Protocol"
       date: 2018
       author:
         -  ins: R. Barnes
            name: Richard Barnes
            organization: Cisco
            email: rlb@ipv.sx
         -
            ins: J. Millican
            name: Jon Millican
            organization: Facebook
            email: jmillican@fb.com
         -
            ins: E. Omara
            name: Emad Omara
            organization: Google
            email: emadomara@google.com
         -
            ins: K. Cohn-Gordon
            name: Katriel Cohn-Gordon
            organization: University of Oxford
            email: me@katriel.co.uk
         -
            ins: R. Robert
            name: Raphael Robert
            organization: Wire
            email: raphael@wire.com

--- abstract

This document describes how Messaging Layer Security (MLS) groups can be used with more than one device per user.


--- middle

# Introduction

Users can have more than one device on which they run an MLS client. Their clients are tied together on the application layer, and collectively represent a "user" or "user account". In the following, two different modes are introduced to achieve multi-device operation in MLS groups.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all
capitals, as shown here.

Client:
: An agent that uses this protocol to establish shared cryptographic
  state with other clients.  A client is defined by the
  cryptographic keys it holds.  An application or user may use one client
  per device (keeping keys local to each device) or sync keys among
  a user's devices so that each user appears as a single client.

User Init Key:
: A short-lived HPKE key pair used to introduce a new
  client to a group.  Initialization keys are published for
  each client (UserInitKey) as defined in {{MLSPROTO}}.

Identity Key:
: A long-lived signing key pair used to authenticate the sender of a
  message as defined in {{MLSPROTO}}.

## Multi-client mode

In this mode, every client occupies a leaf node in the TreeKEM ratcheting tree. The size of the tree is therefore defined by the number of clients that are a member of the group.

The application is responsible for managing the user devices inside the ratcheting tree. It is also responsible for determining whether a user is a member of a group depending on whether their devices are part of the group.

### UserInitKeys

Every client has its own UserInitKeys. UserInitKeys are initially uploaded by the client to the Delivery Service.

### Authentication

As per the MLS protocol draft, clients can have a joint identity key, but it is encouraged that every client has its own dentity key. The identity key is referenced in the UserInitKey.

### Adding and removing of users

When a new user is added to a group, all of their devices have to added as clients to the group. Currently there is no dedicated handshake message to bulk add members, but the DS can ensure that all of them are added atomically by not allowing any other handshake message to be inserted in the series of Add handshake messages.

When a user is removed from a group, all of their devices have to be removed from the group. Bulk removals can be ensure in the same manner as with Add messages (see above).

### Adding and removing of devices

Devices are added and removed with normal Add/Remove handshake messages.

## Single-client mode

In this mode, individual devices of a user are shielded behind a single "virtual" client. Users who are members of a group only use one leaf node of the ratcheting tree.

The leaf node in the ratcheting tree is however the root node of a per-user ratcheting tree. This per-user ratcheting tree has all user devices as leaf nodes. It is maintained by the application whenever devices are added/removed for a certain user.

The root node of the per-user ratcheting tree changes its value whenever a device is added or removed from the user account. The changed root node value leads to a change of the leaf node in the group. This change is propagated to the group by issuing an Update handshake message in the group.

### UserInitKeys

UserInitKeys are not specific to a certain device, they are rather valid for one single "virtual" client. The private part of a UserInitKey has to be known to all devices of a user.
In order to simplify the sharing of the private keys, the UserInitKey could be derived from the per-user ratchet tree key schedule, eliminating the need to synchronize it among devices. For robustness, private values of older UserInitKeys could still be shared among devices of a user.

### Authentication

As per the MLS protocol draft, clients can have a joint identity key, but it is encouraged that every client has its own identity key. One of the identity keys is referenced in the UserInitKey.

### Adding and removing of users

When a new user is added to a group only its single "virtual" client is added to the group with an Add handshake message. The corresponding Welcome handshake message is encrypted with the help of the UserInitKey as usual.

When a user is removed from a group, the corresponding leaf node is removed through a Remove handshake message.

### Adding and removing of devices

Devices are added and removed with normal Add/Remove handshake messages inside the per-user ratcheting tree. Changes of that tree's root node must be propagated to the groups by issuing an Update message in the groups.

# Security Considerations

## Application layer
Both modes require security checks to be implemented on the application layer

# IANA Considerations
This document makes no requests of IANA.
