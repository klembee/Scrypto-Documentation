# A Gumball Machine

We will be building a Candy Store shortly but we will get there by starting out with a traditional example in the world of Radix: The Gumball Machine. Create a new directory in location that you find convenient. Create a new package called candy-store and replace the src/lib.rs file with the gumball machine file located here: [https://github.com/radixdlt/radixdlt-scrypto/blob/main/examples/core/gumball-machine/src/lib.rs](https://github.com/radixdlt/radixdlt-scrypto/blob/main/examples/core/gumball-machine/src/lib.rs)

Bring up the `lib.rs` file in your editor of choice and let's take a look.

```rust
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
            // We specify a divisibility of 0 meaning that we cannot send fractions of a gumball
            let bucket_of_gumballs = ResourceBuilder::new_fungible(DIVISIBILITY_NONE)
                .metadata("name", "Gumball")
                .metadata("symbol", "GUM")
                .metadata("description", "A delicious gumball")
                .initial_supply_fungible(100);

            // populate a GumballMachine struct and instantiate a new component
            Self {
                gumballs: Vault::with_bucket(bucket_of_gumballs),
                collected_xrd: Vault::new(RADIX_TOKEN),
                price: price,
            }
            .instantiate()
        }
```

The `GumballMachine` blueprint looks very similar to the `Hello` blueprint that we just reviewed. This time we have 2 vaults: one for the gumballs that will be sold and the other for the XRD tokens that we will collect from those who purchase gumballs. The `price` in XRD for each Gumball is also maintained as a variable so that it can be changed if we provide code to do that. Notice that `price` is a `Decimal`. This is how we save and use numbers that have a fractional component in Scrypto. Floats are not supported on the ledger. It is usually best not to use them at all.

The constructor also looks similar to what we wrote in `Hello`. One difference is that the `new` constructor takes an argument named `price`. This argument will be saved into the component's state. Notice how we instantiate the two vaults differently. We instantiate the `gumballs` vault with the content of the gumball bucket and we instantiate an empty `collected_xrd` vault that will contain XRD tokens.

Now let's look at the rest of the code:

```rust
        pub fn get_price(&self) -> Decimal {
            self.price
        }

        pub fn buy_gumball(&mut self, payment: Bucket) -> (Bucket, Bucket) {
            // take our price in XRD out of the payment
            // if the caller has sent too few, or sent something other than XRD, they'll get a runtime error
            let our_share = payment.take(self.price);
            self.collected_xrd.put(our_share);

            // we could have simplified the above into a single line, like so:
            // self.collected_xrd.put(payment.take(self.price));

            // return a tuple containing a gumball, plus whatever change is left on the input payment (if any)
            // if we're out of gumballs to give, we'll see a runtime error when we try to grab one
            (self.gumballs.take(1), payment)
        }
    }
}
```

There are just two methods defined. The`get_price` method returns the price which is held as part of the component's state. If by now you are itching to do some coding, try adding another method named `set_price` that can be called to change the gumball price.

Finally we will look at the `buy_gumball` method which is really the heart of this blueprint. This method takes a `payment` Bucket as input. It then takes a portion of the tokens from the bucket as dictated by `self.price`. The caller's change, if any, remains in the `payment` bucket. Notice that the code makes no effort to know if there is any change. Instead it is just going to return the `payment` bucket to the caller without bothering to check whether or not it is empty.

The final line of the method specifies the two buckets that will be returned to the caller. The first bucket is created by using the `take` method from the `self.gumballs` vault. The `take` method returns a bucket with the amount of tokens specified. Of course the second part of the tuple is the previously discussed bucket containing the change. This tuple matches the functions stated return pattern designation of `(Bucket, Bucket)` and so this method is complete.

#### Sending buckets with resim

Notice that the `buy_gumball` method requires a bucket. How do we send a bucket with resim ?

Here is how you would send a bucket containing 10 XRD to the `buy_gumball` method:

```
call-method [component_address] buy_gumball 10,030000000000000000000000000000000000000000000000000004
```

`030000000000000000000000000000000000000000000000000004` is the resource definition of the XRD token.

#### Testing the gumball machine

Let's run a quick smoke test using `resim`.

```bash
resim reset
resim new-account # Remember the account's address
resim publish . # Remember the package's address

resim call-function [package_address] GumballMachine new 0.5 # Remember the component's address
resim call-method [component_address] get_price # Should return Some(0.5)

# Send 3 XRD to the buy_gumbal method
resim call-method [component_address] buy_gumball 3,030000000000000000000000000000000000000000000000000004
resim show [component_address]
resim show [account_address]
# Check that everything adds up.
```

Now that we understand how to process payments and return the change with Scrypto, let's try to build something more challenging. In the next section we will change this Gumball Machine into something a bit closer to the real world: a Candy Store that sells a number of different products that can change over time.
