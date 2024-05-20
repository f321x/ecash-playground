
## Chaumian cash in a Bitcoin world - cashu, Fedimint

### What's this for?

* More scalable/faster than a blockchain (not enough utxos)
* Much better privacy security model than a blockchain
* Same or better theft security model than TTP but much worse than a blockchain

If it's so great, why hasn't it been done yet?

### History

#### Original Chaum papers

1982: ecash using RSA blind signatures and a trusted mint. Every transfer had to be mediated via the bank/mint (yucky?)

https://chaum.com/wp-content/uploads/2022/01/Chaum-blind-signatures.pdf

1990: as above, but with a *blame* protocol: using fancy (and not performant) cut and choose, you can have Alice send Carol a payment *without* it being sent via the bank, and if Alice double spends afterwards, it can be shown that she did it. NOTICE! A blame protocol *only* makes any sense with accounts. (yucky?)

http://blog.koehntopp.de/uploads/chaum_fiat_naor_ecash.pdf

## But how, basically, does it work?

![ecash](https://user-images.githubusercontent.com/4278257/192500280-b04a6ebd-68dc-4871-9886-1cdb341180b3.png)

### Coins and denomination

* Blind signing means the signer doesn't know the message. Are the coins literally all the same?
* Denomination defined by signing pubkeys
* Globally known or defined pubkeys are important for privacy
* Coins are fixed denomination, not variable as utxos
* Messages are sometimes random, but are sometimes used for other things (caveat: privacy if revealed?)
* A coin is 'spent' when redeemed at the mint; it is then added to a 'spent' list so it can't be spent again.
* Never forget - bearer instrument. Not a 'private key', just having the coin itself is enough.

### "Online" payment workflow

1. Alice sends blinded coin to Carol.
2. Carol sends blinded coin to bank to verify and reblind.
3. If 2. successful Carol treats coin as received.
4. Assuming consistency in bank (i.e. one timeline), Carol is safe and Alice can't double spend.
5. Alice is supposedly safe w.r.t. privacy due to blinding.

### Splits

Denominations are fixed. There is nothing stopping anyone from sending bank N coins of denominations n_i and receiving M coins of denominations m_i as long as sum(n_i) == sum(m_i). Helps with privacy, generally. Scalability must be considered.

(Let's not get sidetracked, on with the history!)

1993: Stefan Brands starts to make more elegant designs doing something similar:

http://courses.csail.mit.edu/6.857/2009/handouts/untraceable.pdf

(Brands was Chaum's student at some point)

Important quote:

> More seriously, no realizations of untraceable off-line cash systems have been proposed yet that can offer prior-restraint of double-spending, whereas this property can be trivially attained in fully traceable off-line systems.

His paper purports to solve this problem; but see caveats about blind signature security below.

He uses exactly the same trick as in Dryja's DLC to achieve the punish-if-double-spend outcome that Chaum originally achieved with cut-and-choose. In fact what he creates in "restrictive blind signatures" is more sophisticated but at core it's the same idea: instead of (pubkey: P) and (signature: R,s) you have (pubkey: P, R) and (signature: s), which means that only one signature s can be used; if you make a new one, you can do so, but it will reveal your private key. (cleverer, but still yucky?).

Notice that this trick (and any other "blame" based trick to counter double-spending) only even makes sense **if the user has an account with a private key**.

He then claims: if we have tamper resistant hardware, we can have prior-restraint of double spending (yucky?)

The idea of hardware being part of the e-cash solution was extremely common back then in the 90s. And it persisted: see Hal Finney's RPOW project from the early '00s:

https://nakamotoinstitute.org/finney/rpow/index.html

### Real?

What about real world instantiations of any of these ideas?

Digicash: https://chaum.com/ecash/ has a nice pictorial timeline

For Brands, he later created a project 'Credentica' which was more focused on another aspect of the same thing: allowing people to have credential tokens that revealed something but not their whole identity. See credentica.com and, as noted on that website, the company and the IP it owned was bought by Microsoft in 2003 and converted into their product 'uProve': https://www.microsoft.com/en-us/research/project/u-prove/ (there is almost nothing out there any more about uProve .. i think it was pretty much a failure? Could be wrong!)

So, digicash was a big failure, and so was credentica, as far as getting adopted to any extent, goes, I *believe*. This should not be ignored!

This old piece discussed digicash's collapse/failure in detail: https://cryptome.org/jya/digicrash.htm - highly recommended read.

Coverage of that history by Aaron van Wiirdum: https://bitcoinmagazine.com/culture/genesis-files-how-david-chaums-ecash-spawned-cypherpunk-dream

There are also other, weirder little details of history now mostly lost ... J Orlin Grabbe's writings on the topic of digital cash are a little hard to find nowadays, search for "The End of Ordinary Money" by that author.

Then there is this story I dug up after being told about it by an old bitcoin hand in El Salvador: https://patrifriedman.com/old_writing/lfc_digital_monetary_trust_critique.html (and the money project "DMT" which from my reading seems a slightly inferior implementation of these same ideas).

Other real-world *attempts* at instantiation that I haven't gone into include: Bill St Clair's Truledger (the site is now offline) from early 2000s, Opentransactions ( https://opentransactions.org ) which was post-Bitcoin (and by mostly bitcoiners), and implements a generalized financial cryptography library of which chaumian tokens is only one part (I believe they used the Lucre design from quickly scanning the current code).


### Compact e-cash

A lesser known design is that of and Camenisch and Lysyanskaya from 2006:

https://eprint.iacr.org/2005/060.pdf

Have not studied but it requires: bilinear pairings and strong RSA assumption, creates coins with after-the-fact proof of double spend as discussed above (still yucky?).

> In fact, compared to previous e-cash schemes, our whole wallet of 2^l coins has about the same size as one coin in these schemes. Our scheme also offers exculpability of users, that is, the bank can prove to third parties that a user has double-spent.


Here Matt Green does a great job of summarizing the history of ecash designs up to around 2012, ending with Bitcoin and its privacy limitations:

https://blog.cryptographyengineering.com/category/ecash/

### The blind signature security problem.

Important to understand that blind signatures can be constructed from pretty much all the known basic signature protocols: RSA, Schnorr and also new BLS.

Nadav Kohen has a decent explanation of the basic blind signature scheme in Schnorr here:

https://suredbits.com/schnorr-applications-blind-signatures/

(though Matt Green's is nice too, follow the link above)

If you want to understand Wagner's attack and how it affects collaborative signing protocols like MuSig, you can read my two blog posts:

https://reyify.com/blog/avoiding-wagnerian-tragedies

https://reyify.com/blog/the-soundness-of-musig

(I discuss Wagner's original paper https://www.iacr.org/archive/crypto2002/24420288/24420288.pdf in detail in the former)

This same kind of attack applies to a collaborative blind signing protocol, if the signing server is executing the protocol with many clients at once (in parallel). This is noted in BIP340:

https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki#blind-signatures

Wagner's attack is even substantially improved on by this very recent (2020) paper: https://eprint.iacr.org/2020/945.pdf . The TLDR is you can plausibly extract keys from only 10s of parallel sessions (or maybe 100s, depending).

## Cashu

Code: https://github.com/callebtc/cashu

Outline: https://gist.github.com/callebtc/557e4cc15f9e43d7474c7cb3d31ee8ed

Using a slightly different setup than old blind signature based systems; this one uses blinded Diffie Hellman key exchange.

David Wagner himself, *not* coincidentally, was the one who proposed what we are now discussing as 'Cashu':

http://cypherpunks.venona.com/date/1996/03/msg01848.html

This was unearthed by Ruben Somsen in this gist posting: https://gist.github.com/RubenSomsen/be7a4760dd4596d06963d67baf140406

.. who also pointed out that in 2003 "Lucre" had a similar concept implemented:

https://github.com/benlaurie/lucre/blob/master/html/theory2.pdf

.. though here using old-style DL instead of ECDL. I note here that Ben Laurie seemed not to be aware of the DLEQ primitive, which solves the attack he discusses.

Cashu, at least as currently implemented, reverts back to the "online" not "offline" transfer, i.e. the user can only send coins to a receiver by having that receiver contact the mint to redeem the coin. DLEQ should really be used to get a slightly better guarantee of privacy for the user, as is discussed below re: privacypass.

### Related: privacypass

See excellent explanation here: https://privacypass.github.io/protocol/

Main paper on this system by Ian Goldberg here: https://www.cypherpunks.ca/~iang/pubs/privacypass-popets18.pdf

Privacypass was implemented as a browser extension in things like Firefox and Tor browser. The idea is to replace inconvenient repeated CAPTCHA solvings that are used to counter DDOS attacks or bot spam, with a smaller number of similar "proofs of humanity" that are used to mint tokens that can then be used over a longer period. Hence there is no real intention to create a *transferrable* token from one user to another, representing value (i.e. it's not a 'coin'). But otherwise the construction is very similar. Main differences:

* Multiplicative tweak, not additive tweak
* HMAC of token transferred to avoid MITM
* Batch DLEQ proofs to avoid user-tagging by server

privacypass and Cashu are based on the CDH assumption. Where Schnorr blind sigs are based on the DL assumption. However with these 'blind' systems we cannot talk about standard EUF-CMA security; the concept no longer makes sense (the server doesn't prevent someone "forging" a signature on a message it didn't choose to sign; instead, that's the explicit purpose of the system!). So instead we usually talk about "one-more-forgery" style claims.

If forging one more token, given N tokens have actually been signed by the server, is computationally hard, then we can say that the system is resistant to cheating via inflation *by the users* (but NOT by the server: the server can make as many tokens as it wants!).

So, for both privacypass and Cashu, there is a reduction to one-more-ElGamal decryption forgery, meaning that if the latter is secure, then so are these systems. For privacypass, that is in the paper above, for Cashu, it's a claim I'm making (and, weakly), see: https://gist.github.com/RubenSomsen/be7a4760dd4596d06963d67baf140406?permalink_comment_id=4312092#gistcomment-4312092 .

Argument: just switch the Cashu design to the privacypass design?

### Using Cashu

I've tried it, it's pretty easy. Like:

![cashuCLI](https://user-images.githubusercontent.com/4278257/192499733-19802b2b-cf69-4b5a-9d73-fa0d76173fd6.png)

Code is still bare bones. Mint is just a database, entirely controlled by one entity running a server. Lose the database, kaput. Privacy guarantees are kind of not there yet, imo (from general overview). But still in rapid development.

### Lightning

What makes Cashu Cashu and not just an implementation of a blinded DH token is that the mint can accept Lightning payments to mint tokens and can pay out/redeem in the reverse way. The tooling for this is pretty well established. However there's still a lot you could do to make this more sophisticated.

## Fedimint

A lot of documentation at a level a non-cryptography-geek can follow, here: https://fedimint.org/docs/

Pretty good in-depth video explaining: https://www.youtube.com/watch?v=G4iclApJL0c&t=8s

Example of using:

![photo_2022-09-26_14-56-07](https://user-images.githubusercontent.com/4278257/192501606-b176e102-5a93-429a-8bac-8c74d1c403f7.jpg)

Current operating instance on signet that you can use: https://faucet.sirion.io/

### Threshold blind signatures.

Fedimint uses BLS: https://www.iacr.org/archive/asiacrypt2001/22480516.pdf

These signatures have the following properties: they're based on bilinear pairings, so they use special elliptic curves; they're short (as per the paper), although in practice a similar size to Schnorr/ECDSA; they're linear in the key, like Schnorr; theyre deterministic, unlike Schnorr.

But most striking feature: they're non-interactive aggregatable: a product of signatures is verifiable as such (think about it!).

This is what makes them very attractive for several applications, including Fedimint's (the signature shares can just be multiplied, see below).

Negative: to verify, you actually have to calculate the pairing. This is expensive, some data I saw suggested BLS-verify is about 6 times slower than ECDSA-verify. Both Schnorr and BLS support batch verification (sec 5.2 of paper).

Boldyreva came up with a system to threshold sign in this way in this 2003 paper (although again, threshold is already in the BLS paper), which Fedimint cites: https://faculty.cc.gatech.edu/~aboldyre/papers/b.pdf

It sounds cool to spread trust by having federation threshold-blind-sign, but it's trickier than you might think. Here's the basic attack showing why doing this naively fails:

* Federation with 2 of 3 policy, mints coins by blind signing messages.
* User asks A to sign m1, B to sign m1, and C to sign m2
* User asks A to sign m2, B to sign m3, C to sign m3
* Hey presto: user has *3* coins from 2 signing events!

elsirion's fedimint cleverly combines several things to make this actually work and be secure:

* bitcoin multisig control of base-chain funds against which minted sat-tokens (fedi-sats) are issued
* shamir's secret sharing allows to calculate BLS signatures (that are x*H(m) where x is privkey, so directly linear in the privkeys), from signature *shares* that each of t out of n federation members, sign.
* blinding done multiplicatively (beta * H(m)), but could also be done multiplicatively (H(m) + beta * G).
* to ensure consistency of what is being signed (see attack above), it uses a BFT system: ensure reliable broadcast of the same message to everyone; see https://fedimint.org/docs/CommonTerms/HBBFTConsensus for the system being used (AFAIK)

Apart from the cryptography, also:

* Lightning gateways, the main goal is interoperability such that fedimint users can pay sats to LN wallets, vice versa etc. Federations can also connect up over LN. (although as per above base-chain funds are also involved). Notice as per above that these charge fees.

Here is the actual mechanism for outgoing and incoming LN payments for a user of the federation. Essentially, you have what is essentially a "virtual HTLC" where the contract is controlled by the federation, but it still keeps some of the nice privacy property of the Chaumian ecash (also, it's a little more involved for incoming than outgoing payments, involving threshold encryption, to the federation keys, of the HTLC preimage):

https://github.com/fedimint/fedimint/blob/d44738dece346e52b1f0042275b01945da036b24/modules/minimint-ln/src/contracts/incoming.rs#L24-L54

https://github.com/fedimint/fedimint/blob/d44738dece346e52b1f0042275b01945da036b24/modules/minimint-ln/src/contracts/outgoing.rs#L6-L24

* They plan to have quite a sophisticated 'social backup' system. I don't think that has been worked out in detail yet.

* Role of 'guardians' (that is, federation members). Non profit.

* UX is intended to be 'the same as Lightning', i.e. it can be basically transparent to users whether they're using fedi-sats or LN-sats.

Above docs go into some detail discussing tradeoffs and risk assessment, see https://fedimint.org/docs/category/trade-offs .