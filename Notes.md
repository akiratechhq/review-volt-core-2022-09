 +  Core (ICore, Permissions, Initializable)
    - [x] [Ext] init #
       - modifiers: initializer
       - [x] Why upgradable? Could make `volt` immutable if not upgradable.
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
    - [x] [Int] _allocate #
       - Not clear where this method is used. In PegStabilityModule (?).
    - [x] [Int] _setCeilingBasisPoints #
        - [x] The check that ceiling is != 0 is not needed.
    - [x] [Int] _setFloorBasisPoints #
    - [x] [Int] _validPrice
        - [x] Consider using `greaterThanOrEqualTo`/`lessThanOrEqualTo`. Will reduce awkward floor/ceiling values like `0.079999999999`.
    - [x] [Int] _validatePriceRange

 +  L2Core (ICore, Permissions)
    - [x] [Pub] <Constructor> #
        - [x] Compare `Core.init` with `L2Core.constructor` and find out why `volt` is `immutable` in this case.
    - [x] [Ext] setVcon #
       - modifiers: onlyGovernor

 +  VoltSystemOracle (IVoltSystemOracle)
    - [x] [Pub] <Constructor> #
    - [x] [Pub] getCurrentOraclePrice
       - [x] Check this math
    - [ ] [Ext] compoundInterest #
       - [ ] Can cache value of `getCurrentOraclePrice()` to save gas; useful when emitting the event.
       - [ ] Can use value of `periodEndTime` instead of `periodStartTime` to save gas; useful when emitting the event.

    - [x] Why is timeframe 30.42 days? Can this be exploited?
          365 / 12 = 30.42

 +  OraclePassThrough (IOraclePassThrough, Ownable)
    - [x] [Pub] <Constructor> #
       - modifiers: Ownable
    - [x] [Pub] update #
    - [x] [Ext] read
       - [x] Have to understand the `.div(1e18)` part.
       - [x] Consider having `try/catch`. Might block system.
    - [x] [Ext] getCurrentOraclePrice
        - [x] Why does it exist?
              Historical reasons.
    - [x] [Ext] currPegPrice
        - [x] Why does it exist?
              Historical reasons.
    - [x] [Ext] updateScalingPriceOracle #
       - modifiers: onlyOwner
       - [x] Should this force an update? Can someone backrun the oracle update without a price change? 
             Not really, value not cached.

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
    - [x] [Ext] withdrawAllToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
       - [x] Make sure `.balance()` reports a correct value. Are there any burned tokens on tranfser? Any conversion not linear?
    - [x] [Ext] withdrawERC20ToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
    - [x] [Ext] withdrawAllERC20ToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
       - [x] Make sure `.balance()` reports a correct value. Are there any burned tokens on tranfser? Any conversion not linear?
       - [x] Why is it allowed to withdraw ANY erc20 token?
    - [x] [Int] _withdrawToSafeAddress #
       - Unpauses `PCVDeposit` if it's paused to allow withdraw.
    - [x] [Int] _withdrawERC20ToSafeAddress #
       - [x] Handles ANY erc20 token. Is this safe?
         Doesn't do any accounting, so it's safe.
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
    - [x] [Int] _initialize #
    - [x] [Ext] setContractAdminRole #
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
    - [x] [Int] _setContractAdminRole #

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
      - [x] Will lose precision on scaling down.
    - [x] [Int] _setOracle #
    - [x] [Int] _setBackupOracle #
    - [ ] [Int] _setDoInvert #
       - [ ] Confuses me since seems to work in tandem with `decimalsNormalizer`
    - [x] [Int] _setDecimalsNormalizer #
    - [x] [Int] _setDecimalsNormalizerFromToken #
       - [ ] Not used. Remove?

    - [ ] Discuss invert, backup oracle.
    

 +  ERC20CompoundPCVDeposit (CompoundPCVDepositBase)
    - [x] [Pub] <Constructor> #
       - modifiers: CompoundPCVDepositBase
    - [x] [Ext] deposit #
       - modifiers: whenNotPaused
       - [x] Don't get why approve and mint. Why not transferFrom or burn? Why is approve needed?
       - [x] Something is weird in here
    - [x] [Int] _transferUnderlying #
       - Not used. Remove?
    - [x] [Pub] balanceReportedIn
       - [x] Is this really connected to `CompoundPCVDepositBase.balance()`? It looks like it's not strongly connected.

 +  ERC20Allocator (IERC20Allocator, CoreRef, RateLimitedV2)
    - [x] [Pub] <Constructor> #
       - modifiers: CoreRef,RateLimitedV2
    - [x] [Ext] connectPSM #
       - modifiers: onlyGovernor
    - [x] [Ext] editPSMTargetBalance #
       - modifiers: onlyGovernor
    - [x] [Ext] disconnectPSM #
       - modifiers: onlyGovernor
    - [x] [Ext] connectDeposit #
       - modifiers: onlyGovernor
    - [x] [Ext] deleteDeposit #
       - modifiers: onlyGovernor
    - [x] [Ext] sweep #
       - modifiers: onlyGovernor
    - [x] [Ext] skim #
       - modifiers: whenNotPaused
    - [x] [Int] _skim #
    - [x] [Ext] drip #
       - modifiers: whenNotPaused
    - [x] [Int] _drip #
    - [x] [Ext] doAction #
       - modifiers: whenNotPaused
    - [x] [Ext] targetBalance
    - [x] [Pub] getAdjustedAmount
    - [x] [Pub] getSkimDetails
       - [ ] Check this again
    - [x] [Pub] getDripDetails
    - [x] [Ext] checkDripCondition
    - [x] [Ext] checkSkimCondition
    - [x] [Ext] checkActionAllowed
    - [x] [Int] _checkDripCondition
      - [x] Is `.balance()` always in sync with `.balanceOf()`?
    - [x] [Int] _checkSkimCondition
      - [x] Is `.balance()` always in sync with `.balanceOf()`?



 ($) = payable function
 # = non-constant function
  



 ($) = payable function
 # = non-constant function
  