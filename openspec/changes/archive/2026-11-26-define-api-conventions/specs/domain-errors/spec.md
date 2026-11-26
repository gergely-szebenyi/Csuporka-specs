# Spec Delta

## MODIFIED Requirements

### Requirement: General codes
The catalogue SHALL contain these codes:
- `VALIDATION_FAILED`: input breaks a format or limit rule
- `NOT_FOUND`: the resource doesn't exist or belongs to another household
- `INTERNAL_ERROR`: the server failed unexpectedly

#### Scenario: Another household's meal
- **WHEN** a user changes a meal of another household
- **THEN** the rejection carries `NOT_FOUND`

#### Scenario: Unexpected failure
- **WHEN** the server fails unexpectedly while handling a request
- **THEN** the rejection carries `INTERNAL_ERROR`
