# Progress and Next Steps

This section documents our current progress on the ESPN API data structure and outlines planned future work. It serves as a roadmap for ongoing development and identifies areas that require further attention.

## Current Progress

The following table summarizes the status of each major component of our data structure:

| Entity | Status | Notes |
|--------|--------|-------|
| Leagues | Mapped | Basic structure established |
| Seasons | Mapped | Basic structure established |
| Season Types | Mapped | Basic structure established |
| Weeks | Mapped | Basic structure established |
| Franchises | Mapped | Basic structure established |
| Teams | Mapped | Basic structure established |
| Groups | Mapped | Basic structure established |
| Venues | Mapped | Basic structure established |
| Athletes | Mapped | Basic structure established |
| Positions | Mapped | Basic structure established |
| Events | Mapped | Basic structure established |
| Rankings | Mapped | Basic structure established |
| Statistics | Refined | Enhanced with detailed dimensions and facts |
| Play-by-Play | Implemented | Complete data structure with plays, participants, streaks and game flow |
| Box Scores | Added | Period-by-period scoring |
| Shot Charts | Implemented | Complete spatial shot data structure with defensive tracking |
| Indexing | Added | DuckDB-specific indexing strategy defined |
| Ranking Systems | Added | Complete poll data structures |
| Betting Odds | Implemented | Complete structure for pre-game odds, line movements, and provider tracking |
| Team Performance Metrics | Implemented | Comprehensive framework for efficiency, tempo-free, and comparison metrics |

## Technical Considerations

As we continue development, we need to address these technical considerations:

1. **Reference Resolution**: Create a strategy to properly resolve ESPN API's `$ref` fields into normalized database entities.

2. **Temporal Data Management**: Enforce consistent UTC time formatting for all date/time fields to support historical analysis.

3. **Memory Management**: Implement regular maintenance routines to manage DuckDB's memory usage for indexes.

4. **Data Synchronization**: Design pipelines to handle real-time updates during active game periods.

5. **Data Retention**: Define policies for historical data retention and archiving strategies.

6. **API Rate Limiting**: Implement request throttling to respect ESPN's API rate limits.

7. **Error Handling**: Develop robust error handling for API unavailability or schema changes.

## Open Questions

Several questions remain open and require further investigation:

- How should we handle very sparse statistical fields that only apply to certain athletes or game situations?

- What is the best storage format for historical time-series analysis of player development?

- Should we implement a separate star schema for analytical queries on top of this normalized structure?

- How frequently should aggregate statistics be recalculated?

- What is the optimal backup strategy for a DuckDB-based analytics database?

- How should we handle ESPN API schema changes or deprecations?

- What is the proper approach for handling team identity changes (rebrands, relocations) over time?

## Out of Scope Features

The following features are available in the ESPN API but have been intentionally excluded from our data structure as they do not align with our current analytical goals:

### Media Content and Rich Media

- **Video Content References**: The API includes video content in event summaries, but storing these references adds complexity without analytical value.

- **News Articles and Headlines**: While game headlines are captured in event data, full article content is not stored as it's primarily for display purposes.

- **Image Management System**: Player headshots, venue images, and additional visual content references are not systematically stored beyond basic team logos needed for identification.

### Web and Social Integration 

- **Link Structure**: The API returns various web links for teams, events, and athletes, but these are primarily for UI navigation and not relevant for analysis.

- **Social Media Content**: Any social media references or engagement metrics are not captured as they don't contribute to performance analytics.

### Fan Engagement and User Data

- **User-Generated Content**: Comments, ratings, or other user-generated content are not included in our data model.

- **Attendance Trends**: While game attendance is stored as a basic fact, detailed attendance demographics and engagement patterns are not tracked.

### Supplementary Content

- **Generic Notes Structure**: The API's notes fields for various entities are selectively stored only when they contain analytically relevant information.

- **Documentation References**: Links to rules, commentary, or background information are excluded.

### Broadcast Information

- **Network and Channel Data**: Television network, streaming platform, and broadcast channel information is not tracked due to limited predictive value.

- **Broadcast Teams**: Commentator, analyst, and production team information does not provide analytical benefit for game outcome prediction.

- **Regional Coverage**: Geographic broadcast restrictions and regional network information is excluded as it has no direct relation to game analysis.

### Award Information

- **Individual and Team Awards**: Player of the year, all-conference teams, and other achievement awards are excluded as they represent lagging indicators with minimal predictive value.

- **Award Voting Details**: Voting breakdowns, voter affiliations, and historical award patterns are not stored.

- **Award Ceremonies**: Information about when and where awards are presented holds no analytical value for game prediction.

### Coaching Staff Information

- **Coaching Hierarchies**: Assistant coach relationships, staff structure, and coaching trees are excluded as their impact is difficult to quantify in predictive models.

- **Career Trajectories**: Historical coaching movement between programs does not offer significant predictive value.

- **Coaching Statistics**: Win-loss records, championships, and other coaching accomplishments are only indirectly related to current team performance and are not included.

### Officials and Referees Information

- **Officiating Assignments**: Game official assignments and crew information is excluded as the impact on game outcomes is minimal and inconsistent.

- **Officiating Tendencies**: Referee foul-calling tendencies and game management styles are not tracked due to their limited predictive value.

- **Official Performance Metrics**: Metrics tracking official accuracy, consistency, and bias are excluded as they're not reliably available and offer limited analytical benefit.

These items may be reconsidered in future iterations if analytical use cases emerge that require this data.

## Next Steps: Implementation Recommendations

Based on a thorough review of our database structure, the following recommendations are organized by priority for implementation:

### High Priority

1. **Standardize ID Field Naming**: ✅ Completed
   - ✅ Created comprehensive ID field standardization documentation in `id-field-standardization.md`
   - ✅ Implemented field naming conventions consistently across all database tables
   - ✅ Standardized table naming to PascalCase (e.g., INJURY_TYPES → InjuryTypes)
   - ✅ Unified data types (all ESPN IDs using VARCHAR, proper timestamp formats)
   - ✅ Added missing metadata fields (created_at/updated_at) to all tables
   - ✅ Added proper universal identifier (uid) fields to applicable tables
   - ✅ Fixed all SQL examples to use the standardized naming conventions

2. **Implement Data Integrity Framework**: Develop application-level integrity checks to compensate for DuckDB's limited foreign key constraint support.
   - Create validation procedures that run during data loading
   - Implement automated tests to verify referential integrity
   - Document integrity verification processes

3. **Optimize Constraint Implementation**: Strategically implement foreign key and other constraints considering DuckDB's characteristics.
   - Define constraints after bulk data loading to minimize performance impact
   - Add validation logic to handle DuckDB's "over-eager constraint checking" limitation with UPDATEs
   - Implement integrity checks to ensure data consistency during incremental updates
   - Document constraint strategy with clear rationale for implementation choices

4. **Enhance Statistical Storage Consistency**: Address potential redundancy and consistency challenges in the statistical structure.
   - Develop clear update protocols for statistics at different aggregation levels
   - Implement verification checks to ensure consistency across related tables
   - Create documentation for maintaining data quality during statistical updates

5. **Clarify Complex Relationship Documentation**: Strengthen documentation for complex entity relationships, particularly in tournament structures.
   - Create detailed entity-relationship diagrams for tournament advancement
   - Document the lifecycle of tournament entities with clear state transitions
   - Add examples of how entities relate through a complete tournament cycle

### Medium Priority

1. **Add Missing Entity Relationships**: ✅ Completed
   - ✅ Created `InjuryPlayEvents`, `InjuryGameImpact`, and `InjuryPatternAnalysis` tables to strengthen connections between injuries and specific plays
   - ✅ Created `TeamRankingPerformance`, `RankingPerformanceCorrelation`, and other tables to connect team performance metrics with rankings
   - ✅ Updated entity diagrams and documentation to reflect these new relationships

2. **Develop Materialized View Strategy**: Create a comprehensive approach for managing materialized views.
   - Document refresh schedules based on data change frequency
   - Implement automated view maintenance procedures
   - Balance performance optimization with resource usage

3. **Enhance Partitioning Documentation**: Provide more detailed implementation guidelines for time-based partitioning.
   - Create specific examples for partitioning large fact tables by season/year
   - Document partition management procedures (creation, merging, pruning)
   - Address query optimization across partitions

4. **Refine Data Type Specificity**: Review column data types for opportunities to use more specific types.
   - Create custom enumeration types for fields with fixed value sets (status, home/away)
   - Document domain constraints for numeric fields
   - Standardize handling of temporal data types

### Low Priority

1. **Evaluate Analytical Schema Implementation**: Consider creating a star schema layer optimized for analytical queries.
   - Assess query patterns and performance needs
   - Design dimensional models for common analytical paths
   - Document transformation processes between normalized and star schemas

2. **Develop Team Identity Change Tracking**: Create a formal approach for handling team rebrands, relocations, and identity changes.
   - Design history-tracking mechanism for franchise changes
   - Document how historical records should be associated with current entities
   - Implement versioning system for team identities

3. **Create Comprehensive Update Frequency Guidelines**: Document how often different tables should be refreshed.
   - Define update requirements based on entity volatility
   - Create schedules for recalculating aggregate statistics
   - Balance freshness requirements with processing costs

4. **Implement Systematic Index Performance Review**: Establish a process for regularly reviewing and optimizing the indexing strategy.
   - Create index performance metrics and monitoring
   - Document procedures for evaluating index utility
   - Implement index maintenance routines
