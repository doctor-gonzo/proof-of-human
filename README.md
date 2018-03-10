# Proof of Human
Uses Winograd Schema Challenges to prove humanity of a smart-contract caller.

### Rationale

A Winograd Schema Challenge (WSC) is a type of common-sense reasoning test which has been suggested as an improvement on / alternative to the [Turing Test](https://en.wikipedia.org/wiki/Turing_test). 

If properly implemented, a smart contract leveraging WSCs could be used to verify that a particular transaction or message was initiated by a human, rather than by an automated process. As blockchain technology becomes more widely used, this or a similar verification technique may be used to create a filter similar in function to a [CAPTCHA](https://en.wikipedia.org/wiki/CAPTCHA) for smart-contract interactions. Proof-of-humanity could be useful in a legal context to fortify the validity of a digital signature, or to guarantee that certain transactions and/or transaction types can only be initiated by a human. 

### Format/Examples of WSCs

The following explanation is from [commonsensereasoning.org](commonsensereasoning.org), where a more technical description of WSCs is [also available](http://commonsensereasoning.org/winograd.html). 

Rather than base the test on the sort of short free-form conversation suggested by the Turing Test, the WSC poses a set of multiple-choice questions that have a particular form. Two examples follow:


> I.	The trophy would not fit in the brown suitcase because it was too **big** (*small*). What was too **big** (*small*)?

`Answer 0: the trophy`

`Answer 1: the suitcase` 

and 

> II.	The town councilors refused to give the demonstrators a permit because they **feared** (*advocated*) violence. Who **feared** (*advocated*) violence? 

`Answer 0: the town councilors`

`Answer 1: the demonstrators`


The answers to the questions (in the above examples, `Answer 0` for the sentences if the **bolded** words are used; `Answer 1` for the sentences if the *italicized* words are used) are expected to be obvious to a layperson. A human who answers the first question correctly would likely use their knowledge about the typical size of objects and their ability to do spatial reasoning to solve the first example; they would likely use their knowledge about how political demonstrations unfold and their ability to do interpersonal reasoning to solve the second example. This reasoning ability is common to most humans, but currently remains elusive to formalization by computers.


### Integration of WSCs in context of smart-contracts

This section is intended to lay out a step-by-step description of what the process of using WSCs in the context of a decentralized application might look like. This is early stage and conceptual, if you notice any issues or errors in reasoning please let me know. It will likely change as I learn more about the intricacies of smart-contracts and data storage on the EVM. 

Let's imagine that a `User` is interacting with a `dApp` which requires a proof-of-humanity before allowing a further action to take place. The third party would be the `WSC contract`, which itself communicates with an off-chain `WSC program` (whose properties are described in the [WSC Program section](https://github.com/doctor-gonzo/proof-of-human/blob/master/README.md#decentralization--centralization-nature-of-the-wsc-program)):

1. The `dApp` initiates a transaction to the `WSC contract`, specifying the address of the `User` whose humanity is in question, along with a pseudorandom number encrypted using a public key (PGP, or XMSS as Quantum Computing becomes a more serious threat to blockchain platforms) belonging to the deployer of the `WSC contract` and fixed at genesis of the `WSC contract`
2. The `User` initiates a transaction to the `WSC contract`, including their own pseudorandom number encrypted with the same PGP key belonging to the deployer of the `WSC contract`
3. Once such a pair exists (corresponding `dApp` and `User` transactions), the `WSC contract` generates a pseudorandom number (derived by the combination of the numbers provided by the `User` and the `dApp`, or potentially obtained through something like [RANDAO](https://github.com/randao/randao)) 
4. The random value would be used to choose which version of *n* WSCs (where *n* could be a default value, or specified precisely as part of the initializing transaction from the `dApp`) from a [collection of WSCs](https://cs.nyu.edu/faculty/davise/papers/WinogradSchemas/WSCollection.html). As seen above, the answer to the first example WSC depends on whether the word **big** or *small* is used
5. The `WSC program` would decrypt the pseudorandom number obtained from the `dApp`, and after selecting which *n* WSCs to challenge the `User` with, determine the expected/successful response to those WSCs, it would perform some hashing function on a combination of the random number and the correct responses. The `WSC program` could then encrypt the output of the hash function using the public key of the `dApp` address (which would need to be an EOA account, reasoning further explored in the ["Things I have learned" section](https://github.com/doctor-gonzo/proof-of-human/blob/master/README.md#things-i-have-learned-while-creating-this-document)) obtained by `ecrecover` from [ethereumjs-util library](https://github.com/ethereumjs/ethereumjs-util). The `WSC program` would then write this message to a file and upload it to the [Interplanetary File System, or IPFS](https://ipfs.io/), returning the hash value of that [IPFS object](https://medium.com/@ConsenSys/an-introduction-to-ipfs-9bba4860abd0) from the `WSC contract` to the `dApp`
6. After being encrypted by the `WSC program` using the public key of `User` obtained through `ecrecover` from [ethereumjs-util library](https://github.com/ethereumjs/ethereumjs-util),  or a similar function, through a user-interface, the *n* WSCs would be decrypted by the `Users`'s private key (potentially made smooth by the integration of Metamask or other browser-based wallet systems) presented to the `User` in such a way that only one answer makes logical sense for each WSC, with `Answer 0` and `Answer 1` both clearly visible and intuitively easy to select. They reason for the necessity of encrypting the WSCs (chosen and written to an IPFS object by the `WSC Program`, whose hash-value / link would be stored and visible in the `WSC contract`) to the `User` is to verify that the account which initiated the contact with the `dApp` is the one solving the WSCs, and that off-loading this solving obligation to another human in exchange for money (as is the case with [CAPTCHA farms](http://www.zdnet.com/article/inside-indias-captcha-solving-economy/), where people are paid to answer CAPTCHAs all day) would imply handing over full access to the `User` account before the WSC could be sovled by an untrusted third party, significantly decreasing the incentive to do so 
7. A text field at the bottom of the page would collect the `User`'s answers, which would be in the form of either a `1` or a `0` (potentially derived from their clicks on the correct `Answer` block for ease-of-use). A response to 5 WSCs might look something like `01011`
8. Upon receiving a correct response to the WSCs (which are stored off-chain by `WSC Program`), the `WSC contract` decrypts the `dApp`'s provided random number from earlier and gives it to the `User`
9. The successfully authenticated `User` now hashes their response alongside the recently-obtained secret number, which was originally submitted by `dApp` and locked until the correct response to WSCs on the part of the `User` are received by `WSC contract`

as a last step: 

10. The `User` sends their final message (the correct binary response to the WSCs hashed together with the secret number of the `dApp`). Sending this in a transaction to the `dApp` would guarantee to both parties that they had gone through a safe exchange process, especially if the role of the `WSC Program` could be performed by other smart-contracts in an affordable/efficient way and all the code was open source

or 

10. Some specialised token would be sent to the `User`, and a payment of the exact # of POH tokens required to `dApp` would verify to both parties that they engaged in this process completely. The tokens should ideally be worthless to hoard, as sending a larger number won't prove "more humanity". They could be distributed out of the `WSC contract` over time, and **if there is any way to make it so that an ERC-20 token can only be sent twice in its lifetime it would be an interesting method to explore as well**.

### Additional Thoughts

#### Future proofing and a type of [Canary Clause](https://en.wikipedia.org/wiki/Warrant_canary) for tracking AI progress using incentive structures / crypto-economics: 

From [commonsensereasoning.org](commonsensereasoning.org): 

> Due to the wide variety of commonsense knowledge and commonsense reasoning that would presumably be used by humans to solve Winograd Schema problems, it was proposed during Commonsense-2013 that the Winograd Schema Challenge could be a promising method for tracking progress in automating commonsense reasoning.

If an implementation of the `WSC contract` could create an incentive to attack the contract to earn money (normal people at normal speeds could do them at a rate generating small $, perhaps from a small fee paid by `dApp` for authentication) wheras someone who had solved the underlying WSC problem could make a lot of money all at once (good for them) and drain the contract (good for us, because it shows that the scheme is no longer a feasible means of proving humanity). **As long as the WSC Conract remains un-drained, you are given the guarantee that nobody has built an AI capable of cracking it, OR that the economic value of keeping such an AI a secret exceeds the economic value of breaking the WSC contract + [the $25,000 being offered for solving the underlying WSC problem](http://commonsensereasoning.org/winograd.html)**. A breakthrough like this would show that a large chunk of the necessary work towards general machine intelligence had been done. As the potential monetary prize for breaking this contract increased (perhaps as a function of total use?), it would become more likely that the development of such AI technology would be revealed through the draining of the contract, and a more reliable indicator for the level of our language-processing AI progress.

#### Steps in transforming this from an idea to a reliable program

- [ ] Formatting this [collection of example Winograd Schemas](https://cs.nyu.edu/faculty/davise/papers/WinogradSchemas/WSCollection.html) so as to be usable by a `WSC Program`

- [ ] Learning how to interact more proficiently with github, and building an application which provably references a github repository as a basis for trust / auditability

- [ ] Continuing to explore whether the role of the `WSC Program` can be fully decentralized - the reason it is hard is because of the nature of messages sent over the blockchain and the fact that they cannot be encrypted so as to be visible to the smart contract but nobody else. More information about this particular issue in available in the [section below](https://github.com/doctor-gonzo/proof-of-human#things-i-have-learned-while-creating-this-document) 

- [ ] Guaranteeing that my program can't be solved instead of the underlying WSC: try to formally prove that my implementation is not predictable, which would remove the guarantee of humanity provided by WSCs


#### Problems / Challenges

- Original implementation would be in English, rendering the program useless to much of the world's population. If proven useful there are likely similar constructs in other languages that could be implemented in the same general way

- It might just be better to update current day CAPTCHAs to interact with the blockchain, although [some of my reading](http://www.wired.co.uk/article/captcha-automation-broken-history-fix) has suggested that CAPTCHAs are becoming less reliable due to advances in AI and CAPTCHA farming

- This idea includes multiple transactions between the `WSC contract`, the `dApp` and the `User`. These transactions would have to be affordable and fast, which relies on further scaling in the underlying blockchain protocols

#### Source of idea

My initial thought on this subject was related to the idea of needing to ping an owner of a piece of digital property once per year to ensure that they are still alive, or else the property is put up for auction. It would be trivial to set up a program which would automatically send a given message at any future date, but it would not be trivial to implement such a program with the ability to also solve WSCs

#### Things I have learned while creating this document 

I knew that there were two types of ethereum accounts, but there are nuances that I did not fully grasp. There are Externally Owned Accounts (EOAs), which are controlled by private keys - user wallets for storing and transfering funds fall under this category. The other type is the Contract Account, which is controlled by the code of the contract and does not have a private key. The result of this lack of private key on the Contract Account is that you cannot encrypt a message so that a smart contract can decipher it while a blockchain-onlooker cannot. As I understand it, this means that any secret data must be encrypted before being transmitted to the public chain/network and introduces programming challenges that do not exist in more commonplace centralized server deployments.

As much as centralized architectures are maligned, [they do have a number of benefits](http://blog.iweb.com/en/2015/04/benefits-centralized-storage/14645.html). The art of getting the most out of blockchain systems might be knowing which aspects to adopt, and selecting them based on their ability to remove corruption or censorship, while investigating the potential for "transparent centralization" as a sort of inverse solution to scaling, until improvements in technology allow for the complete decentralization where deemed desirable. I put transparent centralization in quotes because I know it is a phrase that has the potential to receive attention, because it seems so contrary to what blockchain programming should entail.

##### Decentralization / Centralization (Nature of the `WSC program`) 

My idea for a "transparant and centralized" version of this software would include a program that listens for events broadcast by the `WSC Contract`, and performs the selection of *n* WSCs/selection of a version of each WSC prompt/validation of "response to prompt" submitted by `User`, with a friendly and logical user interface that is needed to make such a tool accessible to a large number of future web3 users. The transparent aspect of this solution would be the open-source nature of the software - if desired, power users could set up their own implementations of the exact software used, to verify that there are no intentional security flaws in the software, much like how [MEW](https://www.myetherwallet.com) has been configured.

If it were possible to put the code on github, and the only additional parameter that was needed was the 'validator' pgp key supplied by whoever was initializing the `WSC contract`, most people could safely use the software knowing that the code is completely open sourced, and skeptical users / power-users / those implementing blockchain systems within various industrial and governmental frameworks could modify the code to meet any level of required security.

A proof-of-humanity obtained by a specific `dApp`, might mean nothing about the holder's humanity to any other `dApp`(since any `User` and `dApp` could potentially be the same party). It would only be a gurantee insofar as the seperation of the `dApp` and `User` could be guaranteed, so each `dApp` would need to verify humanity independently and over time. If there was a `dApp 2` that a given `dApp 1` implicitly trusts, a proof-of-humanity on `User` collected by the trusted `dApp 2` could be substituted for a proof-of-humanity collected by `dApp 1`. This would remove potential friction that would come from interacting with different smart-contract whose results work in conjunction, provided by the real-world organizations which are verifiably connected, yet still having to repeatedly prove humanity to the same organization over and over (if it were some sort of government-level blockchain program, for example). 

Perhaps [attention-mining](https://github.com/DemocracyEarth/paper#324_Attention_Mining) could be used to guarantee the correctness of WSCs served by the `WSC program`, and there could be a mechanism by which people get paid for testing the validity of the WSCs being served by the program, and a way for them to audit the program and earn money for reporting any mistakes (which would be a sign that the `WSC program` was not working correctly or had been improperly configured).


##### Misc Questions

- Have there been any attempts to make an ERC-20 token which can only be transacted *n* times? 

Signature: 

{
  "address": "0xb3649ab572cf1dc17293817403bcc4d2f9d6f29b",
  "msg": "Author of this document discussing Winograd Schema Challenges and Smart-Contracts is 0xb3649ab572cf1dc17293817403bcc4d2f9d6f29b",
  "sig": "0xd33d86100400ee7002db0b892a4c9a959f54f55a26488ef3fb88aae35c28a6c90a71ca58e1a1bc2b7863f4f14888a2a47ab7f03040c7d9e27752c8e7e6e256e31c",
  "version": "2"
}
