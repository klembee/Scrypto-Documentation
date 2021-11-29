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

Now let's add the function that adds candy to the candy store and update the prices.

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

This actually handles two cases. If the `candy_addr` is found by the `entry` method then we get back the vault from the HashMap. Otherwise we insert a new empty vault into the HashMap that is prepared to receive tokens that use `candy_addr`. In either case we then add the bucket contents to the vault on the next line. We also update the price whether or not it has a price already.

#### Querying the Candy Store

Next let's uncomment the `get_price` function and update it as shown.

```
 pub fn get_price(&self, candy_addr: Address) -> Decimal {
     scrypto_assert!( candy_addr != self.collected_xrd.resource_address(), "XRD is priceless");
     *self.prices.get(&candy_addr).unwrap()
 }
```

In this new version we pass in the candy's token address. The line of code that actually fetches the price from the HashMap is a bit ugly, but that is due to the extreme care that Rust takes with analyzing all possible paths through your code. Sometimes that extra checking makes the syntax get a bit obscure as we have to peal off safety wrappers to get at our data.

Let's move on to another query which is what candies are available in the store. The easist way to do that is to push them an array of empty buckets with each bucket corresponding to a candy that is for sale but carrying 0. This makes all of the candy resource definitions show up in the current account. From there the user can use the resim show command to get the name, symbol and other metadata for each given token.

```
 pub fn menu(&self) -> Vec<Bucket> {
     let mut buckets = Vec::new();
     for (_addr, vault) in self.candy_vaults.iter() {
         buckets.push(vault.take(0));
     }
     buckets
 }
```

Does this make sense? If not then you might want to look into Vec (short for Vector) which is Rust's version of a dynamic array. Also note that in this case the iter() method loops through the elements of HashMap returning a tuple with the next key and value each time that it is called.

Did you notice that `_addr` has a leading underline character? That is used to let the compiler know that you are aware that this value is not actually used elsewhere in the function. Without the underline you will get a warning from the compiler.

This function has a small error. It will return a bucket for a vault even if that vault doesn't have enough candy available to fulfill a minimal buy request of 1 item. How would you fix that? (Hint: vaults have a method named `amount()` that returns the number of tokens they hold.) Another thing you might want to try is to make a similar function called `free_sample` that returns a tiny portion of every candy in the store. Ir maybe it could return 1 candy of a selected type. Playing around with different possibilities like this is the quickest way to get comfortable with the design, code and test cycle and improve your development speed.

#### Buying Some Candy

To cap off this transformation we will revamp the `buy_gumball` method. We will make this work by sending up the token address to signify which candy we want. We also send a payment bucket as was done in the Gumball Machine. To keep things simple we will only buy one candy at a time, but you can easily extend this logic to handle different purchase quantities if you like.

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

Other than the fact that we are grabbing the vault and the price from the respective HashMaps, (which admittedly is a bit tricky due to Rust's syntax), this function is conceptually unchanged form the `GumballMachine` version.

So there you have it. We took a simple toy example and made the first steps towards turning it into something that might work in the real world. Still there is a major shortcoming. We cannot yet grab the proceeds from selling the candy. Before adding that we first have to think security by controlling access to selected methods. We address topic in the next chapter.
