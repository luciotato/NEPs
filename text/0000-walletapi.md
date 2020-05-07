- Proposal Name: Wallet API
- Start Date: (fill me in with today's date, 2019-07-30)
- NEP PR: [nearprotocol/neps#0000](https://github.com/nearprotocol/NEPs/pull/10)
- Issue(s): n/a

# Summary
[summary]: #summary

We want to have a well-defined API signing transactions to be sent to NEAR protocol via a wallet.
The API needs to support multiple transactions, including but not limited to:
- adding access keys to account
- sending tokens to other accounts
- composite transactions that contain several sub-transactions of potentially different types

In order to support this, we want a generic API for signing arbitrary transactions in the wallet.

# Motivation
[motivation]: #motivation

We want to support multiple wallet implementations, including those by third party developers. We also want to support multiple "styles" of wallets. E.g. web wallets, hardware wallets, browser extensions.
Any specific app may prefer to work with a specific wallet that best suits their use case (security requirements, usability, theme, UX, etc), so there must be a way to configure the app easily to use a particular wallet implementation. We also want a flexible system that allows adding new types of
transactions easily.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

## Login into app with wallet

App requests wallet to login 
`{walletUrl}/login/?publicKey={publicKey}&contractId={contractId}&callback={callbackUrl}`
- `publicKey` optional, base58 encoded public key to use as app specific key. If not specified – login is only choosing `accountId` and not adding access key
- `walletUrl` is a base wallet URL that can be specified when creating wallet connection using nearlib.
- `contractId` optional, account id of the contract app wants to add access key for. Not needed when `publicKey` is not specified. Means app wants unrestricted access (e.g. `near-shell`) otherwise.
- `callbackUrl` callback URL provided by app that gets opened by wallet after flow completion.

When `publicKey` wallet is expected to complete `addKey` transaction to add access key limited to given `contractId`. User can refuse to do it. This is not an error and will mean that callback only sends `accountId` selected by user back to app.

## Sign transaction with Web wallet

App requests wallet to sign transaction by open following URL in the browser:
`{walletUrl}/sign?transactions={transactions}&callback={callbackUrl}`

- `walletUrl` is a base wallet URL that can be specified when creating wallet connection using nearlib.
- `transactions` comma-separatted list of base64-encoded [`Transaction` objects](https://github.com/near/near-api-js/blob/db51150b98f3e55c2893a410ad8e2379c10d8b73/src/transaction.ts#L83) serialized using [Borsh](https://borsh.io). 
- `callbackUrl` callback URL provided by app that gets opened by wallet after flow completion.
- `send` optional, `true` if wallet should send transaction after signing (wallet not required to support this)

### Callback URL

Wallet should open `callbackUrl` provided by app specifying following query paramers.

For error:
- `errorCode` unique identifier string for error
    - TODO: Error codes

For success:
- `accountId` account ID for account used
- `signatures` comma-separatted list of base64-encoded [`Signature` objects](https://github.com/near/near-api-js/blob/db51150b98f3e55c2893a410ad8e2379c10d8b73/src/transaction.ts#L78) serialized using [Borsh](https://borsh.io).
- `sent` indicates whether transaction has been sent, `true` if sent. Wallet that doesn't support sending transactions always returns `false`.

# Drawbacks
[drawbacks]: #drawbacks

TODO

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- Does the wallet always send the transaction directly, or do we need to support a use case of generating a signature and returning it back to the app?
  - Should allow caller to send transaction

- TODO

# Future possibilities
[future-possibilities]: #future-possibilities

- TODO
- Popup?