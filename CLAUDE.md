# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Fit Track** is a personal, data-driven fitness tracking application that uses AI as an intelligent diary. The app is built with Next.js 16, React 19, TypeScript, and Tailwind CSS, featuring a custom dark-first design system.

Core philosophy:
- Chat-first interaction (all user input happens via chat)
- Data-driven (no motivational fluff, just facts)
- Import complements chat (reduces manual work)
- Minimal information per screen
- User control (no automatic actions)

## Development Commands

```bash
# Development
npm run dev              # Start dev server with Turbopack
npm run dev:clean        # Kill existing Node processes, remove .next lock, start dev

# Production
npm run build            # Build for production
npm start                # Start production server

# Quality
npm run lint             # Run ESLint
```

## Architecture

### Tech Stack
- **Framework**: Next.js 16 (App Router)
- **React**: v19.2.3
- **TypeScript**: v5.9.3
- **Styling**: Tailwind CSS 3.4.19
- **UI Components**: Radix UI + custom design system
- **Icons**: Material Symbols Outlined
- **Fonts**: Inter (Google Fonts)
- **AI**: OpenAI API (for chat functionality)

### Directory Structure

```
fit-tracker-master/
├── app/                      # Next.js App Router
│   ├── api/                 # API routes
│   │   └── chat/           # Chat endpoint for AI
│   ├── chat/               # Chat screen (primary interface)
│   ├── home/               # Dashboard/home screen
│   ├── import/             # Data import screen
│   ├── insights/           # Analytics/insights screen
│   ├── profile/            # User profile screen
│   ├── onboarding/         # Onboarding flow (5 screens)
│   └── styleguide/         # Design system reference
├── components/             # Reusable React components
│   ├── chat/              # Chat-specific components
│   ├── feedback/          # Feedback components (Toast, etc)
│   ├── home/              # Home screen components
│   ├── layout/            # Layout components (BottomNav, Header, etc)
│   └── ui/                # Base UI components (shadcn/ui style)
├── lib/                    # Core utilities and services
│   ├── ai.ts              # AI service (OpenAI integration)
│   ├── storage.ts         # localStorage wrapper with types
│   ├── parsers.ts         # Data parsers (Apple Health, Hevy)
│   └── utils.ts           # Utility functions
├── hooks/                  # Custom React hooks
├── schemas/                # Data schemas
├── docs/                   # Product documentation
│   ├── prd/               # Product requirement docs
│   ├── DESIGN_SYSTEM.md   # Visual design tokens
│   ├── COMPONENTS.md      # Component catalog
│   └── PROMPT_CLAUDE_CODE.md  # Implementation guide
└── json_diario/           # Sample data files
```

### Data Architecture

The app uses localStorage for client-side persistence. Key entities (defined in [lib/storage.ts](lib/storage.ts)):

- **User Profile**: Basic user info, gender, birth date, height, BMR
- **Chat Messages**: Conversational history
- **Meals & MealItems**: Food logs with macros
- **Workouts**: Training sessions
- **WeightLog & BodyFatLog**: Body composition tracking
- **Import History**: Record of data imports

All entities include:
- `id` (UUID)
- `timestamp` or `createdAt`
- `source` (enum: 'chat', 'import_apple', 'import_hevy', 'manual')

**Important**: Chat has priority for corrections. Imports never overwrite manual corrections. Always keep the most detailed and most recent record.

### Navigation Structure

Bottom tabs (fixed order):
1. **Chat** (default) - Primary interaction point
2. **Home** - Dashboard with summaries
3. **Importar** - Data import (Apple Health, Hevy)
4. **Insights** - Analytics and trends
5. **Profile** - User settings

Onboarding flow (5 screens):
1. Welcome hero
2. Feature tour (3 screens)
3. Basic profile form

## Design System

The app uses a **dark-first** custom design system. Always reference [docs/DESIGN_SYSTEM.md](docs/DESIGN_SYSTEM.md) for visual tokens.

### Critical Design Rules

**Colors** (never use generic Tailwind grays):
- Primary: `#eb6028` (orange)
- Background: `#211511` (dark brown)
- Surface cards: `#403d39`, `#2f221d`, `#2a2624`
- Borders: `#54423b`
- Text secondary: `#b9a59d`

**Typography**:
- Font: Inter only
- Always apply `antialiased` to body
- Titles: `font-bold tracking-tight`
- Labels: `text-xs font-medium uppercase tracking-wider`

**Spacing**:
- Container: `max-w-md mx-auto` (mobile-first)
- Screen padding: `px-4`
- Bottom nav clearance: `pb-24` on content

**Border Radius**:
- Cards: `rounded-xl` or `rounded-2xl`
- Inputs: `rounded-lg`
- Large buttons: `rounded-xl` or `rounded-2xl`
- Pills/badges: `rounded-full`

**Shadows**:
- Cards in dark mode: usually no shadow or `shadow-sm`
- Primary buttons: `shadow-lg shadow-primary/30`
- FAB: `shadow-xl shadow-primary/30`

**Icons**:
- Material Symbols Outlined
- Navigation: `text-[24px]`
- Cards: `text-[20px]`
- Inline: `text-[18px]`
- Filled (active state): add `fill-1` class with `font-variation-settings: 'FILL' 1`

**Transitions**:
- Always add to interactive elements
- Standard: `transition-colors` or `transition-all duration-200`
- Press effect: `active:scale-95` or `active:scale-[0.98]`

### Tailwind Configuration

Custom colors are defined in [tailwind.config.ts](tailwind.config.ts):
- `primary`, `background-dark`, `surface-dark`, `surface-card`, `surface-input`
- `border-subtle`, `text-floral`, `text-secondary`, `icon-bg`
- Semantic colors: `success`, `error`, `warning`

Custom shadows: `primary`, `primary-lg`, `glow`

## AI Integration

The AI chat system ([lib/ai.ts](lib/ai.ts)) handles:
- Natural language meal logging
- Data extraction and parsing
- Contextual suggestions
- Memory of user preferences (food items only, after explicit confirmation)

**AI Behavior Rules**:
- Never initiates conversation
- Only responds to explicit input
- Direct, human, technical language
- Suggestions appear only when useful (contextual chips)

## Data Import

The app supports importing from:
1. **Apple Health** (ZIP/XML): weight, body fat, workouts, sleep, time series
2. **Hevy** (CSV): workout sessions, sets, reps, weights

Parsers are in [lib/parsers.ts](lib/parsers.ts) and handle deduplication based on the "most detailed and most recent" rule.

## Key Implementation Patterns

1. **Component Structure**: Follow the modular component approach in [docs/COMPONENTS.md](docs/COMPONENTS.md)
2. **File Naming**: PascalCase for components (`InsightCard.tsx`), camelCase for utilities (`dateUtils.ts`)
3. **State Management**: Use React hooks and localStorage via [lib/storage.ts](lib/storage.ts)
4. **Styling**: Always use design system tokens, never arbitrary values
5. **Mobile-First**: All layouts use `max-w-md mx-auto` container
6. **Safe Areas**: Remember `pb-24` on content to avoid bottom nav overlap

## Common Calculations

BMR (Mifflin-St Jeor):
- Male: `BMR = 10 × weight(kg) + 6.25 × height(cm) − 5 × age + 5`
- Female: `BMR = 10 × weight(kg) + 6.25 × height(cm) − 5 × age − 161`

TDEE: `BMR × activity_factor` (sedentary: 1.2, light: 1.375, moderate: 1.55, active: 1.725, very active: 1.9)

## Important Product Principles

When implementing features, always respect:

1. **Chat-first**: All input happens via chat. No screen replaces chat as the action point.
2. **Import complements**: Imports reduce manual work, but chat is the source of truth for corrections.
3. **Minimal screens**: Home and Insights are deliberately sparse.
4. **Data over motivation**: Direct language, no empty phrases.
5. **User control**: Nothing automatic without explicit action.

## Documentation

For detailed product requirements, see:
- [docs/prd/00_MASTER.md](docs/prd/00_MASTER.md) - Product vision and principles
- [docs/prd/01_GLOSSARIO.md](docs/prd/01_GLOSSARIO.md) - Term definitions
- [docs/prd/02_ARQUITETURA.md](docs/prd/02_ARQUITETURA.md) - Technical architecture
- [docs/prd/features/](docs/prd/features/) - Individual feature specs

For design implementation:
- [docs/DESIGN_SYSTEM.md](docs/DESIGN_SYSTEM.md) - Visual design tokens and patterns
- [docs/COMPONENTS.md](docs/COMPONENTS.md) - Complete component catalog with code
- [docs/PROMPT_CLAUDE_CODE.md](docs/PROMPT_CLAUDE_CODE.md) - Step-by-step migration guide
