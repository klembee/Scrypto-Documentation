# Hello World !

To get started quickly, you can generate a new project with the following command

```
scrypto new-package helloworld
```

Now, let's look at the directory structure created by the new-package command:

```
- helloworld
    - src
        - lib.rs
    - tests
        - lib.rs
    - Cargo.toml
```

`src/lib.rs` is where you will write your package's code

`tests/lib.rs` is where you will write the unit tests for your blueprints

`Cargo.toml` is your package's manifest. It allows you to set a name, a version and specify the Rust dependencies

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
            let my_bucket: Bucket = ResourceBuilder::new_fungible(DIVISIBILITY_MAXIMUM)
                .metadata("name", "HelloToken")
                .metadata("symbol", "HT")
                .initial_supply_fungible(1000);
    
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

Everything inside this `lib.rs` file defines your package. Typically you will find a single `blueprint!` section in this file.  (It is possible to include other blueprints defined in other files within this `src` directory using the `mod` declaration but the details are beyond the scope of this tutorial.)

The `blueprint!` macro defines a code template that can be published on the ledger. It has two sections: one that defines the data that generated components will have access to and one that defines the functions and methods.

```rust
struct Hello { 
    // Define what resources and data will be managed by Hello components 
    sample_vault: Vault 
}
```

The `struct` section defines the state information for the components that are instantiated from this blueprint. In this case the generated components will have a vault named `sample_vault` where they will be able to store tokens on the ledger.

Next, is the `impl` section where all of the functions and methods of the instantiated components are defined. Notice that the first function named `new` returns a `Component`. Because it returns a Component, it acts as a constructor which, when called, instantiates and returns a new component that operates according to this blueprint.

```rust
pub fn new() -> Component {
    // Create a new token called "HelloToken," with a fixed supply of 1000, and put that supply into a bucket
    let my_bucket: Bucket = ResourceBuilder::new_fungible(DIVISIBILITY_MAXIMUM)
        .metadata("name", "HelloToken")
        .metadata("symbol", "HT")
        .initial_supply_fungible(1000);

    // Instantiate a Hello component, populating its vault with our supply of 1000 HelloToken
    Self {
        sample_vault: Vault::with_bucket(my_bucket)
    }
    .instantiate()
}
```

The `new` function first creates a new type of token named "HelloToken" with a fixed supply of 1000. The 1000 created tokens are all stored in a Bucket named `my_bucket`. Then it instantiates the component using the syntax shown and, within the body of `Self` specifying all of the initial values of the elements listed above in the `struct`. In this case, there is only one, `sample_vault`, which it creates using the tokens inside `my_bucket`.

Instantiating a component from a blueprint is just the beginning. For a component to be useful it also needs to define additional methods that you can call to perform various tasks.

```rust
pub fn free_token(&mut self) -> Bucket {
    info!("My balance is: {} HelloToken. Now giving away a token!", self.sample_vault.amount());
    // If the semi-colon is omitted on the last line, the last value seen is automatically returned
    // In this case, a bucket containing 1 HelloToken is returned
    self.sample_vault.take(1)
}
```

Here we see a method named `free_token` that returns a `Bucket` containing a single token from the `sample_vault`. As you can see, the method contains a single parameter `&mut self` that is used to get access to the state variables, in this case `sample_vault`. Because this is a method (it has a reference to self), you can only call it on instantiated components.

### Trying the blueprint

In the next section, we will show how you can use the Scrypto CLI `resim` to instantiate this blueprint to make a component and then call its method.
