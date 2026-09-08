# XecVibe

On-chain protocol for [xecvibe.com](https://xecvibe.com/) payments opened in
Cashtab via BIP21 `op_return_raw`.

## LOKAD ID

`XECV` = `0x58454356` (4 bytes, ASCII).

## Output layout

Every XecVibe payment is a single OP_RETURN output:

```
OP_RETURN                 (0x6a)
<push 4> 58454356         LOKAD prefix "XECV"
<pushdata>                utf8 one-time memo (1–75 bytes)
```

The LOKAD alone identifies the app; Cashtab labels the protocol **XecVibe**.
There is one wire shape for all v0 XecVibe Cashtab payments (auth, tips,
live sprays, NFT or marketplace checkout, and other app payments later). The
memo is a server-issued single-use id; XecVibe looks it up off-chain to decide
purpose and settle the payment.

Rules:

- Exactly one pushdata after the LOKAD. Extra pushes are invalid.
- Memo is UTF-8, length 1–75 bytes (fits a standard OP_RETURN push without
  `OP_PUSHDATA1`). Empty memo is invalid.
- Typical memo is a server-issued opaque string (often 32 ASCII hex chars).

## Examples

Valid payment memo `4f200d3504f54a13a4dc856d50f43085`:

```
6a 04 58454356 20 3466323030643335303466353461313361346463383536643530663433303835
```

`op_return_raw` (without leading `6a`):

```
0458454356203466323030643335303466353461313361346463383536643530663433303835
```

Invalid examples:

| Reason | Example `op_return_raw` (no `6a`) |
| ------ | -------------------------------- |
| Missing memo | `0458454356` |
| Extra push after memo | `04584543562034663230306433353034663534613133613464633835366435306634333038350b786563766962652e636f6d` |

## BIP21

Producers open Cashtab with BIP21 `op_return_raw` set to the payload above
(without the leading `6a`). Payment outputs (amounts / addresses) are separate
BIP21 fields; OP_RETURN carries only the XECV marker and memo.
