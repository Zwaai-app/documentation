Cryptography
============

Checking In in Spaces
---------------------

Goal: prevent possibility of somebody publishing where infections have taken
place. It is not desirable that competing supermarkets can give each other a bad
reputation by publishing that there have been infections there.

In order to know that an infection has occurred and who need to act upon it, two
things are needed when checking in at a space: (1) the location identifier, and
(2) the time stamp. When checking in, the protocol ensures that the server does
not know the location identifier, and the client (end-user device) does not know
the time code. The first means that, while the server sees a check in request,
it does not know _where_ the user is. The second means that the resulting check
in codes that are stored on the end user device cannot be correlated to deduce
where and when infections have taken place.

The time codes used by the server are random numbers that are generated every
five minutes. So this random numbers serves as a representation of a 5-minute
time interval.

We accomplish this using finite field arithmetic. The following equations use
lower case variable names for scalars, and upper case variable names for group
elements.

When checking in in a space:

1. The end-user device scans a QR code. This QR code contains the location
   identifier: group element `L`. The end-user device then generates a scalar
   random number `r` and multipliers the two:
   ```
   RL = r * L
   ```
   This group element `RL` is an encrypted representation of the location
   identifier.
2. The encrypted location identifier `RL` is sent to the server.
3. The server takes the current time interval random number, scalar `t`, and
   multiplies it with the code received from the end-user device:
   ```
   TRL = t * RL
   ```
   This number is an encrypted representation of both the time and the location.
4. The number `TRL` is returned to the end-user device.
5. Using the finite field arithmetic, the client removes the factor `r`:
   ```
   TL = TRL / r
   ```
   This code, which is the decrypted representation of both the time and the
   location, is stored on the end-user device.

When the user turns out to be infected, the user's history, as many days back as
is relevant given the virus's properties, is sent to another server. This server
publishes a list of "infected `TL` codes", so that other users may periodically
download the list and, by looking up those codes in their personal history,
determine if they are at risk of infection themselves.

It is important to note that, even though the `TL` code itself cannot be used to
deduce the infected location (because of the finite field arithmetic), if the
server has the list of used time codes `t` the server could compute the location
codes:

```
L = TL / t
```

This can be mitigated by splitting the work to two servers: one that manages the
time codes and performs the encryption, and another that stores the list of
known infections, but doesn't need to have the time codes.

Also, the encrypting server should store the minimum number of time codes. If
the maximum amount of time in a space is 8 hours, this means that the server has
to store a rolling window of 8 hours of time codes, anything older than 8 hours
can be discarded. Still this requires the server to be solidly secured, because
extraction of the time codes allows other parties to compute the infected
locations from the list of `TL` codes.

Also note that the servers probably need to have redundancy, which means that
the instances need to share the same 8 hours worth of time codes, further
increasing the attack surface. The same goes for replication for scalability if
that is needed.
