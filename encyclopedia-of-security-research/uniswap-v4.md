# Uniswap V4

## Edge Cases & Exploit Vectors

- Dust left in Uniswap V4 by the end of the callback will cause a revert
- You could maybe censor async actions by entering a callback on the pool before interacting with the victim protocol
- After a zeroForOne swap the active price can be exactly on a tick, but the activeTick will actually be the previous tick. This merely maintains the invariant that active price is ahead of activeTick, but may cause some issues with protocols — especially for 1 tick spacing pools!
- Sync DoS attack where either native or non-native tokens are donated and not synced, see [L-13 Here](https://github.com/GuardianAudits/Audits/blob/main/GammaStrategies/2025-04-14_Gamma_UniswapV4_LimitOrders.pdf)

## Checklist Items

- Hook functions should be permissioned for only the Uni Pool that uses that hook contract!




## Audit References & Resources

Gamma Uniswap V4 Limit Orders: https://github.com/GuardianAudits/Audits/blob/main/GammaStrategies/2025-04-14_Gamma_UniswapV4_LimitOrders.pdf









