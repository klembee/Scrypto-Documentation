# Scrypto Types

### Vaults

In Scrypto, vaults are used to hold resources. Vaults are initiated with a resource definition and can only hold tokens of that particular resource definition. Here is how you instantiate a vault:&#x20;

`Vault::new(Address)`

### Bucket

Buckets are a way to move resources throughout a transaction. At the end of a transactions, all resources inside buckets must be moved into a vault or burned. This is to make sure no tokens ever gets lost in the cryptosphere.

You can take resources from a vault and put them in a bucket like this:

`let bucket = vault.take(amount);`

`let bigger_bucket = vault.take_all();`

### BucketRef

BucketRefs are used when you write a method that could accept a bucket but you don't need to take ownership over that bucket. For example if you are only looking at the quantity or resource definition of tokens inside the bucket you can define the method argument as BucketRef instead of Bucket. This makes sure that you will not be able to store the tokens in your vault or send them to another account.

Example of a method accepting a bucket\_ref:

```rust
pub fn steal_tokens(&mut self, bucket: BucketRef) {
    // No errors because we are only reading a value
    info!("Amount shown: {}", bucket.amount()); 
    
    // Error ! We are trying to take ownership over the BucketRef
    self.secret_vault.put(bucket);
    bucket.drop();
}
```

You can see that we call `bucket.drop()` at the end of the method. It is required to drop the bucket refs that you receive or else you will get a `ResourceCheckFailure` when calling the method.

### Account



### Component



