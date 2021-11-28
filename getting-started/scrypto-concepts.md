# Scrypto Concepts

## Rust

Scrypto and Rust are joined at the hip. You can't become a Scrypto programmer without learning a good bit about Rust in the process. Fortunately there is a lot of good training material for Rust. To save you time, this tutorial will point out which parts of Rust you need to understand. The rest you can pick up on an as-needed basis.

In the realm of programming languages Rust is considered medium level in complexity. However most of the advanced aspects of Rust do not come into play when coding with Scrypto. Therefore beginning programmers can start with Scrypto and expect to see real results from their efforts. Having said that, truly mastering Scrypto does take you deep into the Rust rabbit hole and so don't expect complete mastery to come about quickly.

## Asset Oriented Programming

Asset-oriented programming incorporates assets as first class types in the language. This allows the compiler and runtime to reason about asset handling using specially designed safety checks that protect programmers and end users alike. The resulting code is generally atomic and can entirely prevent certain types of bugs related to double spending, unauthorized access, misplaced assets, reentrancy and more.

Radix claims that Scrypto goes even further in this regards than earlier experimental languages such as 'flint' and 'Cadence'. For instance, in Radix the set of all tokens protected by the compiler and runtime extends to the core XDR protocol token as well. In addition there are access control entities known as badges that get the same added protections as tokens. Accordingly Radix refers to tokens, badges and similarly handled entities as 'resources' and they all get similar first class treatment in terms of safety and efficacy.

### Resources and Resource Holders

The early release of Scrypto implements tokens and badges. NFTs should also be supported either in the Alexandria release or soon thereafter. We will discuss the details about creating and managing these resources later in this tutorial, but here is the overriding rule: **All resources must be held in a resource holder of some kind at all times!**

A Resource Holder is a container for resources. The Bucket is the workhorse holder. You pass resources around in buckets and manipulate their contents by moving them between buckets and vaults. However buckets are temporary. They must be empty and/or dropped by the end of a given public function or method.

If you need to hold resources more permanently within a component, you must use a Vault. For that reason you will see Vaults in blueprint structs, but never Buckets.

In addition there is Account but it is not used often in blueprints. An Account is part of a wallet or other resource collector that is not controlled by a component. Some of the details about interacting with and defining Accounts are not yet settled and so their importance for day to day Scrypto development is not well understood yet. For now just be aware that they exist.

## Blueprints, Components and Packages versus Smart Contracts

Scrypto has a more sophisticated way to deploy logic to the ledger than first generation smart contract protocols. Let's dive in.

### Blueprints

Blueprints are compiled source code that lives on ledger in a form where it is available for anyone to reuse although there may be an author specified royalty involved with doing so. Blueprints never have state and so in that sense they are not active. Instead they provide one or more constructor functions that allows others to instantiate them. These constructors may have any number of arguments that can parameterize the operation of the blueprint's code. So while blueprints tend to be are highly specialized in terms of their functionality, they may also be used to support many different use cases depending on exactly how they are instantiated.

### Packages

In some cases blueprints are designed to work closely together with other blueprints and in this case they can deployed together as a package. Deploying them together in this way means that a given blueprint can be sure that some other different blueprint exists and therefore it's code may instantiate them as part of their operations. In addition it is reasonable to presume that deploying blueprints together as a package may save on deployment costs. Like blueprints, packages do not manage state information.

Note that the full details on the many way that blueprints can be manipulated in order to make them work together on your behalf are not yet available.

### Components

To activate a blueprint you instantiate it by calling one of its' constructor functions. When this process completes you get the address of the newly created instance which is called a component. Components manage state and can gather, hold and distribute resources according to the logic provided in its' associated blueprint. Technically components never die however there may be logic within a blueprint that makes a component useless after certain conditions are triggered.

So in Scrypto components are the closest thing to what we think of when we think of a smart contract. However all of the smarts in a component derive from the logic that is defined in the blueprint that gave birth to that component via one of its' constructor functions.

### Versus Smart Contracts

So we can say that Radix has smart contracts but, as we have seen above, the situation is quite different since the package/blueprint/component model can be far more configurable and the alignment between components can be far tighter and safer than smart contracts can ever hope to deliver.

## Behind the Scenes: The Radix Engine, Radix API and Cerebus

Part of what makes Radix safer than other protocols is due to the design and implementation of the Radix Engine that is running on the ledger. All tokens, badges and other resources are defined by the Radix Engine making their operations faster and more predictable than if they were defined in third party programmer provided smart contracts. The Radix designers decided that all of the assets running on their protocol deserved that extra level of expedited care and concern.

## Community

Taken as a whole the Radix stack, from the low level protocol to the highest level abstractions, is a lot to digest. No one should expect to master it all by yourself. It is good that you don't have to since already in these early days of Scrypto development, a healthy international community has formed ...
