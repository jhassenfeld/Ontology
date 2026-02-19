# Circles of Knowledge — Implementation Spec

## Overview

An interactive single-page HTML app that visualizes how academic disciplines relate to each other through nested ontology. Philosophy encompasses everything; each subsequent discipline narrows its domain. The app uses concentric circles to show this containment relationship and pulls content from Wikipedia to surface interesting questions within each discipline.

The goal: help college students discover what kinds of questions interest them by exploring real topics across disciplines.

## Output

**Single HTML file** — all CSS, JS, and data inline. No build tools. Only external dependency allowed is a Google Fonts link.

---

## Visual Design

### Aesthetic
- **Light & clean**. White/off-white background, plenty of whitespace, refined typography.
- Avoid generic AI aesthetics (no purple gradients, no Inter/Roboto). Use distinctive but readable fonts — one serif display font paired with a clean sans-serif body font.
- Each discipline gets a distinct color used for its circle ring, heading, and accents.

### Layout
Two-panel layout on desktop:
- **Left panel**: Concentric circles visualization (interactive)
- **Right panel**: Detail view for the selected discipline (definition, domain, questions)

On mobile, stack vertically: circles on top, detail below.

---

## Concentric Circles Visualization

The core visual is a set of 7 nested concentric circles, outermost to innermost:

1. **Philosophy** (outermost) — "All things"
2. **Math** — "Things described by numbers"
3. **Physics** — "Things in time and space"
4. **Chemistry** — "Elements"
5. **Biology** — "Living organisms"
6. **Anthropology** — "People"
7. **History** (innermost) — "People who create societies"

### Behavior
- Each ring is labeled with the discipline name
- Clicking a ring selects that discipline and updates the right panel
- The selected ring should be visually highlighted (thicker stroke, glow, color fill, or similar)
- On hover, rings should have a subtle highlight effect
- Default state on load: Philosophy is selected

### Implementation
Use SVG or Canvas. SVG is preferred for easier click handling and styling.

---

## Right Panel — Discipline Detail

When a discipline is selected, the right panel shows:

### 1. Discipline Name
Large heading in the discipline's color.

### 2. Definition
One-line definition (from the source content below).

### 3. Domain / Ontology
A short statement explaining what subset of reality this discipline studies, and how it narrows from the parent discipline. This is the key pedagogical content.

### 4. Questions (3 displayed)
Three questions related to the discipline. These come from two sources:

- **Default questions** (from the original content) show on initial load while Wikipedia is fetching
- **Wikipedia questions** replace the defaults once loaded

Each question shows:
- The question text (rephrased from the Wikipedia article extract)
- A small link to the full Wikipedia article

### 5. Refresh Button
A button to load 3 new Wikipedia questions for the current discipline. Shows a brief loading state while fetching.

---

## Source Content (Defaults)

Use these as the default display before Wikipedia content loads:

### Introduction 

There are things. 

But that doesn't really tell you much on its own.

If you want to know anything, you have to know what *kind* of things there are.  

Some things, like an **apple** are *red* and *sweet*. They have color and taste.

But not everything is like an **apple**. Some things are very different kinds of things.

Some things, like **the number 6** are *even*, *triangular*, and *perfect*. 

Imagine a big basket. It holds everything that exists. That basket is your ontology.  It is all the things you believe exist. 

What kinds of things are in your basket?

I bet **apples** are in there.  Is **the number 6**? Is **justice**? 

The branches of human knowledge, sometimes called disciplines, have different ontologies, that is, they study different kinds of things. 

This tool organizes human knowledge into a Venn diagram of ontology where each discipline studies a subset of the ontology of the previous discipline.  It shows something about how ontology structures the kinds of questions we ask and the kinds of answers we find.

What kinds of things are you most curious about?  What questions do you have?
### Philosophy

Philosophy studies everything.  Full stop.  Anything that anyone thinks could be a thing could be a thing that philosophy asks a question about. It includes things like minds and bodies as well as numbers and concepts. Everything.

- **Ontology:** All things
- **Questions:**
- What makes an action "right" or "wrong," and how do we tell the difference?
- What IS reality really? How do we know?
### Math

Math studies the subset of things that can be quantified, that is, described with numbers. Even the parts of math that seem to deal with non-numerical things, like sets, still talk in terms of quantities like the set with no elements or the dimensionality of a space.

- **Ontology**: Subset of all things that can be quantified.
- **Questions:**
- Is every even number greater than 2 the sum of two prime numbers?
- Are there more integers or more real numbers?
### Physics

Physics studies only those things that exist in time and space.  That is, for physics to study a thing it has to have a where and a when.  It has to exist in a location at a particular time. It doesn't study the properties of the number two, which, whatever it is, is neither temporal or spatial.  

- **Ontology:** Subset of quantified things that exist in time and space.
- **Questions:**
- What is gravity, and why does it pull objects down to the ground?
- How does a roller coaster work?
### Chemistry

Chemistry studies the things that are atoms (or parts of atoms).  Atoms are just a few of things that exist in time and space.  It turns out though, that these atomic things make up a huge part of the things that we can see, hear, taste, smell, touch, and otherwise interact with. 

- **Ontology:** Subset of spatial and temporal things that are made of atoms
- **Questions:**
- How do the elements in the periodic table relate to each other, and what makes each one unique?
- Why do some metals rust?
### Biology

Biology studies the special group of things that are made of atoms that are alive, whatever that means.  These living collections of atoms do a lot of surprising things like reproduce, metabolize energy, and so much more.

- **Definition:** Study of atomic things that are alive
- **Domain:** Subset of chemicals: living organisms
- **Questions:**
- How does DNA help us inherit traits from our parents?
- How do plants convert sunlight into energy?
### Anthropology

Anthropology studies the subset of living things that are people.  It asks how people got to be the way they are, and how their biology allows them to do certain special people things like make culture. 

- **Ontology:** Subset of living things that are people
- **Questions:**
- How did human brains develop over time, and how did this affect our ability to create tools and solve problems?
- Why do people walk on two legs?
### History

History studies the subset of people that make societies.  It traces how those societies change over time with special focus on the written traces those societies leave behind. 

- **Ontology:** Subset of people that make socieites
- **Domain:** Subset of people: ones that create societies
- **Questions:**

- Why did ancient civilizations like Egypt and Mesopotamia build their cities near rivers?
- How did the Industrial Revolution change the way people lived and worked?

---

## Wikipedia Integration

### Strategy
Each discipline has a hardcoded pool of 30–50 curated Wikipedia article titles. At runtime, 3 are randomly selected, their extracts are fetched from the Wikipedia API, and the extract is rephrased as a question for display.

### API Endpoint
Use the Wikipedia REST API to fetch article extracts:

```
https://en.wikipedia.org/api/rest_v1/page/summary/{title}
```

This returns JSON including:
- `title` — article title
- `extract` — short plain-text summary
- `content_urls.desktop.page` — link to the full article

No API key required. CORS is supported.

### Question Generation
Since we can't use an LLM at runtime to rephrase extracts as questions, use this approach:
- Prepend a question frame based on the discipline. For example: "What is...?", "How does...?", "Why do...?"
- Use the article **title** as the basis for the question, with the **extract** (truncated to ~1–2 sentences) as a description shown below the question
- Format: **"What is [Article Title]?"** + short extract + link

Alternatively, hardcode a question version of each article title in the curated pool alongside the Wikipedia title. This gives better questions at the cost of more upfront data. **This is the preferred approach** — the curated pool should be an array of objects like:

```js
{ question: "Why can't we divide by zero?", article: "Division_by_zero" }
```

### Curated Article Pools
Include 30–50 entries per discipline. Choose topics that are:
- Interesting and accessible to college students
- Well-represented on Wikipedia (good article quality)
- Genuinely representative of the discipline's types of questions

At runtime, pick 3 at random (without repeating until the pool is exhausted), fetch the extract from Wikipedia for the short description, and display.

### Error Handling
- If a Wikipedia fetch fails, show the question without a description
- If all fetches fail, keep showing the default questions
- Show a subtle loading indicator while fetching (spinner or skeleton text)

---

## Interaction Flow

1. Page loads → concentric circles render with Philosophy selected → default Philosophy questions display in right panel
2. Wikipedia fetches begin in background for Philosophy
3. When fetches complete, default questions are replaced with Wikipedia questions (with a subtle fade transition)
4. User clicks a different ring → right panel updates to that discipline with its defaults, then Wikipedia content loads
5. User clicks Refresh → 3 new questions are pulled from the pool and fetched
6. User clicks a question link → opens Wikipedia article in new tab

---

## Technical Notes

- **Single file**: All HTML, CSS, JS inline. No frameworks. Vanilla JS.
- **Fonts**: Load from Google Fonts via `<link>` tag in `<head>`.
- **Responsive**: Two-panel on desktop (>768px), stacked on mobile.
- **Animations**: Use CSS transitions for hover states, selection changes, and content swaps. Keep it subtle and fast.
- **No localStorage**: No state persistence needed.
- **Browser support**: Modern browsers (Chrome, Firefox, Safari, Edge). ES6+ is fine.
