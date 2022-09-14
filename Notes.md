 +  Core (ICore, Permissions, Initializable)
    - [ ] [Ext] init #
       - modifiers: initializer
       - [ ] Why upgradable? Could make `volt` immutable if not upgradable.
    - [x] [Ext] setVcon #
       - modifiers: onlyGovernor

 +  PriceBoundPSM (PegStabilityModule, IPriceBound)
    - [x] [Pub] <Constructor> #
       - modifiers: PegStabilityModule
    - [x] [Ext] setOracleFloorBasisPoints #
       - modifiers: onlyGovernorOrAdmin
    - [x] [Ext] setOracleCeilingBasisPoints #
       - modifiers: onlyGovernorOrAdmin
    - [x] [Ext] isPriceValid
    - [ ] [Int] _allocate #
       - Not clear where this method is used.
    - [x] [Int] _setCeilingBasisPoints #
        - [x] The check that ceiling is != 0 is not needed.
    - [x] [Int] _setFloorBasisPoints #
    - [ ] [Int] _validPrice
        - [x] Consider using `greaterThanOrEqualTo`/`lessThanOrEqualTo`. Will reduce awkward floor/ceiling values like `0.079999999999`.
    - [x] [Int] _validatePriceRange

 +  L2Core (ICore, Permissions)
    - [ ] [Pub] <Constructor> #
        - [ ] Compare `Core.init` with `L2Core.constructor` and find out why `volt` is `immutable` in this case.
    - [x] [Ext] setVcon #
       - modifiers: onlyGovernor

 +  VoltSystemOracle (IVoltSystemOracle)
    - [x] [Pub] <Constructor> #
    - [ ] [Pub] getCurrentOraclePrice
       - [ ] Check this math
    - [ ] [Ext] compoundInterest #
       - [ ] Can cache value of `getCurrentOraclePrice()` to save gas; useful when emitting the event.
       - [ ] Can use value of `periodEndTime` instead of `periodStartTime` to save gas; useful when emitting the event.

    - [ ] Why is timeframe 30.42 days? Can this be exploited?

 +  OraclePassThrough (IOraclePassThrough, Ownable)
    - [x] [Pub] <Constructor> #
       - modifiers: Ownable
    - [x] [Pub] update #
    - [ ] [Ext] read
       - [ ] Have to understand the `.div(1e18)` part.
       - [ ] Consider having `try/catch`. Might block system.
    - [x] [Ext] getCurrentOraclePrice
        - [ ] Why does it exist?
    - [x] [Ext] currPegPrice
        - [ ] Why does it exist?
    - [ ] [Ext] updateScalingPriceOracle #
       - modifiers: onlyOwner
       - [ ] Should this force an update? Can someone backrun the oracle update without a price change?

 +  PCVGuardian (IPCVGuardian, CoreRef)
    - [x] [Pub] <Constructor> #
       - modifiers: CoreRef
    - [x] [Pub] isWhitelistAddress
    - [x] [Pub] getWhitelistAddresses
    - [x] [Ext] addWhitelistAddress #
       - modifiers: onlyGovernor
    - [x] [Ext] addWhitelistAddresses #
       - modifiers: onlyGovernor
    - [x] [Ext] removeWhitelistAddress #
       - modifiers: onlyGuardianOrGovernor
    - [x] [Ext] removeWhitelistAddresses #
       - modifiers: onlyGuardianOrGovernor
    - [x] [Ext] withdrawToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
    - [ ] [Ext] withdrawAllToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
       - [ ] Make sure `.balance()` reports a correct value. Are there any burned tokens on tranfser? Any conversion not linear?
    - [ ] [Ext] withdrawERC20ToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
    - [ ] [Ext] withdrawAllERC20ToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
       - [ ] Make sure `.balance()` reports a correct value. Are there any burned tokens on tranfser? Any conversion not linear?
       - [ ] Why is it allowed to withdraw ANY erc20 token?
    - [ ] [Int] _withdrawToSafeAddress #
       - Unpauses `PCVDeposit` if it's paused to allow withdraw.
    - [ ] [Int] _withdrawERC20ToSafeAddress #
       - [ ] Handles ANY erc20 token. Is this safe?
    - [x] [Int] _addWhitelistAddress #
    - [x] [Int] _removeWhitelistAddress #

   - [ ] PausableLib is a bit awkward. Why not use an interface?

 +  PCVGuardAdmin (IPCVGuardAdmin, CoreRef)
    - [x] [Pub] <Constructor> #
       - modifiers: CoreRef
    - [x] [Ext] grantPCVGuardRole #
       - modifiers: onlyGovernor
    - [x] [Ext] revokePCVGuardRole #
       - modifiers: onlyGuardianOrGovernor

 +  CoreRef (ICoreRef, Pausable)
    - [x] [Pub] <Constructor> #
    - [ ] [Int] _initialize #
    - [ ] [Ext] setContractAdminRole #
       - modifiers: onlyGovernor
    - [x] [Pub] isContractAdmin
    - [x] [Pub] pause #
       - modifiers: onlyGuardianOrGovernor
    - [x] [Pub] unpause #
       - modifiers: onlyGuardianOrGovernor
    - [x] [Pub] core
    - [x] [Pub] volt
    - [x] [Pub] vcon
    - [x] [Pub] voltBalance
    - [x] [Pub] vconBalance
    - [ ] [Int] _burnVoltHeld #
      - [ ] Not used. Remove?
    - [ ] [Int] _mintVolt #
      - [ ] Not used. Remove?
    - [ ] [Int] _setContractAdminRole #

    - [ ] `hasAnyOf{number}Roles` could be replaced with an array of roles.

 +  OracleRef (IOracleRef, CoreRef)
    - [ ] [Pub] <Constructor> #
       - modifiers: CoreRef
    - [x] [Ext] setOracle #
       - modifiers: onlyGovernor
    - [x] [Ext] setDoInvert #
       - modifiers: onlyGovernor
       - [ ] Add comment saying that decimals normalizer is updated.
    - [x] [Ext] setDecimalsNormalizer #
       - modifiers: onlyGovernor
    - [x] [Ext] setBackupOracle #
       - modifiers: onlyGovernorOrAdmin
       - [ ] Should check `_oracle != _backupOracle`.
    - [x] [Pub] invert
    - [x] [Pub] updateOracle #
    - [ ] [Pub] readOracle
      - [ ] Consider `try/catch` in `readOracle`.
      - [ ] Will lose precision on scaling down.
    - [x] [Int] _setOracle #
    - [x] [Int] _setBackupOracle #
    - [ ] [Int] _setDoInvert #
       - [ ] Confuses me since seems to work in tandem with `decimalsNormalizer`
    - [x] [Int] _setDecimalsNormalizer #
    - [x] [Int] _setDecimalsNormalizerFromToken #
       - [ ] Not used. Remove?

    - [ ] Discuss invert, backup oracle.
    

 +  ERC20CompoundPCVDeposit (CompoundPCVDepositBase)
    - [ ] [Pub] <Constructor> #
       - modifiers: CompoundPCVDepositBase
    - [ ] [Ext] deposit #
       - modifiers: whenNotPaused
       - [ ] Don't get why approve and mint. Why not transferFrom or burn? Why is approve needed?
       - [ ] Something is weird in here
    - [ ] [Int] _transferUnderlying #
       - Not used. Remove?
    - [ ] [Pub] balanceReportedIn
       - [ ] Is this really connected to `CompoundPCVDepositBase.balance()`? It looks like it's not strongly connected.

 ($) = payable function
 # = non-constant function
  