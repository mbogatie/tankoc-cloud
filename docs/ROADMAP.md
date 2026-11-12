# Tank OC Development Roadmap

**Status:** Planning baseline  
**Last updated:** 13 November 2026  
**Project:** Tank OC — personal World of Warcraft: Forever assistant

## 1. Product goal

Build Tank OC into a character-aware companion for WoW: Forever. It should combine the player's current character data with reliable game-reference data, then progressively add encounter analysis and practical coaching.

The assistant should explain recommendations, show where items and recipes come from, and distinguish confirmed data from estimates or unsupported assumptions.

## 2. Existing architecture

Keep the current services separate by responsibility:

| Component | Responsibility | Current position |
|---|---|---|
| `tankoc-api` Cloudflare Worker | Retrieve Tank Oc's character sheet from the configured Blizzard character-data integration | Existing service; preserve it |
| `tankoc-mcp` Cloudflare Worker | Expose Tank OC capabilities to ChatGPT through MCP | Existing service; currently exposes the character-sheet tool |
| ForeverDB | Reference data for the WoW: Forever game world | API access is available to the project; confirm exact endpoints, limits and attribution requirements before integration |
| Local combat-log companion (planned) | Read the locally generated combat log and send permitted encounter data to Tank OC | Not built |
| Tank OC analysis layer (planned) | Combine character, game-reference and encounter data into useful advice | Not built |

**Repository responsibilities**

- This public repository, `tankoc-cloud`: project overview, architecture, roadmap and user-facing documentation.
- Private repository, `tankoc-mcp`: MCP server implementation and server-side integrations.
- Existing `tankoc-api`: character-data service. Do not replace or disrupt it as part of the game-data work.
- Future local companion: its own small project/repository, responsible only for local log collection and secure transfer.

Never commit API keys, access tokens, private character data or other secrets to this public repository.

## 3. Development principles

1. **Protect the working system.** Extend the MCP service in small steps; leave the character API unchanged unless a separately justified change is needed.
2. **Use supported data sources.** Use the configured character API for character data and ForeverDB for game-reference data, subject to each provider's terms and technical limits.
3. **Verify before building.** Confirm ForeverDB's current API schema, authentication, attribution, rate limits and coverage before writing integration code. Do not assume that every website feature has a public API endpoint.
4. **Keep collection external to the game client.** Any future combat-log reader should read files produced by the game, not inspect game memory, inject code, automate inputs or control the game.
5. **Privacy by default.** Keep credentials server-side. Treat character details and combat reports as private unless the player deliberately shares them.
6. **Explain confidence.** Label source-backed facts, calculations, estimates and unknowns clearly.
7. **Build useful increments.** Each phase should have a testable outcome before the next begins.

## 4. Roadmap

### Phase 0 — Verify and document the foundations

- [ ] Record the current endpoints, authentication method and deployment configuration for the existing character API and MCP worker (without recording secret values).
- [ ] Confirm the live MCP tool list and test the character-sheet tool end to end.
- [ ] Review ForeverDB's current developer documentation and confirm the exact API base URL, key handling, available resources, query patterns, limits and attribution wording.
- [ ] Identify which ForeverDB resources are sufficient for the first useful gear-advice workflow.
- [ ] Record any API gaps rather than inventing endpoints.

**Exit criteria:** A verified integration note and a safe, testable plan for the first ForeverDB tools.

### Phase 1 — Add ForeverDB game-reference tools

Extend the existing private MCP service; do not create a second general-purpose assistant.

Initial tool candidates, subject to confirmed API support:

- [ ] Search items by name, slot, type or other supported filters.
- [ ] Retrieve item details and stats.
- [ ] Find item sources, drops, quests, vendors or crafting recipes where the API exposes them.
- [ ] Search relevant dungeons, bosses, NPCs and vendors.
- [ ] Search profession recipes and materials where supported.

All displayed ForeverDB-derived information must include the required attribution. Cache only where permitted by ForeverDB's terms and technical rules.

**Exit criteria:** Tank OC can answer a game-data question using a real ForeverDB response, with attribution and clear handling of missing results.

### Phase 2 — Character-aware gear adviser

- [ ] Normalise the character's equipped items and relevant stats from the existing character sheet.
- [ ] Match equipped slots against candidate items from ForeverDB.
- [ ] Compare relevant stats and explain the trade-offs; do not rank items using item level alone.
- [ ] Show how to obtain each recommended upgrade, including source and prerequisites when available.
- [ ] Separate attainable next steps from long-term goals.
- [ ] Add regression examples for the user's current Warrior sword-and-shield progression.

**Exit criteria:** Given a current character sheet, Tank OC can produce a traceable upgrade shortlist with sources and reasoning.

### Phase 3 — Local combat-log companion and encounter records

- [ ] Confirm which combat-log events and encounter identifiers WoW: Forever actually writes.
- [ ] Build a small local companion that watches the configured log file and parses appended lines.
- [ ] Detect encounter start/end and store a per-fight record where the available data permits.
- [ ] Calculate supported metrics such as damage, healing, deaths, interrupts and defensive ability use.
- [ ] Send only the required processed data to a secured Tank OC endpoint.
- [ ] Add local controls for pause, stop, retention and deletion.

The companion must not read game memory, inject into the game, automate gameplay or send game inputs. Review applicable game rules before distributing or relying on the tool; file-based collection is not a guarantee of policy approval.

**Exit criteria:** A test encounter produces a locally inspectable report, and the user can control exactly when collection runs and what is uploaded.

### Phase 4 — Encounter analysis and coaching

- [ ] Combine encounter records with the character's equipment and relevant game-reference data.
- [ ] Explain deaths and damage patterns only where the log supports a conclusion.
- [ ] Identify defensive cooldown, interrupt or ability-usage opportunities when the required events are present.
- [ ] Compare repeated attempts to show improvement or recurring issues.
- [ ] Produce practical next actions rather than generic performance rankings.

**Exit criteria:** Tank OC can analyse a completed encounter, cite the underlying metrics and give appropriately qualified recommendations.

### Phase 5 — Optional live companion and screen analysis

This is a later, separate feature—not a prerequisite for log analysis.

- [ ] Prototype external screen capture initiated and controlled by the user.
- [ ] Interpret visible game and addon panels, such as damage meters, healing breakdowns and threat displays.
- [ ] Keep screen analysis observational only: no game control, input automation, memory reading or injection.
- [ ] Make capture status obvious and provide a one-step stop control.
- [ ] Check privacy, performance and applicable game/platform policies before broader use.

**Exit criteria:** The user can deliberately share a visible game view and receive useful observations without Tank OC interacting with the game.

## 5. First implementation sequence

1. Verify the existing character-sheet MCP call and document the current service boundaries.
2. Verify ForeverDB API details and select the smallest useful first set of endpoints.
3. Add one read-only ForeverDB search tool to the existing MCP worker.
4. Test it against known items and sources, including empty and error responses.
5. Add item detail/source retrieval, then build the first character-aware gear comparison.
6. Only after that workflow is reliable, begin the local combat-log prototype.

## 6. Risks and open questions

- **ForeverDB API coverage:** Website features do not necessarily correspond to public API endpoints. Verify each resource before promising it.
- **Combat-log compatibility:** Confirm the exact WoW: Forever client version, log format, event coverage and encounter identifiers.
- **Live reports:** Do not assume ForeverDB exposes personal live-report data through its developer API. Investigate documented or explicitly authorised access separately.
- **Game policy:** Review applicable Blizzard and WoW: Forever rules before releasing any local collector or screen companion. Do not claim that a design is guaranteed safe from enforcement.
- **Data quality:** Some recommendations may be limited by missing item stats, source records, combat events or character fields.
- **Privacy:** Combat logs and shareable report URLs can contain character names and performance information. Default to private handling.

## 7. Current status

As of 13 November 2026:

- The character-data API and MCP worker already exist.
- The inspected MCP implementation currently exposes the character-sheet retrieval tool.
- ForeverDB game-data integration is the next practical development step, after its API details are verified.
- No local combat-log companion or encounter-analysis engine has been built.
- Screen-based live coaching remains a future research item.

This roadmap is a working plan. Update the checkboxes and status as implementation and testing are actually completed.
