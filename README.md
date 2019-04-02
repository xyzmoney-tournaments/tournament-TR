# Smart contract Tournament-TR

The contract is written in the Solidity language and intended for implementation on Ethereum. It contains function set for collecting and distribution of prize fund of a game tournament. The type of a game does not matter. The tournament should be organized as follows: the prize fund is the sum of fixed entrance fees of players, the one winner receives the predefined share of the prize fund, the rest of the funds is taken away by the tournament organizer.

**The contract contains functions of:**

* control of the tournament statuses;
* entering (re-entering) into the tournament;
* unregistering for the tournament with full refund;
* postponement of entrants registration deadline;
* withdrawal of prize amount by the winner;
* withdrawal of balance of the tournament by its organizer after the winner announcement, minus the winner share;
* full refund to players in case the tournament has been terminated with no winner announced.

## Statuses of the tournament

**The tournament can have one of the following statuses:**

* *NotTerminated* ⏤ the tournament is not complete (proceeds);
* *LackOfPlayers* ⏤ the tournament is terminated due to lack of players;
* *Cancelled* ⏤ the tournament is cancelled;
* *NoWinner* ⏤ the tournament is terminated with no winner announced;
* *Winner* ⏤ the tournament is terminated due to the winner announcement.

During the contract creation the tournament is assigned with the status of *NotTerminated*. When one of the reasons for termination further appears, the tournament is going irreversibly to the corresponding one of four alternative statuses of termination.

The status of *LackOfPlayers* is assigned when after registration deadline passed the number of players appears less than predefined minimum number of players. The due check is performed while executed `withdraw` function called by the organizer.

Any of the statuses of *Cancelled* and *NoWinner* is assigned by the organizer by calling of `terminate` function. The function can be executed even before the registration deadline passed, with immediate effect of entering (re-entering) and unregistering stop.

The status of *Winner* is assigned during execution of `announceWinner` function called by the organizer in order to announce winner.

## About time representation in the contract

Registration deadline (entryEndTime variable) and also `now` property used in the contract code as value of the current time have [Unix time](https://en.wikipedia.org/wiki/Unix_time) format which is the number of seconds that have elapsed since 00:00:00, 1 January 1970, Coordinated Universal Time (UTC).

It should be noted that `now` does not give current astronomical time. It is just alias of `block.timestamp` ⏤ the current block timestamp. Therefore the "current" time is defined in the contract with some share of uncertainty. In the description of the [Solidity language](https://solidity.readthedocs.io/en/v0.5.6/units-and-global-variables.html#index-2) it is noted:

>The current block timestamp must be strictly larger than the timestamp of the last block, but the only guarantee is that it will be somewhere between the timestamps of two consecutive blocks in the canonical chain.

## Typical scheme of the contract implementation

The creator of the contract executes the contract constructor which initializes the following parameters:

* address of the tournament organizer (`organizer` variable);
* minimum number of players (`minNumOfPlayers` variable);
* entrance fee (`entranceFee` variable);
* the winner's percentage share of the prize fund (`winnerShare` variable);
* participants registration deadline (`deadline` variable).

Herewith `organizer` variable is initialized by the address of the constructor's caller, and other variables are initialized by values of the same-name input parameters. Subsequently, registration deadline can be changed by the organizer only once, other variables are not subject to change.

Before the registration starts, the organizer generates a set of tables with unique entrance codes and then distributes these tables among agents that invite participants. If the organizer invites participants directly, he or she uses one of the tables, like the agents. By inviting a participant, an agent provides him or her with one of the available entrance codes. The code given to the participant cannot be re-given to another one, but the participant has the right to use it again while re-entering into the tournament.

To enter the tournament, a participant calls `enter` function, specifying the entrance fee amount (`entranceFee` variable) and the entrance code received from the organizer or from an agent. One can enter into the tournament only while the registration deadline not passed, as long as the tournament has the status of *NotTerminated* (the organizer can terminate the tournament before the deadline if there are grounds for that). Also the participants who has been unregistered for the tournament, can re-enter into it. The person who entered is considered further a player. Besides, if there is his or her first enter into the tournament, then the address of this person is also added into the list of entrants.

To unregister for the tournament, an entrant calls `unregister` function. The unregistering can be performed only by a player and only while the registration deadline not passed, as long as the tournament has the status of *NotTerminated*. The function transfers the entrance fee amount to the address of this participant. At the same time the participant is eliminated players, but remains in the list of entrants.

If necessary the organizer can reassign the registration deadline by shifting it either forward no more than to 72 hours, or backward so there are not less than 24 hours between present time and the new deadline. Such reassignment is possible if the tournament is not terminated yet.

If after the registration deadline passed the number of players (variable `playersCounter`) appears less minimum number of players (variable `minNumOfPlayers`), then the next call of `withdraw` function assignes the tournament with the status of *LackOfPlayers* (if the tournament was not terminated before by `terminate` function). Both the functions call only by the organizer.

If after the registration deadline passed the number of players appears not less minimum number of players, the tournament is considered begun. To terminate the tournament, the organizer should either announce the winner by calling of `announceWinner` function (therefore the tournament is automatically assigned with the status of *Winner*), or to terminate the tournament with no winner announced by calling `terminate` function with explicit assignment the tournament with one of the statuses of *Cancelled* or *NoWinner*. The choice of the status is maid by the organizer depending on the reason of the winner absence. It is possible to terminate the tournament by the winner announcement not earlier than in 24 hours after the registration deadline passed. As opposed to it, it is possible to terminate the tournament with no winner announced even before the deadline.

If the tournament has been terminated with no winner announced (i.e. with one of the statuses of *LackOfPlayers*, *Cancelled* or *NoWinner*), then all players can refund by calling of `refund` function.

From the moment the tournament got the status of *Winner*, the winner can take away the prize by calling `takePrize` function which transfers the prize amount to the winner address. From the same moment the organizer can take away available funds (variable `availableFunds`) from the contract account by calling of `withdraw` function. Available funds are equal to the contract balance (variable `contractBalance`) provided that the winner took away the prize, otherwise the available funds will be reduced by the prize amount.

## Contract variables

**`organizer`** ⏤ address of the organizer. It is is initialized by the address of the constructor's caller during execution of the contract constructor.

**`winner`** ⏤ address of the winner. It is assigned by the organizer at the end of the game in case of winner determination.

**`minNumOfPlayers`** ⏤ minimum number of players. If after the registration deadline passed the number of players (variable `playersCounter`) appears less of `minNumOfPlayers` variable, the tournament automatically get the status of *LackOfPlayers*. The `minNumOfPlayers` variable is initialized during execution of the contract constructor.

**`playersCounter`** ⏤ counter of players. Players are participants who enter into the tournament and does not unregister for it until the registration deadline.

**`entranceFee`** ⏤ the amount of the entrance fee in Wei (1 Ether = 10^18 Wei). It is initialized during execution of the contract constructor.

**`prizeFund`** ⏤ the amount of the prize fund in Wei. It is equal to the sum of all entrance fees of players.

**`winnerShare`** ⏤ the winner's percentage share of the prize fund. It is initialized during execution of the contract.

**`deadline`** ⏤ the registration deadline in Unix time format. It is initialized during execution of the contract constructor.
Afterwards it can be changed, but only once.

**`contractBalance`** ⏤ the balance of the contract.

**`availableFunds`** ⏤ the part of the contract account balance which is available to the organizer for withdrawal.

**`status`** ⏤ current status of the tournament. Can accept one of the following values:

* *NotTerminated* ⏤ the tournament is not complete (proceeds);
* *LackOfPlayers* ⏤ the tournament is terminated due to lack of players;
* *Cancelled* ⏤ the tournament is cancelled;
* *NoWinner* ⏤ the tournament is terminated with no winner announced;
* *Winner* ⏤ the tournament is terminated due to the winner announcement.

Initial status of the tournament ⏤ *NotTerminated*.

**`deadlineIsChanged`** ⏤ boolean variable. Its value is *true* if the registration deadline was reassigned (such reassignment can be maid only once).

**`prizeIsPaid`** ⏤ boolean variable. Its value is *true* if the winner has taken his prize.

**`entrantsList`** ⏤ the dynamic array containing entrants' addresses.

**`entranceCodes`** ⏤ mapping <entrant's address> => <entrant's entrance code>. The organizer identifies the agent who had invite the entrant by his entrance code.

**`entranceCounters`** ⏤ mapping <entrant's address> => <entrant's counter of entrances>.

Counter of entrances allows to define:

* how many times this entrant entered into the tournament;
* whether the entrant is a player at the moment or he (she) has unregistered for the tournament;
* whether the player has refunded if the tournament has terminated with no winner announced.

Such complex informativness is reached by that the counter has the sign: if the counter value is positive then the entrant is among the players of the tournament, else if it is negative then the entrant has unregistered for the tournament. The counter's step is 10. At the first entrance of the participant his (her) counter becomes 10. If after that the participant unregistered for the tournament, the counter inverts its sign. If then the participant enter again, the counter becomes 20, and so on. To put it briefly, at consequent entrances and unregisterings the counter will change in the following sequence: 10, -10, 20, -20, 30, … . If the tournament has terminated with no winner announced and the player has refund after that, his or her counter of entrances increases by 1 in addition.

**`whoseEntryCode`** ⏤ mapping <entrant's entrance code> => <entrant's address>. It is used for fast checking of the entered code ⏤ the new code should not be present among the entrance codes of players (at the same time presence of the code among entrance codes of participants who already has unregistered is allowed). On the participant's unregistering his or her entrance code is removed from the mapping thus releasing this code for new enter.

## Events

**`onCreation`** ⏤ is called during creation of the contract. With this event the following parameters are logged:

* minimum number of players;
* amount of the entrance fee;
* share of the winner;
* the registration deadline.

**`onEnter`** ⏤ is called after each enter into the tournament (see the description of `enter` function). With this event the following parameters are logged:

* the participant's address;
* the participant's entrance code;
* the participant's counter of entrances after this enter.

**`onUnregistering`** ⏤ is called after each unregistrating for the tournament (see the description of `unregister` function). With this event the following parameters are logged:

* the participant's address;
* the participant's counter of entrances after this unregistering.

**`onDeadlineChange`** ⏤ is generated after change of the registration deadline (see the description of `changeDeadline` function). With this event the new value of the deadline is logged.

**`onWinnerAnnounce`** ⏤ is called after storing the winner address into `winner` variable (see the description of `announceWinner` function). With this event the winner address is logged.

**`onPrizePayment`** ⏤ is xalled after the winner took away his prize (see the description of `takePrize` function). With this event the following parameters are logged:

* the address of the player who took away the prize;
* amount of the prize.

**`onTermination`** ⏤ is called after the tournament has been assigned with one of the statuses of *LackOfPlayers*, *Cancelled* or *NoWinner*. The status of *LackOfPlayers* can be assigned in `withdraw` function, the statuses of *Cancelled* and *NoWinner* ⏤ in `terminate` function. With this event the new status of the tournament is logged.

**`onRefund`** ⏤ is called after each refund in case of the tournament termination with no winner announced (see the description of `refund` function). With this event the address of the player who has refunded is logged.

**`onWithdraw`** ⏤ is called after each withdrawal from the contract balance by the organizer (see the description of `withdraw` function). With this event the withdrawal amount is logged.

## Functions

### `enter`

**Purpose:**

* accepts entrance fees from the persons wishing to enter (re-enter) into the tournament.

**Input parameter:**

* entrance code provided from the organizer or from an agent.

**Requirements:**

* current status of the tournament must be *NotTerminated*;
* one can enter only while the registration deadline not passed;
* the entering person's address should not be in the list of players (i.e. either this person enters for the first time or he or she should unregister before this entering);
* the deposit amount should be precisely equal to the entrance fee;
* the entrance code should be 16-digit number;
* the entrance code should be unique among entrance codes of players.

**Execution steps:**

* if the participant enters into the tournament for the first time then the function sets his (her) counter of entrances to 10 and add his (her) address to the list of entrants, else the function advances the counter forward;
* the participant's entrance code is added to `entranceCodes` mapping;
* the participant's address is added to `whoseEntryCode` mapping;
* the counter of players (`playersCounter` variable) increases by 1;
* the prize fund (`prizeFund` variable) increases by the entrance fee amount;
* `onEnter` event is calling;
* the contract balance (`contractBalance` variable) is updated.

### `unregister`

**Purpose:**

* transfers the entrance fee amount to the participant's address and excludes him (her) from the list of players.

There is no input parameters.

**Requirements:**

* current status of the tournament must be *NotTerminated*;
* one can unregister only while the registration deadline not passed;
* the unregistering person's address should be in the list of players.

**Execution steps:**

* the function inverts the sign of the participant's counter of entrances;
* the participant's  entrance code is deleted from `whoseEntryCode` mapping;
* the counter of players (`playersCounter` variable) decreases by 1;
* the prize fund (`prizeFund` variable) decreases by the entrance fee amount;
* `onUnregister` event is calling;
* the function transfers the entrance fee amount to the participant's address;
* the contract balance (`contractBalance` variable) is updated.

### `changeDeadline`

**Purpose:**

* changes the registration deadline.

**Input parameter:**

* new value of the deadline.

**Requirements:**

* only the organizer is authorized to perform the function;
* current status of the tournament must be *NotTerminated*;
* it is possible to execute the function only while the registration deadline not passed;
* the deadline can be changed only once (`deadlineIsChanged` variable must be equal to *false*);
* current value of the deadline must differ from the new one;
* the deadline can be shifted either forward no more than for 72 hours, or backward so there are not less than 24 hours between present time and the new deadline.

**Execution steps:**

* the function changes stores the input value into `deadline` variable;
* `deadlineIsChanged` variable is set to *true*;
* `onDeadlineChange` event is calling.

