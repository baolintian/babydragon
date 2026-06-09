# Rounded List Card Styling Design

## Context

The existing Hugo blog already matches the general Yinwang-inspired structure and typography, but the user wants each list item to gain a rounded visual frame while keeping the restrained, reading-first feel.

The user confirmed:

- The reference remains `https://www.yinwang.org/`
- The change should apply to list-style surfaces rather than the article detail page
- The effect should be subtle, not a heavy product-card layout

## Goals

- Add a rounded border treatment to article list items
- Keep the overall visual tone calm and low-contrast
- Reuse the same visual treatment on home and category detail lists
- Add a lighter rounded treatment to category index items
- Preserve archive page scanability by keeping it list-like rather than turning it into heavy cards

## Non-Goals

- No layout restructuring
- No content model changes
- No Hugo template logic changes unless strictly required
- No shadows, large motion, or bold product-style cards

## Visual Direction

- Article list items should look like lightly framed paper blocks
- Rounded corners should be noticeable but soft, around the high-teens pixel range
- Borders should stay low-contrast and close to the existing palette
- Background fill should be slightly brighter than the page, with no shadow
- Hover feedback should be minimal: small border/background change only

## Scope

- Home page article list items
- Category detail page article list items
- Category index items with a lighter version of the same pattern
- Archive page rows with only a lighter rounded treatment

## Implementation Strategy

- Update `assets/css/main.css`
- Keep current HTML structure and selectors
- Move spacing from separator borders to card spacing
- Use the current typography and color system so the new frames feel integrated

## Verification

- `hugo --gc --minify` succeeds
- Home list items render with rounded borders
- Category detail list items render with the same rounded frame
- Category index items render with lighter rounded frames
- Archive page remains readable and not over-carded
