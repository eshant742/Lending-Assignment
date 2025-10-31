# 🪙 Blockchain Assignment: Simple Lending Protocol

**Name:** Eshant Gupta
**Roll No:** LCB2023016

---

## 📖 Project Description

This project is a `SimpleLending` protocol. It allows users to deposit an ERC20 token (Token A) as lending and, in return, borrow another ERC20 token (Token B) by providing it as collateral. The contract uses OpenZeppelin libraries for security and calculates a health factor to protect against insolvency.

This project was built and tested in the Remix IDE.

---

## 📂 Contracts Overview

This project consists of two main files:

* **`/contracts/SimpleLending.sol`**: The main protocol contract. It handles all logic for deposits, borrowing, repaying, withdrawing collateral, and liquidations. It is secured with several OpenZeppelin contracts.
* **`/contracts/Mocks.sol`**: Contains the mock contracts required for testing in a development environment. This includes `MockERC20` (to create test tokens) and `MockPriceFeed` (to simulate a Chainlink price oracle).

---

## 🛡️ OpenZeppelin Libraries Used

As per the assignment requirements, this project heavily utilizes OpenZeppelin's battle-tested libraries:

* **`@openzeppelin/contracts/token/ERC20/IERC20.sol`**: The standard interface for interacting with ERC20 tokens.
* **`@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol`**: Provides `safeTransfer` and `safeTransferFrom` functions to prevent failed token transfers.
* **`@openzeppelin/contracts/security/ReentrancyGuard.sol`**: Inherited by the contract, and the `nonReentrant` modifier is used to protect all functions that move tokens (`deposit`, `withdraw`, `borrow`, `repay`, `withdrawCollateral`, `liquidate`).
* **`@openzeppelin/contracts/access/Ownable.sol`**: Inherited by the contract to provide a secure access control mechanism. Only the "owner" (deployer) can call critical admin functions.
* **`@openzeppelin/contracts/security/Pausable.sol`**: Inherited by the contract to allow the owner to pause all key functions (`deposit`, `withdraw`, etc.) in case of an emergency.

---

## 🔐 Security Description

The security of the `SimpleLending` contract is centered on preventing common smart contract vulnerabilities, ensuring the correctness of financial logic, and leveraging industry-standard libraries for robust protection.

### 1. Re-entrancy Attack Prevention
This is the most critical security feature for a lending protocol.
* **Implementation**: The contract inherits from OpenZeppelin's `ReentrancyGuard.sol`.
* **How it Works**: The `nonReentrant` modifier is applied to all key functions that involve token transfers and state changes: `deposit()`, `withdraw()`, `borrow()`, `repay()`, `withdrawCollateral()`, and `liquidate()`.
* **Effect**: This modifier locks the contract, preventing a malicious user or contract from calling back into the function before the first call is finished. This directly stops an attacker from repeatedly borrowing or withdrawing funds before their balance or debt can be updated.

### 2. Arithmetic Safety
* **Implementation**: The project is built using **Solidity `^0.8.20`**.
* **How it Works**: All versions of Solidity from 0.8.0 onwards include built-in checks for integer overflow and underflow.
* **Effect**: Any calculation that results in a number wrapping around (e.g., a balance going below zero) will automatically cause the transaction to revert. This eliminates an entire class of critical bugs related to arithmetic.

### 3. Logic and State Validation
The contract enforces a strict order of operations and validates the user's state at every step using `require()` statements.
* **On `borrow()`**: A `require()` statement checks that the user's new position will be sufficiently collateralized (`newTotalDebt <= maxBorrowable`). This ensures no user can take on a loan that is immediately under-collateralized.
* **On `withdrawCollateral()`**: A `require()` statement validates that even after withdrawing, the user's remaining position is still healthy (`currentDebt <= maxBorrowable`). This is the core mechanism that guarantees loans remain solvent.
* **On `liquidate()`**: A `require()` statement checks that the user's health factor is below the threshold (`< 100`) before allowing anyone to liquidate the position.

### 4. Access Control
* **Implementation**: The contract inherits from OpenZeppelin's **`Ownable.sol`** and **`Pausable.sol`**.
* **Effect**: This provides a robust and standard way to manage administrative privileges.
    * **Ownership**: Critical functions that change the contract's parameters (like `setInterestRate`, `setCollateralizationRatio`, and `setPriceFeed`) are marked `onlyOwner`. This prevents any unauthorized user from changing the rules of the protocol.
    * **Pausable**: The contract includes `pause()` and `unpause()` functions (also `onlyOwner`) that can stop all major interactions. This acts as an emergency "off-switch" if a vulnerability is found, protecting user funds.

### 5. Known Assumptions
* **Price Oracle**: The contract correctly uses the `AggregatorV3Interface` but is deployed with a `MockPriceFeed` for this assignment. This is a known trust assumption. In a production environment, this mock address would be replaced with a real, decentralized Chainlink Price Feed to prevent price manipulation attacks.

---

## 🧪 How to Test in Remix

1.  Open Remix and create two files: `SimpleLending.sol` and `Mocks.sol`.
2.  Paste the provided code into each file.
3.  **Deploy `Mocks.sol`**: This will deploy `MockERC20` (for the loan token), `MockERC20` (for the collateral token), and `MockPriceFeed`.
4.  Deploy `MockERC20` twice, giving each a name/symbol (e.g., "LoanToken" / "LOAN" and "CollateralToken" / "COL").
5.  Deploy `MockPriceFeed`.
6.  **Deploy `SimpleLending.sol`**: Copy the three addresses from the previous step and paste them into the constructor fields for `_loanToken`, `_collateralToken`, and `_priceFeed`.
7.  **Run Test Scenario**:
    * Use the `mint` function on your two `MockERC20` contracts to give your test wallet (e.g., `0x5B3...`) funds.
    * Use the `approve` function on both `MockERC20` contracts to give the `SimpleLending` contract address permission to spend your tokens.
    * Follow the transaction steps outlined in the "Proof of Functionality" section below.

---

## ✅ Proof of Functionality (Transaction Logs)

Here is a step-by-step test of the full contract lifecycle.

### 🔑 Key Addresses & Setup

* **SimpleLending Contract**: `0x7EF2e0048f5bAeDe046f6BF797943daF4ED8CB47`
* **Loan Token (Token A)**: `0xd8b934580fcE35a11B58C6D73aDeE468a2833fa8`
* **Collateral Token (Token B)**: `0xf8e81D47203A594245E36C48e151709F0C19fBe8`
* **Mock Price Feed**: `0xD7ACd2a9FD159E69Bb102A1ca21C9a3e3A5F771B`
* **My Address (User)**: `0x5B38Da6a701c568545dCfcB03FcB875f56beddC4`

*(Note: The hashes and gas costs below are from a test environment and may vary.)*

### Step 1: Minting Tokens

(Paste your `mint` transaction logs here)

### Step 2: Approving the Lending Contract

(Paste your `approve` transaction logs here)

### Step 3: Depositing Lending (Token A)

(Paste your `deposit` transaction log here)

### Step 4: Borrowing (Token A) using Collateral (Token B)

(Paste your `borrow` transaction log here)

### Step 5: Checking Health Factor

(Paste your `getHealthFactor` call log here)

### Step 6: Repaying the Loan (with Interest)

(Paste your `repay` transaction log here)

### Step 7: Withdrawing Collateral

(Paste your `withdrawCollateral` transaction log here)
