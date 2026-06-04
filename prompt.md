## Build Prompt

Build **Vibe Design**, an AI design inspiration generator. Users pick a category, type a concept, and get a grid of AI-generated design mockups they can favorite and refine into a single composed result.

### Stack
- React + Vite + TypeScript
- Lovable Cloud (Supabase) for edge functions, storage, and secrets
- cheap LLM on Lovable for prompt variation
- Image generation via **Prodia API** (secret: `PRODIA_API_KEY`)

### Categories
`Product Placement`,`Website`

### Core Features

1. **Header** with app title and a Settings (gear) icon that opens the Prompt Settings modal.

2. **Search bar** with:
   - Category dropdown (starting with product placement)
   - Concept text input
   - Generate button

3. **Prompt variation engine** (`src/lib/promptVariations.ts`):
   - 30 style modifiers per category
   - 8 lighting modifiers 
   - 5 quality suffixes 
   - `buildPrompt(userConcept, category, customSystemPrompt?)` randomizes one style + lighting + quality, prefixes the optional custom system prompt, and puts the user's concept first.

4. **Image grid**:
   - Generates a batch of ~12 images, infinite scroll / "Load more"
   - Per-image retry on failure
   - Logo category renders square (`aspect-square`); others landscape (1024x640)

5. **Selection + Refine**:
   - Tap an image to select (up to N, numbered selection badges)
   - Sticky bottom selection panel with Refine button + model picker
   - Refine calls a separate edge function that composes selections into one image
   - Result shown in a modal with Retry / Start Over

6. **Prompt Settings modal**:
   - One textarea per category for a custom system-prompt prefix
   - Persisted in `localStorage` under `vibe-design-prompt-settings`
   - Seed with the default prompts below (users can edit/reset)

### Product Placement Workflow

Product Placement prompts differently from other categories — all prompts describe **only the empty scene**, never the product itself.

1. **Prompt construction** (client-side, no LLM): `buildPrompt` assembles `{customSystemPrompt}. {userConcept} empty scene photograph with negative space for product placement. {style}. {lighting}. {quality}.`
2. **Style modifiers**: All 30 product-specific modifiers describe only surfaces, surroundings, and negative space (e.g. "Bare weathered wood surface bathed in warm golden hour lighting", "Polished concrete surface with dramatic side lighting and empty foreground") — never the product itself.
3. **Why empty-scene phrasing matters**: The generated image will later be used for product compositing. If modifiers described a product, the model would render a hallucinated product in the scene. By describing only the environment, the output is a clean backdrop ready for compositing.

### Edge Functions

**`generate-image`** — accepts `{ prompt, seed, category }`:
- For all categories → Prodia `inference.flux-2.klein.4b.txt2img.v1`, 512×512, 4 steps

### Default Prompt Settings

**Product Placement**
> "{userConcept} empty scene photograph with negative space for product placement. {style}. {lighting}. {quality}."

{style}: one of 30 empty-scene modifiers (e.g. "Bare weathered wood surface bathed in warm golden hour lighting")
{lighting}: one of 8 shared lighting lines
{quality}: one of 5 standard quality suffixes (e.g. "Professional web design render, high quality")

**Website**
> Modern, professional website design with clean typography, generous whitespace, and a clear visual hierarchy. Focus on conversion and trust.

### UI/UX Notes
- Clean modern aesthetic, semantic tokens only, dark-mode-ready
- Sticky selection panel at the bottom when items are selected
- Back-to-top button after first scroll
- Mobile-first responsive layout
- Loading skeletons + animated dots while generating

### Secrets
- `PRODIA_API_KEY` (required) — prompt the user to add it on first run. (do not forget to set up auto retry for 429s)

---

Build the full app end-to-end with the above behavior, defaults, and styling.
