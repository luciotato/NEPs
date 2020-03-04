## Calculating rewards 

Validator rewards are assigned once per epoch.
- Total reward for the epoch is at most the sum of gas and storage rent paid.
- A percentage of the reward goes to the treasury account.
- The reward is split in proportion to stake and adjusted for each validator depending on
its number of missed blocks and chunks.
- Extra inflation happens in some way (TODO)

### Parameters
Rewards calculator uses the following parameters:
```
   pub max_inflation_rate: u8,
   pub num_blocks_per_year: u64,
   pub epoch_length: u64,
   pub validator_reward_percentage: u8,
   pub protocol_reward_percentage: u8,
   pub protocol_treasury_account: AccountId,
```

The following values for the epoch:
```
    total_storage_rent: Balance,
    total_validator_reward: Balance,
    total_supply: Balance,
```

And values per each validator:
```
    stake
    blocks_expected
    blocks_produced
    chunks_expected
    chunks_produced
```

### Inflation

```
max_inflation = (max_inflation_rate * total_supply * epoch_length) / (100 * num_blocks_per_year)

epoch_fee = total_validator_reward + total_storage_rent

epoch_total_reward = max(max_inflation, epoch_fee)

inflation = epoch_total_reward - epoch_fee
```
The value of `inflation` is propagated and stored (TODO).

### Treasury

```
epoch_protocol_treasury = epoch_total_reward * protocol_reward_percentage / 100

epoch_validator_reward = epoch_total_reward - epoch_protocol_treasury
```

Treasury account gets `epoch_protocol_treasury`.

### Validator rewards

Only the validators who've met the threshold on blocks and chunks get rewarded.
`total_stake` is the total stake of validators in the epoch (including slashed and offline).

The reward is computed based on `stake`, `blocks_produced`, `blocks_expected`, `chunks_produced`, `chunks_expected`
 (0 if either of `blocks_expected` or `chunks_expected` is zero):
```
reward = epoch_validator_reward * stake * (blocks_produced * chunks_expected + chunks_produced * blocks_expected)
/ (total_stake * blocks_expected * chunks_expected * 2)
```
Note that the sum of all rewards is always less than or equal to `epoch_total_reward`.
