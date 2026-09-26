# MatchPulse Requirements

## 1. Purpose

This document defines the functional and non-functional requirements for
the MatchPulse MVP.

MatchPulse is a near-real-time football player performance analysis
platform initially focused on Manchester United.

The purpose of the MVP is to demonstrate the complete flow of receiving
match events, processing those events, calculating player statistics,
displaying live performance information, and retaining completed match
data for historical analysis.

The MVP will initially use simulated football match data rather than a
real football data provider.

---

# 2. Functional Requirements

## 2.1 Matches

MatchPulse will track Manchester United matches while they are being
played.

When a real match begins, MatchPulse should begin receiving and
processing match events.

While the match is live, MatchPulse should:

- Receive match events.
- Process events as they occur.
- Update Manchester United player statistics.
- Update player performance information.
- Send updated information to connected users.

When the match ends, MatchPulse should stop treating the match as live.

The final match events and calculated player statistics should remain
available so they can later be used for historical player performance
analysis.

For the MVP, the real match data source will be simulated.

### Match Recovery

MatchPulse should not permanently lose match events if the application
temporarily becomes unavailable during a live match.

If MatchPulse becomes unavailable and later recovers while the match is
still in progress, it should retrieve and process the events that were
missed during the downtime.

The recovered events should update player statistics so that the current
statistics remain consistent with what actually happened in the match.

The implementation of this recovery mechanism will be decided during
system design.

---

## 2.2 Match Events

While a match is live, MatchPulse should receive and process match
events.

Each event should represent an action that occurred during the match
and contain enough information to determine how the action affects
player statistics.

Examples of events include:

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

Events may have different outcomes.

For example, a SHOT event may have an outcome such as:

- GOAL
- SAVED
- BLOCKED
- OFF_TARGET

The event and its outcome determine which player statistics should
change.

For example:

SHOT + GOAL:

- Shots attempted +1
- Shots on target +1
- Goals +1

SHOT + SAVED:

- Shots attempted +1
- Shots on target +1

SHOT + BLOCKED:

- Shots attempted +1

Related player actions may be represented as separate events.

For example, a goal assisted by another player may produce:

- A SHOT event with outcome GOAL for the goalscorer.
- An ASSIST event for the assisting player.

A goalkeeper save may similarly be represented separately from the
attacking player's shot.

Related events may need to be associated with each other. The exact
mechanism for doing this will be decided during domain and system
design.

### Duplicate Events

MatchPulse should prevent the same match event from affecting player
statistics more than once.

Events should have a way of being uniquely identified.

If an event that has already been processed is received again,
MatchPulse should recognize it as a duplicate and should not update
player statistics again.

The exact mechanism for detecting duplicate events will be decided
during system design.

### Event Ordering

Match events may not always arrive at MatchPulse in the same order in
which they occurred.

Events should contain ordering information, such as a sequence number,
that allows MatchPulse to determine their correct order.

MatchPulse should be able to process late or out-of-order events without
losing them or incorrectly calculating player statistics.

The exact mechanism for handling event ordering will be decided during
system design.

### Missing Events

MatchPulse should detect when one or more match events may have been
missed.

If event ordering information indicates a gap, MatchPulse should attempt
to retrieve the missing event or events from the data source.

Recovered events should still be processed so that player statistics
remain accurate.

MatchPulse should not silently treat incomplete event data as complete.

The exact recovery mechanism will be decided during system design.

---

## 2.3 Player Statistics

MatchPulse should calculate player statistics from the match events it
processes.

Statistics should be maintained separately for each player's
participation in an individual match.

Supported statistics may include:

- Minutes played
- Goals
- Assists
- Passes attempted
- Passes completed
- Shots attempted
- Shots on target
- Dribbles attempted
- Dribbles completed
- Crosses attempted
- Crosses completed
- Tackles
- Interceptions
- Fouls
- Chances created
- Big chances created
- Cards
- Goalkeeper saves where applicable

The exact statistics available may depend on the player's position and
the information available from the match data source.

### Derived Statistics

MatchPulse should avoid storing statistics that can be easily derived
from other stored statistics.

For example, MatchPulse should store:

- Passes attempted
- Passes completed

Pass completion percentage should be calculated from these values when
needed:

    passCompletionPercentage =
        passesCompleted / passesAttempted * 100

This avoids storing duplicate information that could become
inconsistent with the underlying statistics.

### Event Retention

MatchPulse should retain the original match events after they have been
processed.

Player statistics may be calculated and maintained from these events,
but processing an event should not cause the original event to be
deleted.

Keeping the original events should allow MatchPulse to:

- Recalculate statistics if calculation rules change.
- Rebuild statistics if incorrect data is discovered.
- Support historical player analysis.
- Support match timelines.
- Recalculate player ratings when the rating algorithm changes.
- Investigate incorrect statistics or processing problems.

The exact storage and retention strategy will be decided during system
design.

### Per-Match Player Statistics

MatchPulse should maintain player statistics for each individual match.

A player's match statistics should include the underlying values needed
to calculate derived statistics.

For example:

- Passes attempted
- Passes completed
- Shots attempted
- Shots on target
- Goals
- Assists
- Dribbles attempted
- Dribbles completed
- Crosses attempted
- Crosses completed
- Tackles
- Interceptions
- Chances created
- Big chances created

Derived statistics, such as pass completion percentage, should be
calculated from the underlying values when needed.

Historical and season-level statistics can later be calculated by
aggregating the player's individual match statistics.

For example, pass completion across multiple matches should be
calculated using the total completed passes and total attempted passes
rather than simply averaging each match's pass completion percentage.

The MVP does not need to maintain a separate running season-total
record.

### Playing Time and Player Ratings

Playing time should be considered when calculating player performance
ratings.

A player who plays only a small portion of a match should not
necessarily be evaluated in the same way as a player who plays the
entire match.

The rating system should eventually consider situations such as:

- Starting players
- Substitutes
- Players substituted off
- Players receiving limited minutes
- Extra time

The exact effect of playing time on player ratings requires research
and will be defined when the player rating algorithm is designed.

---

## 2.4 Player Performance

During a live match, users should be able to select a Manchester United
player and view their current match performance.

The player view should display relevant statistics such as:

- Player performance rating
- Minutes played
- Goals
- Assists
- Passes attempted
- Passes completed
- Pass completion percentage
- Shots attempted
- Shots on target
- Dribbles attempted
- Dribbles completed
- Crosses attempted
- Crosses completed
- Chances created
- Big chances created
- Tackles
- Interceptions
- Fouls
- Cards

The statistics displayed may depend on the player's position.

For example, goalkeeper statistics may include saves and other
goalkeeper-specific metrics.

Player statistics should update during the match as new events are
processed.

The exact player rating calculation will be defined separately after
research.

### Recent Match Comparison

During a live match, the player performance view should allow users to
compare the player's current performance with their most recent match.

The comparison should show relevant statistics from both matches, such
as:

- Player rating
- Goals
- Assists
- Pass completion
- Shots and shots on target
- Dribbles completed
- Crosses completed
- Chances created
- Tackles
- Interceptions

The previous match should be the player's most recent appearance for
Manchester United.

For example, if Manchester United played a match but the player did not
participate, that match should not be considered the player's previous
appearance.

The MVP will initially compare the current match with only the most
recent match rather than providing full historical trend analysis.

---

## 2.5 Live Updates

During a live match, MatchPulse should automatically update the
information displayed to users when new match events are processed.

Users should not need to manually refresh the page to see updated
player statistics or match information.

Under normal conditions, a processed match event should be reflected
in the user interface within approximately 10 seconds of the event
occurring.

The system should avoid requiring clients to continuously make
unnecessary requests when no match data has changed.

The communication mechanism used to deliver live updates will be
decided during system design.

---

## 2.6 Match Timeline

MatchPulse should provide a live timeline of important events that occur
during a match.

The system may process and retain many detailed match events for player
statistics and performance calculations, but not every processed event
needs to be displayed on the user-facing timeline.

For example, individual passes may be processed for player statistics
without appearing on the match timeline.

The timeline should focus on important events such as:

- Goals
- Assists
- Shots on target
- Big chances
- Penalties
- Yellow cards
- Red cards
- Substitutions

Not every event that contributes to player statistics needs to appear
on the timeline.

For example, a routine goalkeeper save may contribute to the
goalkeeper's statistics without necessarily appearing on the main
timeline.

Timeline events should appear automatically while the match is live
without requiring the user to refresh the page.

The timeline should display events in the order in which they occurred,
even if events are received by MatchPulse out of order.

The exact rules for determining which events are considered important
may evolve as the product is developed.

---

## 2.7 Historical Data

After a match has finished, MatchPulse should retain the match and allow
users to open it later.

A completed match should allow users to view:

- Final match information
- Important match timeline events
- Manchester United players who participated
- Final player statistics
- Final player performance ratings
- Goals and assists
- Passing statistics
- Shooting statistics
- Dribbling statistics
- Crossing statistics
- Defensive statistics
- Other supported performance metrics

The statistics shown for a completed match should represent the final
processed state of that match.

Historical match data should also be available for use when comparing a
player's current performance with their most recent previous appearance.

The MVP does not need to provide advanced historical analytics across
an entire season.

More detailed trends and long-term analysis can be added later.

---

# 3. Non-Functional Requirements

## 3.1 Performance

MatchPulse is a near-real-time system rather than a system requiring
millisecond-level updates.

During a live match, a match event should normally be processed and
reflected in the user interface within approximately 10 seconds of the
event occurring.

This includes the complete flow from receiving the event, processing
it, updating player statistics, and delivering the updated information
to the user.

The MVP does not require sub-second processing.

More detailed internal performance targets may be introduced later if
testing shows that a particular part of the system is causing excessive
delay.

---

## 3.2 Reliability

MatchPulse should not permanently lose valid match events because of a
temporary processing failure.

If processing an event fails, MatchPulse should retry processing the
event rather than immediately discarding it.

A processing failure should not cause the same event to be counted
multiple times.

MatchPulse should support:

- Duplicate event detection
- Out-of-order event handling
- Missing event detection
- Recovery of missed events after downtime
- Retry of temporarily failed events

If an event continues to fail after multiple attempts, MatchPulse
should preserve enough information to investigate or process the event
later rather than silently deleting it.

The exact retry and failure-handling strategy will be decided during
system design.

---

## 3.3 Scalability

MatchPulse is initially intended to support a relatively small number
of concurrent users.

For the MVP, the system should comfortably support approximately
20–30 concurrent users during a live match.

The system should be designed with an initial goal of supporting up to
approximately 100 concurrent users without significant degradation in
the live match experience.

The MVP does not need to be designed for thousands or millions of
concurrent users.

The architecture should avoid unnecessary design decisions that would
make future scaling significantly more difficult.

---

## 3.4 Data Integrity

MatchPulse should maintain player statistics that are as accurate and
consistent as possible with the match events received from the data
source.

The system should avoid introducing incorrect statistics during event
processing.

For example:

- The same event should not affect statistics more than once.
- Derived statistics should remain consistent with their underlying
  data.
- Invalid statistical states should be prevented where possible.
- Missing or failed events should not be silently ignored.
- Original match events should be retained so statistics can be
  investigated or recalculated if necessary.

MatchPulse cannot guarantee that information received from an external
data provider is always correct.

If incorrect source data or a processing error is discovered, the
system should support correcting or recalculating affected statistics.

The exact validation and correction mechanisms will be decided during
system design.

---

## 3.5 Availability

MatchPulse should allow users to access historical match and player
performance data even when no Manchester United match is currently
being played.

The live match processing functionality does not need to operate
continuously.

Before a Manchester United match begins, the live processing components
should be available and ready to receive and process match events.

During a live match, the system should remain available whenever
possible.

If live processing becomes temporarily unavailable during a match,
MatchPulse should recover missed events after returning online and
restore the current match state.

Outside of live match periods, components that are only required for
live event processing may be stopped or scaled down, while historical
data remains accessible.

---

# 4. Constraints

## 4.1 Match Data Source

The MVP will not depend on a real football data provider.

Match events will initially be generated by a simulator that represents
the type of event stream MatchPulse could eventually receive from a
real football data provider.

The simulator should produce enough event information to test the
system's:

- Live processing
- Player statistics
- Match timeline
- Duplicate handling
- Event ordering
- Missing-event recovery
- Downtime recovery
- Historical data functionality

Integration with a real football data provider will be considered after
the MVP.

---

## 4.2 Deployment Cost

MatchPulse should be designed so that the MVP can be deployed and
operated at minimal cost.

Where practical, the deployed MVP should use free-tier or inexpensive
infrastructure.

Technology choices should consider operational cost in addition to
their technical benefits.

Technologies that are useful for learning may still be used locally
even if running them as managed cloud services would be unnecessarily
expensive for the MVP.

The project should avoid expensive infrastructure that is not justified
by the MVP requirements.

---

## 4.3 Team Scope

The MatchPulse MVP will focus exclusively on Manchester United matches
and Manchester United player performance.

The MVP does not need to support multiple teams, leagues, or
competitions as independent products.

The design should avoid unnecessarily preventing future support for
additional teams, but multi-team support is outside the MVP scope.

---

## 4.4 Development Schedule

MatchPulse does not have a fixed development deadline.

The project will be developed incrementally during available personal
development time.

The priority is understanding the engineering decisions and technologies
used in the system rather than completing the project by a specific
date.

Features should be implemented in manageable stages, with each major
technology introduced only when its purpose in the system is understood.

---

# 5. Assumptions

## 5.1 Football Data Availability

MatchPulse assumes that a future football data source will provide
enough information to identify and process the match events required by
the system.

The data source is assumed to provide information such as:

- The match associated with an event
- A way to identify individual events
- The player involved in an event
- The type of event
- The outcome of the event when applicable
- Match time
- Information that can be used to determine event ordering

The data source is also assumed to provide enough event detail to
calculate the player statistics supported by MatchPulse.

If a future data provider does not provide some of the required
information, the supported statistics or event-processing design may
need to be adjusted.

---

## 5.2 Event-Level Data

MatchPulse assumes that a future football data source will provide
individual match events rather than only providing pre-calculated
player statistics.

MatchPulse should use these events to calculate and maintain its own
player statistics.

For example, rather than relying only on a provider to report a
player's final pass completion statistics, MatchPulse should be able to
process individual pass events and calculate:

- Passes attempted
- Passes completed
- Pass completion percentage

The same principle should apply to other supported statistics where
sufficient event data is available.

Provider-calculated statistics may eventually be used for comparison
or validation, but MatchPulse should primarily calculate its own
statistics from the events it receives.

---

## 5.3 Event Recovery Support

MatchPulse assumes that the event data source provides a way to retrieve
or replay previously generated match events.

This capability is required so that MatchPulse can recover events that
were missed because of temporary downtime, processing failures, or
other interruptions.

For example, if MatchPulse receives events:

- 501
- 502
- 504

the system should be able to recognize that event 503 may be missing
and request or retrieve the missing event from the data source.

For the MVP, the match simulator should support retrieving or replaying
previously generated events so that recovery behavior can be tested.

A future real football data provider should ideally provide equivalent
functionality.

If it does not, the event recovery strategy will need to be
reconsidered.

---

# 6. Acceptance Criteria

The MatchPulse MVP will be considered functionally complete when the
following behavior can be demonstrated using the match simulator.

## 6.1 Live Match Processing

Given a simulated Manchester United match, MatchPulse should begin
processing events when the match becomes live.

When supported match events are received, the appropriate player
statistics should be updated.

For example, when a successful PASS event is received for a player:

- Passes attempted should increase by one.
- Passes completed should increase by one.
- Pass completion percentage should reflect the updated values.

---

## 6.2 Live User Updates

When a match event changes player statistics, connected users should
see the updated information automatically without manually refreshing
the page.

Under normal conditions, the update should be visible within
approximately 10 seconds of the simulated event occurring.

---

## 6.3 Player Performance

During a live match, a user should be able to select a Manchester United
player and view their current match statistics and performance
information.

The player's information should update as new relevant events are
processed.

---

## 6.4 Match Timeline

Important match events should automatically appear on the live match
timeline.

Detailed events used only for statistics, such as ordinary passes, do
not need to appear on the user-facing timeline.

---

## 6.5 Duplicate Event Handling

If the same event is delivered more than once, it should affect player
statistics only once.

---

## 6.6 Out-of-Order Event Handling

If events arrive in a different order from the order in which they
occurred, MatchPulse should still process them correctly and maintain
the correct match state.

---

## 6.7 Missing Event Recovery

If MatchPulse detects that an event is missing, it should be able to
retrieve or replay the missing event from the simulator and process it.

The recovered event should contribute to player statistics normally.

---

## 6.8 Downtime Recovery

If MatchPulse temporarily stops processing events during a simulated
live match, it should be able to recover the events missed during that
period after processing resumes.

After recovery, player statistics should reflect the complete set of
match events.

---

## 6.9 Failed Event Processing

If processing an event temporarily fails, MatchPulse should retry the
event rather than silently discarding it.

Retrying an event should not cause its effect to be applied more than
once.

---

## 6.10 Completed Matches

When a simulated match finishes, its final statistics, player
performances, ratings, and important timeline events should remain
available.

A user should be able to open the completed match after the live match
has ended.

---

## 6.11 Previous Match Comparison

When a player has participated in a previous Manchester United match,
the player performance view should allow the current match performance
to be compared with that player's most recent previous appearance.

---

## 6.12 Historical Access

Users should be able to access previously completed matches even when
no Manchester United match is currently live.

---

# 7. Open Questions

The following questions are intentionally left unresolved and should be
investigated during later design and research stages.

## 7.1 Player Rating Algorithm

How should MatchPulse calculate an overall player performance rating?

Research is required to determine:

- Which statistics should contribute to the rating.
- How different statistics should be weighted.
- Whether weighting should differ by player position or role.
- How playing time should affect ratings.
- How substitutes and players with limited minutes should be handled.
- Whether positive and negative actions should have different impacts.

The rating formula should not be finalized until existing football
rating approaches have been researched.

---

## 7.2 Real Football Data Provider

Which football data provider should eventually replace the simulator?

Potential providers should be evaluated based on:

- Availability of event-level data.
- Event detail and supported statistics.
- Event identifiers and ordering information.
- Ability to retrieve previous or missed events.
- Live-data latency.
- API limits.
- Reliability.
- Cost.

The MVP will use the simulator, so selecting a real provider is not
required before development begins.

---

## 7.3 Exact Event Model

What information should every MatchPulse event contain?

Likely information includes:

- Event identifier
- Match identifier
- Event type
- Player
- Match time
- Sequence/order information
- Event outcome
- Related player when applicable

Some events may also need to be associated with other events, such as
an ASSIST event being related to the SHOT event that produced a goal.

The exact event schema will be defined during domain design.

---

## 7.4 Live Update Mechanism

How should live match updates be delivered from MatchPulse to connected
users?

Possible approaches may include:

- Polling
- Server-Sent Events
- WebSockets

The mechanism should be selected based on the actual communication
requirements rather than choosing a technology in advance.

---

## 7.5 Event Processing Architecture

How should incoming match events move through MatchPulse?

The architecture needs to support requirements including:

- Event processing
- Duplicate detection
- Event ordering
- Missing event detection
- Event recovery
- Retry after processing failure
- Event retention
- Player statistic updates
- Player rating updates
- Live user updates

Whether these requirements justify technologies such as a message
broker will be determined during system design.

---

## 7.6 Historical Storage

How should original match events, calculated player statistics, match
information, and historical performance data be stored?

The storage design should support:

- Opening completed matches.
- Comparing current and previous player performances.
- Recalculating statistics from original events.
- Recalculating ratings when the rating algorithm changes.

The database structure and storage technologies will be selected during
system design.

---

## 7.7 Deployment Architecture

Which parts of MatchPulse need to remain continuously available and
which parts can be stopped or scaled down when no Manchester United
match is live?

Historical match data should remain accessible, while components used
only for live event processing may not need to operate continuously.

The final deployment approach should also respect the project's
low-cost constraint.