# Sprocket Identity System

## Overview
Sprocket's identity system can be complex due to its multi-layered structure, which sometimes leads to errors like mismatched player assignments in scrims. This complexity arises from historical design decisions and migrations. Understanding the interplay between the different identity types is crucial, as each part (identity) depends on the broader context of the system's architecture and legacy integrations.

The system currently involves four main types of identities for users, reflecting Sprocket's evolution from supporting multiple organizations and games to its current state. Common errors, such as using the wrong ID type during account reporting, can result in issues like incorrect player mappings. These errors highlight the need for precise handling of IDs.

Below, we distill key learnings from an investigation into such errors, expand on each identity type with examples and implications, and note planned simplifications in Sprocket V2.

## Key Learnings from Investigations
- **Common Error: Mismatched Player Assignments**
  - In certain scrims, the system throws a "mismatched player" error because an account is incorrectly linked to the wrong player.
  - Root Cause: Support personnel often mistakenly use a Sprocket Member ID instead of the Sprocket User ID when reporting accounts via mutations. This is a recurring issue across various incidents.
  - Implication: This error disrupts scrim processing and requires manual investigation using SQL queries to trace IDs across tables (e.g., joining sprocket.user, user_authentication_account, mledb.player, sprocket.member, and sprocket.player).
  - Prevention: Always verify and use the correct ID type (User ID) for reporting. Document and train on ID distinctions to reduce errors.

- **Historical Context**
  - The system's complexity stems from Sprocket v1's design for multiple leagues/organizations sharing a database (e.g., different esports organizations).
  - Incomplete migration of legacy bots (Emilio and Emilia) led to embedding the old MLEDB into the new system, creating redundant player tables.
  - This results in users having multiple IDs, increasing confusion and error potential, especially for players who participate in multiple games like Rocket League (RL) and TrackMania (TM).

## Identity Types Explained
Here, we expand on the four identity types, providing practical examples and implications based on the system's hermeneutic structure—where understanding one type requires context from the others, and the whole system emerges from their interconnections.

1. **Sprocket User**
   - **Description:** The foundational identity for any platform user. It's tied to login credentials and represents a single person across the system.
   - **Example:** When logging in to Sprocket, your User ID (e.g., from the `sprocket.user` table) is used for authentication.
   - **Implications:** This is the core, unique identifier. Errors often occur when it's confused with derived IDs like Member or Player. In mismatch errors, the User ID should be used for accurate mapping via Discord ID linkages.
   - **Practical Note:** Every user has exactly one Sprocket User ID, making it the "whole" from which other identities branch.

2. **Sprocket Member**
   - **Description:** An outdated concept from v1, representing a User attached to a specific organization (e.g., a league or development org).
   - **Example:** A user might have multiple Member IDs if involved in different organizations. In queries, this is represented as `sprocketMemberId`.
   - **Implications:** This layer adds unnecessary complexity in a single-org setup. Misusing Member IDs (e.g., in reporting) causes mismatches, leading to incorrect linkages.
   - **Practical Note:** Users can have multiple Members, but in current usage, it's often one-to-one with Users for most people.

3. **Sprocket Player**
   - **Description:** A Member attached to a specific game (e.g., Rocket League or TrackMania).
   - **Example:** A player participating in multiple games would have separate Player IDs for each, such as one for RL (`sprocketPlayerId`) and one for TM, each linked to their Member ID.
   - **Implications:** This allows game-specific tracking but compounds ID confusion. In multi-game scenarios, ensuring the correct Player ID is used is vital to avoid errors like mismatched scrims.
   - **Practical Note:** Multiple Players per Member reflect game diversity, but this can lead to fragmentation if not managed carefully.

4. **MLEDB Player**
   - **Description:** A legacy artifact from migrating the old MLE database. It's a separate table for RL players only, maintained alongside Sprocket's systems.
   - **Example:** In queries, this is represented as `mledbPlayerId` and associated names. Mismatches occur when legacy entries don't align with Sprocket's IDs.
   - **Implications:** This duplication requires dual maintenance and is prone to desynchronization, where legacy data overrides correct information. It's RL-specific, adding asymmetry.
   - **Practical Note:** Only relevant for RL; it's a historical "bolt-on" that complicates the overall identity circle.

## Future Improvements in Sprocket V2
To address this complexity, Sprocket V2 plans significant simplifications:
- **Remove Multi-Org Support:** Eliminate the Member concept entirely, as the system will focus on single-org databases.
- **Integrate Legacy DB:** Fully migrate Emilio and Emilia into core Sprocket, removing MLEDB and its separate player table.
- **Expected Outcome:** Reduces IDs per person, decreasing dimensionality and confusion. Users will still have multiple IDs (e.g., for different games), but the system will be streamlined.
- **Implications:** This will make error-prone processes like account reporting more straightforward and reduce incidents like mismatched players.

For troubleshooting, refer to example SQL queries for tracing IDs. If issues persist, consult the Development team.

```mermaid
graph TD
    A[Sprocket User] -->|Attached to Org| B[Sprocket Member]
    B -->|Attached to Game| C[Sprocket Player]
    A -->|Legacy Mapping| D[MLEDB Player]
    subgraph "Current Complexity"
        B
        C
        D
    end
    subgraph "V2 Simplification"
        A --> C
    end