# Creating Tokens

Just like with resim, you can create tokens and badges inside your components. You already learned how to create fixed supply tokens if you have done the [HelloWorld](../getting-started/hello-world.md) example. Let's learn how to create badges and mutable supply tokens !

### Fixed supply tokens and badges

To create tokens and badges, you will use the `ResourceBuilder` utility. It allows you to specify metadata on the tokens and specify the supply. When creating tokens and badges of fixed supply, a bucket is returned containing the created resources.&#x20;

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
            let badge = ResourceBuilder::new()
                .metadata("name", "Acces Badge")
                .metadata("symbol", "TB")
                .metadata("icon_url", "https://badge_website.com/icon.ico")
                .metadata("url", "https://badge_website.com")
                .new_badge_fixed(1);

            // Create tokens with fixed supply of 1 000 000 000
            let tokens = ResourceBuilder::new()
                .metadata("name", "Really Cool Token")
                .metadata("symbol", "RCT")
                .new_token_fixed(1_000_000_000);

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
