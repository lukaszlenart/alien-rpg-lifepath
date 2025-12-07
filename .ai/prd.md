# Product Requirements Document (PRD) - Alien RPG Lifepath Generator

## 1. Product Overview

The Alien RPG Lifepath Generator is a web-based application designed to streamline the character creation process for the Alien Tabletop Roleplaying Game (1st edition). By automating the complex series of dice rolls, table lookups, and attribute calculations required by the lifepath system, the tool allows players to focus on the narrative emergence of their character rather than the mechanics.

The MVP focuses on delivering a core, robust generation experience for standard character types (Human, Android, A.W. Soldier), limiting users to 2 character slots to demonstrate value while managing resource costs. It integrates optional AI-powered narrative generation to flesh out the raw data into compelling backstories.

## 2. User Problem

The Pain Point:
Creating a character using the "Lifepath" method in Alien RPG manually is a time-intensive process. It requires:
- Repeatedly referencing multiple tables across different pages.
- Manually rolling dice (D6, 2D6, D66) dozens of times.
- Tracking cumulative attribute and skill point modifications.
- Managing resource caps and aging penalties manually.

The Consequence:
Players often feel overwhelmed or fatigued before the game even begins. The mechanical burden distracts from the storytelling potential of the lifepath system.

The Solution:
An automated generator that:
- Instantly handles all RNG and table lookups.
- Automatically calculates stats and enforces rule limits.
- Presents the results in a clear, chronological narrative flow.
- Allows players to make high-level decisions (like "One more term?") without bogging down in math.

## 3. Functional Requirements

### 3.1. Authentication & User Management
- System: Supabase Auth.
- Features: Sign Up, Sign In, Sign Out.
- Constraint: Access to the generator requires a logged-in account.

### 3.2. Dashboard & Slot Management
- Limit: Each user is strictly limited to 2 concurrent character slots.
- Operations:
    - Create: Only available if slot count < 2.
    - View: List existing characters with basic summary.
    - Delete: Permanently remove a character to free up a slot.

### 3.3. Character Generation Engine
- Supported Natures: Human, Android, A.W. Soldier (determined by forced random roll).
- Phases:
    1.  Nature & Origin: Auto-rolled.
    2.  Early Life & Life Events: Auto-rolled; stats applied automatically.
    3.  Career Application: User chooses attribute bonuses; auto-roll skill check.
        - Success: Proceed to Career Terms.
        - Failure: Default to "Drifter" path (generic fallback).
    4.  Career Terms (1-3):
        - "Push Your Luck" mechanic: After Term 1 and Term 2, user decides to continue or stop.
        - Aging: If continuing to Term 2/3, apply -1 attribute penalty (User choice or random if automated for MVP simplicity). *Refinement: User choice is preferred for agency, but system must enforce it.*
    5.  Final Touches: Name, Appearance, Personal Agenda.

- Rules Enforcement:
    - Caps: Attribute points exceeding max (Human: 5, Android: 8, etc.) are discarded.
    - RNG: All table results are determined by the system.

### 3.4. AI Narrative Integration
- Mechanism: User can optionally provide their own OpenAI API Key.
- Function: If a key is present, the app sends the raw table results (e.g., "Origin: United Americas", "Event: Betrayal") to the LLM to generate a cohesive paragraph of backstory text.
- Fallback: If no key is provided, display the raw text from the rule tables.

### 3.5. Persistence
- Storage: Supabase Database.
- Data: Complete character sheet (Attributes, Skills, Talents, Gear Log, Narrative).
- Inventory: Gear and Cash are stored as a chronological log (e.g., "Term 1: Pistol ($500)", "Term 2: Medkit ($100)").

## 4. Product Boundaries

| In Scope | Out of Scope |
| :--- | :--- |
| Standard Natures (Human, Android, A.W. Soldier) | Homebrew Natures (Kids, Hybrids, Lowlife) |
| Standard Careers | Custom/Homebrew Careers |
| 2 Save Slots per User | Unlimited Slots / Paid Subscriptions |
| Supabase Auth & Database | Payment Processing |
| OpenAI API Key Integration (BYOK) | Built-in Free AI Generation |
| "Drifter" Fallback for failed careers | Complex multi-career branching (switching careers mid-path) |
| Web-based Interface | PDF / JSON Export |

## 5. User Stories

### Authentication

| ID | Title | Description | Acceptance Criteria |
| :--- | :--- | :--- | :--- |
| US-001 | User Sign Up | As a new user, I want to create an account so I can save my characters. | - User can register with email/password via Supabase.<br>- Upon success, user is redirected to the Dashboard. |
| US-002 | User Login | As a returning user, I want to log in to access my saved characters. | - User can log in with valid credentials.<br>- Invalid credentials show an error message. |
| US-003 | User Logout | As a logged-in user, I want to log out to secure my account. | - Clicking logout ends the session and redirects to the landing page. |

### Slot Management

| ID | Title | Description | Acceptance Criteria |
| :--- | :--- | :--- | :--- |
| US-004 | View Dashboard | As a logged-in user, I want to see my current characters and available slots. | - Dashboard shows count (e.g., "1/2 Slots Used").<br>- Lists existing characters with Name and Archetype. |
| US-005 | Create Character Limitation | As a user with 2 characters, I should be prevented from creating a 3rd. | - "Generate New" button is disabled or hidden when 2 characters exist.<br>- Tooltip explaining the limit appears. |
| US-006 | Delete Character | As a user, I want to delete an old character to make room for a new one. | - User can click delete on a character.<br>- Confirmation modal appears.<br>- Upon confirmation, slot count decreases by 1. |

### Generation Flow

| ID | Title | Description | Acceptance Criteria |
| :--- | :--- | :--- | :--- |
| US-007 | Generate Nature & Origin | As a user, I want to start a new path and see my Nature and Origin determined automatically. | - Clicking "Start" rolls Nature (Human/Android/AW) and Origin tables.<br>- Results are displayed immediately.<br>- Initial attributes/skills are updated in background. |
| US-008 | Process Early Life & Events | As a user, I want to see my character's early history and life events auto-generated. | - System auto-rolls Early Life and Life Event tables.<br>- Stat bonuses are applied automatically.<br>- Narrative text for results is displayed. |
| US-009 | Attribute Capping | As a user, I want to ensure my stats don't break the game rules. | - If a bonus would push an attribute above its max (e.g., Human > 5), the surplus point is discarded. |
| US-010 | Career Application | As a user, I want to apply for a career and see if I get accepted. | - User selects preferred Career (or is guided to one).<br>- System rolls skill check.<br>- Success: Proceed to Term 1.<br>- Failure: Character becomes "Drifter". |
| US-011 | Career Term Loop | As a user, I want to simulate a term of service. | - System rolls Posting, Performance, Talent, Gear, Paycheck.<br>- Results are added to character sheet.<br>- Inventory log is updated. |
| US-012 | Term Continuation (Push Your Luck) | As a user, I want to decide whether to serve another term or retire. | - After Term 1 and 2, user is asked: "Retire or Serve Another Term?".<br>- If "Serve": Proceed to next term.<br>- If "Retire": Proceed to Final Touches. |
| US-013 | Aging Penalty | As a user serving extra terms, I must pay the aging cost. | - If starting Term 2 or 3, user must select an attribute to reduce by 1.<br>- System enforces reduction before term start. |
| US-014 | Drifter Fallback | As a user who failed career application, I want a valid playable character result. | - If application fails, career tracks "Drifter" path.<br>- "Drifter" terms are auto-generated to complete the character. |
| US-015 | AI Narrative Generation | As a user with an API key, I want rich stories instead of table results. | - User enters OpenAI Key in a settings modal.<br>- During generation, raw results are replaced/augmented by AI-generated paragraphs.<br>- Fallback to raw text if API call fails. |

### Character Sheet

| ID | Title | Description | Acceptance Criteria |
| :--- | :--- | :--- | :--- |
| US-016 | View Final Character | As a user, I want to view my completed character sheet. | - Displays final Attributes, Skills, Talents, Health/Stress/Radiation (base values).<br>- Displays chronological Gear/Cash log.<br>- Stats are read-only. |
| US-017 | Customize Details | As a user, I want to name and describe my character. | - Name, Appearance, and Narrative/Bio fields are editable.<br>- Changes can be saved to the database. |

## 6. Success Metrics

1.  Generation Speed: Average time to generate a character (from start to save) is under 5 minutes.
2.  Completion Rate: >90% of started generation sessions result in a saved character (indicating no critical blockers or confusion).
3.  Slot Utilization: Track how often users hit the 2-slot limit (validating the potential demand for subscription/expansion).
4.  AI Usage: Percentage of users inputting an API key (validating the value of the narrative feature).
