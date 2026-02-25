# Polaris Structural Patterns for WeWeb Components

Composition recipes for building WeWeb custom components with `polaris-weweb-styles`. These patterns complement the npm package's components and tokens with proven structural layouts.

## Package Setup

```json
{
  "dependencies": {
    "polaris-weweb-styles": "^2.1.0"
  }
}
```

Every component's **root element** must apply `@include polaris-tokens`. Without this, all Polaris components render with missing styles.

```vue
<style lang="scss" scoped>
@import 'polaris-weweb-styles';
.root {
  @include polaris-tokens;
  font-family: var(--p-font-family-sans);
}
</style>
```

---

## Pattern 1: List + Detail View Shell

The standard two-view pattern for CRUD components (list of items → click to edit).

### Structure

```
┌─────────────────────────────────────┐
│ List View (transparent bg)          │
│  ┌─────────────────────────────────┐│
│  │ Header: Title + Create Button   ││
│  ├─────────────────────────────────┤│
│  │ Bordered card table             ││
│  │  th │ th │ th                   ││
│  │  td │ td │ td                   ││
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│ Editor Header (white, shadow)       │
│  ← Back  Title  [Badge] [Cancel][Save]
├─────────────────────────────────────┤
│ Tab Bar (white, bottom border)      │
│  ⚙️ Config  ⚡ Actions  📊 Outcomes │
├─────────────────────────────────────┤
│ Content Area (gray bg)              │
│  ┌─ Card ──────────────────────┐   │
│  │ CardHeader: title + desc    │   │
│  │ CardSection: form fields    │   │
│  └─────────────────────────────┘   │
│  ┌─ Card ──────────────────────┐   │
│  │ ...more sections...         │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

### Template

```vue
<template>
  <div class="my-component">
    <div v-if="currentView === 'list'" class="list-view">
      <slot name="list" />
    </div>
    <div v-else class="editor-view">
      <div class="editor-header">
        <PolarisInline gap="300" blockAlign="center">
          <PolarisButton variant="plain" icon="arrowLeft" @click="goBack">Back</PolarisButton>
          <PolarisText variant="headingLg">{{ title }}</PolarisText>
          <PolarisBadge v-if="isDirty" variant="attention">Unsaved</PolarisBadge>
        </PolarisInline>
        <PolarisInline gap="200">
          <PolarisButton @click="goBack">Cancel</PolarisButton>
          <PolarisButton variant="primary" :loading="isSaving" @click="save">Save</PolarisButton>
        </PolarisInline>
      </div>
      <div class="editor-tabs">
        <button
          v-for="tab in tabs" :key="tab.id"
          class="editor-tab"
          :class="{ 'editor-tab--active': activeTab === tab.id }"
          @click="activeTab = tab.id"
        >
          <span class="editor-tab__icon">{{ tab.icon }}</span>
          <span class="editor-tab__label">{{ tab.label }}</span>
        </button>
      </div>
      <div class="editor-content">
        <slot name="content" />
      </div>
    </div>
  </div>
</template>
```

### Styles

```scss
.list-view {
  flex: 1;
  padding: var(--p-space-600) var(--p-space-500);
  overflow-y: auto;
  background: transparent;
}

.editor-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--p-space-300) var(--p-space-600);
  background: var(--p-color-bg-surface);
  border-bottom: 1px solid var(--p-color-border);
  box-shadow: 0 1px 0 0 rgba(0, 0, 0, 0.04);
  flex-shrink: 0;
}

.editor-tabs {
  display: flex;
  background: var(--p-color-bg-surface);
  border-bottom: 1px solid var(--p-color-border);
  padding: 0 var(--p-space-600);
  flex-shrink: 0;
  overflow-x: auto;
}

.editor-tab {
  display: flex;
  align-items: center;
  gap: var(--p-space-200);
  padding: var(--p-space-300) var(--p-space-400);
  border: none;
  background: transparent;
  cursor: pointer;
  font-size: var(--p-font-size-325);
  font-weight: var(--p-font-weight-medium);
  color: var(--p-color-text-secondary);
  border-bottom: 2px solid transparent;
  transition: all var(--p-motion-duration-150) var(--p-motion-ease);
  white-space: nowrap;
  font-family: var(--p-font-family-sans);

  &:hover {
    color: var(--p-color-text);
    background: var(--p-color-bg-surface-hover);
  }

  &--active {
    color: var(--p-color-text);
    border-bottom-color: var(--p-color-text);
  }
}

.editor-content {
  flex: 1;
  overflow-y: auto;
  padding: var(--p-space-600);
  background: var(--p-color-bg-surface-secondary);
  display: flex;
  flex-direction: column;
  gap: var(--p-space-400);
}
```

---

## Pattern 2: Card-Sectioned Forms

Group related form fields into separate cards. White cards on `bg-surface-secondary` create visual separation.

### Non-collapsible section (always visible)

Use `PolarisCard` + `PolarisCardHeader` + `PolarisCardSection`:

```vue
<PolarisCard>
  <PolarisCardHeader
    title="Action Configuration"
    description="Define the action type and its parameters"
  />
  <PolarisCardSection>
    <PolarisBlockStack gap="400">
      <PolarisTextField label="Name" required ... />
      <PolarisSelect label="Type" ... />
    </PolarisBlockStack>
  </PolarisCardSection>
</PolarisCard>
```

### Collapsible section

Wrap `PolarisConfigSection` inside `PolarisCard`:

```vue
<PolarisCard>
  <PolarisConfigSection
    icon="🛡️"
    title="Guardrails"
    subtitle="Limits specific to this action"
    collapsible
    :defaultOpen="true"
  >
    <PolarisBlockStack gap="400">
      <!-- fields -->
    </PolarisBlockStack>
  </PolarisConfigSection>
</PolarisCard>
```

Without the `PolarisCard` wrapper, `PolarisConfigSection` renders as a bare collapsible with no border/background.

### Multiple cards in a tab

```vue
<PolarisBlockStack gap="400">
  <PolarisCard><!-- Name & Description --></PolarisCard>
  <PolarisCard><!-- Objective & Tone --></PolarisCard>
  <PolarisCard><!-- Context & Settings --></PolarisCard>
</PolarisBlockStack>
```

---

## Pattern 3: List Table in Bordered Card

Tables should be wrapped in a bordered rounded card.

```scss
.table-wrapper {
  overflow-x: auto;
  background: var(--p-color-bg-surface);
  border: 1px solid var(--p-color-border);
  border-radius: var(--p-border-radius-300);
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: var(--p-font-size-325);

  th {
    text-align: left;
    padding: var(--p-space-300) var(--p-space-400);
    font-weight: var(--p-font-weight-semibold);
    font-size: var(--p-font-size-275);
    color: var(--p-color-text-secondary);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    border-bottom: 1px solid var(--p-color-border);
    background: var(--p-color-bg-surface-secondary);
  }

  .th--first { border-top-left-radius: var(--p-border-radius-300); }
  .th--last { border-top-right-radius: var(--p-border-radius-300); }

  td {
    padding: var(--p-space-300) var(--p-space-400);
    border-bottom: 1px solid var(--p-color-border);
    color: var(--p-color-text);
    vertical-align: middle;
  }

  &__row {
    cursor: pointer;
    transition: background var(--p-motion-duration-150) var(--p-motion-ease);
    &:hover { background: var(--p-color-bg-surface-hover); }
    &:last-child td { border-bottom: none; }
  }
}
```

Loading and empty states should also be wrapped in the same bordered card for visual consistency.

---

## Pattern 4: Inline Sub-Page

When clicking "Add" or "Edit" opens a form inside the same tab, use a negative-margin pattern to break out of the parent padding.

```scss
.sub-page {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-400);
  margin: calc(-1 * var(--p-space-600));
  min-height: 100%;

  &__header {
    padding: var(--p-space-300) var(--p-space-600);
    border-bottom: 1px solid var(--p-color-border);
    box-shadow: 0 1px 0 0 rgba(0, 0, 0, 0.04);
    background: var(--p-color-bg-surface);
    position: sticky;
    top: 0;
    z-index: 5;
  }

  &__body {
    padding: 0 var(--p-space-600) var(--p-space-600);
  }
}
```

The header matches the main editor header style. Body padding matches the editor content padding (`space-600`) for alignment.

---

## Pattern 5: Condition Builder

A reusable pattern for filter/condition UIs (eligibility rules, event filters, audience conditions).

### Structure

```
Group 1                                    [AND/OR] ×
  Collection: [Select dropdown ▾]
  ┌──────────┬──────────┬──────────┐
  │ Field ▾  │ Oper ▾   │ Value    │ ×
  └──────────┴──────────┴──────────┘
  + Add Condition

[ + Add Group ]
```

### Components used

- Each group = `PolarisCard subdued` → `PolarisCardSection`
- Group header = `PolarisInline` with `PolarisText` + `PolarisButtonGroup segmented` (AND/OR) + close button
- Collection = `PolarisSelect`
- Each condition row = `PolarisSelect` (field) + `PolarisSelect` (operator) + `PolarisTextField` (value) + close button in a flex row
- Inner add = `PolarisButton variant="plain" fullWidth`
- Outer add = `PolarisButton variant="outline" fullWidth`

### Inline field normalization

```scss
.condition-fields {
  display: flex;
  align-items: flex-end;
  gap: var(--p-space-200);

  :deep(select),
  :deep(input:not([type="checkbox"]):not([type="radio"])) {
    height: 36px;
    box-sizing: border-box;
  }
}
```

---

## Pattern 6: Item Cards with Actions

### Card with header + action slot

```vue
<PolarisCard>
  <PolarisCardHeader :title="`Outcome ${idx + 1}`">
    <template #action>
      <PolarisButton variant="plain" icon="delete" iconOnly @click="remove(idx)" />
    </template>
  </PolarisCardHeader>
  <PolarisCardSection>
    <PolarisBlockStack gap="400">
      <!-- fields -->
    </PolarisBlockStack>
  </PolarisCardSection>
</PolarisCard>
```

### Row-style item (lighter weight)

For denser lists where full cards are too heavy:

```scss
.list-item {
  display: flex;
  align-items: center;
  gap: var(--p-space-300);
  padding: var(--p-space-300) var(--p-space-400);
  background: var(--p-color-bg-surface);
  border: var(--p-border-width-025) solid var(--p-color-border);
  border-radius: var(--p-border-radius-200);
  transition: border-color var(--p-motion-duration-150) var(--p-motion-ease);

  &:hover { border-color: var(--p-color-border-hover); }

  &__name {
    font-size: var(--p-font-size-325);
    font-weight: var(--p-font-weight-semibold);
    color: var(--p-color-text);
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  &__subtitle {
    font-size: var(--p-font-size-275);
    color: var(--p-color-text-secondary);
  }
}
```

---

## Pattern 7: Toggle Switch

Not provided by the package. Custom CSS toggle for status columns and enable/disable controls.

```scss
.toggle-switch {
  position: relative;
  display: inline-block;
  width: 36px;
  height: 20px;
  cursor: pointer;

  input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  &__slider {
    position: absolute;
    inset: 0;
    background: var(--p-color-bg-fill-disabled);
    border-radius: 10px;
    transition: background var(--p-motion-duration-150) var(--p-motion-ease);

    &::before {
      content: '';
      position: absolute;
      left: 2px;
      top: 2px;
      width: 16px;
      height: 16px;
      background: white;
      border-radius: 50%;
      transition: transform var(--p-motion-duration-150) var(--p-motion-ease);
      box-shadow: 0 1px 2px rgba(0, 0, 0, 0.15);
    }
  }

  input:checked + &__slider {
    background: var(--p-color-bg-fill-success);
    &::before { transform: translateX(16px); }
  }
}
```

---

## Pattern 8: Constraint Builder (Natural Language Rules)

For flexible constraint inputs that read like English: "No more than **3** **actions** **per user** **per week**".

```scss
.constraint-row {
  display: flex;
  align-items: center;
  gap: 6px;
  flex-wrap: wrap;
  padding: var(--p-space-200) var(--p-space-300);
  background: var(--p-color-bg-surface-secondary);
  border: var(--p-border-width-025) solid var(--p-color-border);
  border-radius: var(--p-border-radius-200);

  &__text {
    font-size: var(--p-font-size-300);
    color: var(--p-color-text-secondary);
    white-space: nowrap;
  }
}
```

Use `PolarisTextField labelHidden` and `PolarisSelect labelHidden` for inline controls.

---

## Pattern 9: Tag Input (Multi-Value Text)

For fields where the user enters multiple possible values.

```scss
.tag-input-wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: var(--p-space-100);
  padding: var(--p-space-100) var(--p-space-200);
  border: var(--p-border-width-025) solid var(--p-color-border);
  border-radius: var(--p-border-radius-200);
  background: var(--p-color-bg-surface);
  min-height: 36px;
  align-items: center;

  &:focus-within {
    border-color: var(--p-color-border-focus);
    outline: 1px solid var(--p-color-border-focus);
  }

  &__input {
    flex: 1;
    min-width: 100px;
    border: none;
    outline: none;
    background: transparent;
    font-size: var(--p-font-size-325);
    color: var(--p-color-text);
    font-family: var(--p-font-family-sans);
  }
}

.tag-chip {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  background: var(--p-color-bg-fill-info-secondary);
  color: var(--p-color-text-info);
  border-radius: var(--p-border-radius-full);
  font-size: var(--p-font-size-275);
  white-space: nowrap;

  &__remove {
    width: 14px;
    height: 14px;
    border: none;
    background: transparent;
    cursor: pointer;
    color: inherit;
    border-radius: 50%;
    &:hover { background: rgba(0, 0, 0, 0.1); }
  }
}
```

---

## Pattern 10: Visual Rating Picker

For classification or quality scales (e.g., best → worst).

Option shape: `{ value: 'best', label: 'Best', bars: 3, color: 'green' }`

Use `var(--p-color-bg-fill-success)` for green bars, `var(--p-color-bg-fill-critical)` for red. Selected state uses `var(--p-color-border-focus)` + `var(--p-color-bg-surface-selected)`.

---

## Component Choice Guide

| Need | Use |
|---|---|
| Always-visible card section | `PolarisCard` + `PolarisCardHeader` + `PolarisCardSection` |
| Collapsible card section | `PolarisCard` > `PolarisConfigSection collapsible` |
| Rich toggle with description | `PolarisCheckboxCard` |
| Simple checkbox | `PolarisCheckbox` |
| Status indicator | `PolarisBadge` |
| Error/info message | `PolarisBanner` |
| Empty placeholder | `PolarisEmptyState` |
| Horizontal layout | `PolarisInline gap="200"` |
| Vertical layout | `PolarisBlockStack gap="400"` |
| Section heading | `PolarisText variant="headingMd"` |
| Helper text | `PolarisText variant="bodySm" color="subdued"` |
| Enable/disable toggle | Custom `.toggle-switch` (Pattern 7) |
| Multi-value input | Custom `.tag-input-wrapper` (Pattern 9) |
| Natural language rule | Custom `.constraint-row` (Pattern 8) |
| Filter conditions | Condition Builder composition (Pattern 5) |

## Rules

- **No hardcoded colors or pixel values** — always use `var(--p-color-*)`, `var(--p-space-*)`, etc.
- **No global CSS** — all styles must be `<style lang="scss" scoped>`
- **Scoped styles only** — `.vue` files are raw SFCs compiled by `@weweb/cli`
- **Use `?.` optional chaining** for all prop access (WeWeb requirement)
- **Data binding** — always use `:modelValue` + `@update:modelValue` instead of `v-model`
- **`accent-color`** — set `accent-color: var(--p-color-bg-fill-brand, #2C6ECB)` on native radio/checkbox inputs
