---
name: "google-maps-review-writing"
description: "Use when writing or revising Google Maps reviews, especially Japanese 口コミ that should be specific, fact-checked, and 300+ characters."
---

# Google Maps Review Writing

## Workflow

Do not ask the user for basic review inputs when the current Google Maps review page is available. Once this skill is invoked, proceed autonomously: inspect the page, gather missing context from the web, draft, verify character count, and fill the review field. Ask the user only when a true blocker remains, such as the location cannot be identified, the page is unavailable, the requested rating conflicts with the selected rating and cannot be resolved, or the user explicitly asks to choose between alternatives.

1. Inspect the current Google Maps review dialog/page with `snapshot()` and identify the exact location being reviewed.
2. Confirm the location precisely, especially when there may be multiple branches or similarly named places:
   - venue name
   - branch/store name
   - address or neighborhood
   - facility/building name and floor when relevant
3. Read the current form state before drafting:
   - overall rating and sub-ratings
   - selected visit/service details such as dine-in/takeout, meal type, price range, wait time, and group size
   - any existing typed text
   - attached photos or visible photo thumbnails when they can add concrete, reliable details
4. Gather supporting information from the web before writing. Use this priority order when available:
   1. venue official site
   2. official facility/building page
   3. official social accounts
   4. Google Maps business profile
   5. official reservation, booking, or commerce pages
   6. third-party review/listing sites such as Tabelog, Hot Pepper, blogs, and local media
5. Use third-party sources only to supplement, not as the main basis for facts. Check concrete claims such as location, access, branch relationship, floor, signature menu, features, and opening context.
6. Draft the review using only information supported by the page, user-provided context, attached photos, or reliable web sources.
7. Verify the draft is at least 300 Japanese characters using `.length` before entering it.
8. Enter the text into the review field, but do not click the post/submit button unless the user explicitly asks.

## Autonomy Rules

- Do not ask for store name, branch name, star rating, desired points, or whether 300+ characters are needed when those can be read from the current page, user message, attached files, or the skill defaults.
- Default to 300+ Japanese characters for Google Maps review-writing requests unless the user explicitly asks for a shorter draft.
- If desired points are not provided, infer useful angles from the venue type, selected form fields, attached photos, and verified web facts.
- If official facts cannot be found quickly, use page-visible facts and safe, non-specific customer impressions rather than stopping.
- Continue to completion by filling the review field. Report blockers only after trying page inspection and web lookup.

## Review Content Requirements

- Must be 300 characters or more when the user asks for a 300+ character review.
- Match the tone to the selected rating:
  - 5 stars: clearly positive and satisfied.
  - 4 stars: positive overall, but slightly restrained or with a mild caveat.
  - 3 stars or lower: avoid overpraising; reflect the rating honestly.
- Avoid generic praise that could apply to any venue.
- Include venue-specific facts where possible, such as:
  - official branch/store relationship
  - exact facility or neighborhood
  - access details
  - notable menu, coffee beans, cooking style, equipment, seat count, or concept
  - details visible in attached photos, only when reasonably inferable
- Avoid making the review sound like an official brochure or SEO article:
  - keep verified facts to a natural amount, usually 2 to 4 concrete facts
  - blend facts into first-person impressions instead of listing them
  - prioritize what a future visitor would actually find useful
- Prefer natural first-person Japanese that sounds like an actual customer review.
- Do not overstate uncertain details. If a detail is inferred rather than verified, either omit it or soften it.
- Remove or revise claims that fail fact-checking.

## Fact-checking Rules

- Official sources outrank blogs, review sites, and snippets.
- Use exact wording for access/location when official wording is specific. For example, prefer "横浜駅みなみ東口通路直通" over the broader "横浜駅直通" when that is what the official source says.
- Do not invent visited dishes, staff interactions, seating, Wi-Fi, outlets, waiting conditions, or weather unless they are present in the form, photos, user message, or reliable source.
- If using attached photos, describe only what is clearly visible and relevant.
- If a photo shows food but the dish name is unclear, avoid naming it. Use safer wording such as "写真の料理" only if needed.
- Avoid writing "写真を見る限り" in the public review unless it sounds natural. Convert visible photo details into ordinary customer impressions when reliable.

## Genre-specific Cues

Use these as prompts for what to inspect and mention. Do not force every item into the review.

- Cafe or coffee shop: access, seating, crowding, coffee taste, beans/roast, sweets, atmosphere, whether it is suitable for a short break or light work.
- Izakaya or restaurant: food, drinks, staff, atmosphere, group size, price range, wait time, signature cooking style or menu.
- Beauty salon: consultation, staff, finish, atmosphere, cleanliness, ease of explaining preferences.
- Station or public facility: access, routes, nearby landmarks, transfer convenience, crowding, usability.
- Event venue or commercial facility: location, floor/building, flow lines, nearby shops, use cases before/after plans.

## Revision Rules

- If the user says the review is too generic, revise by adding verified venue-specific details: proper nouns, facility/building names, branch relationship, access wording, official features, or photo-supported details.
- If the user asks for fact-checking, inspect sources again, remove unsupported claims, and replace broad wording with source-backed precise wording.
- If a concrete claim cannot be verified quickly, omit it rather than weakening the review with uncertainty.

## Output and Confirmation

- If filling a Google Maps form, confirm the character count and that the post button was not clicked.
- If citing web-based fact checks in chat, cite sources inline using the standard citation format.
