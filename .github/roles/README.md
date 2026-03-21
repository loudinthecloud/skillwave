# Role Registry

Role files live in `.github/roles/{role-name}.md`. skillwave reads the file
before invoking a role and injects its **Role Instructions** section into the
subagent prompt. See the `/role-creation` skill for the full template and
lifecycle conventions.

## Directory Ownership

| Directory               | Typical Role | Contents  |
| ----------------------- | ------------ | --------- | ----------- | --- |
| <!-- ADD DIRECTORY ROW: | `dir/`       | role-name | Description | --> |

## Active Roles

| Role               | Expertise | File                       |
| ------------------ | --------- | -------------------------- | ---------------------------- | --- |
| <!-- ADD ROLE ROW: | role-name | one-line expertise summary | `.github/roles/role-name.md` | --> |

## Inactive Roles

<!-- MOVE RETIRED ROLE ROWS HERE -->

## Decision Log

<!-- ADD ENTRY: YYYY-MM-DD — description -->
