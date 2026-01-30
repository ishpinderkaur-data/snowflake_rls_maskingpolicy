# snowflake_rls_maskingpolicy
Snowflake Security Demo: Row-Level Security + Masking Policies
This project demonstrates Snowflake security features using a small Game of Thrones dataset. The goal is to show how to control:
1. Which rows a role can see (Row-Level Security / Row Access Policy)
2. Which column values a role can view (Masking Policy)

Columns:

HOUSE → used for Row-Level Security
TRUE_IDENTITY → treated as sensitive and protected with a masking policy
Example records include Stark, Lannister, and Targaryen characters.

Roles
This demo creates three roles:
ROLE_NORTH_WATCH
ROLE_COMMONER
ROLE_SMALL_COUNCIL

Each role is granted:
USAGE on database + schema
SELECT on the main table

Row-Level Security (Row Access Policy)
Mapping table

The row access logic uses a lookup table:

GOT_DB.GOT_SCHEMA.GOT_ACCESS_CONTROL (HOUSE, ROLE_ALLOWED)

Example mapping:

('Stark', 'ROLE_NORTH_WATCH')

Meaning:
✅ ROLE_NORTH_WATCH can only see rows where HOUSE = 'Stark'

Policy logic

The Row Access Policy allows access when:

the current role is in (ACCOUNTADMIN, ROLE_SMALL_COUNCIL, ROLE_COMMONER)
OR

the role exists in the mapping table for that house


Masking Policy

A masking policy is applied to the TRUE_IDENTITY column:

ACCOUNTADMIN, ROLE_SMALL_COUNCIL, ROLE_NORTH_WATCH → see the real value

everyone else → sees a hashed value using SHA2()

How to Run

Execute the SQL script as ACCOUNTADMIN

Test results by switching roles and running queries

Test: ROLE_NORTH_WATCH
USE ROLE ROLE_NORTH_WATCH;
SELECT * FROM GOT_DB.GOT_SCHEMA.GOT_CHARACTERS;


Expected:

only Stark rows are visible (Row Access Policy)

TRUE_IDENTITY is visible (Masking Policy allows it)

Test: ROLE_COMMONER
USE ROLE ROLE_COMMONER;
SELECT * FROM GOT_DB.GOT_SCHEMA.GOT_CHARACTERS;


Expected:

rows depend on the RLS logic (in this demo, commoner is privileged)

TRUE_IDENTITY should be hashed (masked)

Test: ROLE_SMALL_COUNCIL
USE ROLE ROLE_SMALL_COUNCIL;
SELECT * FROM GOT_DB.GOT_SCHEMA.GOT_CHARACTERS;


Expected:

full row access

TRUE_IDENTITY visible

