

## Epochs
## Finalizing an epoch

Processing the last block of an epoch has the following additional steps:
1. Aggregate the following changes for the blocks in the epoch:
    1. Collect validator stake proposals. If a validator has multiple, use the last one.
    2. Collect validators that got slashed in this epoch.
    3. Compute the number of blocks/chunks produced by each validator.
    4. Compute the kickout set (see validator kickout section).
    5. Aggregate storage rent and validator reward for the epoch.
2. Calculate validator rewards and `inflation` value for the epoch (see RewardsCalculator section)
3. Assign seats for the next next epoch (T+2).

## Validator kickout
Config has `block_producer_kickout_threshold` and `chunk_producer_kickout_threshold`.
The idea is to kickout validators which missed too many blocks or chunks.

The set of validators to kickout is computed the following way:
1. Slashed validators are kicked out (in the epoch or the epoch before? TODO)
2. Validators and fishermen who remove their stake in the epoch are kicked out
3. A validator is kicked out if the percentage of blocks or chunks produced is below threshold
    - Exception: If all validators are either previously kicked out or to be kicked out, we don't kick out the
validator with the maximum number of blocks produced. If there are multiple, we choose the one with
lowest validator id in the epoch.

## Validator assignment

Validators for epoch `T+2` are assigned during finalization of epoch `T`.
Assignment depends on the following parameters:
- Current set of validators and fishermen
- `proposals`: staking proposals made in the epoch
- `validator_kickout`: kickout set
- `rng_seed`: seed that will be used for shuffling

1. Adjust stake for all current validators and proposals
    - If account is kicked out, change stake to 0
    - If account has a proposal, change stake to the proposal value
    - For current validators without a proposal, keep the stake
2. For current validators not in kickout set, add their reward to the stake
3. Form the set of proposals from the adjusted stakes.
4. Find the validator threshold
    - Number of seats each validator gets is `floor(stake/threshold)`
    - Threshold is the highest number such that the total number of seats is at least `num_block_producer_seats + num_hidden_validator_seats`
    - If the total stake is too low, we error with `ThresholdError` and abort ?! TODO
5. Assign roles for proposals
    - Proposals that meet validator threshold are selected for validator shuffling
    - Remaining proposals that meet fishermen threshold become fishermen
    - Remaining proposals have their stake returned
6. Validator shuffling
    - Validators with stake above threshold are duplicated by their number of seats and
     sorted by `account_id`
    - The array is shuffled using `SeedableRng` with `rng_seed`
    - First `num_block_producer_seats` array elements are chosen as the `block_producers_settlement` array.
    - `chunk_producers_settlement` containing seat assignment for shards is formed from `block_producers_settlement`
7. Accounts who didn't get any seats after shuffling either become fishermen or have their stake returned.
