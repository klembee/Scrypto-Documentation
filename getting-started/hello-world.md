# Hello World !

To get started quickly, you can generate a new project with the following command

```
scrypto new-package helloworld
```

Now, let's look at the generate directory structure:

```
- helloworld
    - src
        - lib.rs
    - tests
        - lib.rs
    - Cargo.toml
```

`src/lib.rs` is where you will write your package's code

`tests/lib.rs` is where you will write the tests for your package

`Cargo.toml` is your package's manifest. It allows you to set a name, a version and specify the Scrypto dependencies

Now open `src/lib.rs`.

```rust
use scrypto::prelude::*;
blueprint! { 
    struct Hello { 
        // Define what resources and data will be managed by Hello components 
        sample_vault: Vault 
    }
    impl Hello {
        // Implement the functions and methods which will manage those resources and data
        
        // This is a function, and can be called directly on the blueprint once deployed
        pub fn new() -> Component {
            // Create a new token called "HelloToken," with a fixed supply of 1000, and put that supply into a bucket
            let my_bucket: Bucket = ResourceBuilder::new()
                .metadata("name", "HelloToken")
                .metadata("symbol", "HT")
                .new_token_fixed(1000);
    
            // Instantiate a Hello component, populating its vault with our supply of 1000 HelloToken
            Self {
                sample_vault: Vault::with_bucket(my_bucket)
            }
            .instantiate()
        }
    
        // This is a method, because it needs a reference to self.  Methods can only be called on components
        pub fn free_token(&mut self) -> Bucket {
            info!("My balance is: {} HelloToken. Now giving away a token!", self.sample_vault.amount());
            // If the semi-colon is omitted on the last line, the last value seen is automatically returned
            // In this case, a bucket containing 1 HelloToken is returned
            self.sample_vault.take(1)
        }
    }
}
```

Everything inside the `blueprint!` section describes your package. Let's go over each sections.

```rust
struct Hello { 
    // Define what resources and data will be managed by Hello components 
    sample_vault: Vault 
}
```

in the struct is where you define the state of the instantiated components. In this case the components will have a vault named `sample_vault` where they will be able to store tokens.

Next, is the `impl Hello` block. This block contains all the methods and functions that your package and instantiated components will provide. The generated code already contains a function named `new` which allows anyone to instantiate a component from the blueprint.&#x20;

```rust
pub fn new() -> Component {
    // Create a new token called "HelloToken," with a fixed supply of 1000, and put that supply into a bucket
    let my_bucket: Bucket = ResourceBuilder::new()
        .metadata("name", "HelloToken")
        .metadata("symbol", "HT")
        .new_token_fixed(1000);

    // Instantiate a Hello component, populating its vault with our supply of 1000 HelloToken
    Self {
        sample_vault: Vault::with_bucket(my_bucket)
    }
    .instantiate()
}
```

The `new` method on this blueprint is quite simple. It first creates a new token name "HelloToken" with a fixed supply of 1000 then it instantiates a component from the blueprint by specifying what the `sample_vault` variable should contain. In that case, it takes the tokens inside my\_bucket (the newly created HelloTokens).

This method acts as a constructor for your components. It is callable on the blueprint itself and generates a new component whenever you call it.

Now, it wouldn't be really useful if that was all you could do with the blueprint. Let's add a method that you can call on the generated components to receive an `HelloToken:`

```rust
pub fn free_token(&mut self) -> Bucket {
    info!("My balance is: {} HelloToken. Now giving away a token!", self.sample_vault.amount());
    // If the semi-colon is omitted on the last line, the last value seen is automatically returned
    // In this case, a bucket containing 1 HelloToken is returned
    self.sample_vault.take(1)
}
```

We created a method named `free_token` that returns a `Bucket` containing a single token from the `sample_vault`. As you can see, the method contains a single parameter `&mut self`. This is used to get access to the state variables, in this case `sample_vault`. Because this is a method ( not a function), you can only call it on instantiated components, not the blueprint itself.

### Trying the blueprint

In the next section, we will show how you can use the Scrypto CLI `resim` to instantiate this component and call its methods !
