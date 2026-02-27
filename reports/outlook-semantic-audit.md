# Semantic/AEO-GEO Audit — Outlook India Article

URL audited:
`https://www.outlookindia.com/national/students-long-march-over-vcs-alleged-casteist-remarks-raf-deployed-outside-jnu`

## Key findings (what is wrong)

### 1) Summary block naming is weak for machine interpretation
The bullet summary appears in this structure:

- Wrapper: `.sb-summary.ole-art-smry`
- Content container: `.content.smry-cnt`
- List: `<ul><li><p>...</p></li></ul>`

What is wrong:
- Class names like `sb-summary`, `ole-art-smry`, and especially `content smry-cnt` are internal shorthand, not semantically descriptive.
- The summary is implemented as generic `<div>` blocks and does not have an explicit semantic role (`<section>`, `aria-labelledby`, or microdata marker).
- The summary is also included inside `#articleBody.sb-article` as a normal `.sb-element`, which can blur where summary ends and body begins for extraction systems.

Why it matters for AEO/GEO:
- LLM retrieval and search extractors look for strong semantic hints for “key points” / “takeaways.”
- Generic class names reduce confidence and increase the chance that summary signals are not prioritized.

### 2) Heading hierarchy noise (ads + modules)
Measured heading counts include `h1=1`, `h2=1`, `h3=54`.

What is wrong:
- Many `h3` headings are unrelated modules or repeated “Advertisement” labels.
- This dilutes topical heading structure of the main news narrative.

Why it matters:
- AI/search systems may infer lower document focus quality when heading graph is noisy.

### 3) Article semantics are fragmented
What is wrong:
- Multiple `article` elements exist on the page, most used for cards/recommendations.
- Main story content is primarily under `#articleBody.sb-article` with repeated `.sb-element` divs rather than a single clean `<article>` containing all primary narrative sections.

Why it matters:
- Extractors can mix primary and secondary content, reducing citation precision.

### 4) OpenGraph type mismatch
What is wrong:
- `og:type` is `website` on a news article URL.

Why it matters:
- `og:type=article` better aligns social/search parsers with editorial content type.

### 5) Structured data quality gaps
`NewsArticle` JSON-LD exists and is broadly good (headline/author/publisher/image/datePublished/dateModified), but:
- `dateCreated` is empty.
- Optional quality enrichers like `wordCount` are missing.

Why it matters:
- Missing/empty fields reduce structured confidence signals.

### 6) Link and module clutter around story
What is wrong:
- High link count and many non-editorial modules around the story area.
- Empty-text links are present.

Why it matters:
- Can reduce signal-to-noise for indexing and summarization pipelines.

## Specific remediation for your summary section (most important)

Use explicit semantic structure for key points:

```html
<article>
  <header>
    <h1>...</h1>
    <p class="dek">...</p>
  </header>

  <section id="key-takeaways" aria-labelledby="key-takeaways-title">
    <h2 id="key-takeaways-title">Key takeaways</h2>
    <ul>
      <li>...</li>
      <li>...</li>
      <li>...</li>
    </ul>
  </section>

  <section id="story-body" aria-labelledby="story-body-title">
    <h2 id="story-body-title">Story</h2>
    <p>...</p>
  </section>
</article>
```

And if class names are needed for styling, make them human-semantic:
- `article-summary`
- `key-takeaways`
- `story-body`

instead of shorthand like `smry-cnt`.

## Priority fixes (ordered)
1. Convert summary block to semantic `<section>` + labeled heading (`Key takeaways`).
2. Ensure summary is not duplicated ambiguously as just another generic `.sb-element` without role separation.
3. Limit heading tags in ad/sidebar modules (use non-heading elements for ad labels).
4. Use one clearly dominant `<article>` for the main story body.
5. Set `og:type=article` on article pages.
6. Fill `dateCreated` and add `wordCount` in `NewsArticle` JSON-LD.
