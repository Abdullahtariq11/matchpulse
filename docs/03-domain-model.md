# MatchPulse Domain Model

## 1. Purpose

This document defines the core domain concepts used by MatchPulse and
the relationships between them.

The domain model is technology-independent. It describes the football
concepts that MatchPulse needs to understand before decisions are made
about databases, APIs, messaging systems, or implementation details.

The model is intentionally kept focused on the current MatchPulse MVP.
Additional concepts should only be introduced when they are required by
the system.

---

## 2. Core Domain Concepts

The core MatchPulse domain concepts are:

- Team
- Player
- Match
- MatchEvent
- PlayerMatchAppearance
- PlayerMatchStats
- PlayerMatchRating

---

### 2.1 Team

A Team represents a football club participating in matches tracked by
MatchPulse.

For the MVP, a Team contains:

- `teamId` — uniquely identifies the team.
- `name` — the team's display name.

Information such as recent form should be derived from historical match
results rather than stored directly as a property of the team.

The MVP focuses on Manchester United, but the domain model should not
require Manchester United to be hardcoded throughout the system.

---

### 2.2 Player

A Player represents a football player who can participate in matches
tracked by MatchPulse.

For the MVP, a Player contains:

- `playerId` — uniquely identifies the player.
- `name` — the player's display name.
- `position` — the player's primary playing position.

A Player does not directly store their team membership.

The team a player represents in a particular match is recorded through
their PlayerMatchAppearance. This preserves historical match information
if the player changes teams in the future.

The player's identifier, rather than their name, is used to uniquely
identify them because player names are not guaranteed to be unique.

---

### 2.3 Match

A Match represents a football match tracked by MatchPulse.

For the MVP, a Match contains:

- `matchId` — uniquely identifies the match.
- `homeTeam` — the home team participating in the match.
- `awayTeam` — the away team participating in the match.
- `homeScore` — the home team's current or final score.
- `awayScore` — the away team's current or final score.
- `status` — the current state of the match.
- `startTime` — the scheduled date and time at which the match begins.

Conceptually, the initial match states are:

- `SCHEDULED`
- `LIVE`
- `FINISHED`

Additional states such as postponed or cancelled can be introduced if
future requirements need them.

The current score is stored on the Match even though goals can also be
determined from MatchEvents.

Keeping the current score provides a direct representation of the match
state and can also help validate event processing. For example, a
difference between the known match score and the goals calculated from
events may indicate that an event was missed or processed incorrectly.

The source of truth for the authoritative match score will be determined
when the event provider and processing architecture are designed.

---

### 2.4 MatchEvent

A MatchEvent represents an action that occurs during a football match.

MatchEvents are the detailed record of what happened during the match
and are used to calculate player statistics, ratings, and the important
match timeline.

For the MVP, a MatchEvent contains:

- `eventId` — uniquely identifies the event.
- `matchId` — identifies the match in which the event occurred.
- `playerId` — identifies the player who performed the action.
- `type` — identifies the type of football action.
- `outcome` — describes the result of the action when applicable.
- `sequenceNumber` — identifies the event's position in the event stream.
- `matchTime` — identifies when the event occurred within the football match.
- `relatedEventId` — optionally references another MatchEvent related to
  this event.

Supported event types initially include:

- PASS
- SHOT
- ASSIST
- DRIBBLE
- TACKLE
- INTERCEPTION
- FOUL
- SAVE
- CROSS
- CARD
- SUBSTITUTION

A SHOT can have outcomes such as:

- GOAL
- SAVED
- BLOCKED
- OFF_TARGET

For example:

A successful pass:

    type: PASS
    outcome: SUCCESS

A goal:

    type: SHOT
    outcome: GOAL

A goal is therefore represented as the outcome of a SHOT rather than as
a separate GOAL event.

#### Event Identity and Ordering

`eventId` and `sequenceNumber` solve different problems.

`eventId` identifies a specific event and allows MatchPulse to recognize
duplicate deliveries.

For example, if the same event is delivered twice, it must still affect
player statistics only once.

`sequenceNumber` represents where the event belongs in the event stream.

For example:

    Event A → sequenceNumber: 101
    Event C → sequenceNumber: 103

Receiving sequence number 103 after 101 indicates that event 102 may be
missing.

If event 102 arrives later, MatchPulse can place it in the correct
logical position.

This allows the system to support:

- duplicate detection
- out-of-order events
- missing event detection
- event recovery

#### Match Time

`matchTime` represents when the event occurred within the football match.

For example:

    67:32

This is different from a real-world timestamp.

`sequenceNumber` determines event ordering, while `matchTime` represents
football match time.

#### Related Match Events

Some MatchEvents may be related to another MatchEvent.

For example, when a goal is assisted:

    SHOT
    eventId: 150
    playerId: Player A
    type: SHOT
    outcome: GOAL

    ASSIST
    eventId: 151
    playerId: Player B
    type: ASSIST
    relatedEventId: 150

The ASSIST event references the SHOT that resulted in the goal.

MatchPulse will initially model these relationships directly between
events rather than introducing a separate Play entity.

If future requirements require grouping many events into a single
football sequence, this decision may be revisited.

---

### 2.5 PlayerMatchAppearance

A PlayerMatchAppearance represents how a player participated in a
specific match.

For the MVP, an appearance contains:

- `playerId` — identifies the player.
- `matchId` — identifies the match.
- `teamId` — identifies the team the player represented in this match.
- `isStarter` — indicates whether the player was part of the starting eleven.
- `subInTime` — the match time when a non-starting player entered the match.
- `subOutTime` — the match time when the player left the match.

For example, a starter substituted after 70 minutes could be represented
as:

    isStarter: true
    subInTime: null
    subOutTime: 70:00

A substitute entering after 70 minutes and finishing the match could be:

    isStarter: false
    subInTime: 70:00
    subOutTime: null

A player who plays the entire match could be:

    isStarter: true
    subInTime: null
    subOutTime: null

Playing time can be derived from the player's starting status and
substitution times rather than stored independently.

A player who remains on the bench and never enters the match does not
have a PlayerMatchAppearance.

The Team relationship is stored on PlayerMatchAppearance instead of
directly on Player. This preserves the historical fact of which team a
player represented during a particular match even if the player later
changes clubs.

---

### 2.6 PlayerMatchStats

PlayerMatchStats represents the measurable actions performed by a player
during a specific match.

For the MVP, PlayerMatchStats contains:

- `playerId` — identifies the player.
- `matchId` — identifies the match.
- `goals`
- `assists`
- `passesAttempted`
- `passesCompleted`
- `shotsAttempted`
- `shotsOnTarget`
- `dribblesAttempted`
- `dribblesCompleted`
- `crossesAttempted`
- `crossesCompleted`
- `tackles`
- `interceptions`
- `chancesCreated`
- `bigChancesCreated`
- `bigChancesMissed`
- `foulsCommitted`
- `foulsSuffered`
- `yellowCards`
- `redCards`
- `saves`

PlayerMatchStats are calculated from MatchEvents.

For example:

    PASS + SUCCESS

results in:

    passesAttempted += 1
    passesCompleted += 1

Similarly:

    SHOT + GOAL

results in:

    shotsAttempted += 1
    shotsOnTarget += 1
    goals += 1

A saved shot results in:

    shotsAttempted += 1
    shotsOnTarget += 1

A blocked or off-target shot contributes to shots attempted but not
shots on target.

Derived statistics should not be stored when they can be calculated
cheaply from the underlying values.

For example:

    passCompletionPercentage =
        passesCompleted / passesAttempted * 100

Pass completion percentage therefore does not need to be stored
independently.

Playing time is derived from PlayerMatchAppearance rather than stored
inside PlayerMatchStats.

The original MatchEvents are retained even after PlayerMatchStats have
been calculated.

This allows statistics to be rebuilt if:

- processing logic changes
- a bug is discovered
- missing events are recovered
- historical statistics need to be recalculated

The distinction is:

> MatchEvents represent what happened.
>
> PlayerMatchStats represent what MatchPulse currently calculates from
> those events.

---

### 2.7 PlayerMatchRating

PlayerMatchRating represents MatchPulse's evaluation of a player's
performance in a specific match.

For the MVP, a PlayerMatchRating contains:

- `playerId` — identifies the player.
- `matchId` — identifies the match.
- `rating` — the calculated performance rating for the player.

The rating is derived from information contained in PlayerMatchStats and
PlayerMatchAppearance.

For example:

    PlayerMatchStats
           +
    PlayerMatchAppearance
           ↓
    PlayerMatchRating

The exact rating algorithm has not yet been defined.

The rating system will require separate research into areas including:

- which statistics affect ratings
- how strongly different statistics affect ratings
- positive and negative contributions
- position-specific weighting
- playing-time requirements
- substitutes and limited appearances
- goalkeeper-specific evaluation

These decisions should be made before implementing the rating algorithm.

---

## 3. Domain Relationships

The core MatchPulse domain concepts have the following relationships:

- A Match is played between two Teams: a home team and an away team.
- A Match contains many MatchEvents.
- A MatchEvent belongs to one Match.
- A MatchEvent is performed by a Player.
- A MatchEvent may optionally reference another related MatchEvent.
- A PlayerMatchAppearance connects a Player, Match, and Team.
- A PlayerMatchAppearance represents the player's actual participation
  in that match.
- A player who never enters the match does not have a
  PlayerMatchAppearance.
- A Player can have one PlayerMatchStats record for each match in which
  they participate.
- A Player can have one PlayerMatchRating for each match in which they
  participate.
- PlayerMatchStats are calculated from MatchEvents.
- PlayerMatchRating is calculated using PlayerMatchStats and
  PlayerMatchAppearance.
- A Player is not directly assigned to a Team in the core model.
- Historical team membership is represented through
  PlayerMatchAppearance.

Conceptually:

    Team
      │
      └── participates in
              │
            Match
              │
              ├── MatchEvent
              │      │
              │      └── performed by → Player
              │
              ├── PlayerMatchAppearance
              │      ├── Player
              │      └── Team
              │
              ├── PlayerMatchStats
              │      └── Player
              │
              └── PlayerMatchRating
                     └── Player

    MatchEvent
         │
         └── may reference → another MatchEvent

---

## 4. Derived Data

MatchPulse distinguishes between stored domain facts and values that can
be derived from those facts.

Examples of values that should normally be derived include:

- pass completion percentage
- minutes played
- recent team form
- season totals
- current-vs-previous appearance comparisons

For example, season pass completion percentage should be calculated
using total completed passes and total attempted passes:

    totalPassesCompleted / totalPassesAttempted * 100

It should not be calculated by averaging individual match percentages.

Derived data may later be cached or stored for performance reasons, but
that should be an implementation decision based on an actual
performance requirement.

---

## 5. Domain Principles

The MatchPulse domain model follows several principles.

### Events Are Historical Facts

MatchEvents represent what happened during a match and should be
retained after processing.

### Statistics Are Calculations

PlayerMatchStats are calculated from MatchEvents and should be
rebuildable when necessary.

### Ratings Are Evaluations

PlayerMatchRating represents MatchPulse's evaluation of a player's
performance rather than an objective match fact.

The rating algorithm may evolve independently of the underlying events
and statistics.

### Historical Information Must Remain Correct

Changes to current information, such as a player's future team, should
not alter historical match information.

PlayerMatchAppearance therefore records which Team a Player represented
in a specific Match.

### Avoid Unnecessary Stored Data

Values that can be cheaply and reliably calculated from existing domain
data should generally be derived rather than stored independently.

### Keep the Model Focused

The domain model should contain concepts required by MatchPulse rather
than attempting to model every aspect of professional football.

New fields and entities should be introduced when requirements justify
them.

---

## 6. Open Domain Questions

The following domain questions remain intentionally unresolved:

### Player Rating Algorithm

Research is required to determine:

- rating scale
- statistical weights
- position-specific weighting
- playing-time effects
- substitute handling
- goalkeeper-specific rules
- positive and negative contributions

### Event Provider Mapping

The eventual real football data provider may represent events
differently from MatchPulse.

A mapping strategy will be needed between provider events and the
MatchPulse MatchEvent model.

### Event Corrections

A real provider may later correct an event after it has already been
processed.

The exact approach for handling corrected or withdrawn events will be
designed when the event processing architecture is explored.

### Additional Football Concepts

Concepts such as competitions, seasons, detailed player roles, and
multi-team tracking are outside the current MVP and should only be added
when required.

---

## 7. Current Domain Flow

At a high level, the MatchPulse domain behaves as follows:

    Match
      ↓
    MatchEvents
      ↓
    PlayerMatchStats
      +
    PlayerMatchAppearance
      ↓
    PlayerMatchRating

MatchEvents provide the detailed record of what happened.

PlayerMatchStats summarize measurable player actions.

PlayerMatchAppearance provides participation context such as starting
status and substitution timing.

PlayerMatchRating evaluates the player's overall performance using those
facts.

This domain model defines the football concepts MatchPulse needs without
making decisions about databases, APIs, messaging systems, deployment,
or other infrastructure.