---
description: >-
  In this section, we will go over the different commands you can run with the
  resim CLI to deploy and test the HelloWorld example !
---

# Deploying HelloWorld

### Accounts

The first thing to do is to create an account from which you will call the functions and methods. You can achieve this with this command: `resim new-account`. This will return the created account's public key and address. If it is the first time that you call this command, the account will be set as the default one when calling functions and methods.

You can get the resources at a specific account's address with the following command: `resim show [account_address]`:

```
> resim show [account_address]
Resources:
└─ { amount: 1000000, resource_def: 030000000000000000000000000000000000000000000000000004, name: "Radix", symbol: "XRD"}
```

As you can see, when you create a new account, it starts with 1 000 000 XRD to make it easier to test.

### Deploy and test

Now you will learn how to deploy the "Hello World!" code and call its method.

We first need to build and deploy the blueprint:

```
# Inside the helloworld directory
resim publish .
```

The publish command will build the code and deploy it to the ledger. It returns the blueprint's address. Save it somewhere, you will need it in the next step.

Now that the blueprint is on the ledger, we can call its functions. In the HelloWorld blueprint, we specified a function acting as a constructor named `new`. You can call it like this:

```
resim call-function [blueprint_address] Hello new
```

This will output a new ResourceDef's address and the instantiated component's address:

```
New Entities: 2
├─ ResourceDef: 03af54cb529183218ca2485fbc2097a265005924ef64a4bcfe6953
└─ Component: 020e46c3811a8e3c24e1b306599686e7bb2990a5eb24fbf8ce38af
```

The ResourceDef is an address representing the HelloToken that was created.

Now that we have a component, we can call its method `free_token` which should give us one HelloToken:

```
resim call-method [component_address] free_token
```

If you now look at the resources at your account's address, you should see the new token:

```
> resim show [account_address]
Resources:
└─ { amount: 1000000, resource_def: 030000000000000000000000000000000000000000000000000004, name: "Radix", symbol: "XRD"}
└─ { amount: 1, resource_def: 03af54cb529183218ca2485fbc2097a265005924ef64a4bcfe6953, name: "HelloToken", symbol: "HT"}
```

### Conclusion

Good job ! You are now able to write Scrypto code, deploy it and test it using the CLI !
