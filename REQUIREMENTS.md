# stackademic — Requirements

This file owns WHAT the system must do. It is written from the project notes and the owner's interview answers.

The owner answered the first questions of the interview directly, then asked for the remaining open questions to be answered on their behalf. Every requirement that comes from one of those delegated answers ends with *(assumed)*. An assumed requirement is a proposed decision, not something the owner stated; the owner can change any of them.

Requirement IDs (`FR-`, `NFR-`, `OQ-`) are permanent. They are never renumbered or reused.

## Required Requirement Inputs

- **Project purpose:** A Texas Hold'em strategy website called "stackademic" that teaches strategy through quizzes, simulated hands, win probability, one-on-one tutoring, and bluffing sessions. It is for people who already understand the game, not for new players.
- **Users and actors:** Users (learners); "smart students" (users who passed the initial exam and so can access tutors); expert players (users who completed the expert tutorials and passed the expert exams, and so can tutor); administrators *(assumed)*.
- **Core workflows:** Account creation and passkey sign-in; hand quiz; practice-hand simulation; win probability; initial exam that unlocks tutors; expert tutorials and exams; picking a tutor; tutoring chat; tutoring live game; bluffing sessions; content management by administrators *(assumed)*.
- **Data entities:** Account, passkey, role, quiz hand, quiz answer, practice hand, exam, exam attempt, tutorial, bluffing session, session completion, tutor selection, chat message, live game. *(assumed)*
- **External integrations:** Email delivery, for email confirmation and lost-passkey recovery. No others. *(assumed)*
- **Authentication and roles:** Accounts are required; sign-in is by passkey. Roles: user, smart student, expert player, administrator *(administrator assumed)*.
- **Regulatory or privacy constraints:** Users must confirm they are 18 or older; no real money is involved anywhere; personal data is limited to email address and display name; users can delete their account. *(assumed — not reviewed by a lawyer)*
- **Target surfaces (web UI, APIs, background jobs):** Web UI only. No mobile app and no public API. No background jobs are required by these requirements. *(assumed)*

## Functional Requirements

### 1. Hand quizzes

- **FR-1.1** The system MUST present the user with a Texas Hold'em hand.
- **FR-1.2** The system MUST ask the user which move they should make next for the presented hand.
- **FR-1.3** The system MUST accept the user's chosen move as their answer to the quiz.
- **FR-1.4** For each quiz hand, the system MUST show the user: their two private cards, the shared cards dealt so far, the number of players at the table, the user's seat position, each player's chip stack, the pot size, and every move made so far in the hand. *(assumed)*
- **FR-1.5** The system MUST support quiz hands at each of the four points in a hand: before the flop, on the flop, on the turn, and on the river. *(assumed)*
- **FR-1.6** The system MUST offer as answer choices only the moves that are legal in the presented hand, drawn from: fold, check, call, bet, raise. *(assumed)*
- **FR-1.7** The system MUST NOT ask the user to choose a bet size as part of a quiz answer. *(assumed)*
- **FR-1.8** The system MUST reject an answer that is not one of the offered choices and leave the quiz unanswered. *(assumed)*
- **FR-1.9** The system MUST NOT let a user change an answer after submitting it. *(assumed)*
- **FR-1.10** The system MUST mark an answer correct when it matches one of the accepted moves stored with that quiz hand, and incorrect otherwise. *(assumed)*
- **FR-1.11** After the user answers, the system MUST show whether the answer was correct, the accepted move or moves, and the written explanation stored with that quiz hand. *(assumed)*
- **FR-1.12** The system MUST save each quiz answer with the user, the quiz hand, the chosen move, whether it was correct, and when it was given. *(assumed)*
- **FR-1.13** The system MUST show a user their own past quiz answers and their overall percentage of correct answers. *(assumed)*
- **FR-1.14** The system MUST NOT show one user's quiz answers or results to another user, except as FR-4.19 allows. *(assumed)*

### 2. Situations and simulations

- **FR-2.1** The system MUST present the user with situations at each of these table sizes: 2 players, 4 players, and 8 players.
- **FR-2.2** The system MUST let the user choose the table size for quizzes and simulations, from 2 to 9 players. *(assumed)*
- **FR-2.3** The system MUST cover no-limit Texas Hold'em in cash-game form only. *(assumed)*
- **FR-2.4** The system MUST let the user play a simulation: a full practice hand, from the deal to the end of the hand, against computer-controlled players. *(assumed)*
- **FR-2.5** In a simulation, the system MUST deal the cards at random. *(assumed)*
- **FR-2.6** In a simulation, the system MUST let the user make only moves that are legal at that point in the hand, and MUST reject any other move. *(assumed)*
- **FR-2.7** In a simulation, the system MUST let the user choose a bet size when they bet or raise, and MUST reject a size that the rules of no-limit Texas Hold'em or the user's chip stack do not allow. *(assumed)*
- **FR-2.8** When a simulation hand ends, the system MUST show the user who won, the amount of play chips won, and the private cards of every player who reached the showdown. *(assumed)*
- **FR-2.9** The system MUST use play chips that have no monetary value in every simulation and live game. *(assumed)*
- **FR-2.10** A computer-controlled player MUST make only moves and bet sizes that are legal at that point in the hand. *(assumed)*
- **FR-2.11** A computer-controlled player MUST choose its move from only what a human player in its seat could know: its own private cards, the shared cards dealt so far, the chip stacks, the pot size, and the moves made so far. It MUST NOT use the user's or any other player's private cards, or cards not yet dealt. *(assumed)*
- **FR-2.12** A computer-controlled player MUST choose its move by a fixed set of rules that weighs the strength of its hand against the cost of staying in the hand, with some random variation so that it does not always make the same move in the same situation. *(assumed)*
- **FR-2.13** The system MUST NOT offer a choice of difficulty or playing style for computer-controlled players. *(assumed)*

### 3. Win probability

- **FR-3.1** The system MUST show the user the win probability for the current hand.
- **FR-3.2** The system MUST calculate win probability as the chance that the user's hand wins or ties at the showdown, given the user's private cards, the shared cards dealt so far, and the number of other players still in the hand, treating those players' cards as unknown. *(assumed)*
- **FR-3.3** The system MUST show win probability as a percentage. *(assumed)*
- **FR-3.4** In a hand quiz, the system MUST show win probability only after the user has answered. *(assumed)*
- **FR-3.5** In a simulation, the system MUST show an updated win probability each time new shared cards are dealt or a player leaves the hand. *(assumed)*

### 4. Tutoring

- **FR-4.1** The system MUST offer every user an initial exam.
- **FR-4.2** The system MUST NOT give a user access to tutors until that user has passed the initial exam.
- **FR-4.3** The system MUST give a user access to tutors once that user has passed the initial exam.
- **FR-4.4** The system MUST provide a series of tutorials and exams that a user completes to become an expert player.
- **FR-4.5** The system MUST make a user an expert player only after that user has gone through the series of tutorials and passed the exams.
- **FR-4.6** The system MUST NOT let a user act as a tutor unless that user is an expert player.
- **FR-4.7** The system MUST let an expert player and a user with tutor access exchange chat messages for tutoring.
- **FR-4.8** The system MUST let an expert player tutor a user with tutor access in a live game.
- **FR-4.9** The system MUST show a user with tutor access a list of expert players to choose a tutor from.
- **FR-4.10** The system MUST let a user with tutor access pick one expert player from that list as their tutor.
- **FR-4.11** The system MUST keep each tutoring chat and each tutoring live game one-on-one: one expert player and one student.
- **FR-4.12** The initial exam MUST consist of 20 hand quiz questions, and the system MUST count it as passed when the user answers at least 16 correctly. *(assumed)*
- **FR-4.13** The system MUST let a user who failed the initial exam retake it, but not sooner than 24 hours after the failed attempt. *(assumed)*
- **FR-4.14** The system MUST NOT let a user start the expert tutorials until that user has passed the initial exam. *(assumed)*
- **FR-4.15** The system MUST follow each expert tutorial with its own exam, and MUST count that exam as passed when the user answers at least 90% of its questions correctly. *(assumed)*
- **FR-4.16** The system MUST let a user who failed an expert exam retake it, but not sooner than 24 hours after the failed attempt. *(assumed)*
- **FR-4.17** The system MUST list an expert player in the tutor list from the moment they become an expert player. *(assumed)*
- **FR-4.18** The system MUST let a student have one tutor at a time and change to a different tutor at any time. *(assumed)*
- **FR-4.19** The system MUST show a tutor the quiz results of their current student, and MUST stop showing them when that student changes tutor. *(assumed)*
- **FR-4.20** The system MUST NOT let anyone other than the two participants read a tutoring chat. *(assumed)*
- **FR-4.21** The system MUST reject a chat message that is empty or longer than 2,000 characters, and MUST tell the sender why. *(assumed)*
- **FR-4.22** The system MUST let a student invite their tutor to a live game, and MUST start the live game only when the tutor joins. *(assumed)*
- **FR-4.23** In a live game, the student MUST play practice hands against computer-controlled players under the simulation rules in section 2, and the tutor MUST NOT be dealt cards or make moves. *(assumed)*
- **FR-4.24** In a live game, the system MUST show the tutor the student's private cards and every move as it happens. *(assumed)*
- **FR-4.25** In a live game, the system MUST let the tutor and the student exchange chat messages while the hand is being played. *(assumed)*
- **FR-4.26** The system MUST end a live game when either the student or the tutor leaves it, and MUST tell the other participant that it ended. *(assumed)*

### 5. Bluffing strategy education

- **FR-5.1** The system MUST provide educational sessions on bluffing strategy.
- **FR-5.2** Each bluffing session MUST consist of written lesson material followed by hand quizzes about bluffing decisions. *(assumed)*
- **FR-5.3** The system MUST show a signed-in user the list of bluffing sessions and let them open any of them. *(assumed)*
- **FR-5.4** The system MUST record a bluffing session as completed for a user once that user has answered every quiz in it, and MUST show the user which sessions they have completed. *(assumed)*

### 6. Accounts and sign-in

- **FR-6.1** The system MUST require a person to be signed in to an account before they can use hand quizzes, situations, simulations, win probability, tutoring, or bluffing sessions.
- **FR-6.2** The system MUST let a person sign in to their account with a passkey.
- **FR-6.3** The system MUST NOT sign a person in when their passkey sign-in fails or is cancelled.
- **FR-6.4** To create an account, the system MUST require an email address, a display name, a confirmation that the person is 18 or older, and the registration of a passkey. *(assumed)*
- **FR-6.5** The system MUST refuse to create an account when the email address is already used by another account, when the email address is not validly formed, or when the person does not confirm they are 18 or older. *(assumed)*
- **FR-6.6** The system MUST send a confirmation link to the email address given at account creation, and MUST NOT let the account be used until that link has been followed. *(assumed)*
- **FR-6.7** The system MUST let a person who lost their passkey request a recovery link sent to their account's confirmed email address, and MUST let them register a new passkey after following it. *(assumed)*
- **FR-6.8** A recovery link MUST work only once and MUST stop working 30 minutes after it is sent. *(assumed)*
- **FR-6.9** When recovery is requested, the system MUST show the same response whether or not the email address belongs to an account. *(assumed)*
- **FR-6.10** The system MUST let a signed-in user delete their account, and MUST remove their email address, display name, passkeys, quiz answers, exam attempts, and chat messages when they do. *(assumed)*
- **FR-6.11** The system MUST NOT collect personal data beyond email address and display name. *(assumed)*

### 7. Administration and content

- **FR-7.1** The system MUST have an administrator role. *(assumed)*
- **FR-7.2** The system MUST NOT let any user give themselves or anyone else the administrator role through the website. *(assumed)*
- **FR-7.3** The system MUST let an administrator create, edit, and remove quiz hands, each with its accepted move or moves and its written explanation. *(assumed)*
- **FR-7.4** The system MUST refuse to save a quiz hand that has no accepted move or no explanation. *(assumed)*
- **FR-7.5** The system MUST let an administrator create, edit, and remove expert tutorials, exams, and bluffing sessions. *(assumed)*
- **FR-7.6** The system MUST let an administrator remove a user's expert player status, and MUST take that user off the tutor list when they do. *(assumed)*
- **FR-7.7** The system MUST let an administrator deactivate an account, and MUST NOT let a deactivated account sign in. *(assumed)*
- **FR-7.8** The system MUST refuse every administrator action attempted by a user who is not an administrator. *(assumed)*

### 8. Money

- **FR-8.1** The system MUST NOT accept payments, deposits, or wagers of real money, and MUST NOT pay out anything of monetary value. *(assumed)*
- **FR-8.2** The system MUST NOT charge for any feature, including tutoring. *(assumed)*

## Non-Functional Requirements

- **NFR-1.1** The system MUST work in the current versions of Chrome, Safari, Firefox, and Edge. *(assumed)*
- **NFR-1.2** The system MUST be usable on both desktop-sized and phone-sized screens. *(assumed)*
- **NFR-1.3** The system MUST meet WCAG 2.2 level AA. *(assumed)*
- **NFR-2.1** The system MUST show a quiz hand within 2 seconds of the user asking for one. *(assumed)*
- **NFR-2.2** The system MUST show a win probability within 2 seconds of the point where FR-3.4 or FR-3.5 calls for it. *(assumed)*
- **NFR-2.3** A displayed win probability MUST be within 1 percentage point of the true value in at least 999 of every 1,000 calculations. *(assumed)*
- **NFR-2.4** The system MUST deliver a chat message or a live-game move to the other participant within 2 seconds. *(assumed)*

## Open Questions

None open. All questions were answered by the owner or, at the owner's request, answered on their behalf; the delegated answers are the requirements marked *(assumed)* above.
