# Sprocket V2: Unified Matchmaking System Design Proposal

## Executive Summary

This proposal outlines a fundamental transformation of MLE's matchmaking system from isolated skill-group-based queues to a unified, cross-league matchmaking platform. By implementing a single Elo rating pool across all skill groups and introducing intelligent matchmaking algorithms, we can significantly improve player experience, reduce queue times, and create more balanced matches while maintaining competitive integrity across all skill levels.

## Current System Analysis

### The Fragmented Landscape

MLE's current matchmaking system operates like a collection of isolated islands, each skill group (Foundation League, Academy League, Champion League, etc.) maintains its own separate Elo rating pool. A player rated 1300 in Foundation League cannot meaningfully compare their skill to a 900-rated player in Champion League, despite potentially similar actual skill levels. This fragmentation creates several critical issues:

**Skill Group Silos**: Players are locked into their respective skill groups with no mechanism for cross-group competition. The system enforces rigid boundaries where a 10.0 salary player in Foundation League (approaching promotion) might have nearly identical skill to a 10.5 salary player in Academy League (at risk of demotion), yet they can never compete against each other. This creates artificial barriers that don't reflect the continuous nature of skill development.

**Queue Fragmentation**: Each skill group operates independent queues for different game modes (2v2, 3v3, LFS). A player seeking any competitive match must manually check each skill group's queue separately, leading to reduced match frequency and longer wait times. The current system processes matchmaking by finding players within expanding skill ranges (starting at ±100 rating, expanding to ±500), but this expansion happens within isolated skill groups rather than across the entire player base.

**Bubble Player Instability**: Players near skill group boundaries experience constant promotion/demotion cycles without adequate hysteresis mechanisms. The current system lacks sufficient buffer zones, causing players to oscillate between skill groups based on small rating fluctuations rather than sustained performance changes.

### Technical Architecture Overview

The existing system centers around several key entities: [`PlayerEntity`](https://github.com/SprocketBot/sprocket/tree/dev/core/src/db/player/player.entity.ts) contains a [`salary`](https://github.com/SprocketBot/sprocket/tree/dev/core/src/db/player/player.entity.ts) field (string-based rating representation) and links to [`SkillGroupEntity`](https://github.com/SprocketBot/sprocket/tree/dev/core/src/db/skill_group/skill_group.entity.ts). The [`ScrimQueueEntity`](https://github.com/SprocketBot/sprocket/tree/dev/core/src/db/scrim_queue/scrim_queue.entity.ts) manages individual queue entries with [`skillRating`](https://github.com/SprocketBot/sprocket/tree/dev/core/src/db/scrim_queue/scrim_queue.entity.ts) values, while the [`QueueService`](https://github.com/SprocketBot/sprocket/tree/dev/core/src/matchmaking/queue/queue.service.ts) implements the current matchmaking logic with expanding skill ranges.

Match results flow through a submission system where replay files are validated, ratified by players, and then processed for rating updates. However, the current implementation shows several TODO comments indicating incomplete Elo update logic: [`// TODO: Elo Side Effects`](https://github.com/SprocketBot/sprocket/tree/dev/core/src/gql/player/player.resolver.ts) appears in player mutation handlers, suggesting the rating update system requires significant development.

## Industry Best Practices Analysis

### League of Legends: Tier-Based MMR with Promotion Series

Riot Games implements a sophisticated hybrid system combining visible tiers (Iron through Challenger) with hidden Match Making Rating (MMR). Players compete within divisions (I-IV) while their MMR operates on a continuous scale. The system includes promotion series that require sustained performance to advance, preventing random fluctuation-based promotions. Most importantly, players can be matched across tier boundaries when their MMR aligns, creating larger matchmaking pools while maintaining competitive integrity.

### Dota 2: Visible MMR with Seasonal Recalibration

Valve's approach uses fully visible MMR numbers ranging from 0 to over 10,000, with separate ratings for solo and party play. The system implements seasonal recalibration, allowing players to reset their ratings periodically while maintaining some historical data. Dota 2's matchmaking algorithm considers not just MMR but also uncertainty values, matching new accounts with wider skill ranges to quickly establish accurate ratings.

### Counter-Strike: Global Offensive: Glicko-2 Implementation

CS:GO utilizes the Glicko-2 rating system, which maintains both a rating value and a Rating Deviation (RD) that represents uncertainty. Higher RD values allow for larger rating changes and broader matchmaking ranges. The system naturally implements hysteresis through the RD mechanism—players with stable performance have lower RD, making their ratings more resistant to change.

## Proposed Unified System Architecture

### Core Philosophy: Continuous Skill Spectrum

The new system replaces discrete skill group silos with a continuous Elo rating scale spanning all skill groups. Rather than maintaining separate rating pools, all players exist on a single spectrum where skill group boundaries represent ranges rather than absolute barriers. This approach acknowledges that skill development is continuous, not discrete, and allows for more nuanced player matching.

### Unified Elo Pool Implementation

**Single Rating System**: All players share one Elo pool ranging from 0 to 3000+, with skill groups mapped to specific rating ranges:

- Foundation League: 0-800
- Academy League: 700-1200  
- Champion League: 1100-1600
- Master League: 1500-2000
- Premier League: 1900+

The overlapping ranges (100-point buffers) create natural transition zones where cross-group matches can occur, while the unified pool ensures meaningful skill comparisons across all levels.

**Hysteresis Mechanisms**: Implement promotion/demotion thresholds with built-in buffers. Players promote when exceeding the upper threshold (e.g., 820 for Foundation → Academy) but only demote when falling below the lower threshold (e.g., 680 for Academy → Foundation). This 140-point buffer prevents constant oscillation and requires sustained performance changes for skill group transitions.

### Single Master Queue System

**Multi-Queue Capability**: Players can simultaneously queue for multiple scrim types (2v2 Round Robin, 3v3 Round Robin, Team 2v2/3v3, LFS) across all skill groups. The system maintains separate queue entries but processes them through a unified matchmaking algorithm that finds the best available match regardless of skill group boundaries.

**Intelligent Matchmaking Algorithm**: The proposed algorithm expands search parameters based on queue time:

1. **Initial Search (0-2 minutes)**: Match players within ±100 rating points of the same skill group
2. **Expanded Search (2-5 minutes)**: Extend to ±150 rating points, allow cross-group matches within 50 rating points of boundaries
3. **Broad Search (5+ minutes)**: Expand to ±250 rating points, prioritize match quality over skill group boundaries
4. **Emergency Match (10+ minutes)**: Maximum expansion to ±400 rating points, create matches with available players

### Cross-Skill-Group Matchmaking Benefits

**Bubble Player Solution**: The unified system naturally addresses bubble player issues. A 795-rated Foundation League player can match against a 805-rated Academy League player—these players are only 10 rating points apart despite being in different skill groups. The system recognizes their similar skill levels and creates balanced matches that wouldn't be possible in the current siloed approach.

**Expanded Matchmaking Pools**: By removing skill group barriers, the effective player pool for matchmaking increases dramatically. During off-peak hours, when individual skill groups might have only 2-3 queued players, the unified system can draw from 15-20 players across all skill groups, significantly reducing queue times.

**Skill Development Acceleration**: Cross-group matches provide valuable learning opportunities. Foundation League players competing against Academy League opponents experience higher-level play, accelerating skill development. Conversely, Academy League players mentoring Foundation League opponents reinforce fundamental concepts.

## Technical Implementation Strategy

### Database Schema Evolution

The current [`PlayerEntity`](https://github.com/SprocketBot/sprocket/tree/dev/core/src/db/player/player.entity.ts) requires modification to support the unified system:

```typescript
@Entity('player')
export class PlayerEntity extends BaseEntity {
    @Column()
    unifiedEloRating: number; // Single Elo value (0-3000+)
    
    @Column()
    skillGroup: SkillGroup; // Current skill group for display/organization
    
    @Column()
    eloUncertainty: number; // Glicko-2 RD equivalent
    
    @Column()
    lastRatingUpdate: Date;
    
    @Column({ nullable: true })
    promotionThreshold: number; // Personal promotion threshold
    
    @Column({ nullable: true })
    demotionThreshold: number; // Personal demotion threshold
}
```

### Matchmaking Service Architecture

The [`QueueService`](https://github.com/SprocketBot/sprocket/tree/dev/core/src/matchmaking/queue/queue.service.ts) requires fundamental restructuring:

```typescript
@Injectable()
export class UnifiedQueueService {
    private readonly INITIAL_SKILL_RANGE = 100;
    private readonly MAX_SKILL_RANGE = 400;
    private readonly CROSS_GROUP_THRESHOLD = 50; // Allow cross-group within 50 points
    
    async findMatches(gameId: string): Promise<MatchmakingResult[]> {
        const allQueuedPlayers = await this.getAllQueuedPlayers(gameId);
        
        // Group by skill level rather than skill group
        const skillSortedPlayers = allQueuedPlayers.sort((a, b) => a.skillRating - b.skillRating);
        
        // Implement time-based expansion
        const currentTime = new Date();
        const expansionFactor = this.calculateExpansionFactor(skillSortedPlayers, currentTime);
        
        return this.createBalancedMatches(skillSortedPlayers, expansionFactor);
    }
}
```

### Rating Update Algorithm

Implement a Glicko-2 inspired system that maintains both rating and uncertainty:

```typescript
interface RatingUpdateResult {
    newRating: number;
    newUncertainty: number;
    skillGroupChange?: SkillGroupChange;
}

async function updateRating(
    player: PlayerEntity,
    matchResult: MatchResult,
    opponentRatings: number[]
): Promise<RatingUpdateResult> {
    // Calculate rating change based on expected vs actual outcome
    const expectedScore = calculateExpectedScore(player.unifiedEloRating, opponentRatings);
    const actualScore = matchResult.playerScore;
    
    // Apply Glicko-2 style update with uncertainty adjustment
    const ratingChange = (actualScore - expectedScore) * player.eloUncertainty;
    const uncertaintyChange = calculateUncertaintyChange(player, matchResult);
    
    return {
        newRating: player.unifiedEloRating + ratingChange,
        newUncertainty: Math.max(player.eloUncertainty + uncertaintyChange, MIN_UNCERTAINTY),
        skillGroupChange: determineSkillGroupChange(player, newRating)
    };
}
```

## User Experience Enhancements

### Multi-Queue Interface

Players can select multiple scrim types simultaneously through an intuitive interface:

- **Primary Queue**: Player's preferred scrim type (highest priority)
- **Secondary Queues**: Additional scrim types they're willing to play
- **Cross-Group Preference**: Option to enable/disable cross-skill-group matches
- **Maximum Skill Gap**: Player-defined maximum rating difference for matches

### Transparent Skill Group Boundaries

Display current skill group ranges and personal thresholds:

```
Your Rating: 1,247 (Academy League)
Promotion to Champion League: 1,250 (3 points needed)
Demotion to Foundation League: 1,150 (97 points buffer)
Cross-group matches: Enabled (±50 rating points)
```

### Matchmaking Feedback

Provide real-time feedback during queue:

- Estimated wait time based on current queue population
- Number of players in compatible skill ranges
- Cross-group match opportunities identified
- Skill rating changes from recent matches

## Risk Assessment and Mitigation

### Competitive Integrity Concerns

**Risk**: Cross-group matches might create unfair competitive environments.

**Mitigation**: Implement strict rating-based matching regardless of skill group. A 1200-rated Foundation League player should never face a 1600-rated Champion League player, even if both are queued. The system prioritizes rating proximity over skill group boundaries.

### Player Confusion About Skill Groups

**Risk**: Players might not understand why they're matching against different skill groups.

**Mitigation**: Provide clear educational materials explaining the continuous rating system. Show pre-match information: "This opponent is 45 rating points higher—an even match!" Emphasize that skill groups are organizational tools, not absolute skill barriers.

### Rating Inflation/Deflation

**Risk**: Unified system might experience rating drift over time.

**Mitigation**: Implement rating decay for inactive players and periodic recalibration events. Monitor rating distributions and adjust algorithms if drift exceeds acceptable thresholds (±5% from baseline distributions).

## Implementation Considerations

This system represents a fundamental architectural change that will require careful planning and execution. Key implementation phases should include:

- **Database schema migration** to support unified rating system
- **Unified rating system backend implementation** with proper Elo update logic
- **Basic cross-group matchmaking algorithm** development and testing
- **Multi-queue interface development** for enhanced user experience
- **Rating update algorithm integration** with existing submission system
- **Skill group transition logic** with proper hysteresis mechanisms
- **Advanced matchmaking algorithms** optimization
- **Beta testing** with select groups before full rollout
- **Gradual deployment** across all leagues with continuous monitoring

The complexity of this transformation requires thorough testing and gradual implementation to ensure system stability and competitive integrity throughout the transition.
## Success Metrics

### Primary Metrics
- **Average Queue Time**: Target 50% reduction across all skill groups
- **Match Balance**: 70% of matches within ±100 rating points
- **Player Satisfaction**: 80%+ positive feedback on new system
- **Cross-Group Match Rate**: 25% of matches involve different skill groups

### Secondary Metrics
- **Skill Group Transition Stability**: <10% of players experience promotion/demotion cycles within 30 days
- **Rating Accuracy**: Predicted match outcomes align with actual results 75% of the time
- **System Utilization**: 90%+ of queued players find matches within maximum time limits

## Conclusion

The proposed unified matchmaking system represents a fundamental evolution in MLE's competitive infrastructure. By embracing the continuous nature of skill development and leveraging industry-proven matchmaking techniques, we can create a more engaging, efficient, and fair competitive environment for all players.

The system's success depends on careful implementation of rating algorithms, clear communication with players, and continuous monitoring of competitive integrity. However, the potential benefits—reduced queue times, improved match quality, and enhanced player development—justify the significant architectural changes required.

This transformation aligns with MLE's mission to provide the best possible competitive experience while maintaining the organizational structure that makes our leagues unique. The unified system preserves skill group identities while removing artificial barriers that currently limit competitive opportunities.

Through thoughtful application of Heidegger's hermeneutic circle, we've examined both the technical components and the broader player experience context, ensuring that each detail supports the overall goal of creating a more connected and responsive matchmaking ecosystem. The interplay between individual player ratings and the collective matchmaking pool creates emergent properties that benefit the entire competitive community.

The path forward requires careful execution, but the destination—a more vibrant, accessible, and competitive MLE ecosystem—is well worth the journey.