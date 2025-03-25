# ID Field Standardization

This document defines the standard naming conventions for identifier fields across all tables in our ESPN API data structure. These standards ensure consistency, improve code maintainability, and enhance query reliability.

## Primary Standards

### Identifier Types

| ID Type | Standard Name | Data Type | Description |
|---------|---------------|-----------|-------------|
| Primary Keys | `<entity>_id` | INTEGER | Auto-incrementing integer primary key for each table |
| ESPN IDs | `espn_id` | VARCHAR | ESPN's identifier from the API, stored as VARCHAR regardless of appearance |
| Universal IDs | `uid` | VARCHAR | ESPN's universal identifier with pattern `s:<sport_id>~l:<league_id>~<entity_type>:<entity_id>` |
| Global IDs | `guid` | VARCHAR | UUID format used by certain entities |
| Natural Keys | *descriptive name* | *appropriate type* | Business-meaningful identifiers (abbreviations, slugs, etc.) |

### Data Type Standardization

* **INTEGER** for all:
  * Primary key fields
  * Foreign key references
  * Numeric counters, quantities, and measurements
  * Boolean flags (0/1)

* **VARCHAR** for all:
  * ESPN IDs (even when numeric in the API)
  * UIDs
  * GUIDs
  * Names, abbreviations, and text identifiers
  * Formatted display values

* **TIMESTAMP** for all:
  * Date and time values (always stored in UTC)

## Specific Entity Conventions

### Common ID Fields

| Field | Tables | Data Type | Description |
|-------|--------|-----------|-------------|
| `athlete_id` | Athletes, AthleteStatistics, etc. | INTEGER | Primary key for athlete records |
| `team_id` | Teams, TeamStatistics, etc. | INTEGER | Primary key for team records |
| `league_id` | Leagues, Seasons, etc. | INTEGER | Primary key for league records |
| `event_id` | Events, PlayByPlay, etc. | INTEGER | Primary key for events (games) |
| `espn_id` | All entities | VARCHAR | ESPN's identifier, always VARCHAR for consistency |
| `uid` | All entities | VARCHAR | Universal identifier, always VARCHAR |

### Foreign Key Naming

Foreign keys always follow the pattern `<referenced_entity>_id` and use the INTEGER data type:

```sql
CREATE TABLE AthleteTeams (
    athlete_team_id INTEGER PRIMARY KEY,
    athlete_id INTEGER, -- References Athletes
    team_id INTEGER,    -- References Teams
    season_id INTEGER,  -- References Seasons
    -- Other columns
);
```

## Rationale for Standards

### Why VARCHAR for ESPN IDs

While many ESPN IDs appear as numeric values in the API, we standardize on VARCHAR for the following reasons:

1. **Future Compatibility**: ESPN may change ID formats in the future
2. **Consistency**: Some entities already use non-numeric IDs
3. **Prefix Support**: Some systems may add prefixes to IDs
4. **Avoiding Confusion**: Prevents misinterpreting IDs as quantitative values

### Why INTEGER for Primary Keys

1. **Performance**: Integer primary keys provide better performance for joins
2. **Space Efficiency**: Require less storage than UUIDs or strings
3. **Simplicity**: Easier to work with in query conditions and joins
4. **Auto-increment**: Simple sequential generation without external dependencies

## Implementation Notes

### Field Order Consistency

Fields should appear in a consistent order in all table definitions:

1. Primary key
2. Foreign keys (alphabetically)
3. ESPN identifier fields (espn_id, uid, guid)
4. Natural keys and business identifiers
5. Descriptive attributes
6. Temporal attributes
7. Metadata/system fields (created_at, updated_at)

### Metadata Fields

All tables should include these standard metadata fields:

| Field | Data Type | Description |
|-------|-----------|-------------|
| `created_at` | TIMESTAMP | When the record was created |
| `updated_at` | TIMESTAMP | When the record was last updated |

## Implementation Status

All tables in the ESPN API data structure have been updated to follow these standardization guidelines. Specifically:

- All tables use PascalCase naming (e.g., `Athletes`, `TeamStatistics`)
- All ESPN IDs are stored as VARCHAR type
- All tables include created_at and updated_at timestamp fields
- All foreign key references follow the `<referenced_entity>_id` pattern
- All tables include appropriate universal identifier (uid) fields
- Field ordering is consistent across all table definitions
- SQL examples have been updated to reflect standardized field names

## Exceptions

The following exceptions to the standards are permitted:

1. **Junction Tables**: May use composite primary keys instead of surrogate keys when appropriate
2. **Historical Data**: Legacy data imports may maintain original field names in transformation views
3. **Special Cases**: Documented exceptions based on specific analytical requirements

Any exception must be documented with clear rationale in the table definition. 