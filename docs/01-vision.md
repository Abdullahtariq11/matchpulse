# MatchPulse — Product Vision

## 1. Why Am I Building This?

### Personal Motivation

I am building MatchPulse because I am interested in football and want
to build a project around something I genuinely enjoy.

I am a Manchester United fan, so the initial scope of MatchPulse will
focus on Manchester United matches and players.

From a technical perspective, I want this project to help me understand
how real-time and event-driven systems are designed and built.

The goal is not simply to use technologies such as Kafka, RabbitMQ,
Redis, or Elasticsearch. I want to understand what problems these
technologies solve and determine whether MatchPulse actually needs them.

### Learning Goals

Through this project, I want to improve my understanding of:

- Event-driven architecture
- Real-time data processing
- Backend development
- Frontend development
- API design
- Database design
- Asynchronous processing
- Real-time communication with clients
- System design
- Testing
- Containerization and deployment

I also want to practice making and documenting engineering decisions
instead of selecting technologies simply because they are popular.


---

## 2. What Problem Am I Solving?

Football matches generate many events during a game, including passes,
shots, goals, assists, tackles, dribbles, fouls, saves, and other actions.

Individual events alone do not provide a complete picture of how a
player is performing.

MatchPulse will process match events and transform them into useful
player statistics and performance information.

The application should allow a football fan to understand how a
Manchester United player is performing during a match and how that
performance compares with the player's previous matches.


---

## 3. Who Is the User?

The initial target users are:

- Manchester United supporters
- Football enthusiasts
- Fantasy Premier League players
- Users interested in player statistics and performance

A user should be able to open MatchPulse during or after a Manchester
United match and understand how individual players are performing using
data rather than only the final score.


---

## 4. Core Product Idea

MatchPulse is primarily a:

**Manchester United player performance analysis platform.**

It is not intended to initially compete with full live-score
applications covering every team, league, and competition.

The primary focus is individual player performance.

MatchPulse will receive or generate football match events, process those
events, calculate player statistics, and present the resulting
information to users.

Over time, historical match data should allow users to see how a
player's performance changes across multiple matches.


---

## 5. Player Performance

Player performance will initially be represented using measurable
football statistics.

Examples include:

- Passes attempted
- Passes completed
- Pass completion percentage
- Shots
- Shots on target
- Goals
- Assists
- Dribbles attempted
- Dribbles completed
- Crosses attempted
- Crosses completed
- Tackles
- Fouls
- Chances created
- Big chances created
- Saves for goalkeepers

Some statistics will be directly counted from events while others will
be derived.

For example:

Pass Completion Percentage =
Passes Completed / Passes Attempted

The exact set of statistics may evolve as the project develops.


---

## 6. What Is a Match Event?

A match event represents something that happened during a football
match.

Examples include:

- Pass
- Shot
- Goal
- Assist
- Dribble
- Tackle
- Foul
- Penalty
- Save
- Cross
- Yellow card
- Red card
- Substitution

An event should contain enough information for MatchPulse to understand
what happened and which player or players were involved.

Possible information includes:

- Match
- Player or players involved
- Team
- Event type
- Match time
- Timestamp
- Event outcome

The exact event model will be designed later.


---

## 7. Event Processing

MatchPulse should derive player statistics from match events rather
than receiving only pre-calculated statistics.

For example, if the system receives:

PASS -> Bruno -> SUCCESS
PASS -> Bruno -> SUCCESS
PASS -> Bruno -> FAILED
SHOT -> Bruno -> ON_TARGET

MatchPulse should be able to derive:

Passes Attempted: 3
Passes Completed: 2
Pass Completion: 66.7%

Shots: 1
Shots on Target: 1

This means the application will need to process events and maintain
player statistics as a match progresses.


---

## 8. Real-Time Expectations

MatchPulse should provide near-real-time updates during a match.

For the initial version, an event should normally be reflected in the
user interface within approximately:

**10 seconds**

For example, if a player takes a shot, the player's shot statistics and
the match timeline should update within approximately 10 seconds.

The exact technical implementation required to achieve this has not yet
been decided.


---

## 9. Data Source

The long-term goal is for MatchPulse to operate using real football
match data.

However, the initial development version may use a match event
simulator.

The simulator will allow controlled events to be generated while the
event-processing system is being developed and tested.

Example:

Match Simulator
|
| PASS
| SHOT
| GOAL
| TACKLE
v
MatchPulse

Once the core system works correctly, real football data sources can be
researched and potentially integrated.

The real-data provider has not yet been selected.


---

## 10. Player Performance Rating

MatchPulse should eventually provide an overall player performance
rating.

Example:

Player Rating: 8.2 / 10

However, the rating algorithm will NOT be invented without research.

Football players perform different roles, and players in different
positions should not necessarily be evaluated using identical metrics.

For example:

A striker may be strongly influenced by:

- Goals
- Shots
- Shots on target
- Chances created
- Dribbles

A midfielder may be influenced by:

- Passing
- Chances created
- Assists
- Ball progression
- Tackles

A defender may be influenced by:

- Tackles
- Interceptions
- Clearances
- Duels
- Defensive mistakes

A goalkeeper may be influenced by:

- Saves
- Goals conceded
- Distribution
- Other goalkeeper-specific actions

Therefore, the player rating system requires additional research before
implementation.

The final MatchPulse rating model should be documented separately,
including the reasoning behind the metrics and weightings selected.


---

## 11. Historical Performance

MatchPulse should eventually allow users to compare a player's
performance across matches.

For example:

Bruno Fernandes

                 ARS    LIV    CHE    TOT    NEW

Shots             4      2      5      1      3
Shots on Target   2      1      3      0      2
Pass Completion  85%    79%    88%    91%    84%
Chances Created   3      1      4      2      2

This will allow users to identify trends in player performance rather
than looking at only one match.


---

## 12. MVP

The first version of MatchPulse will focus on Manchester United.

A successful MVP should demonstrate the following scenario:

1. A Manchester United match is started in the system.
2. Match events begin arriving from a simulator.
3. A user opens the MatchPulse frontend.
4. The user can see the current match.
5. Events occur during the simulated match.
6. MatchPulse processes those events.
7. Player statistics are updated.
8. The match timeline is updated.
9. The frontend receives the updated information.
10. The user can view the performance statistics of Manchester United
    players without manually refreshing the page.

The goal of the MVP is to demonstrate the complete flow:

Match Event
->
Event Processing
->
Player Statistics
->
Backend
->
Live Frontend


---

## 13. Not Part of the MVP

The following features are intentionally excluded from the initial MVP.

### Multiple Leagues

The MVP will not attempt to support every football league or team.

The initial focus is Manchester United.


### Player Position Tracking

Continuous player position data will not initially be processed.

Therefore, metrics requiring tracking data, such as:

- Distance covered
- Heat maps
- Average position
- Running speed
- Player movement

are outside the MVP.

These features may be explored later if appropriate tracking data
becomes available.


### AI Performance Analysis

AI-generated analysis will not initially be included.

A future version could potentially analyze historical player data and
identify trends such as:

- Improving performance
- Declining performance
- Changes in playing style
- Strengths and weaknesses
- Differences between recent matches

AI should operate on meaningful historical data already collected by
MatchPulse rather than being the foundation of the MVP.


### Real Data Integration

The first development version may use simulated match events.

Integration with a real football data provider will happen after the
core event-processing system works.


---

## 14. Research Required

The following areas require research before implementation decisions
are made.

### Player Rating System

Questions include:

- How are football player ratings typically calculated?
- What metrics matter for different positions?
- How should positive events affect a rating?
- How should negative events affect a rating?
- Should players start with a baseline rating?
- How should playing time affect ratings?
- How should substitutes be evaluated?
- How frequently should a live rating change?
- How do existing football analytics platforms approach player ratings?


### Real Football Data

Research is required to determine:

- What real football data providers are available?
- Which providers expose event-level data?
- Is live data available?
- What does access cost?
- What restrictions exist on storing the data?
- What rate limits exist?
- Which statistics are available?
- Is detailed event data available for Manchester United matches?


---

## 15. Open Questions

The following questions have not yet been answered:

- How exactly should a player performance rating be calculated?
- Should different player positions use different rating models?
- What real football data provider should eventually be used?
- What should the final match event model look like?
- How should related events be represented?
- For example, is a saved shot represented as a SHOT event followed by
  a SAVE event?
- How should duplicate events be handled?
- What happens if events arrive out of order?
- How should MatchPulse recover if processing fails?
- How should historical statistics be stored?
- How should live updates reach the frontend?


---

## 16. Technologies I Want to Explore

The following technologies are technologies I am interested in
exploring during the project:

- Spring Boot
- React
- PostgreSQL
- Kafka
- RabbitMQ
- Elasticsearch
- Redis
- WebSockets
- Docker

These are NOT currently architecture decisions.

A technology should only be introduced when MatchPulse has a problem
that justifies using it.

Before introducing a major technology, I should be able to answer:

1. What problem am I trying to solve?
2. Can the existing architecture solve it?
3. What alternatives exist?
4. Why am I choosing this technology?
5. What additional complexity does it introduce?
6. What would happen if I removed it?


---

## 17. Development Philosophy

MatchPulse is intended to be a learning project as much as a software
project.

The development process should follow:

Understand
->
Decide
->
Document
->
Implement
->
Test
->
Review

Architecture should evolve from requirements rather than selecting
technologies first.

The objective is not simply to make MatchPulse work.

The objective is to understand why it works and why each major
engineering decision was made.