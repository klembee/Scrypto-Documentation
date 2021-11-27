# A Gumball Machine

We will be building a Candy Store shortly but we will get there by starting out with a traditional example in the world of Radix: The Gumball Machine. Create a new directory in location that you find convenient and go to that directory and create a new package called candy-store and replace the src/lib.rs file with a copy of the source file from the gumball-machine example.

```
cd <some_folder>
scrypto new-package candy-store
cd candy-store
rm src/lib.rs
cp <radix-scrypto-folder>/examples/core/gumball-machine/src/lib.rs src/lib.rs
```

Bring up the source file in your editor of choice and let's take a look.

```
use scrypto::prelude::*;

blueprint! {
    struct GumballMachine {
        gumballs: Vault,
        collected_xrd: Vault,
        price: Decimal,
    }

    impl GumballMachine {
        // given a price in XRD, creates a ready-to-use gumball machine
        pub fn new(price: Decimal) -> Component {
            // create a new Gumball resource, with a fixed quantity of 100
            let bucket_of_gumballs = ResourceBuilder::new()
                .metadata("name", "Gumball")
                .metadata("symbol", "GUM")
                .metadata("description", "A delicious gumball")
                .new_token_fixed(100);

            // populate a GumballMachine struct and instantiate a new component
            Self {
                gumballs: Vault::with_bucket(bucket_of_gumballs),
                collected_xrd: Vault::new(RADIX_TOKEN),
                price: price,
            }
            .instantiate()
        }
```

(Describe the header, struct and constructor here.)

Now let's look at the rest of the code:

```
        pub fn get_price(&self) -> Decimal {
            self.price
        }

        pub fn buy_gumball(&mut self, payment: Bucket) -> (Bucket, Bucket) {
            // take our price in XRD out of the payment
            // if the caller has sent too few, or sent something other than XRD, they'll get a runtime err
or
            let our_share = payment.take(self.price);
            self.collected_xrd.put(our_share);

            // we could have simplified the above into a single line, like so:
            // self.collected_xrd.put(payment.take(self.price));

            // return a tuple containing a gumball, plus whatever change is left on the input payment (if 
any)
            // if we're out of gumballs to give, we'll see a runtime error when we try to grab one
            (self.gumballs.take(1), payment)
        }
    }
}

```

(Discuss this code and then have the user publish it and try it out using the CLI.)

Now that we understand how to process payments and return the change with Scrypto, let's try to build something more challenging. In the next section we will change this Gumball Machine into a simple Candy Store.
