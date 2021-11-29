# More on the CLI

### Accounts

As you saw on the last page, you can create a new account with `resim new-account`. If it is the first account you create, resim will automatically set it as the default. You can later change the default account by running: `resim set-default-account [address] [public_key]`. This last command takes the address and public key of the account you want to switch to. The default account is the one you call the components from.

### Fixed supply tokens

Let's create a new token from the CLI ! To create a "Scrypto" token with a fixed supply of 1000, you can run the following command:

```
resim new-token-fixed --name Scrypto --symbol SCO 1000
```

This command creates a new token and inserts all of them in the default account. Now if you run `resim show [account_address]`, you should see the newly created token in the resources:

```
> resim show [account_address]
Resources:
└─ { amount: 1000000, resource_def: 030000000000000000000000000000000000000000000000000004, name: "Radix", symbol: "XRD"}
└─ { amount: 1000, resource_def: 036732a7f237e668965b058e2011d372ce03938bd6e6fa1d0fac87, name: "Scrypto", symbol: "SCO"}
```

You can attach more metadata to a token like an icon url, a description and the url of a website. Type `resim new-token-fixed --help` for the full list of parameters.

### Badges

Before I explain how to create tokens with a mutable supply, you first need to know about badges and how to generate them, because the ability to mint and burn tokens is protected through badges.

You can create a badge with the following command:

`resim new-badge-fixed --name TokenMinter 1`

This command acts just like `new-token-fixed` and offers the same parameters for the metadata. The difference is that badges are not divisible.

Take note of the returned ResourceDef, we will use this address to generate mutable supply tokens!

### Mutable supply tokens

Now that we have a minter badge, we can create a mutable supply token like this:

`resim new-token-mutable --name MutableScrypto [badge_address]`

This creates a new token with an initial supply of zero and with the specified badge required to be able to mint and burn the tokens. You can view the current supply of the created token with `resim show [resource_def_of_token`].

Let's increate the supply !

`resim mint 1000 [resource_def_of_token] [resource_def_of_badge]`

Make sure to run this command while the default account is the one that owns the badge.

### Debugging

Resim comes with other useful commands that will help you debug and test your components.

To show information about an account, component, package or token definition, use: `resim show [an_address]`

To set the current epoch, use: `resim set-current-epoch [epoch_number]`

To find the current epoch and the default account use `resim show-configs`

To list all packages, components and resource definitions: `resim show-ledger`

### Shortcuts

There is a shortcut you can use when redeploying your blueprint if you only changed its methods and functions, not its struct data.

`resim publish . --address [already_published_package_address]`

This will build and deploy your changes while storing the blueprint at the same address. This way, you can continue to use the old package and component address. Note that if you changed your blueprint's struct since the last time you published this package, then this shortcut will fail.
