# Mina zkApp: Fund pool

Fund pool is a zkApp that accepts deposits of a custom token, but only if you can prove you're a part of a whitelist. Deposits are tracked in an off-chain merkle tree, for each user/address respectively. Whitelist membership is proven via a recursive proof, and the proof is validated through it's public input via a smart contract on-chain state precondition (whitelist tree root hash matches on-chain commitment).

## Keys used:

```
FundPool: B62qijxqZ9mcYF7djeyJ4Uvku8y1MxrS1c7zGfhKLaZfTdd82XKzWAP
```

## Approximate test running times (M1 Max)

| zkApp            | Compile    | Without proofs | With proofs | Berkeley  |
| ---------------- | ---------- | -------------- | ----------- | --------- |
| Token            | 18s        | -              | -           | -         |
| FundPool         | 3min       | 70s            | 7min        | -         |
| WhitelistProgram | 27s        | -              | 60s         | -         |
| **Combined**     | **3-4min** | -              | **8min**    | **25min** |

## How to create a Mina account

For the purpose of the _zkApps e2e testing_ program, this project comes with a pre-funded account in `keys/berkeley.json`. However, if you wish to generate a new account and request funds from a faucet, please run:

```sh
npm run init:account
```

The script creates a Mina account and requests a small amount of Mina from the faucet on Berkeley, and then writes the private and public key to `keys/berkeley.json`.

## How to build

```sh
npm run build
```

## How to run tests locally

```sh
npm run test
npm run testw # watch mode
```

## How to run tests on Berkeley

```sh
npm run test:berkeley
# or edit .env file and set TEST_ON_BERKELEY=true
npm run test # with edited .env
```

## Surface / Testing areas

1. Recursion

The FundPool contract only allows deposits as long as a valid proof of being whitelisted is submitted.

2. Call stack composability

The FundPool contract transfers tokens through `Token.transfer()`, from the user to the FundPool's token holder account.

3. Actions

Every deposit it accompanied with a `DepositAction`, which is subsequently processed during `FundPool.rollup()`

4. Events

Rolled up deposit actions update a tree of deposits, and emit each write to the deposits tree through events, with the help of the ZKFS offchain storage library.

5. Preconditions (account)

FundPool stores a `whitelistRoot` as on-chain state, when the `FundPool.deposit()` is proven locally, a precondition for the `whitelistRoot` to match the on-chain state is applied.

Additionally, `provedState` is used to prevent changing of

6. Preconditions (network)

The FundPool allows deposits only during a specific time window, which is enforced based on a blockheight / chainlength range (from - to blockchainLength). A precondition is used as `network.blockchainLength.assertBetween`.

7. Permissions

FundPool's state is only editable with proofs, a signature authorization is not enough. This is tested by manually injecting an account update signed by the zkApp, but not generated by a provable method.

8. Deploy Smart Contract

The FundPool smart contract is deployed either on LocalBlockchain or the Berkeley testnet respectively.

9. Tokens

The FundPool smart contract accepts deposits of a custom token, not MINA. The token contract comes from an external repository `stove-labs/mip-token-standard`.

## License

[Apache-2.0](LICENSE)