# GameForGood
Game for Good - Gaming platform add-ons to process research for non-profit organizations

A self-hosted volunteer compute network that enables distributed scientific computing through volunteer networks, reducing research costs and accelerating discovery while maintaining full data control and minimizing operational expenses.

## Mission Statement
**Constitution Alignment:** Community-driven design approach with humanity-focused outcomes
**Humanity Impact:** Enable distributed scientific computing through volunteer networks, reducing research costs and accelerating discovery while maintaining full data control and minimizing operational expenses

## Functional Requirements

### User Roles and Permissions
- **Volunteer Clients:** Download and process work units, upload results, view progress
- **Researchers:** Submit work units, monitor results, access research data
- **Coordinators:** Manage work unit distribution, validate results, assign credits
- **System Administrators:** Manage system configuration, monitor health, handle security

### Primary User Journey
**Volunteer Client Workflow:**
1. **Registration:** Client registers with coordinator service
2. **Authentication:** Client authenticates using JWT tokens
3. **Work Unit Assignment:** Client requests and receives work unit
4. **Processing:** Client processes work unit with periodic checkpoints
5. **Result Upload:** Client uploads completed result
6. **Validation:** System validates result integrity and correctness
7. **Credit Assignment:** Client receives credit for validated work
8. **Repeat:** Client can request additional work units
