+++
title = "Enigma2017 CTF Broken Encryption Writeup"
Categories = ["cryptography", "CTF", "Python"]
Tags = ["cryptography", "CTF", "Python", "Enigma2017", "cryptanalysis"]
Description = "Block Cipher Cryptanalysis 101"
date = "2017-02-12T13:40:38-05:00"
+++

There is a new "Jeopardy style" security CTF web framework (CTF-as-a-Service?) called [HackCenter](http://hackcenter.com/) that just debuted from For All Secure, the CMU-affiliated security startup known for [winning last year's DARPA Cyber Grand Challenge Final Event](https://www.defense.gov/News/Article/Article/906931/three-teams-earn-prizes-in-darpa-cyber-grand-challenge) with their game-playing "automated exploit generation" system they called Mayhem CRS. HackCenter is their "other" technology, I guess, and right now the only CTF they've hosted is/was the one that occurred at [Enigma2017 USENIX](https://www.usenix.org/conference/enigma2017) conference at the end of January. It seemed to be marketed as educational: "learn to hack!" and not as unfriendly and elitist as some of the more competitive CTFs, so I gave it a look. Also, this was a chance to refresh myself on some Python.

## The Challenge

They give us a telnet server that prompts us to send whatever string we want, and then it sends back an encrypted version of that string. Also they give us this source code for the server:

```python
#!/usr/bin/python -u
from Crypto.Cipher import AES

flag = open("flag", "r").read().strip()
key = open('enc_key', 'r').read().strip().decode('hex')

welcome = """
************ MI6 Secure Encryption Service ************
                  [We're super secure]
       ________   ________    _________  ____________;_
      - ______ \ - ______ \ / _____   //.  .  ._______/ 
     / /     / // /     / //_/     / // ___   /
    / /     / // /     / /       .-'//_/|_/,-'
   / /     / // /     / /     .-'.-'
  / /     / // /     / /     / /
 / /     / // /     / /     / /
/ /_____/ // /_____/ /     / /
\________- \________-     /_/

"""

def pad(m):
  m = m + '1'
  while len(m) % 16 != 0:
    m = m + '0'
  return m

def encrypt():
  cipher = AES.new(key,AES.MODE_ECB)

  m = raw_input("Agent number: ")
  m = "agent " + m + " wants to see " + flag

  return cipher.encrypt(pad(m)).encode("hex")

print welcome
print encrypt()
```

We also get a web shell on hackcenter.com: literally an in-browser terminal emulator connected to the remote server (we do not have read access to the directory with "flag"), but for this problem we will just open our local Terminal app and poke around.

## Anything ECB is Bad Mmmkay

Look at the source: basically, `"agent " + yourinput + " wants to see " + flag` is padded out to the next nearest AES block length (128 bits == 16 bytes) and then encrypted with AES-ECB using whatever the key is. Now, basically the first thing you learn about block ciphers is to never use the Electronic Code Book (ECB) mode. You'll see a photo of Tux the Linux mascot encrypted with AES-ECB and how you can still see the edges of the image in the encrypted version. But that's about it. It's rare to see an explanation of why this is relevant or how to break it. Just, "everyone knows it's bad."

The reason why ECB mode of any block cipher is bad is that the same input always encrypts to the same output. The input is broken into fixed-length blocks and encrypted, and all of the blocks of identical input will create similarly equal output blocks. The data is all encrypted, but we know where their plaintexts were the same. There is *no key recovery attack against this issue*, at least not that I am aware of, but the problem is that the plaintext can be guessed. There are two basic attacks against ECB:

1. Given enough encrypted blocks and some partial knowledge of the plaintext (known offsets of fixed data, like as defined by filetype formats or communication protocols), statistical and frequency analysis (and some guessing, then confirming) can reveal partial plaintext.
2. Given the ability to prefix or circumfix (that means insert in the middle somewhere) arbitrary plaintext, and then have it encrypted and view the resulting ciphertext, an attacker can stage what cryptographers call a Chosen Plaintext Attack (CPA). The scenario of passing arbitrary plaintext to a remote encryptor and receiving the ciphertext back is also called an [Oracle](https://en.wikipedia.org/wiki/Oracle_machine#Applications_to_cryptography). This is the attack we will discuss in this post.

The reason why this is *relevant* is that to the average programmer who can't be bothered, ECB looks like a valid mode choice for AES, a cipher that people generally recommend: "military grade crypto," right? They might use it to encrypt the cookie their web site stores in your browser. Or if they're especially ignorant in security like the people who work at Adobe, [they might use it to encrypt their users' passwords on the server](https://arstechnica.com/security/2013/11/how-an-epic-blunder-by-adobe-could-strengthen-hand-of-password-crackers/).

## Breaking ECB with the Chosen Plaintext Attack

Being able to circumfix our arbitrary input into the plaintext (at a known location in that string) means that we can choose an input such that we can fully align *our* *known* *substring* on an AES block boundary. Thus allowing us to test what the ciphertext is for any arbitrary block that we choose.

    "agent " + yourinput + " wants to see " + flag + padding
    (6 chars)  (n chars)    (14 chars)   <—- if you want to test-encrypt a single block of arbitrary input, put your test input on a 16-byte block boundary, like so: yourinput = "01234567891000000000000000". "1000000000000000" is at bytes 16 through 31 of the input, aka the second AES (128-bit, 16-byte) block.

We don't know how long the flag is, but we know how the padding is applied: if the plaintext message does not end on a 16-byte boundary, then it is extended by a single "1" and up to 14 "0" characters. If the plaintext message *does* end on a 16-byte boundary, then it is extended by a full block of padding: `1000000000000000`. This may seem counter-intuitive, but there always has to be padding in a block cipher, even when the message length already is a multiple of the block length: otherwise how would you know if the last block is padding or if `1000000000000000` was part of the message?

See where we're going with this? We will give the above plaintext, and observe the output's 2nd block. That is the exact same output we would expect to see as the last block of ciphertext if the flag ends at a block boundary and the final block were AES padding.

```
Agent number: 01234567891000000000000000
ceaa6fa24a71971f21413c1ea39f4e7c53b1c1d36d11a2c20dfc3913bb299f11c9777890922460e74fefb1a94f5c95df0ebb6d7bc5a7922f0857283feb2b068dc5148be36b7670e2ca4fe52c3f65c37612b88acbe4bbd5a9f2588bbc4e0ea92453b1c1d36d11a2c20dfc3913bb299f11
```

Note the second block (32 hex characters = 16 bytes) of ciphertext is `53b1c1d36d11a2c20dfc3913bb299f11c` and, through a stroke of luck, we've already aligned the overall message on a block boundary too, as we see `53b1c1d36d11a2c20dfc3913bb299f11c` is also the last block of ciphertext!

The game now is to insert one *additional* byte of arbitray text in order to push a single byte of the "flag" portion of the string rightward into the padding block. The final padding block will be `n100000000000000` where `n` is the unknown byte of flag.

What will we do then to guess that byte? We'll brute-force it: send new plaintext messages for all 255 possibilities of `n` in our block-aligned arbitrary input (which is the 2nd block). When the ciphertext's 2nd block matches the ciphertext's 7th block, then we know we guessed correctly. Then we'll insert one additional byte again at the same location, and repeat this process. In other words, we expect to send a series of messages like the following:

```
0123456789a100000000000000
0123456789b100000000000000
0123456789c100000000000000
0123456789d100000000000000
0123456789e100000000000000 ... let's say that ciphertext blocks 2 and 7 match at this point!
0123456789ae10000000000000
0123456789be10000000000000
0123456789ce10000000000000
0123456789de10000000000000
0123456789ee10000000000000
0123456789fe10000000000000 ... they match again. We so far know last block = fe10000000000000
0123456789afe1000000000000
0123456789bfe1000000000000
and so on, and so on... up to 255 guesses per byte and as many bytes as we need to discover
```

In practical terms, we can try guessing only in the ASCII range of 0x20-0x7E or so, since we expect the secret in this case to be plaintext (the "flag"). This will speed things up by more than double.

## Putting it All Togther: A Solution in Python

Knowing what to do is half the battle. The other half is coding it up and tearing your hair out over data alignment issues and dynamic typing issues.

```python
#!/usr/bin/python

# Enigma2017 CTF, "Broken Encryption"

import sys
import time       # for using a delay in network connections
import telnetlib  # don't try using raw sockets, you'll tear your hair out trying to send the right line feed character

__author__ = 'michael-myers'

# TODO: I'm interested in any more elegant way to block-slice a Python string like this.
# Split out every 16-byte (32-hex char) block of returned ciphertext:
def parse_challenge(challenge):
    ciphertext_blocks = [challenge[0:32], challenge[32:64], challenge[64:96],
                         challenge[96:128], challenge[128:160], challenge[160:192],
                         challenge[192:224], challenge[224:]]
    return ciphertext_blocks


# To attack AES-ECB, we will be exploiting the following facts:
#   * we do not know all of the plaintext but we control a substring of it.
#	* the controlled portion is at a known offset within the string.
#   * by varying our input length we can force the secret part onto a block boundary.
#   * we can choose our substring to be a full block of padding & align it at a boundary.
#   * if the message ends at a block boundary, the last 16-byte block will be all padding.
#   * thus we know when the secret part is block aligned; we'll see the same ciphertext.
#   * there is no nonce or IV or counter, so ciphertext is deterministic.
#   * by varying length of plaintext we can align the secret part such that there 
#		is only one unknown byte at a time being encrypted in the final block of output. 
#	* by varying one byte at a time, we can brute-force guess input blocks until we
#       match what we see in the final block, thus giving us one byte of the secret.
#   * we will limit our guesses to the ASCII range 0x20-0x7E for this particular challenge.
#
# Begin by changing the 2nd block of plaintext to n100000000000000, where n is a guess. 
# If the ciphertext[2nd block] == ciphertext[7th block] then the guess is correct,
# otherwise increment n.
def main():
    # If the Engima2017 servers are still up: enigma2017.hackcenter.com 7945
    if len(sys.argv) < 3:   # lol Python doesn't have an argc
        print 'Usage : python CTF-Challenge-Response.py hostname port'
        sys.exit()
    host = sys.argv[1]
    port = int(sys.argv[2])
    
    guessed_secret = ""

    # Our input pads to the end of the 1st block, then aligns a guess at block 2.
    # Because we need to constantly alter this value, we are making it a bytearray. 
    # Strings in Python are immutable and inappropriate to use for holding data.
    chosen_plaintext = bytearray("0123456789" + "1000000000000000")

    # Guess each byte of the secret, in succession, by manipulating the 2nd plaintext
    # block (bytes 10 through 26) and looking for a matched ciphertext in the final block:
    for secret_bytes_to_guess in range(0, 64):
        # Add in a new guessing byte at the appropriate position:
        chosen_plaintext.insert(10, "?")

        # Guess over and over different values until we get this byte:
        for guessed_byte in range(0x20, 0x7E):  # this is the printable ASCII range.
            chosen_plaintext[10] = chr(guessed_byte)

            tn = telnetlib.Telnet("enigma2017.hackcenter.com", 7945)
            tn.read_until("Agent number: ")

            # Telnet input MUST BE DELIVERED with a \r\n line ending. If you send
            # only the \n the remote end will silently error on your input and send back
            # partially incorrect ciphertext! Untold hours debugging that bullshit.
            # Here we carefully convert the bytearray to ASCII and then to a string type, 
            # or else telnetlib barfs because of the hell that is dynamic typing.
            send_string = str(chosen_plaintext.decode('ascii') + "\r\n")
            tn.write(send_string)

            challenge = tn.read_all()
            tn.close()
            # time.sleep(0.5)   # (optional) rate-limit if you're worried about getting banned.

            ciphertext_blocks = parse_challenge(challenge)
            print "Currently guessing: " + chosen_plaintext[10:26]  # 2nd block holds the guess
            print "Chosen vs. final ciphertext blocks: " + ciphertext_blocks[1] + " <- ? -> " + ciphertext_blocks[6]

            # We're always guessing in the 2nd block and comparing result vs the 7th block:
            if ciphertext_blocks[1] == ciphertext_blocks[6]:
                print "Guessed a byte of the secret: " + chr(guessed_byte)
                guessed_secret = chr(guessed_byte) + guessed_secret
                break   # Finish the inner loop immediately, back up to the outer loop.

    print "All guessed bytes: " + guessed_secret

    print("Done")


if __name__ == "__main__":
    main()
```

And, after all of this, we uncover the flag: `54368eae12f64b2451cc234b0f327c7e_ECB_is_the_w0rst`