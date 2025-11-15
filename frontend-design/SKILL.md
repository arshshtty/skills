---
name: frontend-design
description: Creates minimalistic, production-grade frontend interfaces using shadcn/ui with themes from tweakcn.com. Apply when building React components, landing pages, dashboards, or any frontend UI work.
---

# Frontend Design Skill

Generate distinctive, minimalistic, production-grade frontend interfaces that avoid generic AI aesthetics.

## Core Stack

- **Component Library**: shadcn/ui (https://ui.shadcn.com)
- **Theming**: Use themes from tweakcn.com (https://tweakcn.com)
- **Styling**: Tailwind CSS with CSS variables
- **Framework**: React with TypeScript

## Design Philosophy: Minimalism

### Key Principles
- **Less is more**: Remove all non-essential elements
- **Whitespace**: Use generous spacing to create breathing room and visual hierarchy
- **Clarity**: Clear, purposeful layouts that guide the user's eye
- **Restraint**: Avoid decoration for decoration's sake
- **Subtle details**: Small, intentional touches that enhance without overwhelming

## Avoid Common AI Slop

Never default to these overused patterns:

❌ **Typography**: Inter or Roboto fonts
❌ **Colors**: Purple gradients (#8B5CF6 → #3B82F6) on white backgrounds
❌ **Layout**: Everything centered with uniform padding
❌ **Borders**: Uniform `rounded-lg` on all elements
❌ **Depth**: Excessive shadows and layering
❌ **Components**: Every section wrapped in a card with shadow

## Instead, Apply These Patterns

### Typography
- Choose distinctive but readable fonts:
  - **Sans-serif**: Geist, Satoshi, Space Grotesk, DM Sans
  - **Serif**: Newsreader, Lora, Fraunces, Crimson Pro
- Use 2-3 font sizes per view maximum
- Generous line-height: 1.6-1.8 for body text
- Subtle weight variations for hierarchy

### Color Palettes
- **Start with tweakcn.com**: Browse and select a minimal theme
- **Limited palette**: 2-3 primary colors maximum
- **Neutral foundations**: Off-white, subtle grays, or muted tones
- **High contrast**: Ensure text is readable
- **Semantic colors**: Use theme variables for success/error/warning states

### Layout & Spacing
- **Asymmetric balance**: Not everything needs to be centered
- **Generous whitespace**: Let content breathe
- **Intentional grouping**: Use proximity to show relationships
- **Varied spacing**: Mix tight and loose spacing for rhythm
- **Minimal containers**: Don't wrap everything in cards

### Components
- **Use shadcn/ui** as the foundation
- **Selective rounding**: Vary border-radius (some sharp, some rounded)
- **Subtle borders**: Often better than shadows for separation
- **Flat or minimal depth**: Avoid heavy shadows
- **Clean edges**: Crisp, intentional boundaries

### Animations
- **Minimal motion**: Only when it serves a purpose
- **Fast transitions**: 150-300ms
- **Simple effects**: Subtle hover states and focus indicators
- **Performance-first**: Use CSS transforms and opacity

## Implementation Workflow

### 1. Choose a Theme from tweakcn.com
- Visit https://tweakcn.com
- Select a minimal, clean theme
- Copy CSS variables for your Tailwind config
- Customize if needed for your specific use case

### 2. Set Up shadcn/ui
```bash
npx shadcn-ui@latest init
```

Configure with:
- TypeScript: Yes
- Tailwind CSS: Yes
- CSS variables: Yes

### 3. Add Only Needed Components
```bash
# Install components as you need them
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add input
# etc.
```

### 4. Build with Minimalist Principles
1. Start with a clean, simple layout
2. Add content with clear hierarchy
3. Use whitespace intentionally
4. Apply consistent typography scale
5. Add subtle interactions last

## Context-Aware Design

Adapt the approach based on the interface type:

**Dashboard**: Data-focused, scannable, functional over decorative. Use tables, charts, and clean data visualization.

**Landing Page**: Clear value proposition, ample whitespace, visual interest through layout not decoration.

**Settings Panel**: Organized sections, clear labels, minimal chrome, easy to scan.

**E-commerce**: Product-focused, clean imagery, subtle CTAs, easy navigation.

**SaaS Application**: Professional, consistent, efficiency-focused, keyboard-friendly.

**Portfolio**: Content-first, unique layouts, personality through typography and spacing.

## Code Quality

- Use TypeScript for type safety
- Mobile-first responsive design
- Semantic HTML and ARIA labels for accessibility
- Modular, reusable components
- Semantic Tailwind classes
- Extract repeated patterns into components
- Keep component files focused and small

## Example Patterns

### Hero Section (Not Centered!)
Instead of centered hero with gradient background:
```tsx
<section className="pt-24 pb-16">
  <div className="max-w-3xl"> {/* Aligned left, not centered */}
    <h1 className="text-5xl font-bold tracking-tight">
      Clear, Bold Statement
    </h1>
    <p className="mt-6 text-lg text-muted-foreground max-w-2xl">
      Supporting text with generous spacing and breathing room.
    </p>
    <Button className="mt-8" size="lg">
      Simple CTA
    </Button>
  </div>
</section>
```

### Card Alternative (Minimal Container)
Instead of shadowed cards everywhere:
```tsx
<div className="border-l-2 border-border pl-6 py-4">
  <h3 className="font-semibold">Section Title</h3>
  <p className="text-sm text-muted-foreground mt-2">
    Content with subtle left border instead of full card.
  </p>
</div>
```

### Data Display (Clean Tables)
Prefer clean tables over card grids for data:
```tsx
<table className="w-full border-collapse">
  <thead className="border-b">
    <tr className="text-left text-sm text-muted-foreground">
      <th className="pb-3 font-medium">Column</th>
    </tr>
  </thead>
  <tbody className="divide-y">
    <tr className="text-sm">
      <td className="py-4">Clean, readable data</td>
    </tr>
  </tbody>
</table>
```

## Resources

- shadcn/ui Docs: https://ui.shadcn.com/docs
- tweakcn Theme Editor: https://tweakcn.com/editor/theme
- Tailwind CSS: https://tailwindcss.com/docs

---

**Remember**: Create interfaces that feel intentional, refined, and distinctively minimal—not generic or overdesigned.
