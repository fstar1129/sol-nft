# NFT packs Solana program

Supports creation of "mystery" packages of NFTs that are not revealed until after "opening", similar to the experience you have when purchasing a pack of baseball cards at the store.

## NFT packs actions

- Admin init a pack
    - pack account is PDA with seeds [“nft-pack”, nft_pack_program_id, store, pack_admin_key, pack_name]
    - store address is saved in pack
    - set distribution type(max_supply, weighted, unlimited)
    - set allowed amount to redeem
    - set if it’s mutable
    - set dates(redeem start and end)
- Add cards
    - adding a card means we transfer MasterEdition to program account so we are able to mint Edition once user open a pack
    - every card account is PDA with seeds [pack_key, "card", index]
- Add voucher
    - save MasterEdition data(keys) so we can match Editions with this Master when users will open a pack
    - pack can have multiple different vouchers and every voucher has the same value and gives users the same amounts of cards from the pack
    - voucher is Edition in terms of Metaplex but in terms of nft-packs program it's PDA account with seeds [pack_key, "voucher", index] which stores some data
    - we can add only voucher which we are own
    - to sum up, when we add voucher to the pack we save MasterEdition key to the pack and every user who has Edition from that MasterEdition owns a voucher for created pack and can open it
- Activate
    - in activated state admin can't change any pack data
    - users can start to open a pack (using `RequestCardForRedeem` and `ClaimPack` methods)
- Deactivate
    - when pack is deactivated users can't interact with it and admin can change data
- CleanUp
    - sort weights Vec which is stored in PackConfig account
- Request card for redeem
    - user calls this instruction to receive index of card which he can redeem
    - program burns user's voucher token account
    - program is using RandomOracle program to count probability to decide which card user will receive
    - probability is calculating using weighted list from PackConfig account
    - index of next card to redeem is written to ProvingProcess account
    - ProvingProcess is a PDA account with seeds [pack, "proving", voucher_mint_key]
    - once user call this instruction weights Vec should be sorted with `CleanUp` instruction
- Claim
    - user call this instruction after they receive a card index from `Request card for redeem`
    - program mints new Edition to user wallet
- Edit pack
    - can be called only if pack is in deactivated state
    - allows changing pack `name`, `description`, `URI`(pack wallpaper) and `mutable` fields
- Close pack
    - can be called at any time if pack doesn't have redeem end date and if it has only after redeem end date
    - if admin tries to call this instruction before redeem end date program will return `EndDateNotArrived` error
    - irreversible pack state changing
- Delete card
    - cards can be deleted only if pack is in closed state
    - deleting cards means transferring MasterEdition back to the admin, zeroing PackCard account and emptying the card balance
- Delete voucher
    - vouchers can be deleted only if pack is in closed state
    - empty the balance
- Delete pack
    - pack can be deleted only when all the cards and vouchers were deleted
    - empty the balance
