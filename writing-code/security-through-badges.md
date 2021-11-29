# Security Through Badges

FIXME: This page needs editing and formatting.

For this example we will add the simplest possible security model using a store owner's badge.

Someone with the store owner's badge will be able to stock and restock the candy store and also claim the collected\_xrd.

One obvious way to set this up is to have the creator of the component also be considered to be the owner of the store and therefore get a store owner's badge.

First let's create the badge.

Add this to the constructor:

```
    let owners_badge = ResourceBuilder::new()
            .metadata("name", "Owner's Badge")
            .new_badge_fixed(1);
```

Also in the struct let's record the badge's resource definition

```
    owners_badge: ResourceDef,
```

This in turn means that we have to set that up when creating the component and so we add this:

```
    owners_badge: owners_badge.resource_def();
```

Finally we need to return this badge when we return the component and that changes the new() function even more.

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

Now let's update the stock\_candy method to require the owner's badge. You do this by adding this macro on the line before the method is defined:

```
        #[auth(owners_badge)]
```

This macro does all of the work to protect this function. The caller must include their badge as an additional last argument which is sent up n a bucketref (which, in resim, is specified in the same manner as a bucket.) Try it with both 1,$badge and 0,$badge and you will see that having the badge address is not sufficient. You actually have to include a badge.

Just one more thing to do now. Let's add a secure function to claim the collected XRD. While we are at it, Let's also track the amount of XRD that has been claimed over the lifetime of the CabdyStore. To do this add this to the struct:

```
    total_claimed: Decimal,
```

and this to the constructor:

```
    total_claimed: 0.0.into(),
```

Again we will secure this function with the auth macro making it a snap to write:

```
    #[auth(owners_badge)]
    pub fn claim (&mut self) -> Bucket {
        self.total_claimed += self.collected_xrd.amount();
        self.collected_xrd.take_all()
    }
```

That's it. The owner can now claim the CandyStore proceeds in a safe manner.
