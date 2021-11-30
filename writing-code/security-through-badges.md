# Security Through Badges

For this example we will add the simplest possible security model using a store owner's badge. With this badge the candy store owner will be able to stock and restock the candy store and also claim the `collected_xrd`.

One obvious way to set this up is to have the creator of the component also be considered to be the owner of the store and therefore give them a store owner's badge.

#### Creating the Badge

First let's create the badge by adding this to the constructor:

```rust
let badge_bucket = ResourceBuilder::new()
        .metadata("name", "Store Owner's Badge")
        .new_badge_fixed(1);
```

Also in the Candy Store's struct let's save the badge's resource definition ...

```rust
owners_badge: ResourceDef,
```

... and set that up when creating the component like this:

```rust
Self {
    candy_vaults: HashMap::new(),
    collected_xrd: Vault::new(RADIX_TOKEN),
    prices: HashMap::new(),
    owners_badge: badge_bucket.resource_def();
}
.instantiate()
```

#### Delivering the Badge

We need to change the `new()` function to return the created badge along with the component.

Here is the completely revised struct and new().

```rust
struct CandyStore {
    candy_vaults: HashMap<Address, Vault>,
    collected_xrd: Vault,
    prices: HashMap<Address, Decimal>,
    owners_badge: ResourceDef,
}

impl CandyStore {

    pub fn new() -> (Bucket, Component) {
        // Create the badge
        let badge_bucket = ResourceBuilder::new()
            .metadata("name", "Store Owner's Badge")
            .new_badge_fixed(1);

        let component = Self {
            candy_vaults: HashMap::new(),
            collected_xrd: Vault::new(RADIX_TOKEN),
            prices: HashMap::new(),
            owners_badge: badge_bucket.resource_def(),
        }
        .instantiate();

        // Return the badge and the component
        (badge_bucket, component)
    }
```

#### Securing the stock_candy Method

Now let's update the `stock_candy` method to require the owner's badge when calling it. You do this by adding this macro on the line before the method's definition:

```rust
#[auth(owners_badge)]
pub fn stock_candy(&mut self, candy: Bucket, new_price: Decimal) {
```

This macro does all of the work to protect the method. The caller must include their badge as an additional last argument which is sent up in a `BucketRef` (which, in `resim`, is specified in the same manner as a `Bucket`.) Try it with both `1,[badge_address]` and `0,[badge_address]`. You will see that having the badge address is not sufficient. You actually have to provide a quantity of least one.

#### Claiming the Proceeds

There is just one more thing to do now. Let's add a secure method allowing the owner to claim the collected XRD. While we are at it, let's also track the amount of XRD that has been claimed over the lifetime of the CandyStore. To do this add this to the CandyStore struct:

```rust
total_claimed: Decimal,
```

and this to the constructor:

```rust
let component = Self {
    candy_vaults: HashMap::new(),
    collected_xrd: Vault::new(RADIX_TOKEN),
    prices: HashMap::new(),
    owners_badge: badge_bucket.resource_def(),
    // Add this line !
    total_claimed: 0.into(),
}
.instantiate();
```

Again we will secure this new `claim` method with the `auth` macro making it a snap to write:

```rust
#[auth(owners_badge)]
pub fn claim (&mut self) -> Bucket {
    self.total_claimed += self.collected_xrd.amount();
    self.collected_xrd.take_all()
}
```

That's it. The owner can now claim the CandyStore proceeds in a safe manner.

To see the final version of CandyStore along with all of the other code presented in this tutorial, visit the Scrypto Tutorial repository that is maintained by the Radix Programmer's Guild at their RadGuild home on GitHub.
