---
name: use-gamma-slides
description: |
  Use when the user wants to create, generate, or build slide presentations
  using the Gamma API. Covers client setup, slide generation with prompts,
  customization options, and multi-slide deck creation.
  Trigger with phrases like "create slides", "gamma presentation",
  "generate a deck", "make slides with gamma", "build a slide deck".
allowed-tools: Read, Write, Edit
version: 1.0.0
license: MIT
---

# Use Gamma Slides

## Overview
Generate slide presentations using the Gamma API. Supports prompt-driven slide creation, style customization, and multi-slide deck generation.

## Prerequisites
- Gamma SDK installed (`npm install @gamma/sdk` or `pip install gamma`)
- `GAMMA_API_KEY` environment variable set
- Completed `gamma-install-auth` setup

## Instructions

### Step 1: Initialize the Gamma Client

**TypeScript:**
```typescript
import { GammaClient } from '@gamma/sdk';

const gamma = new GammaClient({
  apiKey: process.env.GAMMA_API_KEY,
});
```

**Python:**
```python
from gamma import GammaClient

client = GammaClient()
```

### Step 2: Generate a Slide Deck

Provide a title and a prompt describing the content. Gamma generates the slides from the prompt.

**TypeScript:**
```typescript
const deck = await gamma.presentations.create({
  title: 'Quarterly Business Review',
  prompt: 'Create a 5-slide deck covering Q1 revenue, customer growth, product milestones, challenges, and Q2 priorities',
  style: 'professional',
});

console.log('Deck URL:', deck.url);
```

**Python:**
```python
deck = client.presentations.create(
    title='Quarterly Business Review',
    prompt='Create a 5-slide deck covering Q1 revenue, customer growth, product milestones, challenges, and Q2 priorities',
    style='professional',
)

print(f'Deck URL: {deck.url}')
```

### Step 3: Customize Style and Slide Count

Control the output with additional parameters:

```typescript
const deck = await gamma.presentations.create({
  title: 'Product Launch',
  prompt: 'Present our new AI assistant: problem it solves, key features, demo walkthrough, pricing, and call to action',
  style: 'modern',        // Options: professional, modern, minimal, bold
  slideCount: 6,          // Target number of slides
  audience: 'executives', // Tailors language and depth
});
```

### Step 4: Generate from Structured Content

Pass structured data instead of a freeform prompt for precise control:

```typescript
const deck = await gamma.presentations.create({
  title: 'Team Onboarding',
  slides: [
    { heading: 'Welcome', content: 'Overview of the team mission and values' },
    { heading: 'Tools & Access', content: 'Development environment, CI/CD, and communication channels' },
    { heading: 'First Week Plan', content: 'Day-by-day onboarding schedule with key milestones' },
  ],
});
```

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| Import Error | SDK not installed | Run `npm install @gamma/sdk` or `pip install gamma` |
| Auth Error | Invalid or missing API key | Verify `GAMMA_API_KEY` is set |
| Timeout | Network issues or large deck | Increase timeout or reduce slide count |
| Rate Limit | Too many requests | Wait and retry with exponential backoff |
| Validation Error | Invalid style or parameters | Check supported values in API docs |

## Resources
- [Gamma Getting Started](https://gamma.app/docs/getting-started)
- [Gamma API Reference](https://gamma.app/docs/api)
- [Gamma Examples](https://gamma.app/docs/examples)
