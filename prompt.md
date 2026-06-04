## Build Prompt

Build **Vibe Design**, an AI design inspiration generator. Users pick a category, type a concept, and get a grid of AI-generated design mockups they can favorite and refine into a single composed result.

### Stack
- React + Vite + TypeScript
- Tailwind CSS + shadcn/ui (semantic design tokens in `index.css` and `tailwind.config.ts`, HSL only — no hardcoded colors in components)
- Lovable Cloud (Supabase) for edge functions, storage, and secrets
- Image generation via **Prodia API** (secret: `PRODIA_API_KEY`)

### Categories
`Product Placement`,`Website`

### Core Features

1. **Header** with app title and a Settings (gear) icon that opens the Prompt Settings modal.

2. **Search bar** with:
   - Category dropdown
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

### Edge Functions

**`generate-image`** — accepts `{ prompt, seed, category, imageUrl? }`:
- For all categories → Prodia `inference.flux-2.klein-4b.txt2img.v1`, 512x512, 4 steps
- Returns `{ imageUrl }`

**`refine-image`** — accepts selected image URLs + prompt, composes them into a single image, returns `{ imageUrl }`.

### Storage
Create a public bucket `product-images` for Product Placement uploads (Prodia img2img requires a public URL). Add RLS policies on `storage.objects` allowing public read and anon/authenticated insert for that bucket.

### Default Prompt Settings (seed values)

**Product Placement**
> Versatile product placement image featuring the product naturally incorporated into a believable real-world setting. Vary the scene, environment, lighting, camera angle, background, and lifestyle context to suit the product type. Keep the product visually prominent, clearly identifiable, and accurately represented while making the overall image feel authentic, polished, and commercially appealing.

**Website**
> Modern, professional website design with clean typography, generous whitespace, and a clear visual hierarchy. Focus on conversion and trust.

### UI/UX Notes
- Clean modern aesthetic, semantic tokens only, dark-mode-ready
- Sticky selection panel at the bottom when items are selected
- Back-to-top button after first scroll
- Mobile-first responsive layout
- Loading skeletons + animated dots while generating

### Secrets
- `PRODIA_API_KEY` (required) — prompt the user to add it on first run.

---

Build the full app end-to-end with the above behavior, defaults, and styling.
