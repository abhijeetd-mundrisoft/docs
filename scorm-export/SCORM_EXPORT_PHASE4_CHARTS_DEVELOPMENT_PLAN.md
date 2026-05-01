# Phase 4 - Charts Development Plan (SCORM Export)

**Scope:** `BAR_CHART`, `TREND_CHAT`, `DISTRIBUTION_CHAT` for course SCORM export (`SCORM_1_2`, `SCORM_2004`).

This plan is aligned with:
- `docs/scorm-export/SCORM_EXPORT_REQUIREMENT_ANALYSIS.md`
- `docs/scorm-export/SCORM_EXPORT_SOLUTION_DESIGN.md`
- `docs/scorm-export/SCORM_EXPORT_PHASE4_DEVELOPMENT_PLAN.md`

And references frontend behavior from:
- `course-forge-frontend/src/features/editor/components/blocks/preview/chart/ChartPreview.tsx`
- `course-forge-frontend/src/features/editor/components/blocks/editor/chart/ChartEditor.tsx`
- `course-forge-frontend/src/features/editor/components/blocks/chart/chartUtils.ts`

---

## 1) Goal

Render chart blocks in exported SCORM lessons with consistent learner-visible output, without depending on React/Recharts runtime inside LMS packages.

Key outcomes:
- No `Unsupported block type` for the 3 chart blocks.
- Deterministic offline rendering in LMS.
- Backward-compatible handling of legacy chart payload keys.

---

## 2) Frontend Findings (what we should mirror)

Current frontend preview uses `recharts` and supports:
- Chart title: `chartTitle` (legacy fallback `title`)
- Axis labels:
  - x-axis: `xAxisLabel` (legacy fallback `itemsLabel`)
  - y-axis: `yAxisLabel` (legacy fallback `valuesLabel`)
- Items array with fallback aliases:
  - `title` (fallback `label` or `name`)
  - `value`
  - `color` (optional)
  - `orderIndex`
- Sorting by `orderIndex`.
- Block type mapping:
  - `BAR_CHART` -> bar chart
  - `TREND_CHAT` -> line/trend chart
  - `DISTRIBUTION_CHAT` -> pie/distribution chart

Important constraints from UI utils:
- max items ~13
- title length cap (~30)
- numeric `value` required

---

## 3) SCORM Rendering Strategy Decision

## Recommended approach: **Pure HTML + SVG renderer (no chart JS dependency)**

Why:
- SCORM packages should remain lightweight and deterministic.
- Avoid introducing chart libraries into LMS runtime.
- No external script dependencies; low compatibility risk.
- Easy to test with plain string/snapshot assertions.

Fallback approach (if product needs richer animation later):
- Add tiny namespaced JS for progressive enhancement only, but keep static SVG as baseline.

---

## 4) Canonical content shape (SCORM parser)

Renderer should parse map content with tolerant fallbacks:

- `chartTitle` | `title`
- `xAxisLabel` | `itemsLabel`
- `yAxisLabel` | `valuesLabel`
- `items` array:
  - `id` (optional)
  - `title` | `label` | `name`
  - `value` (number)
  - `color` (optional hex/rgb/hsl string)
  - `orderIndex` (optional)

Validation/filtering at render-time:
- Keep only items with non-empty title and finite numeric value.
- Sort by `orderIndex` ascending (fallback array order).
- Clamp item count to safe max (same as UI cap).
- For distribution chart: only positive values contribute to slices.

---

## 5) Block-specific rendering behavior

### `BAR_CHART`
- Render title + axis labels + SVG bars.
- Compute relative heights from max value.
- Show item labels (truncated if needed).
- Show value labels over/near bars.

### `TREND_CHAT`
- Render title + axis labels + SVG polyline path.
- Plot points by item order.
- Draw dot markers and optional value labels.

### `DISTRIBUTION_CHAT`
- Render title + SVG pie chart.
- Render legend list with color chip + label + value.
- Empty-state message when no positive values.

---

## 6) Accessibility and UX expectations

- Include semantic wrapper classes:
  - `.lesson-chart`, `.lesson-chart-title`, `.lesson-chart-empty`
- Use `<figure>` + `<figcaption>` for chart region.
- Provide readable fallback text when no valid data.
- Respect reduced-motion (no animation in v1).

---

## 7) Implementation plan (backend + templates)

1. Add renderer classes under:
   - `.../export/render/blocks/BarChartScormRenderer.java`
   - `.../export/render/blocks/TrendChartScormRenderer.java`
   - `.../export/render/blocks/DistributionChartScormRenderer.java`
2. Reuse `BlockRendererSupport` for parsing + string extraction.
3. Add shared chart math helper if needed (internal static utility).
4. Add mirrored CSS in:
   - `src/main/resources/export-templates/scorm12/css/blocks.css`
   - `src/main/resources/export-templates/scorm2004/css/blocks.css`
5. No JS needed for v1 charts.
6. Ensure dispatcher picks renderers via `@Component`.

---

## 8) Media extraction impact

Current chart payload is data-only (no fileId media expected).
- No new extraction branch required initially.
- If future payload adds image/icon refs, extend extraction in both strategies together.

---

## 9) Test plan

### Unit tests (renderer)
- valid payload renders expected SVG container and labels.
- legacy keys (`title/itemsLabel/valuesLabel`) are respected.
- invalid/missing values are filtered.
- item ordering by `orderIndex`.
- distribution chart handles zero/negative-only data with empty state.

### Regression tests
- Update `JourneyBlockRenderersTest` with 3 new chart cases.
- Confirm existing renderer tests remain green.

### Integration smoke
- Export fixture lesson containing all 3 chart blocks.
- Verify SCORM 1.2 and 2004 output equivalence.

---

## 10) Acceptance criteria

- All 3 chart block types render without unsupported fallback.
- Equivalent output for SCORM 1.2 and 2004.
- Legacy and canonical chart payload variants render safely.
- No added runtime dependency on Recharts/React in exported package.
- Automated chart renderer tests pass.

---

## 11) Risks and mitigations

| Risk | Mitigation |
|------|------------|
| Visual mismatch vs frontend Recharts | Define v1 export style baseline (not pixel-perfect parity). |
| Overly long labels breaking layout | Truncate + wrap rules in CSS and renderer text constraints. |
| Negative/edge numeric values | Explicit normalization rules per chart type. |
| SCORM 12/2004 style drift | Mirror CSS edits in same change set and test both exports. |

---

## 12) Suggested delivery slices

1. Slice A: `BAR_CHART` renderer + css + tests
2. Slice B: `TREND_CHAT` renderer + tests
3. Slice C: `DISTRIBUTION_CHAT` renderer + legend/empty-state tests
4. Slice D: dual-SCORM manual smoke + docs/tickets update

---

## 13) Document control

| Version | Date | Notes |
|---------|------|-------|
| 1.0 | 2026-04-30 | Initial SCORM chart plan based on frontend Recharts behavior and Phase 4 constraints. |
