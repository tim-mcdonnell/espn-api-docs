# Injury to Play/Event Relationships

This section documents the enhanced relationships between injuries and specific game events and plays. These relationships allow for precise tracking of when and how injuries occur during games, enabling detailed analysis of injury patterns and their impact on game outcomes.

## InjuryPlayEvents

Links specific injuries to the exact plays where they occurred.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| injury_play_id | INTEGER | Primary key (auto-increment) | 1 |
| athlete_injury_id | INTEGER | Foreign key to AthleteInjuries | 253 |
| play_id | INTEGER | Foreign key to Plays | 6721 |
| event_id | INTEGER | Foreign key to Events | 401524661 |
| certainty_level | VARCHAR | Confidence in play connection (confirmed, probable, possible) | confirmed |
| detected_mechanism | VARCHAR | Injury mechanism detected from play | contact |
| contact_type | VARCHAR | Type of contact if applicable | player-to-player |
| video_evidence | BOOLEAN | If video evidence exists | true |
| video_url | VARCHAR | URL to video evidence if available | https://media.espn.com/injury/401524661-6721.mp4 |
| notes | VARCHAR | Additional contextual notes | Non-contact injury on landing |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12T18:23:01Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12T18:23:01Z |

**Indexing:**
```sql
CREATE INDEX idx_injury_play_athlete ON InjuryPlayEvents(athlete_injury_id);
CREATE INDEX idx_injury_play_event ON InjuryPlayEvents(event_id);
CREATE INDEX idx_injury_play_play ON InjuryPlayEvents(play_id);
```

## InjuryGameImpact

Tracks the impact of injuries on specific games.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| impact_id | INTEGER | Primary key (auto-increment) | 1 |
| event_id | INTEGER | Foreign key to Events | 401524661 |
| team_id | INTEGER | Foreign key to Teams | 2509 |
| athlete_injury_id | INTEGER | Foreign key to AthleteInjuries | 253 |
| minutes_missed | DECIMAL | Minutes missed due to injury in this game | 32.5 |
| preexisting_injury | BOOLEAN | If injury occurred before this game | true |
| replacement_athlete_id | INTEGER | Athlete who replaced the injured player | 4066490 |
| team_performance_impact | DECIMAL | Estimated impact on team performance (-10.0 to 10.0) | -4.2 |
| key_position | BOOLEAN | If injury was to a key position | true |
| game_plan_adjustment | VARCHAR | How team adjusted game plan | Modified defensive assignments |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12T18:23:01Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12T18:23:01Z |

**Indexing:**
```sql
CREATE INDEX idx_injury_impact_event ON InjuryGameImpact(event_id);
CREATE INDEX idx_injury_impact_team ON InjuryGameImpact(team_id, event_id);
CREATE INDEX idx_injury_impact_athlete ON InjuryGameImpact(athlete_injury_id);
```

## InjuryPatternAnalysis

Aggregates injury patterns for analytical purposes.

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| pattern_id | INTEGER | Primary key (auto-increment) | 1 |
| injury_type_id | INTEGER | Foreign key to InjuryTypes | 12 |
| season_id | INTEGER | Foreign key to Seasons | 2024 |
| play_type_id | INTEGER | Foreign key to PlayTypes | 404 |
| occurrence_count | INTEGER | Number of occurrences | 15 |
| court_zone_id | INTEGER | Foreign key to CourtZones (if applicable) | 3 |
| period_distribution | VARCHAR | JSON distribution by period | {"1":5,"2":8,"OT":2} |
| game_time_distribution | VARCHAR | JSON distribution by game time | {"0-5":2,"5-10":4,"35-40":9} |
| fatigue_correlation | DECIMAL | Correlation with minutes played (0.0-1.0) | 0.72 |
| contact_pct | DECIMAL | Percentage involving contact | 65.0 |
| common_scenarios | VARCHAR | JSON array of common scenarios | ["fast break defense","landing after rebound"] |
| created_at | TIMESTAMP | Record creation timestamp | 2024-03-12T18:23:01Z |
| updated_at | TIMESTAMP | Record update timestamp | 2024-03-12T18:23:01Z |

**Indexing:**
```sql
CREATE INDEX idx_injury_pattern_type ON InjuryPatternAnalysis(injury_type_id, season_id);
CREATE INDEX idx_injury_pattern_play ON InjuryPatternAnalysis(play_type_id, season_id);
```

## Implementation Guidelines

When implementing these injury-play relationships, follow these guidelines:

1. **Automated Detection**: Develop algorithms to automatically detect potential injury plays based on:
   - Play descriptions containing injury-related terms
   - Substitution patterns immediately following plays
   - Timeout patterns and medical staff entering the court
   - Player tracking data showing sudden stops or unusual movements

2. **Manual Verification**: Establish a process for manual verification of injury-play connections:
   - Review game footage for confirmation
   - Update certainty_level as additional evidence becomes available
   - Document verification in notes field

3. **Historical Backfilling**: For historical injuries:
   - Prioritize significant injuries to key players
   - Use available game logs and play-by-play data to identify potential plays
   - Mark historical connections with appropriate certainty levels

4. **Privacy Considerations**: Ensure that:
   - Sensitive medical details remain in the core injury tables, not in the relationship tables
   - Public-facing applications respect athlete privacy regarding injury details

## Example Queries

**Find all plays that resulted in knee injuries this season:**

```sql
SELECT p.text, p.clock_time, p.period, e.name as game, a.full_name as player, 
       ipe.detected_mechanism, ipe.certainty_level
FROM InjuryPlayEvents ipe
JOIN Plays p ON ipe.play_id = p.play_id
JOIN Events e ON ipe.event_id = e.event_id
JOIN AthleteInjuries ai ON ipe.athlete_injury_id = ai.athlete_injury_id
JOIN Injuries i ON ai.injury_id = i.injury_id
JOIN BodyParts bp ON i.body_part_id = bp.body_part_id
JOIN Athletes a ON ai.athlete_id = a.athlete_id
WHERE bp.name = 'Knee'
AND e.season_id = ?
ORDER BY e.start_date DESC;
```

**Analyze which play types most commonly result in injuries:**

```sql
SELECT pt.name as play_type, it.name as injury_type, 
       COUNT(*) as occurrences,
       AVG(DATEDIFF('day', ai.injury_date, COALESCE(ai.actual_return_date, CURRENT_DATE))) as avg_recovery_days
FROM InjuryPlayEvents ipe
JOIN Plays p ON ipe.play_id = p.play_id
JOIN PlayTypes pt ON p.play_type_id = pt.play_type_id
JOIN AthleteInjuries ai ON ipe.athlete_injury_id = ai.athlete_injury_id
JOIN Injuries i ON ai.injury_id = i.injury_id
JOIN InjuryTypes it ON i.injury_type_id = it.injury_type_id
GROUP BY pt.name, it.name
ORDER BY occurrences DESC;
``` 