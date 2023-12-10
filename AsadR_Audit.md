# MixBytes Farm: Maincard Audit Report 

## 1. INTRODUCTION

### 1.1 Disclaimer
Here could be a disclaimer, but I don't have one

### 1.2 Security Assessment Methodology

One student-auditor were involved into the work on the audit. Auditor check the code with the methodology described below:

* Checking each file independently for a specific for this project vulnerabilities
* Checking the code in accordance with the vulnerabilities checklist
* Checking the correctness of code when possibile bad scenarios of cross-contracts function calling is happening
* Bug-fixing and reaudit

#### Funding Severity breakdown

All vulnerabilities discovered during the audit are classified based on their potential severity and have the following classification:

| Severity | Description                                                                                                                             |
|----------|-----------------------------------------------------------------------------------------------------------------------------------------|
| Critical | Bugs leading to assets theft, fund access locking, or any other loss of funds                                                           |
| High     | Bugs that can trigger a contract failure. Further recovery is possible only by manual modification of the contract state or replacement |
| Medium   | Bugs that can break the intended contract logic or expose it to DoS attacks, but do not cause direct loss funds                         |
| Low      | Bugs that do not have a significant immediate impact and could be easily fixed                                                          |

### 1.3 Project Dashboard

#### Project Summary

| Title              | Description                        |
|--------------------|------------------------------------|
| Client             | MixBytes farms' alumini?           |
| Project name       | maincard                           |
| Timeline           | December 5 2023 - December 10 2023 |
| Number of Auditors | 1                                  |

#### Project log

| Date       | Commit Hash                              | Note                 |
|------------|------------------------------------------|----------------------|
| 14.11.2023 | d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8 | Commit for the audit |

#### Project scope

| File name  | Link                                                                                                                    |
|------------|-------------------------------------------------------------------------------------------------------------------------|
| Arena      | https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Arena.sol      |
| Auction    | https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Auction.sol    |
| Card       | https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Card.sol       |
| Lottery    | https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Lottery.sol    |
| MagicBox   | https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/MagicBox.sol   |
| MainToken  | https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/MainToken.sol  |
| Tournament | https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Tournament.sol |

### 1.4 Summary of Findings

| Severity | # of Findings |
|----------|---------------|
| CRITICAL | 0             |
| HIGH     | 3             |
| MEDIUM   | 3             |
| LOW      | 0             |

## FINDINGS

### 2.1 CRITICAL

Not found

### 2.2 HIGH

#### H-1 Inverse require for balance check

##### Description

* At line https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Tournament.sol#L77C13-L77C76
* The require statement which blocks winners to get their rewards
* there is a require statement, which checks that sum of payouts should be more than rewards collected and then substract totalPayouts from rewardsCollected, which would lead to the function revert. Moreover, this would make it impossible to function finish successfully **in any case**, because of if the payouts is less than rewards it would also lead to the revert this time because of require
* It's HIGH because it will be impossible to execute this function never, which will make impossible for users to winners to get their rewards. The only way to fix this issue is to fix and redeploy contract 

##### Recommendation
I recommend to change the require statement to the opposite:  require(totalPayout <= rewards);

#### H-2 Impossible to register more than 1 card

##### Description

* At lines https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Tournament.sol#L127C13-L131C61
* Require statement at loop will make it impossible for user to register more than one card for tournament
* In cases if there is requireCards > 1 user needs to register >1 cards. If he passes the list of 2+ cards, the first iteration of a for loop will pass the require statement and set the boolean flag of _registeredPlayers_ to _True_. On the next iteration the require will fail. Which means it is only possible to register with only one card
* It's HIGH because for the tournaments with require cards amount > 1 it would be impossible for users to register for it. And the only way to fix it is contract manual fix and redeploy

##### Recommendation

I recommend to move the require statement from the loop. So it checks if the player already registered before the for loop and setting the true flag after the for loop   

#### H-3 Wrong text message for signature

#### Description

* At line https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Lottery.sol#L29C17-L29C45
* Wrong string for permit signature will make it impossible to sign with vrs.
* Instead of "\x19Ethereum Signed Message:\n32" at that line is "\x19E Signed Message:\n32". Because of it the require statement _signer == msg.sender_ will always fail
* It's HIGH because this will make it impossible for any user to use this function, and since there is no other function to buy tickets, the user won't be able to do this also. The only way to fix this issue it to change the contract and redeploy it

##### Recommendation

I recommend to replace the current string to "\x19Ethereum Signed Message:\n32"

### 2.3 MEDIUM

#### M-1 No check for bad timestamp

##### Description

* At line https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Arena.sol#L124C1-L129C71
* Owner of the contract can set the betsAcceptedUntilTs parameter to the earlier time then it was, which can cause problems for people who made bets after new timestamp
* Suppose the user make bet at time X and later the owner called updateEvent with time X-10. That is supposed to be wrong, because in function validateBet() is requirement for bet's timestamp to be lower than betsAcceptedUntilTs. But with this vulnerability we can bypass this defense
* It's MEDIUM, because it's bypass the contract defense for late bets, but fixable if owner of the contract will notice his mistake

##### Recommendation

I recommend to add require statement which checks, that time parameter is not lower than current block.timestamp

#### M-2 Extra big commission

##### Description

* At lines https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Auction.sol#L52C5-L54C6
* No check for a commission bigger than 100, which can lead to underflow
* If the contract owner set the commission >100, later when function [takeCard](https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Auction.sol#L190C36-L190C55) will be called, the evaluation `100 - _commission` will trigger underflow and revert of the transaction. Thus, it will be impossible to call the function takeCard until the owner set the normal commission
* It's MEDIUM, because it's cause the inability to use an important contract function, but fixable if contract owner notice this issue

##### Recommendation

I recommend to add require statement at function `setCommission`, that commission < 100. Despite it's only 8bit: 2^8-1 = 255

#### M-3 Interface Mismatch Blindspot

##### Description

* At lines [Arena L95C5-L97C6](https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Arena.sol#L95C5-L97C6), [Arena L99C5-L101C6](https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Arena.sol#L99C5-L101C6), [Auction L48C5-L50C6](https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Auction.sol#L48C5-L50C6), [Card L78C5-L89C6](https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Card.sol#L78C5-L89C6), [Card L110C5-L115C6](https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Card.sol#L110C5-L115C6), and [Auction L297C5-L299C6](https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Auction.sol#L297C5-L299C6)
* No check for sanity of a passing contract/interface/address which can lead to an overall contract inability to work
* Owner of a contract can pass wrong(i.e. zero address) in a setCard function at Arena contract, which will lead to inability to use almost all contract functions
* It's MEDIUM because only owner can pass the wrong parameter, but also he can fix it if he notices it

##### Recommendation

I recommend to add check for zero address or sanity checks in all functions above

### 2.4 LOW

#### L-1 Unverified Input Lengths in Function

##### Description

* At line https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Arena.sol#L182C1-L184C42
* No check for a length of the input lists, which will cause function revert or unexpected behavior 
* If user passes parameters with eventIds.length < cardIds.length and eventIds.length < choiceIds.length, then the function will end without revert, but the bets will be made only for the first eventIds.length cards, not all, which were passed and user wont notice it. Also, if eventIds.length > any of other two lists, the transaction will fail
* It's LOW because user won't lose his money, but he can be misled by the function behavior

##### Recommendation

I recommend to add require statement, which checks that input lists has the same length

#### L-2 Contract Name Spelling Mistake

##### Description

* At line https://github.com/maincard-io/maincard-contract/blob/d0a0029411f5dc6e58c2029e2707ba5d02dc2ab8/contracts/Auction.sol#L9C19-L9C40
* The contract's name misspell 
* It won't be so cool to have a simple spelling mistake forever in your contract, I guess
* It's LOW because it's not very important, but everyone agree that it's better to fix this 

##### Recommendation

I recommend to fix this 1-letter misspell