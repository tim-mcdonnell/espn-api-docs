# Database Design Principles

This section outlines the core principles and approach that guide our database design for the ESPN API data structure. These principles ensure consistency, performance, and extensibility as the database evolves.

## Design Philosophy

Our database design follows a hybrid approach that combines:

1. **Normalized Dimensional Model**: We primarily follow dimensional modeling techniques with clear fact and dimension tables, but maintain a higher degree of normalization than typical star schemas.

2. **Domain-Driven Design**: Tables and relationships closely reflect the real-world sports domain rather than purely technical considerations.

3. **Performance-Oriented**: While maintaining normalization, we make strategic denormalization decisions when query performance requires it.

4. **Future-Proof Structure**: The design accommodates the addition of new sports, leagues, and statistical categories without structural changes.

## Core Principles

### 1. Clear Separation of Dimensions and Facts

- **Dimensions**: Represent descriptive elements that provide context (teams, athletes, venues, etc.)
- **Facts**: Represent measurable events or statistics tied to dimensions

This separation allows for efficient querying and aggregation while maintaining a clear semantic structure.

### 2. Appropriate Normalization Levels

We follow a modified 3NF (Third Normal Form) approach:

- **1NF**: All tables have a primary key, and each column contains atomic values
- **2NF**: All non-key attributes are fully functionally dependent on the primary key
- **3NF**: All attributes are non-transitively dependent on the primary key

However, we selectively denormalize when:

- Query performance would significantly benefit
- The denormalization creates minimal redundancy
- The data is relatively stable and unlikely to create update anomalies

### 3. Consistent Surrogate Keys

- All entities have synthetic surrogate keys (typically auto-incrementing integers or UUIDs)
- Natural keys are preserved in separate columns and unique constraints
- This approach simplifies key management and allows for entity evolution

### 4. Temporal Tracking

- All significant entities track creation and modification timestamps
- Historical states are preserved for important dimensions like team compositions
- Effective dating is used for time-variant relationships

### 5. Referential Integrity

- Foreign key constraints are enforced wherever possible
- Cascading updates are used selectively and documented
- Deletion constraints default to RESTRICT to prevent data loss

### 6. Indexing Strategy

- Indexes are created based on query patterns rather than blindly on all foreign keys
- Composite indexes are designed for common query paths
- We leverage DuckDB's zone maps for range-based queries

For detailed indexing information, see the [Indexing Strategy](indexing-strategy.md) section.

## Data Organization

### Dimension Tables

These tables represent the fundamental entities in our sports data domain:

- Core entities (leagues, teams, athletes, venues)
- Classification dimensions (positions, status types, game types)
- Time dimensions (seasons, weeks, dates)

Dimensions typically have:
- A synthetic primary key
- Natural business keys with unique constraints
- Descriptive attributes
- Hierarchical relationships to other dimensions

### Fact Tables

These tables contain the measurable events in our domain:

- Events (games, matches)
- Statistics (box scores, player stats)
- Relationships (athlete team memberships, coaching assignments)

Fact tables typically have:
- A synthetic primary key
- Foreign keys to relevant dimensions
- Measurement values
- Temporal attributes

### Junction Tables

These tables resolve many-to-many relationships:

- Team season relationships
- Athlete position assignments
- Event participant mappings

Junction tables typically have:
- A composite primary key from the entities they connect
- Additional attributes that describe the relationship
- Temporal attributes for when the relationship is effective

## Naming Conventions

Our naming conventions ensure consistency and clarity:

### Tables

- **Dimension tables**: Plural nouns (e.g., `Athletes`, `Teams`, `Venues`)
- **Fact tables**: Plural nouns describing the measurements (e.g., `Events`, `AthleteStatistics`)
- **Junction tables**: Concatenation of the related tables (e.g., `AthleteTeams`, `EventOfficials`)

### Columns

- **Primary keys**: `id` (e.g., `athlete_id`, `team_id`)
- **Foreign keys**: Entity name + `_id` (e.g., `athlete_id`, `league_id`)
- **Natural keys**: Descriptive of the business key (e.g., `espn_id`, `abbreviation`)
- **Timestamps**: Use `created_at`, `updated_at`, `effective_from`, `effective_to`

### Indexes

- **Primary key indexes**: Table name + `_pkey`
- **Foreign key indexes**: Table name + `_` + foreign key column + `_idx`
- **Composite indexes**: Table name + `_` + included columns + `_idx`

## Query Optimization

Our design optimizes for these common query patterns:

1. **Time-series analysis**: Efficient filtering and aggregation over time periods
2. **Entity lookups**: Fast retrieval of entity details by identifiers
3. **Hierarchical navigation**: Traversal of related entities (teams → athletes)
4. **Statistical aggregation**: Calculation of summary statistics and rankings
5. **Spatial analysis**: For location-based queries like shot charts

## Sport-Specific Considerations

While our design is sport-agnostic, we accommodate sport-specific features through:

- Extensible statistics categories and types
- Sport-specific extension tables when required
- Flexible play-by-play structures that handle different game flows

## Evolution Strategy

The database is designed to evolve through:

- Addition of new dimension attributes without structural changes
- Extension tables for new analytical dimensions
- Version tracking for significant schema changes
- Data migration patterns for historical consistency

## Technical Implementation Notes

For specific implementation details regarding DuckDB, see the [Technical Implementation](technical-implementation.md) section.

For an overview of our indexing approach, see the [Indexing Strategy](indexing-strategy.md) section. 