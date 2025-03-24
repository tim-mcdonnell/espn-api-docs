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
| Awards & Honors | Added | Award categories and recipients |
| Broadcast Data | Added | Networks, markets, and broadcast details |
| Officials & Referees | Added | Complete officiating structure with assignments, tendencies, and performance metrics |
| Coaching Staff | Implemented | Complete coaching data structure with roles, history, and coaching trees |
| Betting Odds | Implemented | Complete structure for pre-game odds, line movements, and provider tracking |
| Team Performance Metrics | Implemented | Comprehensive framework for efficiency, tempo-free, and comparison metrics |

## Next Steps

Based on our current progress, we've identified several areas for future development. These are categorized by priority level.

### High Priority

1. **~~Implement Betting Odds Structure~~** ✓
   - ~~Create tables for pre-game odds and lines~~
   - ~~Track line movements over time~~
   - ~~Include over/under and point spread data~~
   - ~~Store odds provider information~~

2. **~~Develop Team Performance Metrics Framework~~** ✓
   - ~~Create advanced metrics calculation framework~~
   - ~~Implement efficiency ratings and tempo-free statistics~~
   - ~~Develop team comparison tools~~
   - ~~Create visualization-ready data structures~~

3. **Develop Injury Tracking System**
   - Implement `Injuries` table with injury types and severity
   - Create `AthleteInjuries` table linking injuries to players
   - Track timelines, return dates, and status updates
   - Support historical injury analysis

### Medium Priority

4. **Implement Media Resource Tracking**
   - Create `MediaResources` table for logos, headshots, and venue images
   - Develop metadata for image dimensions and types
   - Track resource currency and updates
   - Include alt text and accessibility information

5. **Create Event Status History Tracking**
   - Implement temporal tracking of game status changes
   - Store clock, period, and status type information
   - Support timeline reconstruction
   - Include status change reasons

6. **Develop Reference Resolution Strategy**
    - Create approach for handling ESPN API's `$ref` fields
    - Implement reference caching mechanisms
    - Establish rules for reference staleness
    - Develop hierarchical resolution patterns

7. **Add News/Headlines/Notes Structure**
    - Create `News` table for articles and stories
    - Implement `EventHeadlines` for game-specific headlines
    - Track news sources and publication timestamps
    - Include relevance scores and categorization

### Lower Priority

8. **Ensure Consistent UTC Time Handling**
    - Standardize all temporal fields in UTC
    - Implement display time zone conversion
    - Create timestamp precision standards
    - Ensure date range query optimization

9. **Add Venue Capacity and Attendance Tracking**
    - Enhance `Venues` table with detailed capacity information
    - Add historical attendance tracking
    - Calculate attendance percentages
    - Include venue configuration information

10. **Establish Data Refresh Patterns**
    - Define incremental vs. full refresh strategies
    - Create data freshness metadata tracking
    - Implement priority-based update scheduling
    - Develop change detection mechanisms

11. **Create View Layer for Analytics**
    - Develop materialized views for common analytical patterns
    - Create data marts for specific analysis domains
    - Implement role-based access controls
    - Support visualization-friendly data structures

12. **Consider Partitioning Strategy**
    - Implement partitioning for large historical tables
    - Develop season-based partitioning scheme
    - Create hot/cold data management strategy
    - Optimize partition sizes for DuckDB

13. **Develop Approach for Derived Statistics**
    - Create calculation methodology documentation
    - Implement derived statistic triggers or procedures
    - Establish calculation provenance tracking
    - Develop statistic version control

14. **Create Test Queries for Analytics**
    - Develop benchmark query suite
    - Create validation test cases
    - Implement performance testing framework
    - Document common analytical patterns

15. **Implement Team Conference Alignment History**
    - Track historical conference membership
    - Create effective dating for alignment changes
    - Support realignment analysis
    - Include division classification history

16. **Develop Memory Management Routines**
    - Create scheduled database detach/reattach jobs
    - Implement index usage monitoring
    - Develop memory pressure handling
    - Create index rebuild optimization

17. **Add Alternate Names and Identifiers**
    - Create historical name tracking for teams/venues
    - Implement external ID mapping system
    - Support alias resolution
    - Track brand changes and renamings

18. **Develop Game Highlight Structure**
    - Create `Highlights` table for key moments
    - Link highlights to plays and athletes
    - Include media references and timestamps
    - Support highlight categorization

19. **Implement Records and Milestones Tracking**
    - Track team and player records
    - Create milestone achievement history
    - Implement record-breaking detection
    - Support historical significance scoring

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
