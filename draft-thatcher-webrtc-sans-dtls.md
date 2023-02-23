---
title: "WebRTC sans DTLS"
abbrev: NO-DTLS
docname: draft-thatcher-webrtc-sans-dtls-latest
date: {DATE}
category: info

ipr: trust200902
area: General
submissionType: IETF
workgroup: Independent Submission
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs, docmapping]

author:
  -
    ins: P. Thatcher
    name: Peter Thatcher
    organization: Microsoft
    email: pthatcher@microsoft.com

--- abstract

This document proposes how two WebRTC peers can establish an SRTP session without DTLS by doing a Diffie-Helman Exchange over signaling.

--- middle


# Introduction

WebRTC uses DTLS-SRTP to negotiate SRTP keys.  This requires a DTLS handshake, which requires roundtrips and complexity vs. negotiating SRTP keys over signaling.

Negotiating SRTP keys was rejected by the IETF RTCWEB working group due to the ease of accidentally leaking SRTP keys.  

Can we have avoid the complexity and roundtrips of DTLS-SRTP while avoiding the problems of negotiating the SRTP keys over signaling?

We can by doing a Diffie-Helman Exchange over signaling.

# Using out-of-band signaling (no SDP)

WebRTC endpoints can signal the following pieces of information out-of-band (in an implementation-specific manner):

1. SRTP profile
2. DHE algorithm
3. DHE public key
4. HKDF salt
5. HKDF info fragment
6. Which endpoint is "first"

If these pieces of information are signaling from both sides, then the endpoints use a Diffie-Helman exchange followed by an HDKF expansion to generate SRTP keys rather than DTLS-SRTP. The endpoints must agree on a single value for the SRTP profile, the DHE algorithm, the HKDF salt, and which endpoint is "first".  Each endpoint will have its own value for the DHE Public Key and HKDF info fragment.

# Using JSEP SDP

In an SDP offer, an endpoint can add any number of lines in the following format:

```
a=srtp-dhe-hkdf SRTP_PROFILE:DHE_ALGORITHM:DHE_PUBLIC_KEY:HKDF_SALT:HKDF_INFO_FRAGMENT
```

In and SDP answer, an endpoint can add one such line with a matching SRTP_PROFILE, DHE_ALGORITHM, and HKDF_SALT.  But it will use its own value for DHE_PUBLIC_KEY and HKDK_INFO_FRAGMENT.  The endpoint sending the SDP offer is considered "first".

DHE_PUBLIC_KEY, HKDF_INFO_FRAMENT, and HKDF_SALT are hex-encoded byte strings. DHE_ALGORITHM is an ASCII string, such as "X25519", and SRTP_PROFILE is an ASCII string such as "AEAD_AES_256_GCM".  HKDF_INFO_FRAMENT and HKDF_SALT can be empty. 

An SDP offer MUST NOT have multiple a=srtp-dhe-hkdf lines that differ only by HKDF_INFO_FRAGMENT.

# Generating SRTP keys from DHE and HKDF

Assuming that values have been signaled from each endpoint, each endpoint should have the following pieces of information:

- A shared SRTP profile with keys of size K and salts of size S.
- A shared DHE algorithm
- A local DHE secret
- A remote DHE public key
- Which endoint is "first" and which is "second"
- A "first" HKDF info fragment (the HKDF info fragment of the "first" endpoint)
- A "second" HKDF info fragment (the HKDF info fragment of the "second" endpoint)

Using these, each endpoint derives SRTP keys in the following way:

1. Derive a shared secret from the DHE algorithm, local DHE secret, and remote DHE public key.
2. Derive an expanded shared secret by doing an HKDF expansion using the shared secret, the HKDF salt, and an HKDF info which is the concatentation of the fisrt HKDF info framgent and the second HKDF info fragment.
3. Extract SRTP keys from the the expanded shared secret by using using the first K bytes as the SRTP key for the "first" endpoint, the following S bytes as the SRTP key for the "first" endpoint, the following K bytes as the SRTP key for the "second" endpoint, and the following S bytes as the SRTP salt for the "second" endpoint.


# Contributors
{:numbered="false"}

- Peter Thatcher
