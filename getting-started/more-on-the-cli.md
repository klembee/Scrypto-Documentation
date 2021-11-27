# More on the CLI

### Tokens

Let's create a new token from the CLI ! There are two commands to achieve this. `resim new-token-fixed` and `resim new-token-mutable` wether you want to create a token with a fixed supply or one with a mutable supply.&#x20;

To create a Scrypto token with a fixed supply of 1000, you can run the following command:&#x20;

```
resim new-token-fixed --name Scrypto --symbol SCO 1000
```

Now if you run `resim show [account]`, you should see the newly created token in the resources:

```
> resim show 0280ffdb839344b14898b083b43a9286b36f5da6b1c68cbeeb133e # replace with your address
Resources:
└─ { amount: 1000000, resource_def: 030000000000000000000000000000000000000000000000000004, name: "Radix", symbol: "XRD"}
└─ { amount: 1000, resource_def: 036732a7f237e668965b058e2011d372ce03938bd6e6fa1d0fac87, name: "Scrypto", symbol: "SCO"}
```

To create a mutable token, it is a little bit different. You have to specify the address of the account that has the right to mint or burn the tokens:
