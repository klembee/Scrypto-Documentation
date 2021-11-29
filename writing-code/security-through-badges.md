# Security Through Badges

For this example we will add the simplest possible security model using a store owner's badge. With this badge the candy store owner will be able to stock and restock the candy store and also claim the collected\_xrd.

One obvious way to set this up is to have the creator of the component also be considered to be the owner of the store and therefore get a store owner's badge.

#### Creating the Badge

First let's create the badge by adding this to the constructor:

```
    let owners_badge = ResourceBuilder::new()
            .metadata("name", "Owner's Badge")
            .new_badge_fixed(1);
```

Also in the Candy Store's struct let's save the badge's resource definition ...

```
    owners_badge: ResourceDef,
```

... and set that up when creating the component like this:

```
    owners_badge: owners_badge.resource_def();
```

#### Delivering the Badge

We need to return this badge in the constructor along with the component and that changes the `new()` function's definition and requires a little refactoring.

Here is the completely revised struct and new().

```
struct CandyStore {
    candy_vaults: HashMap<Address, Vault>,
    collected_xrd: Vault,
    prices: HashMap<Address, Decimal>,
    owners_badge: ResourceDef,
}

impl CandyStore {
    // given a price in XRD, creates a ready-to-use gumball machine
    pub fn new() -> (Bucket, Component) {
        let badge_bucket = ResourceBuilder::new()
            .metadata("name", "Stpre Owner's Badge")
            .new_badge_fixed(1);
        let component = Self {
            candy_vaults: HashMap::new(),
            collected_xrd: Vault::new(RADIX_TOKEN),
            prices: HashMap::new(),
            owners_badge: badge_bucket.resource_def(),
        }
        .instantiate();
        (badge_bucket, component)
    }
```

#### Securing the stock\_candy Method

Now let's update the stock\_candy method to require the owner's badge. You do this by adding this macro on the line before the method is defined:

```
        #[auth(owners_badge)]
```

This macro does all of the work to protect this function. The caller must include their badge as an additional last argument which is sent up in a `BucketRef` (which, in `resim`, is specified in the same manner as a `Bucket`.) Try it with both 1,$badge and 0,$badge and you will see that having the badge address is not sufficient. You actually have to include the badge to pass muster with the auth macro.

#### Claiming the Proceeds

There is just one more thing to do now. Let's add a secure function to claim the collected XRD. While we are at it, let's also track the amount of XRD that has been claimed over the lifetime of the CandyStore. To do this add this to the CandyStore struct:

```
    total_claimed: Decimal,
```

and this to the constructor:

```
    total_claimed: 0.0.into(),
```

Again we will secure this new `claim` function with the `auth` macro making it a snap to write:

```
    #[auth(owners_badge)]
    pub fn claim (&mut self) -> Bucket {
        self.total_claimed += self.collected_xrd.amount();
        self.collected_xrd.take_all()
    }
```

That's it. The owner can now claim the CandyStore proceeds in a safe manner.

To see the final version of CandyStore along with all of the other code presented in this tutorial, visit the Scrypto Tutorial repository that is maintained by the Radix Programmer's Guild at their RadGuild home on GitHub.
