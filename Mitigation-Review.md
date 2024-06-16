## Code4Rena DYAD Auditing Contest Mitigation Review.

> Auditor: Al-Qa'qa'

- 🟩️ => Issue mitigated without introducing further issues || Issues Acknowledged by the sponsor
- 🟧️ => Issues mitigated but introduced little Impact
- 🟥️ => Issues that were not mitigated by the sponsor, and have a great impact || Issues mitigated but introduced HIGH Impact

### Summary


|ID|Title|Status|
|:-:|----|:----:|
|[H&#8209;01](#h-01-design-flaw-and-mismanagement-in-vault-licensing-leads-to-double-counting-in-collateral-ratios-and-positions-collateralized-entirely-with-kerosine)|Design flaw and mismanagement in vault licensing leads to double counting in collateral ratios and positions collateralized entirely with kerosine|🟩️|
|[H-02](#h-02-inability-to-perform-partial-liquidations-allows-huge-positions-to-accrue-bad-debt-in-the-system)|Inability to perform partial liquidations allows huge positions to accrue bad debt in the system|🟥️|
|[H-03](#h-03-attacker-can-make-0-value-deposit-calls-to-deny-user-from-redeeming-or-withdrawing-collateral)|Attacker can make 0 value deposit() calls to deny user from redeeming or withdrawing collateral|🟩️|
|[H-04](#h-04-attacker-can-frontruns-users-withdrawals-to-make-them-reverts-without-costs)|Attacker Can Frontruns User's Withdrawals To Make Them Reverts Without Costs|🟩️|
|[H-05](#h-05-unable-to-withdraw-kerosene-from-vaultmanagerv2withdraw-as-it-expects-a-vaultoracle-method-which-is-missing-in-kerosene-vaults)|Unable to withdraw Kerosene from `vaultmanagerv2::withdraw` as it expects a `vault.oracle()` method which is missing in Kerosene vaults|🟩️|
|[H-06](#h-06-user-can-get-their-kerosene-stuck-because-of-an-invalid-check-on-withdraw)|User can get their Kerosene stuck because of an invalid check on withdraw|🟩️|
|[H-07](#h-07-missing-enough-exogeneous-collateral-check-in-vaultmanagerv2liquidate-makes-the-liquidation-revert-even-if-dyad-minted--non-kerosene-value)|Missing enough exogeneous collateral check in `VaultManagerV2::liquidate` makes the liquidation revert even if (DYAD Minted > Non Kerosene Value)|🟥️|
|[H-08](#h-08-users-can-get-their-kerosene-stuck-until-tvl-becomes-greater-than-dyads-supply)|Users can get their Kerosene stuck until TVL becomes greater than Dyad's supply|🟧️|
|[H-09](#h-09-kerosene-collateral-is-not-being-moved-on-liquidation-exposing-liquidators-to-loss)|Kerosene collateral is not being moved on liquidation, exposing liquidators to loss|🟩️|
|[H-10](#h-10-flash-loan-protection-mechanism-can-be-bypassed-via-self-liquidations)|Flash loan protection mechanism can be bypassed via self-liquidations|🟩️|
||||
|[M&#8209;01](#m-01-liquidation-bonus-logic-is-wrong)|Liquidation bonus logic is wrong|🟧️|
|[M-02](#m-02-no-incentive-to-liquidate-when-cr--1-as-asset-received--dyad-burned)|No incentive to liquidate when CR <= 1 as asset received < dyad burned|🟩️|
|[M-03](#m-03-setunboundedkerosinevault-not-called-during-deployment-causing-reverts-when-querying-for-kerosene-value-after-adding-it-as-a-kerosene-vault)|setUnboundedKerosineVault not called during deployment, causing reverts when querying for Kerosene value after adding it as a Kerosene vault|🟩️|
|[M-04](#m-04-liquidating-positions-with-bounded-kerosen-could-be-unprofitable-for-liquidators)|Liquidating positions with bounded Kerosen could be unprofitable for liquidators|🟩️|
|[M-05](#m-05-no-incentive-to-liquidate-small-positions-could-result-in-protocol-going-underwater)|No incentive to liquidate small positions could result in protocol going underwater|🟩️|
|[M-06](#m-06-attacker-can-frontrun-to-prevent-vaults-from-being-removed-from-the-dnft-owners-position)|Attacker can frontrun to prevent vaults from being removed from the dNFT owner's position|🟩️|
|[M-07](#m-07-vaultmanagerv2solburndyad-function-is-missing-an-isdnftowner-modifier-allowing-a-user-to-burn-another-users-minted-dyad)|`VaultManagerV2.sol::burnDyad` function is missing an `isDNftOwner` modifier, allowing a user to burn another user's minted DYAD|🟩️|
|[M-08](#m-08-incorrect-deployment--missing-contract-will-break-functionality)|Incorrect deployment / missing contract will break functionality|🟩️|
|[M-09](#m-09-value-of-kerosene-can-be-manipulated-to-force-liquidate-users)|Value of kerosene can be manipulated to force liquidate users|🟩️|

---

### [H-01]: Design flaw and mismanagement in vault licensing leads to double counting in collateral ratios and positions collateralized entirely with kerosine

- Status: 🟩️

**Brief description**

Licensing Vaults were made incorrectly in the deployment Script, where `non-kersone` vaults were added into both `kerosine` and `non-kerosine` vaults, leading to doubling users' `CR` ratio.

**Mitigation**

The Sponsor handled that case in the deployment script, and deployed the protocol successfully.

---

### [H-02]: Inability to perform partial liquidations allows huge positions to accrue bad debt in the system

- Status: 🟥️

**Brief description**

The old liquidate mechanism Forces the liquidator to pay for all the minted DYAD tokens to liquidate the bad position. But this will make Whales positions, which are positions that has huge amounts, hard to get liquidated as not all people will have that amount of `DYAD` to pay.

**Mitigation**

The sponsor allowed partial liquidation by allowing users to put the amount of `DYAD` he will burn and he will partially liquidate the position by the amount he paid.

This led to a problem, where some bad positions (with cr = 0) will be `unliquidatable` in some situations, and I explained it in `H-01` issue.

---

### [H-03]: Attacker can make 0 value deposit() calls to deny user from redeeming or withdrawing collateral

- Status: 🟩️

**Brief description**

To protect against FlashLoans attacks, `deposit()` function implements a check that prevents depositing and withdrawing in the same block. Since the `deposit()` function was allowed to be called by anyone, even ones that do not own the NFT. This allows malicious actors, to front-run any Liquidity provider from withdrawing by just depositing `1 wei` and the withdrawing will get reverted because of the FlashLoan protection check.

**Mitigation**

`deposit()` now is only callable by the owner of the NFT.

--- 

### [H-04]: Attacker Can Frontruns User's Withdrawals To Make Them Reverts Without Costs

- Status: 🟩️

**Brief description**

`deposit/withdraw` functions do not check if the vault provided is a valid and licensed vault or not. This allows malicious actors to prevent withdrawing by just calling the `deposit()` function with the face vault address, preventing the Liquidity provider from withdrawing his money.

**Mitigation**

Although `deposit/withdraw` functions still accept any value of `vault` and do not check if it is licensed or not. The issue is mitigated by only letting the owner of the NFT have the ability to fire `deposit()` to the given `ID`.

---

### [H-05]: Unable to withdraw Kerosene from `vaultmanagerv2::withdraw` as it expects a `vault.oracle()` method which is missing in Kerosene vaults

- Status: 🟩️

**Brief description**

kerosine vaults were not implementing `oracle()` interface, which will make the tx revert if the user tries to withdraw from them.

**Mitigation**

The Sponsor made a specific Oracle contract for kerosine Vaults and used them as the oracle for them.

---

### [H-06]: User can get their Kerosene stuck because of an invalid check on withdraw

- Status: 🟩️

**Brief description**

In the `withdraw()` function, the check for `exogenous` collateral was done by subtracting the amount to be withdrawn from the `non-Kerosine` vaults and checking the `exogenous` ratio. This is not true in all cases, as if the user withdraws from kerosine vaults, this check should not subtract the withdrawn value, as it is `endogenous` collaterals not `exogenous`

**Mitigation**

The sponsor fixed the issue, by reading the values on-chain after withdrawing and checking for `exogenous` collaterals correctly.

---

### [H-07]: Missing enough exogeneous collateral check in `VaultManagerV2::liquidate` makes the liquidation revert even if (DYAD Minted > Non Kerosene Value)

- Status: 🟥️

**Brief description**

The liquidation only occurs if the CR goes below `150%`, However, from the protocol invariants, the position should have at least `100%` of the position ratio with `exogenous` collaterals. If the position has `exogenous` collaterals below `100%` the position will still be `unliquidatable`, which breaks protocol invariants. 

**Mitigation**

The sponsor did not mitigate this issue, and the liquidate function still only allows liquidating if the `CR` goes below `150%`. And the mitigation of issue [`H-09`](#h-09-kerosene-collateral-is-not-being-moved-on-liquidation-exposing-liquidators-to-loss) is independent of this issue, and will not fix it.

- issue link: [here](https://github.com/code-423n4/2024-04-dyad-findings/issues/338)
- Important info: [here](https://github.com/code-423n4/2024-04-dyad-findings/issues/338#issuecomment-2104323958)

---

### [H-08]: Users can get their Kerosene stuck until TVL becomes greater than Dyad's supply

- Status: 🟧️

**Brief description**

The kerosine value is determined by the amount of collaterals in the Vaults and the total Minted DYAD. Since it subtracts them, if the `tvl` is smaller than the `total DYAD minted` the function will revert

**Mitigation**

The sponsor lets the function return `0` (the value of kerosine will be `zero`) if the `tvl` is smaller than `total DYAD minted`.

This is fine, but it introduced an issue. if the `tvl` became less than `minted DYAD` in the process of removing vaults, unsupporting them, or in mitigations. The liquidation process will be DoS'ed as there will be `kerosineVault::assetPrice()` which is returning `0` and we divide by it.

```solidity
      uint asset = cappedValue 
                     * (10**(vault.oracle().decimals() + vault.asset().decimals())) 
@>                   / vault.assetPrice() 
                     / 1e18;
```


---

### [H-09]: Kerosene collateral is not being moved on liquidation, exposing liquidators to loss

- Status: 🟩️

**Brief description**

When liquidating `undercollateralized` positions, only `exogenous` collaterals move to the liquidator without sending `endogenous` collaterals (kerosine).

**Mitigation**

The liquidate function sends all collaterals in both `kerosine/nonkerosine` vaults (`endogenous/exogenous`) to the liquidator.

---

### [H-10] Flash loan protection mechanism can be bypassed via self-liquidations

- Status: 🟩️

**Brief description**

`liquidate()` function was not protected from FlashLoan attacks, which opens up the ability for the user to liquidate himself, or another user tries to liquidate him by manipulating the kerosine price via FlashLoan. 

**Mitigation**

`Liquidate()` function saves the blockId of the receiver `NFT token ID` of the collaterals, which will prevent the attacker from withdrawing from that `ID` in the same block (preventing the FlashLoan attack).

---
---
---

### [M-01]: Liquidation bonus logic is wrong

- Status: 🟧️

**Brief description**

`liquidate()` function should give the liquidator `100%` worth of the DYAD he burned + `20%` bonus, but this was implemented incorrectly.

**Mitigation**

Although the sponsor acknowledges this issue, he chose to mitigate it, and it is mitigated successfully, where the liquidator receives `120%` as `max reward` of the undercollateralized position when getting liquidated.

But there is a little problem, which is the rounding division is not in the favour of the liquidator. The old version rounds `UP` to be in the favour of the liquidator. But in the new logic, it rounds `Down`.

---

### [M-02]: No incentive to liquidate when CR <= 1 as asset received < dyad burned

- Status: 🟩️

**Brief description**

If the position `CR` goes below `100%`, there may be no incentive to liquidate it, as they will not gain any bonus.

**Mitigation**

The sponsor acknowledges the issue. and I agree with him. As even if there is no bonus, the liquidator still gets the position collaterals. and for the DYAD he burned, he can re-mint them since his `CR` ratio will increase when he burns DYAD.

---

### [M-03]: setUnboundedKerosineVault not called during deployment, causing reverts when querying for Kerosene value after adding it as a Kerosene vault

- Status: 🟩️

**Brief description**

`BoundedkerosineVault` depends on `UnBoundedkerosineVault`, but this was not added in the deployment script before transferring the ownership to the Multisig wallet.

**Mitigation**

The issue is Acknowledged by the sponsor, besides that the `UnBoundedKerosineVault` is deprecated now and is not supported.

---

### [M-04]: Liquidating positions with bounded Kerosen could be unprofitable for liquidators

- Status: 🟩️

**Brief description**

In `BoundedkerosineVaults`, the users can not withdraw their funds. so in the liquidation process, the funds will get transferred to the liquidator and will be locked permanently.

**Mitigation**

That the `UnBoundedKerosineVault` is deprecated now and is not supported.

---

### [M-05]: No incentive to liquidate small positions could result in protocol going underwater

- Status: 🟩️

**Brief description**

If the bad position is not worth a lot of money, Liquidating it may cost more than the reward gained by the liquidator, which may keep it `unliquidatable`.

**Mitigation**

The sponsor acknowledged the issue. Although I see that the issue impact is too low, having a minimum amount for depositing is a good approach.

---

### [M-06]: Attacker can frontrun to prevent vaults from being removed from the dNFT owner's position

- Status: 🟩️

**Brief description**

To remove a vault, there should be no funds in it. but since the deposit funds is available for anyone, an attacker can front-run the removing function and prevents the removal of that vault.

**Mitigation**

The issue is mitigated by only letting the owner of the `NFT token` the ability to deposit for this `token ID`.

---

### [M-07]: `VaultManagerV2.sol::burnDyad` function is missing an `isDNftOwner` modifier, allowing a user to burn another user's minted DYAD

- Status: 🟩️

**Brief description**

burnDyad is callable by anyone. so position modification can be made by users who have DYAD not related to that position, which can manipulate redeem/withdraw/liquidate processes, either if is unprofitable to the attacker.

**Mitigation**

The burner of the DYAD should be the owner of the `NFT token ID` now.

---

### [M-08]: Incorrect deployment / missing contract will break functionality

- Status: 🟩️

**Brief description**

Incorrect setting Kerosine vaults in `kerosineManager`.

**Mitigation**

The sponsor fixed the issue and deployed the `KerosineManager` with the correct vaults.

---

### [M-09]: Value of kerosene can be manipulated to force liquidate users

- Status: 🟩️

**Brief description**

Because of the nature of the kerosine Price, which represents the Market Cap of the DYAD total Supply, its price can goes up and down by Whales, leading to positions getting liquidated.

**Mitigation**

The sponsor acknowledged the issue, and this issue is by design, and I do not think it can even be mitigated.
