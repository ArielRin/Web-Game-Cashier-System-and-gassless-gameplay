# **Web3 Game Cashier System**

A comprehensive and secure token management system designed for Web3 gaming platforms. The **GameCashier** contract facilitates deposits, game balances, bet deductions, winnings distribution, and more, while ensuring robust security and flexibility.

---

## **Features**

### **1. User Deposits**
- Users can deposit ERC-20 tokens into their `gameBalances` for seamless gameplay.
- Configurable deposit fees (default: 2%) sent to a dedicated `feesAddress`.
- Event: `Deposit` logs the net amount and fees for each deposit.

### **2. Game Balances**
- Tracks user balances (`gameBalances`) used for placing bets.
- Supports **pending bets**, ensuring unresolved bets must be cleared before placing new ones.
- Prevents double spending or over-betting.

### **3. Bet Deduction and Winnings Distribution**
- Operators deduct bets using `placeBet` and resolve outcomes with `resolveBet`.
- Winnings are directly sent to the user’s wallet on a win.
- Event logs: `BetPlaced`, `BetResolved`, `WinningsDistributed`.

### **4. Reward System**
- Operators can reward users directly using `rewardTokens`:
  - Tokens are sent directly to the user's wallet.
  - Event: `RewardGiven`.

### **5. Emergency Withdrawals**
- Users can withdraw their entire `gameBalances` in emergencies.
- Pending bets are excluded from emergency withdrawals.
- Event: `EmergencyWithdrawal`.

### **6. Operator Management**
- Admins can add/remove multiple operators dynamically.
- Operators manage bets, distribute winnings, and handle blacklist actions.
- Publicly accessible operator list (`getOperatorList`).
- Event logs: `OperatorAdded`, `OperatorRemoved`.

### **7. Blacklist Functionality**
- Blacklists malicious users, preventing interactions (including withdrawals).
- Only admins can remove users from the blacklist.
- Event logs: `Blacklisted`, `Unblacklisted`.

### **8. Administrative Controls**
- Admins can pause/unpause user interactions.
- Configurable deposit fee and `feesAddress`.
- Admin-only recovery of mistakenly sent ERC-20 tokens or native ETH.
- Event logs: Fee and state changes are logged for auditing.

### **9. Transparency**
- Public read functions for:
  - User `gameBalances`, `pendingBets`, and wallet balances.
  - List of active operators.

---

## **Usage**

### **1. Depositing Tokens**
Users deposit tokens to fund their `gameBalances`.

#### Example:
```solidity
deposit(100); // User deposits 100 tokens with a 2% fee.
```
- Fee: 2 tokens sent to `feesAddress`.
- Net amount credited to `gameBalances`: 98 tokens.

Event emitted:
```plaintext
Deposit(user: 0xUserAddress, amount: 98, fee: 2)
```

---

### **2. Placing a Bet**
Operators deduct tokens from `gameBalances` for gameplay.

#### Example:
```solidity
placeBet(0xUserAddress, 50); // Operator deducts a 50-token bet.
```
- User `gameBalances` reduced by 50 tokens.
- `pendingBets[user]` set to 50 tokens.

Event emitted:
```plaintext
BetPlaced(user: 0xUserAddress, amount: 50)
```

---

### **3. Resolving a Bet**
Operators resolve a bet, distributing winnings on a win or clearing the bet on a loss.

#### Example (Win):
```solidity
resolveBet(0xUserAddress, true, 100); // User wins 100 tokens.
```
- 100 tokens sent to the user’s wallet.
- `pendingBets[user]` reset to 0.

Event emitted:
```plaintext
WinningsDistributed(user: 0xUserAddress, amount: 100)
BetResolved(user: 0xUserAddress, betAmount: 50, won: true, winnings: 100)
```

#### Example (Loss):
```solidity
resolveBet(0xUserAddress, false, 0); // User loses the bet.
```
- `pendingBets[user]` reset to 0.

Event emitted:
```plaintext
BetResolved(user: 0xUserAddress, betAmount: 50, won: false, winnings: 0)
```

---

### **4. Rewarding Tokens**
Operators reward tokens directly to user wallets.

#### Example:
```solidity
rewardTokens(0xUserAddress, 25); // Operator rewards 25 tokens.
```
- 25 tokens transferred to the user’s wallet.

Event emitted:
```plaintext
RewardGiven(user: 0xUserAddress, amount: 25)
```

---

### **5. Emergency Withdrawal**
Users withdraw their entire `gameBalances`.

#### Example:
```solidity
emergencyWithdraw(); // User withdraws all tokens.
```
- Entire `gameBalances` sent to the user’s wallet.
- `gameBalances` reset to 0.

Event emitted:
```plaintext
EmergencyWithdrawal(user: 0xUserAddress, amount: <balance>)
```

---

### **6. Operator and Blacklist Management**
#### Add an Operator:
```solidity
addOperator(0xOperatorAddress);
```
Event emitted:
```plaintext
OperatorAdded(operator: 0xOperatorAddress)
```

#### Remove an Operator:
```solidity
removeOperator(0xOperatorAddress);
```
Event emitted:
```plaintext
OperatorRemoved(operator: 0xOperatorAddress)
```

#### Blacklist a User:
```solidity
blacklistAddress(0xUserAddress);
```
Event emitted:
```plaintext
Blacklisted(user: 0xUserAddress)
```

#### Unblacklist a User (Admin-only):
```solidity
unblacklistAddress(0xUserAddress);
```
Event emitted:
```plaintext
Unblacklisted(user: 0xUserAddress)
```

---

### **7. Administrative Actions**
#### Pause Contract:
```solidity
setPaused(true);
```
Event emitted:
```plaintext
Paused(state: true)
```

#### Update Deposit Fee:
```solidity
setDepositFee(3); // Set fee to 3%.
```
Event emitted:
```plaintext
FeeUpdated(newFeePercent: 3)
```

#### Update Fees Address:
```solidity
setFeesAddress(0xFeesAddress);
```
Event emitted:
```plaintext
FeesAddressUpdated(newFeesAddress: 0xFeesAddress)
```

---

## **Security Features**

1. **Pending Bets**:
   - Prevents new bets if a user has unresolved bets.
2. **Non-Reentrancy**:
   - Protects all state-changing functions using OpenZeppelin's `ReentrancyGuard`.
3. **Role-Based Access**:
   - Operators and admins manage critical functions securely.
4. **Blacklist**:
   - Denies blacklisted addresses access to the contract.
5. **Paused State**:
   - Admins can halt user interactions during emergencies.
6. **Auditability**:
   - Comprehensive event logging ensures full transparency.

---

## **Public Read Functions**

1. **Get Game Balance**:
   ```solidity
   getGameBalance(address user) external view returns (uint256);
   ```
   - Returns the user's available game balance.

2. **Get Pending Bet**:
   ```solidity
   getPendingBet(address user) external view returns (uint256);
   ```
   - Returns the user's unresolved bet amount.

3. **Get Operator List**:
   ```solidity
   getOperatorList() external view returns (address[] memory);
   ```
   - Returns a list of active operators.

4. **Get Wallet Balance**:
   ```solidity
   getWalletBalance(address user) external view returns (uint256);
   ```
   - Returns the user's ERC-20 wallet token balance.

---

## **Conclusion**
The **GameCashier** contract provides a comprehensive, secure, and flexible solution for managing in-game tokens in Web3 gaming platforms. With features like deposits, bet handling, rewards, blacklist management, and operator roles, it serves as a robust foundation for decentralized gaming ecosystems.

### **Contact**
For support or inquiries, feel free to open an issue or contribute to the repository.
Full Web3 Game Cashier Package approx $2,500usd setup per project for integration to current web3 game. (Please note: We dont make game software, thats on your team. We can provide integration support to make your games web3 compatible ).
