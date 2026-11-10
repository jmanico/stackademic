---
version: alpha
name: stackademic
description: Study-desk design system for a Texas Hold'em strategy trainer for experienced adult players.
colors:
  primary: "#17202E"
  secondary: "#4A5568"
  tertiary: "#FFD84D"
  neutral: "#EEF1F4"
  surface: "#FFFFFF"
  on-surface: "#17202E"
  on-primary: "#FFFFFF"
  outline: "#6B7789"
  suit-spades: "#17202E"
  suit-hearts: "#B42318"
  suit-diamonds: "#1F4FB8"
  suit-clubs: "#1E6B3A"
  focus: "#1F4FB8"
  correct: "#146C43"
  correct-container: "#E3F1E8"
  error: "#B42318"
  error-container: "#FBE6E3"
  highlight-container: "#FFF3B8"
typography:
  headline-lg:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 32px
    fontWeight: 700
    lineHeight: 1.15
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 24px
    fontWeight: 700
    lineHeight: 1.2
  headline-sm:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 19px
    fontWeight: 700
    lineHeight: 1.3
  body-lg:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 18px
    fontWeight: 400
    lineHeight: 1.55
  body-md:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.5
  label-md:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 15px
    fontWeight: 700
    lineHeight: 1.2
  label-sm:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 13px
    fontWeight: 400
    lineHeight: 1.3
  card-rank:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 28px
    fontWeight: 800
    lineHeight: 1
    fontFeature: '"tnum" 1'
  stat-figure:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 40px
    fontWeight: 800
    lineHeight: 1
    fontFeature: '"tnum" 1'
rounded:
  none: 0px
  sm: 3px
  md: 6px
  full: 9999px
spacing:
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 40px
  gutter: 16px
  measure: 68ch
  page-max: 1120px
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label-md}"
    rounded: "{rounded.sm}"
    padding: 12px
    height: 44px
  button-secondary:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.primary}"
    typography: "{typography.label-md}"
    rounded: "{rounded.sm}"
    padding: 12px
    height: 44px
  button-danger:
    backgroundColor: "{colors.error}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label-md}"
    rounded: "{rounded.sm}"
    height: 44px
  move-choice:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.primary}"
    typography: "{typography.label-md}"
    rounded: "{rounded.sm}"
    height: 48px
  move-choice-selected:
    backgroundColor: "{colors.tertiary}"
    textColor: "{colors.primary}"
  playing-card:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.suit-spades}"
    typography: "{typography.card-rank}"
    rounded: "{rounded.md}"
    width: 52px
    height: 72px
  playing-card-hearts:
    textColor: "{colors.suit-hearts}"
  playing-card-diamonds:
    textColor: "{colors.suit-diamonds}"
  playing-card-clubs:
    textColor: "{colors.suit-clubs}"
  page:
    backgroundColor: "{colors.neutral}"
    textColor: "{colors.on-surface}"
  focus-ring:
    backgroundColor: "{colors.focus}"
    width: 3px
  input-field:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body-md}"
    rounded: "{rounded.sm}"
    height: 44px
  feedback-correct:
    backgroundColor: "{colors.correct-container}"
    textColor: "{colors.correct}"
  feedback-incorrect:
    backgroundColor: "{colors.error-container}"
    textColor: "{colors.error}"
  chat-bubble:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body-md}"
    rounded: "{rounded.md}"
  role-badge:
    backgroundColor: "{colors.highlight-container}"
    textColor: "{colors.primary}"
    typography: "{typography.label-sm}"
    rounded: "{rounded.full}"
---

# stackademic — Design

## Overview

**Brief.** *Subject:* no-limit Texas Hold'em cash-game strategy, taught through hand quizzes, practice hands, win probability, exams, one-on-one tutoring and bluffing lessons. No real money anywhere. *Audience:* adults (18+) who already know the rules and want to make better decisions; some become expert players who tutor others. *Primary job of the main screens:* the **hand quiz** shows one decision point completely (cards, board, seats, stacks, pot, action so far), takes one move, then explains it. Simulations, exams and live games reuse that same hand view. Chat, tutor list, account and admin screens support it.

**Rationale.** The product is a study desk, not a casino. The first draft was green felt, gold accents and an oval table. It was dropped because it would fit any poker site and points at gambling, which this product doesn't do. What replaced it:

- **The hand as a worked problem.** Each hand is laid out like a chess or bridge problem in a strategy book: the problem statement first, then the board, a seat strip, and the action written as a numbered line of moves. This is the one bold element. Everything else stays quiet.
- **A four-color deck.** Spades ink, hearts red, diamonds blue, clubs green. Serious players use this to stop misreading flushes, so it is a choice that belongs to this subject. Every suit also shows its symbol, so color is never the only cue.
- **Highlighter yellow** marks the user's own decision: the selected move and the user's seat, the way a student marks a textbook. It is never used for text.
- Cool index-card paper with dark-navy ink. Left-aligned with a single reading measure.

ASSUMPTION: the voice is a direct coach (plain, short, no hype), because the audience is experienced. There are no existing brand assets; this system and `logo.svg` are the brand *(assumed)*. Light theme only at launch *(assumed)*.

## Colors

- **Ink (#17202E, `primary`):** text, primary buttons, spade suit. 14.4:1 on paper, 16.4:1 on surface.
- **Slate (#4A5568, `secondary`):** secondary text and metadata. 6.6:1 on paper.
- **Highlighter (#FFD84D, `tertiary`):** the user's selected move and own seat. Always has ink text on it (11.8:1). Never used as a text color.
- **Paper (#EEF1F4, `neutral`):** page background. **Surface (#FFFFFF):** cards, panels, inputs.
- **Outline (#6B7789):** input and control borders, at least 3:1 on paper and surface.
- **Suits:** hearts #B42318 (5.8:1 on paper), diamonds #1F4FB8 (6.5:1), clubs #1E6B3A (5.8:1), spades = ink.
- **Focus (#1F4FB8):** a 3px ring with a 2px offset (6.5:1 against paper).
- **Correct (#146C43 on #E3F1E8) / Error (#B42318 on #FBE6E3):** answer feedback and errors (5.5:1 and 5.5:1). Always paired with an icon and a word ("Correct", "Not the best move"), never color alone.

All text pairs above pass 4.5:1. All UI and graphic pairs pass 3:1.

## Typography

One family: **Atkinson Hyperlegible Next**. It was designed for legibility, so 8 and 6, Q and O, and 1 and l are easy to tell apart. That matters when reading ranks and chip counts at a glance. Weight carries the hierarchy: 800 for card ranks and big figures, 700 for headings and labels, 400 for body. Numbers use tabular figures so stacks and pots line up. The scale is 13 / 15 / 16 / 18 / 19 / 24 / 32 / 40px. Body text is capped at 68ch. Labels use sentence case, never all caps. ASSUMPTION: the font files are self-hosted. Until then, the fallback stack is `system-ui, sans-serif`.

## Layout

- The page is a single left-aligned column up to 1120px wide, with 16px gutters on phones.
- **Hand view (desktop):** two columns. On the left: the problem, board, seat strip and action line. On the right: the move choices, then the feedback, explanation and win probability. **Phone:** one column in the same order, with the move choices shown right after the action line.
- **Seat strip:** seats in table order as a row of compact tiles (position, name, stack). It wraps onto two rows above 5 seats. There is no oval table drawing.
- Spacing uses a 4px base: 4 / 8 / 16 / 24 / 40.
- Touch targets are at least 44×44px.

## Elevation & Depth

The design is flat. Hierarchy comes from tone (white surface on paper) and 1px outline borders. Playing cards are the only element with a shadow (`0 1px 0 #6B7789`), so they read as physical objects. Dialogs sit on a scrim of ink at 60% opacity.

## Shapes

Radius follows the object. Playing cards are 6px, like a real card. Buttons, inputs and move choices are 3px. Panels have square corners. Badges are pill-shaped. Suit symbols are drawn shapes, not emoji. The logo has a dark-background variant (`logo-dark.svg`): paper-colored chips with ink dots under a lighter blue diamond (#6E98F5). It is used only where the page around it is dark, such as GitHub in dark mode.

## Components

- **Playing card:** a white face with the rank (800 weight) and the suit symbol in its suit color. The accessible name is spelled out, for example "Ace of spades". A face-down card is ink with a fine diagonal hatch. An empty board slot is a dashed outline.
- **Seat tile:** position (BTN, SB, BB, UTG…), display name, stack and last action. The user's seat has a highlighter background and the word "You". A folded seat uses slate text with "Folded". The current actor gets a 2px ink border.
- **Action line:** an ordered list, grouped by street (Preflop, Flop, Turn, River), for example "UTG raises to 6", "BTN calls 6".
- **Move choices:** a radio group showing only the legal moves (FR-1.6). States: default, hover (outline turns ink), focus (ring), selected (highlighter), disabled after submit, and locked showing the chosen answer. The submit button is labelled "Lock in answer", because answers can't be changed (FR-1.9).
- **Feedback panel:** Correct or incorrect, the accepted moves, the explanation, and then win probability as a large tabular percentage with "Win or tie at showdown vs N players". Per FR-3.4, win probability is hidden until the user answers.
- **Bet-size input (simulation):** a number field plus quick buttons. The error text states the allowed range, for example "Raise must be between 12 and 840."
- **Buttons:** primary (ink), secondary (white with outline), danger (red, used only inside confirmation dialogs). States: hover, focus, pressed, disabled (slate on paper, with a "why" hint nearby).
- **Input field:** label above, help text below, error text below in red with an icon. The field is linked to its error message for screen readers.
- **Chat:** messages show the sender's display name and time. The user's own messages are aligned right with a highlighter-tinted edge. System notices ("Live game ended — Dana left") are centered slate text with no bubble, so they can't be mistaken for a person's message. A character counter appears near 2,000 (FR-4.21).
- **Role badge:** a pill reading "Smart student", "Expert player" or "Admin".
- **Lock notice:** shown on gated features, for example "Tutors unlock when you pass the initial exam (16 of 20)." Retake timers show the exact time when the user can try again.
- **Dialog:** a title that states the action, a body that lists the consequences, and the safe action placed first.

**Security-relevant interactions**

- **Signed-in state:** the header always shows the display name, the role badge and "Sign out". Admin pages add a full-width ink bar reading "Admin — changes affect all users". Signed-out pages show only "Sign in" and "Create account".
- **Destructive and privileged actions** (delete account, remove a quiz hand or lesson, remove expert status, deactivate an account): these always open a dialog that names the exact target and lists what will be lost. Deleting an account lists email, display name, passkeys, quiz answers, exam attempts and chat (FR-6.10), and the user must type their display name before the red button becomes active. Switching tutors and leaving a live game use a plain confirmation without the red button. Account deletion and every admin action also require a fresh passkey check right before they run *(assumed)*.
- **Error messages:** say what happened and what to do next. They never show internals such as stack traces, IDs, or which check failed. Failed or cancelled passkey sign-in shows "Sign-in didn't complete. Try again or use another passkey." Recovery always shows "If that address has an account, a link is on its way. It works once, for 30 minutes." (FR-6.9). Messages about other users never say whether a given account exists.
- **User-supplied content** (display names, chat messages, and anything admins write): always shown as plain text, never as markup. Long words wrap, and display names are truncated with the full name available on focus or hover. Links in chat stay plain text and are not clickable *(assumed)*. Admin-written explanations and lessons allow a limited format: paragraphs, bold, lists, and card notation (for example `Ad`), which is shown as playing-card components. Raw markup is never rendered *(assumed)*.
- **Sign-up with an email already in use:** the account is still refused (FR-6.5), but the screen shows the same "Check your email to finish" message as any other sign-up, and the email itself tells the owner that an account already exists. The screen never confirms that an account exists *(assumed)*.

## Do's and Don'ts

- Do show a suit symbol with every suit color, and spell out cards for screen readers.
- Do keep highlighter yellow for the user's own choices only.
- Do name each action the same way everywhere: "Lock in answer" leads to "Answer locked".
- Don't use felt green, gold, chips-as-decoration, or casino language like "Bet now" or "Win big". No real money is involved.
- Don't show win probability before a quiz answer is locked.
- Don't render user content as markup, and don't style system notices like chat messages.
- Don't use all-caps labels, numbered section markers, or more than one bold element per screen.

## Open Questions

None open. DQ-1 to DQ-6 were answered on the owner's behalf at their request; those answers are marked *(assumed)* above.
