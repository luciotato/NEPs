# Epoch

Blocks are split into epochs.
Genesis block is in its own epoch. After that, a block is either in its parent's epoch or
starts a new epoch if it meets certain conditions.

## BlockHeight
Each block has a height. Genesis block has height 0, the height of each block is greater than
the height of its parent. Within one epoch, each height has a validator assigned to it as
the block producer.

## Epoch id
Every block stores the id of its epoch - `epoch_id`.

Epoch id is defined as
- For epoch 0 it's `0` (genesis block only)
- For epoch 1 it's `0`
- For epoch `T+2` it's the hash of the last block in epoch `T`

## End of an epoch
A block is defined to be the first block in the new epoch if the following conditions are met:
- There is a final block in its parent's epoch
- `height(block) >= height(first_block_in_parent_epoch) + epoch_length`

When the epoch ends, the validator set rotates and the block in the new epoch is
produced by a validator from the new set. Applying this block, in addition to a normal
transition, also applies validator rewards and per-epoch state changes.

## Fishermen
Which blocks can be challenged?
