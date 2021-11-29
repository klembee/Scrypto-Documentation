# Building a Candy Store

In this section we will improve the `GumballMachine` by building a `CandyStore`. Start by changing the `struct` and `impl` names to `CandyStore`.

Next we are going to change the struct. First, instead of one Vault, we want a collection of Vaults, one for each token type. Since token names and symbols can actually clash, let's go the safe route and map between the token's address and the corresponding vault as shown by `candy_vaults`. Similarly the candy prices, which can be different for each candy vaullt, are kept in a map between the token address and the price.

```rust
 struct CandyStore {
    // token address mapped to valut holding that token
    candy_vaults: HashMap<Address, Vault>,

    collected_xrd: Vault,

    // token address mapped to the price in XRD for that token
    prices: HashMap<Address, Decimal>
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

We also updated the new() function as shown above making it super simple. We construct the component with no active vaults or prices. In a moment we will add a method to add those dynamically.

Comment out the `get_price` and `buy_gumball` methods. You can now publish and test what we have so far to make sure it works.

#### Stocking the store

Now let's write the method that adds candy to the candy store and update the prices.

```rust
pub fn stock_candy(&mut self, candy: Bucket, new_price: Decimal) {
    let candy_addr: Address = candy.resource_address();

    // We can write assertions. If it fails, the whole transaction is safely rolled back.
    // Here, we make sure that the provided bucket does not contain XRD
    // and that the price is greater than zero.
    assert!( candy_addr != self.collected_xrd.resource_address(), "cannot stock XRD as candy");
    assert!(new_price > 0.into(), "new price must be a positive value");

    // Try to find the vault with candy_addr as key.
    // If it does not exist, it creates a new empty vault.
    let vault = self.candy_vaults.entry(candy_addr).or_insert(Vault::new(candy_addr));

    // Insert the candies in the vault
    vault.put(candy);

    // Update the price
    self.prices.insert(candy_addr, new_price);
}
```

Line 12 handles two cases. If the `candy_addr` is present in the `candy_vaults` keys, then we get back the vault from the HashMap. Otherwise we insert a new empty vault into the HashMap that is prepared to receive tokens. In both cases we then add the candy bucket to the vault on the next line. We also update the price whether or not it has a price already.

#### Querying the Candy Store

Next let's uncomment the `get_price` method and update it as shown.

```rust
pub fn get_price(&self, candy_addr: Address) {
    // Make sure the candy_addr is not XRD
    assert!(candy_addr != self.collected_xrd.resource_address(), "XRD is priceless");

    // Display the price if present, display error otherwise
    match self.prices.get(&candy_addr) {
        Some(price) => info!("Price: {}", price),
        None => info!("Could not find candy in stock !")
    };
}
```

Let's write a `menu` method that shows what candies are available in the store. An easy way to do that is to return an array of empty buckets for each candy we have in store. This makes all candy resource definitions show up in the current account's resources. After calling `menu`, the user can use the `resim show [account_address]` command to get the name, symbol and other metadata of each given token.

```rust
pub fn menu(&self) -> Vec<Bucket> {
    let mut buckets = Vec::new();
    for (_addr, vault) in self.candy_vaults.iter() {
        buckets.push(vault.take(0));
    }
    buckets
}
```

Does this make sense? If not then you might want to look into Vec (short for Vector) which is Rust's version of a dynamic array. Also note that the iter() method allows us to loop through the elements of our candy_vaults HashMap.

Did you notice that `_addr` has a leading underline character? This is used to let the compiler know that you are aware that this value is not actually used elsewhere in the method. Without the underline you will get a warning from the compiler.

This method has a small problem. It will return a bucket for a vault even if it doesn't have enough candy available to fulfill a minimal buy request of 1 item. How would you fix that? (Hint: vaults have a method named `amount()` that returns the number of tokens they hold.) Another thing you might want to try is to make a similar method called `free_sample` that returns a tiny portion of every candy in the store. It could return 1 candy of a selected type. Playing around with different possibilities like this is the quickest way to get comfortable with the design, code and test cycle and improve your Scrypto skills.

#### Buying Some Candy

To cap off this component's transformation we will improve the `buy_gumball` method. We will add a parameter allowing the user to specify which candy to buy. To keep things simple we will only buy one candy at a time, but you can easily extend this logic to handle different purchase quantities if you like.

```rust
pub fn buy_candy(&mut self, candy_addr: Address, payment: Bucket) -> (Bucket, Bucket) {
    // take our price in XRD out of the payment
    // if the caller has sent too few, or sent something other than XRD, they'll get a runtime error
    let price = match self.prices.get(&candy_addr) {
        Some(price) => price,
        None => {
            info!("Candy not in stock !");
            std::process::abort()
        }
    };

    self.collected_xrd.put(payment.take(*price));

    let candy_bucket: Bucket = match self.candy_vaults.get(&candy_addr) {
        Some(vault) => vault.take(1),
        None => {
            info!("Candy not in stock !");
            std::process::abort()
        }
    };

    (candy_bucket, payment)
}
```

Other than the fact that we are grabbing the vault and the price from the respective HashMaps, this method is conceptually unchanged form the `GumballMachine` version.

So there you have it. We took a simple toy example and made the first steps towards turning it into something that might work in the real world. Still there is a major shortcoming. We cannot yet grab the proceeds from selling the candies. Before adding that we first have to learn how to secure our components to make methods only callable by specific people. We address this topic in the next chapter.
