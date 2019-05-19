# The application/branch Media Type and the .well-known URIs suffix "shrubbery"

draft-shrub.fr-shrubbery

## Abstract

This document specifies the application/branch Media Type, which can be used to transfer a branch of a shrub in various protocols and applications. This document also defines a mapping between branches and .well-known URIs which is usefull for [the retrieval of branches with low latency][shrub-fetch] which is required by protocols and applications such as [the Shrub https subscheme protocol][shrub].

## Status of this memo

This document is a pre-draft which will be submited to the IETF in order to register the definition of the application/branch Media Type and of the .well-known URIs suffix "shrubbery".

## Copyright Notice

[![Creative Commons 0](https://i.creativecommons.org/p/zero/1.0/80x15.png)](https://creativecommons.org/publicdomain/zero/1.0/) To the extent possible under law, the editors have waived all copyright and related or neighboring rights to this work. In addition, as of May 19, 2019, the editors have made this specification available under the Open Web Foundation Agreement Version 1.0, which is available at <http://www.openwebfoundation.org/legal/the-owf-1-0-agreements/owfa-1-0>. Parts of this work may be from another specification document. If so, those parts are instead covered by the license of that specification document.

## Introduction

TODO

## Cryptographic primitives

[_Gimli_](https://gimli.cr.yp.to/spec.html) is a permutation of 48-bytes-long strings.

_Gimli-Absorb_ is a function which takes as inputs a sponge state _S_ and a byte-string _A_, and updates the sponge state _S_. The sponge state _S_ is a 48-bytes-long string. The length of _A_ MUST be a multiple of 16. _Gimli-Absorb_ is defined by the following algorithm:

1. Let _l_ be the length of _A_
1. Let _k_ be the integer 0.
1. Let _pos_ be the integer 0.
1. While _k_ < _l_:
   1. _S_[_pos_] ^= _A_[_k_]
   1. _k_ += 1
   1. _pos_ += 1
   1. If _pos_ == 16:
      1. If _k_ == _l_, then _S_[47] ^= 0x01
      1. _S_ = _Gimli_(_S_)
      1. _pos_ = 0

_Gimli-Encrypt_ is a function which takes as inputs a sponge state _S_ and a plaintext _P_, updates the sponge state _S_ and returns the ciphertext _C_. The sponge state _S_ is a 48-bytes-long string. The length of _P_ MUST be a multiple of 16. _Gimli-Encrypt_ is defined by the following algorithm:

1. Let _l_ be the length of _P_
1. Let _C_ be a byte-string of length _l_.
1. Let _k_ be the integer 0.
1. Let _pos_ be the integer 0.
1. While _k_ < _l_:
   1. _S_[_pos_] ^= _P_[_k_]
   1. _C_[_k_] ^= _S_[_pos_]
   1. _k_ += 1
   1. _pos_ += 1
   1. If _pos_ == 16:
      1. If _k_ == _l_, then _S_[47] ^= 0x01
      1. _S_ = _Gimli_(_S_)
      1. _pos_ = 0
1. Return _C_

_Gimli-Decrypt_ is a function which takes as inputs a sponge state _S_ and a ciphertext _C_, updates the sponge state _S_ and returns the plaintext _P_. The sponge state _S_ is a 48-bytes-long string. The length of _C_ MUST be a multiple of 16. _Gimli-Decrypt_ is defined by the following algorithm:

1. Let _l_ be the length of _C_
1. Let _P_ be a byte-string of length _l_.
1. Let _k_ be the integer 0.
1. Let _pos_ be the integer 0.
1. While _k_ < _l_:
   1. _P_[_k_] = _S_[_pos_] ^ _C_[_k_]
   1. _S_[_pos_] = _C_[_k_]
   1. _k_ += 1
   1. _pos_ += 1
   1. If _pos_ == 16:
      1. If _k_ == _l_, then _S_[47] ^= 0x01
      1. _S_ = _Gimli_(_S_)
      1. _pos_ = 0
1. Return _P_

_Gimli-Finalize_ is a function which takes as input a sponge state _S_ and returns a tag _T_. The sponge state _S_ is a 48-bytes-long string and the tag _T_ is a 32-bytes-long string. _Gimli-Finalize_ is defined by the following algorithm:

1. Let _T_ be a byte-string of length 32.
1. Let _pos_ be the integer 0.
1. While _pos_ < 16:
   1. _T_[_pos_] = _S_[_pos_]
   1. _pos_ += 1
1. _S_ = _Gimli_(_S_)
1. _pos_ = 0
1. While _pos_ < 16:
   1. _T_[16 + _pos_] = _S_[_pos_]
   1. _pos_ += 1
1. Return _T_

_Pad_ is a padding function which takes as input a byte-string _In_ and returns a byte-string _Out_. _Pad_ is defined by the following algorithm:

1. Let _len_ be the length of _In_
1. Let _Out_ be a byte-string of length (_len_/16 + 1) \* 16.
1. Let _k_ be the integer 0.
1. While _k_ < _len_:
   1. _Out_[_k_] = _In_[_k_]
   1. _k_ += 1
1. _Out_[_k_] = 0x01
1. _k_ += 1
1. While _k_ < (_len_/16 + 1) \* 16:
   1. _Out_[_k_] = 0x00
   1. _k_ += 1
1. Return _Out_

## Shoots

A shoot is a tuple (_K_, _Path_, _P_, _N_) where:

- _K_ is a 32-bytes-long randomly generated key.
- _Path_ is a byte-string.
- _P_ is a sequence of _p_+1 byte-strings, such that for all _k_ between 0 and _p_:
  - The length of _P_[_k_] is a multiple of 16, and lesser or equal to 2400 bytes.
  - If _k_ < _p_, then the length of _P_[_k_] is equal to 2400 bytes.
  - _P_[_k_] == _content_ || _pad_ || _A_ || _B_.
  - _pad_ is a sequence of null bytes of length _len_.
  - _A_ is a byte equal to _len_ % 256.
  - If 0 < _k_ < _p_, then _B_ is a byte equal to _len_ / 256.
  - If 0 == _k_ < _p_, then _B_ is a byte equal to _len_ / 256 + 128.
  - If 0 < _k_ == _p_, then _B_ is a byte equal to _len_ / 256 + 64.
  - If 0 == _k_ == _p_, then _B_ is a byte equal to _len_ / 256 + 128 + 64.
- _N_ is a 32-bytes-long randomly generated nonce.

A shoot has associated attributes:

A 48-bytes-long sponge state _SPath_ which is computed by the following algorithm:

1. Let _SPath_ be the null sponge state.
1. _Gimli-Absorb_(_SPath_, _Pad_(0x73 || 0x61 || 0x70))
1. _Gimli-Absorb_(_SPath_, _K_)
1. _Gimli-Absorb_(_SPath_, _Pad_(_Path_))
1. Return _SPath_

A 32-bytes-long path tag _TPath_ which is equal to _Gimli-Finalize_(_SPath_).

A 32-bytes-long tag _T_[_p_+1] which is equal to _N_.

A sequence of ciphertexts _C_ and a sequence of tags _T_, such that for all _k_ between _p_ and _0_, _C_[_k_] and _T_[_k_] are computed by the following algorithm:

1. Let _S_ be a copy of _SPath_.
1. _Gimli-Absorb_(_S_, _T_[_k_+1])
1. _C_[_k_] = _Gimli-Encrypt_(_S_, _P_[_k_])
1. _T_[_k_] = _Gimli-Finalize_(_S_)
1. Return _C_[_k_], _T_[_k_]

A leaf node _Leaf_ which is the concatenation _C_[_0_] || _T_[_1_] || _C_[_1_] || _T_[_2_] || _C_[_2_] || _T_[_3_] || ... || _C_[_p_] || _T_[_p_+1]

A 32-bytes-long leaf tag _TLeaf_ which is computed by the following algorithm:

1. Let _S_ be the null sponge state.
1. _Gimli-Absorb_(_S_, _Pad_(0x73 || 0x61 || 0x70))
1. Let _k_ be the integer 0.
1. While _k_ < _p_ + 1:
   1. Let _STemp_ be the null sponge state.
   1. _Gimli-Absorb_(_STemp_, _Pad_(0x73 || 0x61 || 0x70))
   1. _Gimli-Absorb_(_STemp_, _T_[_k_+1])
   1. _Gimli-Absorb_(_STemp_, _C_[_k_])
   1. _Gimli-Absorb_(_S_, _Gimli-Finalize_(_STemp_))
   1. k += 1
1. Return _Gimli-Finalize_(_S_)

A stem node _Stem_ which is the concatenation _TLeaf_ || _T_[0].

A shoot node _Shoot_ which is the concatenation _Stem_ || _Leaf_.

A 32-bytes-long shoot tag _TShoot_ which is computed by the following algorithm:

1. Let S be the null sponge state.
1. _Gimli-Absorb_(_S_, _Pad_(0x73 || 0x61 || 0x70))
1. _Gimli-Absorb_(_S_, _Stem_)
1. _Gimli-Absorb_(_S_, _TPath_)
1. Return _Gimli-Finalize_(_S_)

## Shrubs

A shrub is a binary Merkle tree composed of internal nodes and shoot nodes.

An internal node _Node_ is the concatenation _T0_ || _T1_, where _T0_ (resp. _T1_) is the tag of another internal node, or the tag of a shoot node.

The tag of an internal node _Node_ is computed by the following algorithm:

1. Let S be the null sponge state.
1. _Gimli-Absorb_(_S_, _Pad_(0x73 || 0x61 || 0x70))
1. _Gimli-Absorb_(_S_, _Node_)
1. Return _Gimli-Finalize_(_S_)

## The application/branch Media Type

A file which media type is application/branch represents a branch of a shrub. The branch (and the file which represents it) has three attributes: a tag _TRoot_, a key _K_ and a path _Path_. _TRoot_ is the tag of the root of the shrub containing the branch. _K_ and _Path_ are the key and path of the shoot node at the end of the branch.

The file which represents a branch is the concatenation of the ancestry and the shoot.\
The shoot is the shoot node which is at the end of the branch represented by the file.\
The ancestry is the concatenation of all the internal nodes which are ancestors to the shoot node in depth-first-search pre-order (starting from the root of the shrub containing the shoot).

## ShootStreaming Algorithm

_ShootStreaming_ is a generator which takes four inputs, generates byte-strings, and returns one output. The four inputs are the tag _TRoot_ of the root of a shrub, the key _K_ and the path _Path_ of a shoot of the shrub, and a readable byte-stream _Str_ of the application/branch file which represents the branch of the shrub containing the shoot. _ShootStreaming_ generates the parts (_content_[0], ..., _content_[_p_]) of the plaintexts of the shoot. If all the parts have been generated, then _ShootStreaming_ returns the _EOF_ error, else it returns _STREAM_ERROR_.

The algorithm of _ShootStreaming_ is the following:

1. Let _Ssap_ be a null sponge state.
1. _Gimli-Absorb_(_Ssap_, _Pad_(0x73 || 0x61 || 0x70))
1. Let _SPath_ be a copy of _Ssap_.
1. _Gimli-Absorb_(_SPath_, _K_)
1. _Gimli-Absorb_(_SPath_, _Pad_(_Path_))
1. Let _Parent_ and _Child_ be two 64-bytes-long string of null bytes.
1. _Parent_[0:32] = _TRoot_
1. Let _S_ be a null sponge state.
1. While _true_, run the following steps:
   1. Let _bytes_ be the result of reading 64 bytes from _Str_.
   1. If the length of _bytes_ is not 64, then: Return _STREAM_ERROR_
   1. _Child_ = _bytes_
   1. Let _STemp_ be a copy of _Ssap_.
   1. _Gimli-Absorb_(_STemp_, _Child_)
   1. Let _S_ be a copy of _STemp_
   1. Let _T_ be a 32-bytes-long string of null bytes.
   1. _T_ = _Gimli-Finalize_(_STemp_)
   1. If (_T_ != _Parent_[0:32]) and (_T_ != _Parent_[32:64]), then: _Break_
   1. _Parent_ = _Child_
1. Let _STemp_ be a copy of _SPath_.
1. _Gimli-Absorb_(_S_, _Gimli-Finalize_(_STemp_))
1. Let _T_ be a 32-bytes-long string of null bytes.
1. _T_ = _Gimli-Finalize_(_S_)
1. If (_T_ != _Parent_[0:32]) and (_T_ != _Parent_[32:64]), then: Return _STREAM_ERROR_
1. _T_ = _Child_[32:64]
1. Let _isFirstChunck_ be a boolean equal to _true_.
1. While _true_, run the following steps:
   1. Let _bytes_ be the result of reading 2432 bytes from _Str_.
   1. Let _len_ be the length of _bytes_.
   1. If _len_ is strictly lesser than 48 or is not a multiple of 16, then: Return _STREAM_ERROR_
   1. _Nonce_ = _bytes_[_len_ - 32:_len_]
   1. _C_ = _bytes_[0:_len_ - 32]
   1. Let _S_ be a copy of _SPath_
   1. _Gimli-Absorb_(_S_, _Nonce_)
   1. _P_ = _Gimli-Decrypt_(_S_, _C_)
   1. If (_T_ != _Gimli-Finalize_(_S_)): Return _STREAM_ERROR_
   1. If (_len_ != 2432) and (_P_[_len_ - 33] & 64 == 0), then: Return _STREAM_ERROR_
   1. If _isFirstChunck_ and (_P_[_len_ - 33] & 128 == 0), then: Return _STREAM_ERROR_
   1. _isFirstChunck_ = _false_
   1. _l_ = _P_[_len_ - 34] + (_P_[_len_ - 33] & 63) \* 256
   1. If _len_ - 32 < _l_ + 2, then: Return _STREAM_ERROR_
   1. Yield P[0:_len_ - 32 - _l_ - 2]
   1. If (_P_[_len_ - 33] & 64) != 0, then: _Break_
   1. _T_ = _Nonce_
1. Return _EOF_

## The .well-known URIs suffix "shrubbery"

We define a mapping between branches of shrubs and HTTPS URLs that makes use of the .well-known URIs by defining a "shrubbery" suffix. These .well-known URIs are usefull for protocols and applications (such as [the Shrub https subscheme][shrub]) which need [to retrieve branches with low latency][shrub-fetch].

For a branch with the attributes (_TRoot_, _K_, _Path_), and a hostname _HOST_, the corresponding HTTPS URL produced by this mapping is "https://<_HOST_>/.well-known/shrubbery/<_shrubID_>/<_branchID_>" where _shrubID_ is equal to _base32_(_TRoot_) and _branchID_ is equal to _base32_(_TPath_), with _TPath_ the path tag of the shoot at the end of the branch (computed according to the algorithm in [Definition of Shoots][shoots]).

base32 is the Base32 encoding function defined in [RFC 4648 section 6](https://tools.ietf.org/html/rfc4648#section-6) with alphabet "0123456789abcdefghijklmnopqrstuv" (0-9 a-v) and without padding.

## IANA Considerations

### Assignment of the application/branch Media Type

Type name: application

Subtype name: branch

Required parameters: N/A

Optional parameters: N/A

Encoding considerations: "binary"

Security considerations: see [Security Considerations][security]

Interoperability considerations: N/A

Published specification: This specification (see [The application/branch Media Type][media]).

Applications that use this media type: [Shrub https subscheme][shrub].

Fragment identifier considerations: N/A

Additional information:

- Deprecated alias names for this type: N/A
- Magic number(s): N/A
- File extension(s): N/A
- Macintosh file type code(s): N/A

Intended usage: COMMON

Restrictions on usage: N/A

### Assignment of the .well-known URIs suffix "shrubbery"

## Security Considerations

TODO

## Privacy Considerations

TODO

[shrub]: shrub.md "Shrub https subscheme"
[shrub-fetch]: shrub.md#modifications-of-the-fetch-standard "fetch protocol for Shrub https subscheme"
[shoots]: #shoots "Definition of Shoots"
[security]: #security-considerations "Security Considerations"
[media]: #the-applicationbranch-media-type "The application/branch Media Type"
