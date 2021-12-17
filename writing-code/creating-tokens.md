# More About Token Creation

Just like with resim, you can create tokens and badges inside your components. You already learned how to create fixed supply tokens if you have done the [HelloWorld](../getting-started/hello-world.md) example. Let's learn how to create badges and mutable supply tokens !

### Fixed supply tokens and badges

To create tokens and badges, you will use the `ResourceBuilder` utility. It allows you to specify metadata, specify the supply and define who can mint, burn and update its metadata. When creating tokens and badges of fixed supply, a bucket is returned containing the created resources.

Here is an example of a component creating an access badge and a token. The tokens are stored in a vault of the component and the badge is returned to the caller.

```rust
use scrypto::prelude::*;

blueprint! {
    struct TokenCreator {
        token_vault: Vault
    }

    impl TokenCreator {
        pub fn new() -> (Component, Bucket) {

            // Create a badge with fixed supply of 1
            let badge = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
                .metadata("name", "Acces Badge")
                .metadata("symbol", "TB")
                .metadata("icon_url", "https://badge_website.com/icon.ico")
                .metadata("url", "https://badge_website.com")
                .initial_supply_fungible(1);

            // Create tokens with fixed supply of 1 000 000 000
            let tokens = ResourceBuilder::new_fungible(DIVISIBILITY_MAXIMUM)
                .metadata("name", "Really Cool Token")
                .metadata("symbol", "RCT")
                .initial_supply_fungible(1_000_000_000);

            let component = Self {
                token_vault: Vault::with_bucket(tokens)
            }
            .instantiate();

            // Don't forget! At the end of a function of method,
            // all buckets must be stored in a vault or returned.
            (component, badge)
        }
    }
}
```

### Mutable supply tokens and badges

As with resim, to create a mutable supply resource, you first need a minter badge. If you then want your component to be the only one allowed to mint and burn the resources, you can store the minter badge inside one of its vaults like in the next example:

```rust
use scrypto::prelude::*;

blueprint! {
    struct TokenCreator {
        minter_badge: Vault,
        token_vault: Vault
    }

    impl TokenCreator {
        pub fn new() -> (Component, Bucket) {
            let minter_badge = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
                .initial_supply_fungible(1);

            // Create a mutable supply token and specify
            // the resource definition of the badge allowed to mint and burn.
            // Notice that new_token_mutable does not return a bucket but only a resource_definition.
            let token_resource_def = ResourceBuilder::new_fungible(DIVISIBILITY_MAXIMUM)
                .metadata("name", "Really Cool Token - but mutable")
                .metadata("symbol", "RCTM")
                .flags(MINTABLE | BURNABLE)
                .badge(minter_badge.resource_def(), MAY_MINT | MAY_BURN)
                .no_initial_supply();

            // Now we can mint tokens
            let tokens = token_resource_def.mint(1000, minter_badge.present());

            // It's the same when creating mutable badges (divisibility of 0)
            let badge_resource_def = ResourceBuilder::new_fungible(DIVISIBILITY_ZERO)
                .metadata("name", "Mutable Badge")
                .flags(MINTABLE | BURNABLE)
                .badge(minter_badge.resource_def(), MAY_MINT | MAY_BURN)
                .no_initial_supply();

            let badge = badge_resource_def.mint(1, minter_badge.present());

            let component = Self {
                minter_badge: Vault::with_bucket(minter_badge),
                token_vault: Vault::with_bucket(tokens)
            }
            .instantiate();


            (component, badge)
        }
    }
}
```

Notice that we do `minter_badge.present()` when calling `mint`. That's because the mint method expect a `BucketRef` and not a `Bucket`. Calling present`()` transforms the bucket into a BucketRef.

Because we saved the minter badge in one of the component's vault, we are able to mint and burn the resources in another method:

```rust
pub fn burn_badge(&mut self, badge_to_burn: Bucket) {
    // Take the minter badge out of the vault
    let badge_bucket = self.minter_badge.take(1);

    // Burn the provided badge
    badge_to_burn.burn(badge_bucket.present());

    // Put the badge back into its vault
    self.minter_badge.put(badge_bucket);
}

pub fn mint_tokens(&mut self) -> Bucket {
    // Take the badge out of the vault
    let badge_bucket = self.minter_badge.take(1);

    // Let's mint 100 RCTM
    let bucket = self.token_vault.resource_def().mint(100, badge_bucket.present());

    // Put the badge back into its vault
    self.minter_badge.put(badge_bucket);

    // Return the minted tokens
    bucket
}
```

Because badges are used a lot, it can become annoying to always take the badge from the vault and having to put it back at the end. That's why the Radix team added the `authorize` method on vaults:

```rust
pub fn burn_badge(&mut self, badge_to_burn: Bucket) {
    self.minter_badge.authorize(|badge| {
        badge_to_burn.burn_with_auth(badge);
    });
}

pub fn mint_tokens(&mut self) -> Bucket {
    self.minter_badge.authorize(|badge| {
        self.token_vault.resource_def().mint(100, badge)
    })
    // Notice, there is no semicolon. The created bucket will be returned.
}
```

Much cleaner don't you think ?
