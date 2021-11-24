# Scrypto Concepts

## Rust

Scrypto and Rust are joined at the hip. You can't become a Scrypto programmer without learning a good bit about Rust in the process. Fortunately there is a lot of good training material for Rust. To save you time, this tutorial will point out which parts of Rust you need to understand. The rest you can pick up on an as-needed basis.

In the realm of programming languages Rust is considered medium level in complexity. However most of the advanced aspects of Rust do not come into play when coding with Scrypto. Therefore beginning programmers can start with Scrypto and expect to see real results from their efforts. Having said that, truly mastering Scrypto does take you deep into the Rust rabbit hole and so don't expect complete mastery to come about quickly.

## Asset Oriented Programming

Asset-oriented programming incorporates assets as first class types in the language. This allows the compiler and runtime to reason about asset handling using specially designed safety checks that protect programmers and end users alike. The resulting code is generally atomic and can entirely prevent certain types of bugs related to double spending, unauthorized access, misplaced assets, reentrancy and more.

Radix claims that Scrypto goes even further in this regards than earlier experimental languages such as 'flint' and 'Cadence'. For instance, in Radix the set of all tokens protected by the compiler and runtime extends to the core XDR protocol token as well. In addition there are access control entities known as badges that get the same added protections as tokens. Accordingly Radix refers to tokens, badges and similarly handled entities as 'resources' and they all get similar first class treatment in terms of safety and efficacy.

### Resources

* Tokens
* Badges
* NFTs

### Resource Holders
* Vault
* Bucket
* Account

## Blueprints, Components and Smart Contracts

## Behind the Scenes: The Radix Engine, Radix API and Cerebus

## Community

Taken as a whole the Radix stack, from the low level protocol to the highest level abstractions, is a lot to digest. No one should expect to master it all by yourself.  It is good that you don't have to since already in these early days of Scrypto devlopment, a healthy internnational community has formed ...
