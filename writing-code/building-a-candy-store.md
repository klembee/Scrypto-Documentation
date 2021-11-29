# Building a Candy Store

In this section we will transform the `GumballMachine` into a `CandyStore`. Start by changing the `struct` and `impl` names to `CandyStore`.

Next we are going to change the struct. First. instead of one Vault, we want a collection of Vaults, one for each token type. Since token names and symbols can actually clash, let's go the safe route and map between the token's address and the corresponding vault as shown by `candy_vaults`. Similarly the candy prices, which can be different for each candy vaullt, are kept in a map between the token address and the price in the `prices` field.

```rust
 struct CandyStore {
     candy_vaults: HashMap<Address, Vault>, // token address mapped to valut holding that token
     collected_xrd: Vault,
     prices: HashMap<Address, Decimal> // token address mapped to the price in XRD for that token
 }

impl CandyStore {
     pub fn new() -> Component {
         Self {
             candy_vaults: HashMap::new(),
             collected_xrd: Vault::new(RADIX_TOKEN),
             prices: HashMap::new()
         }
         .instantiate()
     }
```

We also update the new() constructor as shown above making it super simple. We construct the component with no active vaults or prices. In a moment we will add a function to add those dynamically.

Comment out the `get_price` and `buy_gumball` methods and go ahead and publish and test what we have so far.

#### Stocking the store

FIXME: The rest of this page needs fixing up.

Now let's add the function that adds candy to the candy store and update the prices.

1.  Now let's add the function that adds candy to the candy store and update the prices.

    ```
     pub fn stock_candy(&mut self, candy: Bucket, new_price: Decimal ) {
         let candy_addr: Address = candy.resource_address();
         scrypto_assert!( candy_addr != self.collected_xrd.resource_address(), "cannot stock XRD as candy");
         scrypto_assert!(new_price > 0.into(), "new price must be a positive value");
         let v = self.candy_vaults.entry(candy_addr).or_insert(Vault::new(candy_addr));
         v.put(candy);
         self.prices.insert(candy_addr, new_price);
     }
    ```

This actually handles two cases. If the candy token is found we get the vault. Otherwise we insert a new vault into the candy\_vaults HashMap. In either case we then add the bucket contents to the vault. We also update the price whether or not it has a orice already.

1.  Next let's uncomment the get\_price function and update it as shown.

    ```
     pub fn get_price(&self, candy_addr: Address) -> Decimal {
         scrypto_assert!( candy_addr != self.collected_xrd.resource_address(), "XRD is priceless");
         *self.prices.get(&candy_addr).unwrap()
     }
    ```

In this new version we pass in the candy's token address. The line of code that fetches the price from the HashMap is a bit ugly, but that is due to the ultra car that Rust takes with analyzing all possible paths through your code. Sometimes that makes the syntax get a bit obscure.

1.  The easist way to show someone what candies are available is to push them an array of empty bucekts with each bucket corresponding to a candy for sale but carrying 0 tokens so that nothing is actually transferred. This makes all of the candy resourcedefs show up in the current account. From there you can essily shoe the name, symbol and other metadata for each given token.

    ```
     pub fn menu(&self) -> Vec<Bucket> {
         let mut buckets = Vec::new();
         for (addr, vault) in self.candy_vaults.iter() {
             buckets.push(vault.take(0));
         }
         buckets
     }
    ```
2.  To cap off this transformation we shall update the buy\_gumball method. We will make this work by sending up the token addrees to signify which candy we want. We also send a payment bucker as in the Gumball Machine. To keep things simple we will only buy one candy at a time, but you can easily extend this logic to handle different purchase quantities if you like.

    ```
     pub fn buy_candy(&mut self, candy_addr: Address, payment: Bucket) -> (Bucket, Bucket) {
         // take our price in XRD out of the payment
         // if the caller has sent too few, or sent something other than XRD, they'll get a runtime error
         // (add asserts here if you want to see friendly messages)
         let price = *self.prices.get(&candy_addr).unwrap();
         // info!("The candy price is {}", price);
         let our_share = payment.take(price);
         self.collected_xrd.put(our_share);
         // info!("Your payment change is {}", payment.amount());

         // we could have simplified the above into a single line, like so:
         // self.collected_xrd.put(payment.take(price));

         // return a tuple containing the requested candy, plus whatever change is left on the input payment (if any)
         // if we're out of stock for that candy, we'll see a runtime error when we try to buy one
         let candy_bucket: Bucket = (*self.candy_vaults.get(&candy_addr).unwrap()).take(1);
         (candy_bucket, payment)
     }
    ```

Other than the fact that we are grabbing the vault and the price from the respective HashMaps, (which admittedly is a bit tricky due to Rust's synax), this function is conceptually uncanged form the GumballMachine version.

So there you have it. We took a simple toy example and made the first steps towards turning it into something that mihgt work in the real world. Still there is a major shortcoming and that is access control. We will address that in teh next chapter.
