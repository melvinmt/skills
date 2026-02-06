# Prompting Patterns for Agentic Website Design

Example prompts for each phase of the design process. Use these as templates — adapt to your specific project.

## Phase 1: Project Setup

```
Create a new Astro site for [product/company]. Set up the project structure with:
- Layout component with proper meta tags, Open Graph, and font loading
- Global CSS with CSS custom properties for colors, spacing, and typography
- Component files for each section: Hero, Features, Pricing, Testimonials, Footer
- Use [color palette / brand guidelines] for the design system
```

## Phase 2: Hero Section

### Initial Generation

```
Use the frontend-design skill to build the hero section. Requirements:
- Bold headline that communicates [core value prop]
- Subtitle that expands on the headline
- Primary CTA button and secondary text link
- Hero visual (gradient background / illustration placeholder / screenshot)
- Entry animations: headline fades in first, then subtitle, then CTA
```

### Copywriting Pass

```
Use the copywriting skill to rewrite the hero headline and subtitle.
Current: "[current headline]"
Target audience: [describe audience]
Tone: [confident / friendly / technical / bold]
The headline should be benefit-driven, not feature-driven.
```

### Iteration

```
Screenshot the hero section. Now improve it:
- The headline needs more visual weight — increase size and adjust letter-spacing
- Add a subtle gradient or color accent to the key word in the headline
- The CTA button needs more presence — larger padding, bolder color
- Improve the entry animation timing — it feels too fast
```

```
Ok, now let's push this further. What would a top design agency do differently here?
Make the hero feel more premium and intentional.
```

## Phase 3: Features Section

### Initial Generation

```
Use the frontend-design skill to design the features section. Include:
- Section heading with subtitle
- 3-4 feature cards in a responsive grid
- Each card: icon, title, description
- Scroll-triggered staggered fade-in animation
- Features: [list your features]
```

### Refinement

```
Use the copywriting skill to improve the feature descriptions. Each one should:
- Lead with the benefit, not the feature name
- Be 1-2 sentences max
- Use active voice
- Make the reader feel the problem being solved
```

```
Now screenshot and improve the visual design:
- Add more whitespace between cards
- The icons need to be more distinctive — use different colors or styles
- Add a hover state that lifts the card with a shadow
- Stagger the scroll animation so cards appear left-to-right
```

## Phase 4: Social Proof

```
Design the social proof section with:
- Logo bar of [companies/publications]
- 2-3 testimonial cards with name, title, photo placeholder, and quote
- A key metric ("10,000+ users" or "99.9% uptime")
- Subtle background differentiation from adjacent sections
```

```
The testimonials feel generic. Use the copywriting skill to make them feel authentic:
- Vary the length (one short, one medium)
- Include specific details or metrics in each quote
- Make each voice distinct
```

## Phase 5: Pricing

```
Use the frontend-design skill to design the pricing section:
- 3 tiers: [Free/Pro/Enterprise] or [Starter/Growth/Scale]
- Recommended tier should be visually prominent
- Feature comparison list in each card
- CTA button per tier
- Annual/monthly toggle if applicable
```

```
Improve the pricing cards:
- The recommended tier needs a badge or highlight border
- Add a subtle scale difference — recommended card slightly larger
- Check mark icons for included features, dash for excluded
- Hover animation on each card
```

## Phase 6: Final Polish

### Animation Pass

```
Review all animations on the page and improve them:
- Ensure consistent easing (use cubic-bezier(0.16, 1, 0.3, 1) as default)
- Check animation timing — nothing should feel abrupt or sluggish
- Add micro-interactions I might have missed: button hovers, link underlines, input focus
- Verify animations respect prefers-reduced-motion
```

### Responsive Pass

```
Screenshot the entire page at 375px width. Fix any issues:
- Text overflow or horizontal scrolling
- Touch targets smaller than 44px
- Images breaking layout
- Navigation usability on mobile
```

### Performance Pass

```
Optimize the site for performance:
- Ensure images use proper formats (WebP/AVIF) and lazy loading
- Minimize CSS — remove unused styles
- Check that no unnecessary JavaScript is shipped
- Add proper font loading strategy (font-display: swap)
- Target Lighthouse 90+ on all categories
```

## Iteration Phrases

Use these to keep pushing quality higher:

| Prompt | Purpose |
|--------|---------|
| "Now improve what you just did" | General quality push |
| "This is good but not great — what would make it exceptional?" | Raise the bar |
| "The animations need more personality" | Motion refinement |
| "Make this feel more premium" | Overall polish |
| "Screenshot and tell me what's wrong" | Self-critique |
| "What would [Apple / Stripe / Linear] do differently?" | Design benchmark |
| "Push the typography harder — more hierarchy, more contrast" | Typography focus |
| "The spacing feels inconsistent — audit and fix all padding/margin" | Layout precision |
