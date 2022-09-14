<div id="splash">
    <div id="project">
          <span class="splash-title">
               Project
          </span>
          <br />
          <span id="project-value">
               Project name
          </span>
    </div>
     <div id="details">
          <div id="left">
               <span class="splash-title">
                    Client
               </span>
               <br />
               <span class="details-value">
                    Client name
               </span>
               <br />
               <span class="splash-title">
                    Date
               </span>
               <br />
               <span class="details-value">
                    September 2022
               </span>
          </div>
          <div id="right">
               <span class="splash-title">
                    Reviewers
               </span>
               <br />
               <span class="details-value">
                    Daniel Luca
               </span><br />
               <span class="contact">@cleanunicorn</span>
               <br />
               <span class="details-value">
                    Andrei Simion
               </span><br />
               <span class="contact">@andreiashu</span>
          </div>
    </div>
</div>


## Table of Contents
 - [Details](#details)
 - [Issues Summary](#issues-summary)
 - [Executive summary](#executive-summary)
     - [Week 1](#week-1)
     - [Week 2](#week-2)
 - [Scope](#scope)
 - [Issues](#issues)
     - [Users can lose funds if they call ERC20CompoundPCVDeposit.deposit() directly](#users-can-lose-funds-if-they-call-erc20compoundpcvdepositdeposit-directly)
     - [_validPrice can include floor and ceiling values when checking the validity](#_validprice-can-include-floor-and-ceiling-values-when-checking-the-validity)
     - [Setting ceiling basis points does not need a non zero validation](#setting-ceiling-basis-points-does-not-need-a-non-zero-validation)
 - [Artifacts](#artifacts)
     - [Surya](#surya)
     - [Coverage](#coverage)
     - [Tests](#tests)
 - [License](#license)


## Details

- **Client** Client name
- **Date** September 2022
- **Lead reviewer** Daniel Luca ([@cleanunicorn](https://twitter.com/cleanunicorn))
- **Reviewers** Daniel Luca ([@cleanunicorn](https://twitter.com/cleanunicorn)), Andrei Simion ([@andreiashu](https://twitter.com/andreiashu))
- **Repository**: [Project name](https://github.com/volt-protocol/volt-protocol-core.git)
- **Commit hash** `86868c6f77fca67ed6586674437cc9bfc85c1963`
- **Technologies**
  - Solidity
  - Node.JS

## Issues Summary

| SEVERITY       |    OPEN    |    CLOSED    |
|----------------|:----------:|:------------:|
|  Informational  |  0  |  0  |
|  Minor  |  2  |  0  |
|  Medium  |  0  |  0  |
|  Major  |  1  |  0  |

## Executive summary

This report represents the results of the engagement with **Client name** to review **Project name**.

The review was conducted over the course of **2 weeks** from **October 15 to November 15, 2020**. A total of **5 person-days** were spent reviewing the code.

### Week 1

During the first week, we ...

### Week 2

The second week was ...

## Scope

The initial review focused on the [Project name](https://github.com/volt-protocol/volt-protocol-core.git) repository, identified by the commit hash `86868c6f77fca67ed6586674437cc9bfc85c1963`. ...

<!-- We focused on manually reviewing the codebase, searching for security issues such as, but not limited to, re-entrancy problems, transaction ordering, block timestamp dependency, exception handling, call stack depth limitation, integer overflow/underflow, self-destructible contracts, unsecured balance, use of origin, costly gas patterns, architectural problems, code readability. -->

**Includes:**
- PriceBoundPSM
- Core
- L2Core
- VoltSystemOracle
- OraclePassThrough
- TimelockController - from OpenZeppelin
- PCVGuardian
- PCVGuardAdmin
- CoreRef
- OracleRef
- ERC20CompoundPCVDeposit
- ERC20Allocator - was not found in the repository

Documents:

- [Market Governance Whitepaper](https://docs.google.com/document/d/1eSlmC_010gedoYoxN702lDU_2Y6atMfuOnvXxsITcOs/edit)
- [Market Governance Alpha](https://docs.google.com/document/d/1wmK0esBaIxHx_iDXXHWjMo7RaELo3RVBjImhMdefyJg/edit)



## Issues


### [Users can lose funds if they call `ERC20CompoundPCVDeposit.deposit()` directly](https://github.com/akiratechhq/review-volt-core-2022-09/issues/4)
![Issue status: Open](https://img.shields.io/static/v1?label=Status&message=Open&color=5856D6&style=flat-square) ![Major](https://img.shields.io/static/v1?label=Severity&message=Major&color=ff3b30&style=flat-square)

**Description**

We can see the current `deposit` implementation:


[code/contracts/pcv/compound/ERC20CompoundPCVDeposit.sol#L24-L37](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/pcv/compound/ERC20CompoundPCVDeposit.sol#L24-L37)
```solidity
    /// @notice deposit ERC-20 tokens to Compound
    function deposit() external override whenNotPaused {
        uint256 amount = token.balanceOf(address(this));

        token.approve(address(cToken), amount);

        // Compound returns non-zero when there is an error
        require(
            CErc20(address(cToken)).mint(amount) == 0,
            "ERC20CompoundPCVDeposit: deposit error"
        );

        emit Deposit(msg.sender, amount);
    }
```

The method expects the tokens to already be in the contract's possession:


[code/contracts/pcv/compound/ERC20CompoundPCVDeposit.sol#L26](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/pcv/compound/ERC20CompoundPCVDeposit.sol#L26)
```solidity
        uint256 amount = token.balanceOf(address(this));
```

Proceeds to allow [`cToken`](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/CToken.sol) to move tokens on its behalf:


[code/contracts/pcv/compound/ERC20CompoundPCVDeposit.sol#L28](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/pcv/compound/ERC20CompoundPCVDeposit.sol#L28)
```solidity
        token.approve(address(cToken), amount);
```

And calls `cToken.mint()`:


[code/contracts/pcv/compound/ERC20CompoundPCVDeposit.sol#L30-L34](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/pcv/compound/ERC20CompoundPCVDeposit.sol#L30-L34)
```solidity
        // Compound returns non-zero when there is an error
        require(
            CErc20(address(cToken)).mint(amount) == 0,
            "ERC20CompoundPCVDeposit: deposit error"
        );
```

Under the hood, `mint()` takes the tokens from the caller's account. We can see that by following the call stack in the canonical implementation:

Mint calls `mintInternal`:


[contracts/CErc20.sol#L43-L52](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/CErc20.sol#L43-L52)
```solidity
    /**
     * @notice Sender supplies assets into the market and receives cTokens in exchange
     * @dev Accrues interest whether or not the operation succeeds, unless reverted
     * @param mintAmount The amount of the underlying asset to supply
     * @return uint 0=success, otherwise a failure (see ErrorReporter.sol for details)
     */
    function mint(uint mintAmount) override external returns (uint) {
        mintInternal(mintAmount);
        return NO_ERROR;
    }
```

Which calls `mintFresh`:


[contracts/CToken.sol#L381-L390](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/CToken.sol#L381-L390)
```solidity
    /**
     * @notice Sender supplies assets into the market and receives cTokens in exchange
     * @dev Accrues interest whether or not the operation succeeds, unless reverted
     * @param mintAmount The amount of the underlying asset to supply
     */
    function mintInternal(uint mintAmount) internal nonReentrant {
        accrueInterest();
        // mintFresh emits the actual Mint event if successful and logs on errors, so we don't need to
        mintFresh(msg.sender, mintAmount);
    }
```

The method `mintFresh` does a few things, but we're interested in the part where tokens are moved, which happens in `doTransferIn`:


[contracts/CToken.sol#L392-L398](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/CToken.sol#L392-L398)
```solidity
    /**
     * @notice User supplies assets into the market and receives cTokens in exchange
     * @dev Assumes interest has already been accrued up to the current block
     * @param minter The address of the account which is supplying the assets
     * @param mintAmount The amount of the underlying asset to supply
     */
    function mintFresh(address minter, uint mintAmount) internal {
```


[contracts/CToken.sol#L416-L424](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/CToken.sol#L416-L424)
```solidity
        /*
         *  We call `doTransferIn` for the minter and the mintAmount.
         *  Note: The cToken must handle variations between ERC-20 and ETH underlying.
         *  `doTransferIn` reverts if anything goes wrong, since we can't be sure if
         *  side-effects occurred. The function returns the amount actually transferred,
         *  in case of a fee. On success, the cToken holds an additional `actualMintAmount`
         *  of cash.
         */
        uint actualMintAmount = doTransferIn(minter, mintAmount);
```

Which tries to transfer the tokens from the caller (`ERC20CompoundPCVDeposit`) in the `CToken` implementation:


[contracts/CErc20.sol#L152-L182](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/CErc20.sol#L152-L182)
```solidity
    /**
     * @dev Similar to EIP20 transfer, except it handles a False result from `transferFrom` and reverts in that case.
     *      This will revert due to insufficient balance or insufficient allowance.
     *      This function returns the actual amount received,
     *      which may be less than `amount` if there is a fee attached to the transfer.
     *
     *      Note: This wrapper safely handles non-standard ERC-20 tokens that do not return a value.
     *            See here: https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca
     */
    function doTransferIn(address from, uint amount) virtual override internal returns (uint) {
        // Read from storage once
        address underlying_ = underlying;
        EIP20NonStandardInterface token = EIP20NonStandardInterface(underlying_);
        uint balanceBefore = EIP20Interface(underlying_).balanceOf(address(this));
        token.transferFrom(from, address(this), amount);

        bool success;
        assembly {
            switch returndatasize()
                case 0 {                       // This is a non-standard ERC-20
                    success := not(0)          // set success to true
                }
                case 32 {                      // This is a compliant ERC-20
                    returndatacopy(0, 0, 32)
                    success := mload(0)        // Set `success = returndata` of override external call
                }
                default {                      // This is an excessively non-compliant ERC-20, revert.
                    revert(0, 0)
                }
        }
        require(success, "TOKEN_TRANSFER_IN_FAILED");
```

Following this complete call stack we see that the tokens are indeed moved in the `CToken` implementation.

Thus, when a user wants to deposit tokens in the contract `ERC20CompoundPCVDeposit`, they need to do these two actions:

- send tokens to `ERC20CompoundPCVDeposit`
- call `ERC20CompoundPCVDeposit.deposit()`

The method `deposit()` expects the tokens to be in the contract before the method is executed. This isn't obvious, but we made sure that's the case by discussing it with the development team and checking the canonical Compound implementation.

Having the tokens in the contract before `deposit()` is called, leaves room for attackers to front-run the execution of `deposit()`.

Even if Compound implements this approach, it doesn't mean the same pattern needs to be implemented in `ERC20CompoundPCVDeposit`.

However, having the tokens in `ERC20CompoundPCVDeposit` exposes the user to front-running attacks.

We can protect the user in a few ways.

1. Create a proxy contract that bundles together `transfer` and `deposit`. This will be similar to how the tests currently work, since both actions happen synchronously and there is no possibility to front-run the execution of `deposit`. We can see this approach in the tests included in the repository that the `transfer` and `deposit` happen one after the other:


[code/contracts/test/integration/IntegrationTestCompoundPCVDeposits.t.sol#L44-L52](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/test/integration/IntegrationTestCompoundPCVDeposits.t.sol#L44-L52)
```solidity
    function setUp() public {
        daiBalance = daiDeposit.balance();

        usdcBalance = usdcDeposit.balance();

        vm.prank(MainnetAddresses.CUSDC);
        usdc.transfer(address(usdcDeposit), usdcBalance);
        usdcDeposit.deposit();
    }
```

But this does not correctly simulate an external user interacting with the system because a user will likely create two separate transactions.

We can fix this if we only allow the users to interact with `ERC20CompoundPCVDeposit` through a contract.

Going in this direction, the development team needs to do a few things:

- create a proxy actions contract that implements the desired interaction with the protocol;
- add limits in `deposit()` to ensure `msg.sender` is a contract, reducing the risk of users sending the transactions one after the other.

Implementing a proxy actions contract is a lot of work since the dev team will need to implement the action bundle described above and also every other action possible in the system. It also adds additional gas costs since a proxy action contract might be needed for each user before interacting with the system.

Taking this approach doesn't seem to be the ideal one, since a lot of additional work needs to be put in. However, we have an alternative 👇 

2. Modify `ERC20CompoundPCVDeposit` to use `transferFrom`

A different implementation of `deposit` can protect the user from being front-run while reducing the amount of development work.

Ideally we would add an argument to `deposit()` to specify the amount of tokens to deposit. This allows us to transfer the amount of tokens to `ERC20CompoundPCVDeposit` and call `CErc20(address(cToken)).mint(_amount)` which would safely move the tokens around.

```solidity
/// @notice deposit ERC-20 tokens to Compound
function deposit(uint256 _amount) external override whenNotPaused {
    token.transferFrom(msg.sender, address(this), _amount);

    // Compound returns non-zero when there is an error
    require(
        CErc20(address(cToken)).mint(_amount) == 0,
        "ERC20CompoundPCVDeposit: deposit error"
    );

    emit Deposit(msg.sender, _amount);
}

```

If the method signature can't be changed because `IPCVDeposit` needs a `deposit()` method with no arguments, we can take a different approach.

- Find out the amount of tokens the user has in their possession;
- Find out the approval amount the user allows `ERC20CompoundPCVDeposit` to transfer in their behalf;
- Deposit the minimum of the two values.

An example implementation would look like this:

```solidity
/// @notice deposit ERC-20 tokens to Compound
function deposit() external override whenNotPaused {
    // We need the minimum of the two values, because the user 
    // might approve more than their balance 
    uint256 amount = min(
        // The amount of tokens we are allowed to move
        token.allowance(msg.sender, address(this)),
        // The user's total balance
        token.balanceOf(msg.sender)
    );

    token.transferFrom(msg.sender, address(this), amount);

    // Compound returns non-zero when there is an error
    require(
        CErc20(address(cToken)).mint(amount) == 0,
        "ERC20CompoundPCVDeposit: deposit error"
    );

    emit Deposit(msg.sender, amount);
}
```

Please do not use the example code verbatim since they haven't been tested at all and make sure to thoroughly test before doing the update.

**Recommendation**

Consider the two options and see if any presented approaches make sense to implement to protect the users from being front-run.

**References**

- [MakerDAO's DSSProxy](https://etherscan.io/address/0x82ecd135dce65fbc6dbdd0e4237e0af93ffd5038#code)
- [OpenZeppelin Proxy](https://docs.openzeppelin.com/contracts/4.x/api/proxy#Proxy)
- [FIATDAO Action implementation](https://github.com/fiatdao/actions)

---


### [`_validPrice` can include floor and ceiling values when checking the validity](https://github.com/akiratechhq/review-volt-core-2022-09/issues/3)
![Issue status: Open](https://img.shields.io/static/v1?label=Status&message=Open&color=5856D6&style=flat-square) ![Minor](https://img.shields.io/static/v1?label=Severity&message=Minor&color=FFCC00&style=flat-square)

**Description**

A price is considered valid if the internal method `_validPrice` returns true.


[code/contracts/peg/PriceBoundPSM.sol#L122-L135](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/peg/PriceBoundPSM.sol#L122-L135)
```solidity
    /// @notice helper function to determine if price is within a valid range
    function _validPrice(Decimal.D256 memory price)
        internal
        view
        returns (bool valid)
    {
        valid =
            price.greaterThan(
                Decimal.ratio(floor, Constants.BASIS_POINTS_GRANULARITY)
            ) &&
            price.lessThan(
                Decimal.ratio(ceiling, Constants.BASIS_POINTS_GRANULARITY)
            );
    }
```

This method checks if the provided price is strictly greater than the floor and strictly less than the floor.


[code/contracts/peg/PriceBoundPSM.sol#L129-L134](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/peg/PriceBoundPSM.sol#L129-L134)
```solidity
            price.greaterThan(
                Decimal.ratio(floor, Constants.BASIS_POINTS_GRANULARITY)
            ) &&
            price.lessThan(
                Decimal.ratio(ceiling, Constants.BASIS_POINTS_GRANULARITY)
            );
```

These methods are taken from the `Decimal` library.


[code/contracts/external/Decimal.sol#L163-L177](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/external/Decimal.sol#L163-L177)
```solidity
    function greaterThan(D256 memory self, D256 memory b)
        internal
        pure
        returns (bool)
    {
        return compareTo(self, b) == 2;
    }

    function lessThan(D256 memory self, D256 memory b)
        internal
        pure
        returns (bool)
    {
        return compareTo(self, b) == 0;
    }
```

Using a strictly greater/less check will exclude the values of floor and ceiling as being valid. This forces the governance to add floor and ceiling values similar to `floor = 0.7999999999999` and `ceiling = 1.199999999999` to describe a valid interval from `0.8` to `1.2`, included.

If the check would accept the limits as valid, the governance could set `floor = 0.8` and `ceiling = 1.2`, which are a lot easier to reason about and verify.

**Recommendation**

Consider using the methods `greaterThanOrEqualTo` and `lessThanOrEqualTo` provided by [`Decimal`](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/external/Decimal.sol#L179-L193) to avoid using awkward values.


[code/contracts/external/Decimal.sol#L179-L193](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/external/Decimal.sol#L179-L193)
```solidity
    function greaterThanOrEqualTo(D256 memory self, D256 memory b)
        internal
        pure
        returns (bool)
    {
        return compareTo(self, b) > 0;
    }

    function lessThanOrEqualTo(D256 memory self, D256 memory b)
        internal
        pure
        returns (bool)
    {
        return compareTo(self, b) < 2;
    }
```

**[optional] References**


---


### [Setting ceiling basis points does not need a non zero validation](https://github.com/akiratechhq/review-volt-core-2022-09/issues/2)
![Issue status: Open](https://img.shields.io/static/v1?label=Status&message=Open&color=5856D6&style=flat-square) ![Minor](https://img.shields.io/static/v1?label=Severity&message=Minor&color=FFCC00&style=flat-square)

**Description**

A governor or an admin can set the price floor and the ceiling between which swaps are enabled.

To set the ceiling, they call `setOracleCeilingBasisPoints`


[code/contracts/peg/PriceBoundPSM.sol#L62-L69](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/peg/PriceBoundPSM.sol#L62-L69)
```solidity
    /// @notice sets the ceiling price in BP
    function setOracleCeilingBasisPoints(uint256 newCeilingBasisPoints)
        external
        override
        onlyGovernorOrAdmin
    {
        _setCeilingBasisPoints(newCeilingBasisPoints);
    }
```

The internal method `_setCeilingBasisPoints` make sure the provided value is 

- non zero

[code/contracts/peg/PriceBoundPSM.sol#L84-L87](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/peg/PriceBoundPSM.sol#L84-L87)
```solidity
        require(
            newCeilingBasisPoints != 0,
            "PegStabilityModule: invalid ceiling"
        );
```
- greater than the floor

[code/contracts/peg/PriceBoundPSM.sol#L88-L98](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/peg/PriceBoundPSM.sol#L88-L98)
```solidity
        require(
            Decimal
                .ratio(
                    newCeilingBasisPoints,
                    Constants.BASIS_POINTS_GRANULARITY
                )
                .greaterThan(
                    Decimal.ratio(floor, Constants.BASIS_POINTS_GRANULARITY)
                ),
            "PegStabilityModule: ceiling must be greater than floor"
        );
```

The check that makes sure the value is non-zero is not required since the ceiling has to be greater than the floor, and the floor has to be greater than zero:


[code/contracts/peg/PriceBoundPSM.sol#L105-L107](https://github.com/akiratechhq/review-volt-core-2022-09/blob/5826c86d97f928c23d23c5d2b6cabefba4649e67/code/contracts/peg/PriceBoundPSM.sol#L105-L107)
```solidity
    /// @notice helper function to set the floor in basis points
    function _setFloorBasisPoints(uint256 newFloorBasisPoints) internal {
        require(newFloorBasisPoints != 0, "PegStabilityModule: invalid floor");
```

To increase readability and to help the developer reason about the code, the check can stay there. Still, to slightly decrease the gas cost, a comment can be added that explains why the non-zero check is not needed, retaining readability.

**Recommendation**

Consider replacing the `require` with a comment that explains why the check is not necessary.


---


## Artifacts

### Surya

Sūrya is a utility tool for smart contract systems. It provides a number of visual outputs and information about the structure of smart contracts. It also supports querying the function call graph in multiple ways to aid in the manual inspection and control flow analysis of contracts.

<!-- **Contracts Description Table**

```text
surya mdreport report.md Contract.sol
```

-->

#### Graphs

<!-- ***Contract***

```text
surya graph Contract.sol | dot -Tpng > ./static/Contract_graph.png
```

![Contract Graph](./static/Contract_graph.png)

```text
surya inheritance Contract.sol | dot -Tpng > ./static/Contract_inheritance.png
```

![Contract Inheritance](./static/Contract_inheritance.png)

```text
Use Solidity Visual Auditor
```

![Contract UML](./static/Contract_uml.png) -->

#### Describe

We used surya to generate these outputs:

```shell
$ npx surya describe ./Contract.sol
```

##### Core

```shell
 +  Core (ICore, Permissions, Initializable)
    - [Ext] init #
       - modifiers: initializer
    - [Ext] setVcon #
       - modifiers: onlyGovernor
```

##### PriceBoundPSM

```shell
 +  PriceBoundPSM (PegStabilityModule, IPriceBound)
    - [Pub] <Constructor> #
       - modifiers: PegStabilityModule
    - [Ext] setOracleFloorBasisPoints #
       - modifiers: onlyGovernorOrAdmin
    - [Ext] setOracleCeilingBasisPoints #
       - modifiers: onlyGovernorOrAdmin
    - [Ext] isPriceValid
    - [Int] _allocate #
    - [Int] _setCeilingBasisPoints #
    - [Int] _setFloorBasisPoints #
    - [Int] _validPrice
    - [Int] _validatePriceRange
```

##### L2Core

```shell
 +  L2Core (ICore, Permissions)
    - [Pub] <Constructor> #
    - [Ext] setVcon #
       - modifiers: onlyGovernor
```

##### VoltSystemOracle

```shell
 +  VoltSystemOracle (IVoltSystemOracle)
    - [Pub] <Constructor> #
    - [Pub] getCurrentOraclePrice
    - [Ext] compoundInterest #
```

##### OraclePassThrough

```shell
 +  OraclePassThrough (IOraclePassThrough, Ownable)
    - [Pub] <Constructor> #
       - modifiers: Ownable
    - [Pub] update #
    - [Ext] read
    - [Ext] getCurrentOraclePrice
    - [Ext] currPegPrice
    - [Ext] updateScalingPriceOracle #
       - modifiers: onlyOwner
```

##### PCVGuardian

```shell
 +  PCVGuardian (IPCVGuardian, CoreRef)
    - [Pub] <Constructor> #
       - modifiers: CoreRef
    - [Pub] isWhitelistAddress
    - [Pub] getWhitelistAddresses
    - [Ext] addWhitelistAddress #
       - modifiers: onlyGovernor
    - [Ext] addWhitelistAddresses #
       - modifiers: onlyGovernor
    - [Ext] removeWhitelistAddress #
       - modifiers: onlyGuardianOrGovernor
    - [Ext] removeWhitelistAddresses #
       - modifiers: onlyGuardianOrGovernor
    - [Ext] withdrawToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
    - [Ext] withdrawAllToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
    - [Ext] withdrawERC20ToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
    - [Ext] withdrawAllERC20ToSafeAddress #
       - modifiers: hasAnyOfThreeRoles,onlyWhitelist
    - [Int] _withdrawToSafeAddress #
    - [Int] _withdrawERC20ToSafeAddress #
    - [Int] _addWhitelistAddress #
    - [Int] _removeWhitelistAddress #
```

##### PCVGuardAdmin

```shell
 +  PCVGuardAdmin (IPCVGuardAdmin, CoreRef)
    - [Pub] <Constructor> #
       - modifiers: CoreRef
    - [Ext] grantPCVGuardRole #
       - modifiers: onlyGovernor
    - [Ext] revokePCVGuardRole #
       - modifiers: onlyGuardianOrGovernor
```

##### CoreRef

```shell
 +  CoreRef (ICoreRef, Pausable)
    - [Pub] <Constructor> #
    - [Int] _initialize #
    - [Ext] setContractAdminRole #
       - modifiers: onlyGovernor
    - [Pub] isContractAdmin
    - [Pub] pause #
       - modifiers: onlyGuardianOrGovernor
    - [Pub] unpause #
       - modifiers: onlyGuardianOrGovernor
    - [Pub] core
    - [Pub] volt
    - [Pub] vcon
    - [Pub] voltBalance
    - [Pub] vconBalance
    - [Int] _burnVoltHeld #
    - [Int] _mintVolt #
    - [Int] _setContractAdminRole #
```

##### OracleRef

```shell
 +  OracleRef (IOracleRef, CoreRef)
    - [Pub] <Constructor> #
       - modifiers: CoreRef
    - [Ext] setOracle #
       - modifiers: onlyGovernor
    - [Ext] setDoInvert #
       - modifiers: onlyGovernor
    - [Ext] setDecimalsNormalizer #
       - modifiers: onlyGovernor
    - [Ext] setBackupOracle #
       - modifiers: onlyGovernorOrAdmin
    - [Pub] invert
    - [Pub] updateOracle #
    - [Pub] readOracle
    - [Int] _setOracle #
    - [Int] _setBackupOracle #
    - [Int] _setDoInvert #
    - [Int] _setDecimalsNormalizer #
    - [Int] _setDecimalsNormalizerFromToken #
```

##### ERC20CompoundPCVDeposit

```shell
 +  ERC20CompoundPCVDeposit (CompoundPCVDepositBase)
    - [Pub] <Constructor> #
       - modifiers: CompoundPCVDepositBase
    - [Ext] deposit #
       - modifiers: whenNotPaused
    - [Int] _transferUnderlying #
    - [Pub] balanceReportedIn
```


Legend
```shell
 ($) = payable function
 # = non-constant function
```

### Coverage

<!-- ```text
$ npm run coverage
``` -->

### Tests

<!-- ```text
$ npx buidler test
``` -->

## License

This report falls under the terms described in the included [LICENSE](./LICENSE).

<!-- Load highlight.js -->
<link rel="stylesheet"
href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/10.4.1/styles/default.min.css">
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/10.4.1/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/highlightjs-solidity@1.0.20/solidity.min.js"></script>
<script type="text/javascript">
    hljs.registerLanguage('solidity', window.hljsDefineSolidity);
    hljs.initHighlightingOnLoad();
</script>
<link rel="stylesheet" href="./style/print.css"/>
