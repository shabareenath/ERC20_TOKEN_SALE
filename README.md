### 1\. **ERC20 Standard Overview**

The **ERC20 standard** is the technical specification for creating fungible tokens on the Ethereum blockchain. It defines a set of functions and events that a compliant token must implement, such as:

-   **`balanceOf`**: Returns the token balance of a specific account.
-   **`transfer`**: Transfers tokens between accounts.
-   **`approve`** and **`allowance`**: Manage allowances where an owner authorizes another address to spend tokens on their behalf.
-   **`transferFrom`**: Used by the approved spender to transfer tokens on behalf of the owner.

The **CoffeeToken** contract is an ERC20-compliant token, and the **TokenSale** contract allows users to buy this token.

* * * * *

### 2\. **CoffeeToken Contract**

The `CoffeeToken` contract inherits from OpenZeppelin's **ERC20** and **AccessControl** contracts. It has features like minting, burning, and access control, ensuring that only authorized users can mint tokens.

#### Key Features:

1.  **ERC20 Inheritance:**

    -   The contract uses OpenZeppelin's implementation of ERC20 to simplify token creation. It includes all standard ERC20 functions like `transfer`, `approve`, `transferFrom`, etc.
    -   The `CoffeeToken` has a name ("CoffeeToken") and a symbol ("CFE"). The token also follows the standard decimals, typically 18, which is the default in the ERC20 standard.
2.  **AccessControl:**

    -   The contract implements role-based access control through OpenZeppelin's **AccessControl** contract.
    -   The contract defines a `MINTER_ROLE`, which allows specific addresses (those with this role) to mint new tokens.
3.  **Minting:**

    -   The `mint` function is restricted to only those addresses with the `MINTER_ROLE`. This allows them to create new tokens by calling `_mint`, a built-in ERC20 function.
    -   Tokens are minted to the provided address and in an amount multiplied by `10**decimals()` to account for the token's smallest unit.
4.  **Burning:**

    -   The `buyOneCoffee` and `buyOneCoffeeFrom` functions burn tokens. This means the tokens are destroyed, reducing the total supply.
    -   `buyOneCoffee` allows the user to burn 1 token from their own balance.
    -   `buyOneCoffeeFrom` allows the user to burn 1 token from someone else's account using the `allowance` mechanism, which first checks if the user has been given the right to spend 1 token from the account using `_spendAllowance`.

#### Key Functions:

-   **`mint(address to, uint256 amount)`**: Mints new tokens to a specified account.
-   **`buyOneCoffee()`**: Burns 1 token from the sender's balance.
-   **`buyOneCoffeeFrom(address account)`**: Burns 1 token from the specified account after checking allowances.

* * * * *

### 3\. **TokenSale Contract**

The `TokenSale` contract facilitates the sale of `CoffeeToken` in exchange for Ether (ETH). It allows users to buy tokens by sending ETH to the contract.

#### Key Features:

1.  **ERC20 Interaction:**

    -   The `TokenSale` contract interacts with the `CoffeeToken` (or any ERC20 token) through an abstract ERC20 interface. This interaction is done through `transferFrom` to handle the actual transfer of tokens.
    -   The token sale contract uses the address of the `CoffeeToken` contract passed in the constructor to interact with it.
2.  **Token Price:**

    -   The price of each token is fixed at `1 ether`, which is stored in the variable `tokenPriceInWei`. For every Ether sent to the contract, the user will receive one token.
3.  **Allowance Mechanism:**

    -   The `purchase` function uses `transferFrom` to transfer tokens from the `tokenOwner` to the buyer. For this to work, the `TokenSale` contract must first be given an allowance by the `tokenOwner`. The owner calls `approve` on the `CoffeeToken` contract, authorizing the `TokenSale` contract to spend a certain amount of tokens on their behalf.
4.  **Ether Handling:**

    -   The contract accepts Ether through the `purchase` function and calculates the number of tokens that the user will receive.
    -   It also refunds any excess Ether if the user sends more than the exact price of the tokens.

#### Key Functions:

-   **`purchase()`**:
    -   Users can send Ether to this function to purchase tokens.
    -   The number of tokens is calculated as `msg.value / tokenPriceInWei` (i.e., the amount of Ether sent divided by the token price in Wei).
    -   The contract transfers tokens to the buyer using `transferFrom` and sends any leftover Ether back to the buyer.

* * * * *

### 4\. **How the Contracts Work Together**

1.  **Deployment:**

    -   First, the `CoffeeToken` contract is deployed. The deployer (typically the owner or admin) gets the `MINTER_ROLE` and can mint tokens.
    -   The `TokenSale` contract is deployed next, and in its constructor, the address of the `CoffeeToken` is passed to link it.
2.  **Setting Allowances:**

    -   The `tokenOwner` (who deployed both contracts or has control over the tokens) needs to approve the `TokenSale` contract to transfer tokens on their behalf.
    -   The owner calls the `approve` function on `CoffeeToken`, specifying the `TokenSale` contract address and the number of tokens it is allowed to sell.
3.  **Buying Tokens:**

    -   Once the allowance is set, users can send Ether to the `TokenSale` contract's `purchase` function.
    -   The contract checks if enough Ether is sent, calculates the number of tokens to send, and uses `transferFrom` to transfer tokens from the `tokenOwner` to the buyer.
    -   Any excess Ether is refunded to the buyer.

* * * * *

### 5\. **Example Flow for Token Purchase:**

1.  **Deploy CoffeeToken:**

    -   The deployer deploys the `CoffeeToken` contract, which starts with no tokens.
    -   The deployer mints tokens to their own address (or any other address).
2.  **Deploy TokenSale:**

    -   The deployer deploys the `TokenSale` contract and passes the address of the `CoffeeToken`.
3.  **Set Allowance:**

    -   The `tokenOwner` (who minted the tokens) approves the `TokenSale` contract to spend a certain number of tokens on their behalf using the `approve` function of the `CoffeeToken`.
4.  **Purchase Tokens:**

    -   A user sends Ether to the `purchase` function of the `TokenSale` contract.
    -   The contract checks if the user sent enough Ether and then transfers tokens from the `tokenOwner` to the buyer.
    -   If the user overpaid, the contract refunds the excess Ether.

* * * * *

### Summary

-   **CoffeeToken**: Implements an ERC20 token with minting and burning functionality. Only users with `MINTER_ROLE` can mint tokens. Tokens can be burned when a user buys coffee, and events are emitted for transparency.

-   **TokenSale**: Facilitates the purchase of CoffeeTokens using Ether. It handles payments, allowance checks, and refunds if excess Ether is sent.

These contracts interact via the ERC20 `approve` and `transferFrom` functions, ensuring that token transfers are authorized and can be executed securely.
