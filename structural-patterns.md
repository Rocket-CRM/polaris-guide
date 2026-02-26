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

## Pattern 11: Node Palette (Action Picker Sidebar)

A draggable sidebar list for workflow/flow builders. Each item has a colored icon circle, title + description, and a hover chevron — modeled on the Shopify Flow trigger selector.

### Structure

```
┌─────────────────────────────────┐
│ Add action                      │
│                                 │
│ MESSAGES                        │
│ ┌ 💬 ─ LINE Message ──── › ┐  │
│ │     Send a LINE message   │  │
│ └───────────────────────────┘  │
│ ┌ 📱 ─ SMS ──────────── › ┐  │
│ │     Send an SMS text      │  │
│ └───────────────────────────┘  │
│                                 │
│ LOYALTY                         │
│ ┌ 🪙 ─ Award Currency ── › ┐  │
│ │     Give points or tickets│  │
│ └───────────────────────────┘  │
│ ...                             │
└─────────────────────────────────┘
```

### Data shape

```js
const nodeGroups = [
  {
    label: 'Messages',
    nodes: [
      { type: 'action', subType: 'send_line', label: 'LINE Message', desc: 'Send a LINE message', icon: '💬', color: '#06C755' },
      { type: 'action', subType: 'send_sms', label: 'SMS', desc: 'Send an SMS text message', icon: '📱', color: '#10B981' },
    ],
  },
  // ...more groups
];
```

### Template

```vue
<div class="node-palette">
  <div class="palette-heading">Add action</div>
  <div v-for="group in nodeGroups" :key="group.label" class="palette-group">
    <div class="palette-group__label">{{ group.label }}</div>
    <div
      v-for="item in group.nodes"
      :key="item.subType || item.type"
      class="palette-item"
      draggable="true"
      @dragstart="onDragStart($event, item)"
    >
      <span class="palette-item__icon" :style="{ '--icon-bg': item.color + '14', '--icon-color': item.color }">{{ item.icon }}</span>
      <div class="palette-item__text">
        <span class="palette-item__label">{{ item.label }}</span>
        <span v-if="item.desc" class="palette-item__desc">{{ item.desc }}</span>
      </div>
      <svg class="palette-item__chevron" width="16" height="16" viewBox="0 0 20 20" fill="currentColor"><path d="M7.293 4.293a1 1 0 0 1 1.414 0l5 5a1 1 0 0 1 0 1.414l-5 5a1 1 0 0 1-1.414-1.414L11.586 10 7.293 5.707a1 1 0 0 1 0-1.414z"/></svg>
    </div>
  </div>
</div>
```

### Styles

```scss
.node-palette {
  display: flex;
  flex-direction: column;
  padding: var(--p-space-300);
  height: 100%;
  overflow-y: auto;
  gap: var(--p-space-100);
}

.palette-heading {
  font-size: var(--p-font-size-350);
  font-weight: var(--p-font-weight-bold);
  color: var(--p-color-text);
  padding: var(--p-space-100) var(--p-space-200) var(--p-space-200);
}

.palette-group__label {
  font-size: var(--p-font-size-275);
  font-weight: var(--p-font-weight-semibold);
  color: var(--p-color-text-secondary);
  padding: var(--p-space-200) var(--p-space-200) var(--p-space-100);
  text-transform: uppercase;
  letter-spacing: 0.3px;
}

.palette-item {
  display: flex;
  align-items: center;
  gap: var(--p-space-300);
  padding: var(--p-space-200);
  cursor: grab;
  transition: background var(--p-motion-duration-150) var(--p-motion-ease);
  border-radius: var(--p-border-radius-200);
  min-height: 48px;

  &:hover {
    background: var(--p-color-bg-surface-hover);
    .palette-item__chevron { opacity: 1; }
  }
  &:active { cursor: grabbing; background: var(--p-color-bg-surface-active); }

  &__icon {
    width: 36px;
    height: 36px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 16px;
    flex-shrink: 0;
    border-radius: var(--p-border-radius-200);
    background: var(--icon-bg, var(--p-color-bg-fill-secondary));
    color: var(--icon-color, var(--p-color-text));
  }

  &__text { flex: 1; min-width: 0; display: flex; flex-direction: column; gap: 1px; }
  &__label { font-size: var(--p-font-size-300); font-weight: var(--p-font-weight-semibold); color: var(--p-color-text); }
  &__desc { font-size: var(--p-font-size-275); color: var(--p-color-text-secondary); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  &__chevron { flex-shrink: 0; color: var(--p-color-icon-secondary); opacity: 0; transition: opacity var(--p-motion-duration-150) var(--p-motion-ease); }
}
```

Tint the icon background by appending `14` (8% alpha) to the hex color via inline style: `'--icon-bg': color + '14'`.

---

## Pattern 12: Entry Mode Radio Cards

Selectable cards for choosing between 2–3 mutually exclusive modes. Each card has a radio input, icon, title, and description. Only one can be active at a time.

### Use cases

- Trigger entry type: "Audience" vs "Custom condition"
- Export format: "CSV" vs "JSON" vs "PDF"
- Schedule mode: "One-time" vs "Recurring"

### Template

```vue
<div class="entry-type-selector">
  <span class="entry-type-selector__label">Entry type</span>
  <div class="entry-type-options">
    <label
      v-for="option in options" :key="option.value"
      class="entry-type-option"
      :class="{ 'entry-type-option--active': selected === option.value }"
    >
      <input type="radio" :value="option.value" :checked="selected === option.value" @change="select(option.value)" />
      <div class="entry-type-option__content">
        <span class="entry-type-option__icon">{{ option.icon }}</span>
        <div>
          <span class="entry-type-option__title">{{ option.label }}</span>
          <span class="entry-type-option__desc">{{ option.desc }}</span>
        </div>
      </div>
    </label>
  </div>
</div>
```

### Styles

```scss
.entry-type-selector {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-200);

  &__label {
    font-size: var(--p-font-size-300);
    font-weight: var(--p-font-weight-semibold);
    color: var(--p-color-text);
  }
}

.entry-type-options {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-150);
}

.entry-type-option {
  display: flex;
  align-items: flex-start;
  gap: var(--p-space-200);
  padding: var(--p-space-300);
  border: 1px solid var(--p-color-border);
  border-radius: var(--p-border-radius-200);
  cursor: pointer;
  transition: all var(--p-motion-duration-150) var(--p-motion-ease);
  background: var(--p-color-bg-surface);

  input[type="radio"] {
    width: 16px;
    height: 16px;
    margin-top: 2px;
    cursor: pointer;
    flex-shrink: 0;
    accent-color: var(--p-color-bg-fill-brand);
  }

  &:hover { background: var(--p-color-bg-surface-hover); }
  &--active { border-color: var(--p-color-border-brand); background: var(--p-color-bg-surface-selected); }

  &__content { display: flex; align-items: flex-start; gap: var(--p-space-200); flex: 1; }
  &__icon { font-size: 18px; flex-shrink: 0; line-height: 1.2; }
  &__title { display: block; font-size: var(--p-font-size-300); font-weight: var(--p-font-weight-medium); color: var(--p-color-text); }
  &__desc { display: block; font-size: var(--p-font-size-275); color: var(--p-color-text-secondary); margin-top: 1px; }
}
```

When the selected mode reveals additional UI (a dropdown, a condition builder, etc.), render that below the radio cards in a separate `PolarisCard`.

---

## Pattern 13: Config Panel Chrome

Standard header/footer for a side panel that edits a selected item (workflow node, record, entity). The header shows a colored icon badge, type label, editable title input, and a close button. The footer has Cancel + Save buttons.

### Structure

```
┌──────────────────────────────────────┐
│ [🎯] TRIGGER                    [×] │
│      Entry Trigger                   │
├──────────────────────────────────────┤
│                                      │
│  (config content — subdued bg)       │
│  ┌─ Card ───────────────────────┐   │
│  │ ...form fields...            │   │
│  └──────────────────────────────┘   │
│                                      │
├──────────────────────────────────────┤
│              [Cancel] [Save]         │
└──────────────────────────────────────┘
```

### Styles

```scss
.config-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.config-panel__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: var(--p-space-400);
  border-bottom: var(--p-border-width-025) solid var(--p-color-border);
  flex-shrink: 0;
}

.config-panel__icon {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: var(--p-border-radius-200);
  font-size: 18px;
  flex-shrink: 0;
  // Set background per node type: e.g. --trigger: #E0E7FF, --condition: #DBEAFE
}

.config-panel__type-label {
  font-size: var(--p-font-size-275);
  color: var(--p-color-text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.config-panel__title-input {
  width: 100%;
  border: none;
  background: transparent;
  padding: 0;
  outline: none;
  font-size: var(--p-font-size-325);
  font-weight: var(--p-font-weight-semibold);
  color: var(--p-color-text);
}

.config-panel__content {
  flex: 1;
  overflow-y: auto;
  padding: var(--p-space-400);
  background: var(--p-color-bg-surface-secondary);
  display: flex;
  flex-direction: column;
  gap: var(--p-space-400);
}

.config-panel__footer {
  display: flex;
  justify-content: flex-end;
  gap: var(--p-space-200);
  padding: var(--p-space-400);
  border-top: var(--p-border-width-025) solid var(--p-color-border);
  flex-shrink: 0;
}
```

The content area uses `bg-surface-secondary` so child `PolarisCard` blocks pop visually. Use `PolarisButton` for footer actions — default variant for Cancel, primary for Save.

---

## Pattern 14: Variable Reference Panel

A collapsible panel showing available template variables (e.g., `{{user.firstname}}`). Common in messaging, action, and agent config panels. Sits outside the main card sections, typically at the bottom of the config content.

### Template

```vue
<div class="variable-ref">
  <button class="variable-ref__toggle" @click="showVars = !showVars">
    <svg width="12" height="12" viewBox="0 0 12 12" fill="currentColor">
      <path v-if="showVars" d="M2 4l4 4 4-4"/>
      <path v-else d="M4 2l4 4-4 4"/>
    </svg>
    Available variables
  </button>
  <div v-if="showVars" class="variable-ref__list">
    <div v-for="group in varGroups" :key="group.label" class="variable-ref__group">
      <span class="variable-ref__group-label">{{ group.label }}</span>
      <code v-for="v in group.vars" :key="v">{{ v }}</code>
    </div>
  </div>
</div>
```

### Styles

```scss
.variable-ref {
  border-top: 1px solid var(--p-color-border);
  padding-top: var(--p-space-300);
}

.variable-ref__toggle {
  display: flex;
  align-items: center;
  gap: var(--p-space-150);
  background: none;
  border: none;
  padding: 0;
  font-size: var(--p-font-size-275);
  font-weight: var(--p-font-weight-medium);
  color: var(--p-color-text-secondary);
  cursor: pointer;

  &:hover { color: var(--p-color-text); }
  svg { transition: transform var(--p-motion-duration-150) var(--p-motion-ease); }
}

.variable-ref__list {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-300);
  padding-top: var(--p-space-200);
}

.variable-ref__group {
  display: flex;
  flex-wrap: wrap;
  gap: var(--p-space-100);
  align-items: center;
}

.variable-ref__group-label {
  font-size: var(--p-font-size-275);
  font-weight: var(--p-font-weight-semibold);
  color: var(--p-color-text-secondary);
  width: 100%;
  margin-bottom: 2px;
}

.variable-ref code {
  background: var(--p-color-bg-surface-secondary);
  padding: 2px 6px;
  border-radius: var(--p-border-radius-100);
  font-family: var(--p-font-family-mono);
  font-size: 11px;
  color: var(--p-color-text);
  white-space: nowrap;
  cursor: pointer;

  &:hover { background: var(--p-color-bg-surface-hover); }
}
```

---

## Pattern 15: Status Badge (Toolbar)

Inline status indicator for toolbars and headers. A colored dot + label inside a pill shape.

### Template

```vue
<span class="toolbar__badge" :class="isActive ? 'toolbar__badge--live' : 'toolbar__badge--draft'">
  <span class="toolbar__badge-dot"></span>
  {{ isActive ? 'Live' : 'Draft' }}
</span>
```

### Styles

```scss
.toolbar__badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 3px 10px;
  border-radius: 20px;
  font-size: var(--p-font-size-275);
  font-weight: var(--p-font-weight-medium);

  &--draft {
    background: var(--p-color-bg-surface-secondary);
    color: var(--p-color-text-secondary);
  }
  &--live {
    background: var(--p-color-bg-fill-success-secondary);
    color: var(--p-color-text-success);
  }
}

.toolbar__badge-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;

  .toolbar__badge--draft & { background: var(--p-color-icon-secondary); }
  .toolbar__badge--live & { background: var(--p-color-icon-success); }
}
```

Extend with additional variants as needed (e.g., `--warning`, `--critical`).

---

## Pattern 16: Multi-View Navigation Shell

Extension of Pattern 1 for components with 3+ views (e.g., list / builder / detail). Uses WeWeb internal variables for state and exposed component actions for programmatic navigation.

### State management

```javascript
const { value: currentView, setValue: setCurrentView } = wwLib.wwVariable.useComponentVariable({
  uid: props.uid,
  name: 'currentView',
  type: 'string',
  defaultValue: 'list',
});

const { value: selectedItemId, setValue: setSelectedItemId } = wwLib.wwVariable.useComponentVariable({
  uid: props.uid,
  name: 'selectedItemId',
  type: 'string',
  defaultValue: '',
});
```

### Template

```vue
<template>
  <div class="component-root">
    <ListView
      v-if="currentViewValue === 'list'"
      :items="itemsData"
      @create="handleCreate"
      @view="handleViewDetail"
      @edit="handleEdit"
    />
    <BuilderView
      v-else-if="currentViewValue === 'builder'"
      :item="editingItem"
      @save="handleSave"
      @cancel="handleBackToList"
    />
    <DetailView
      v-else-if="currentViewValue === 'detail'"
      :item="selectedItemObj"
      @edit="handleEdit"
      @back="handleBackToList"
    />
  </div>
</template>
```

### Navigation helpers

```javascript
const navigateTo = (view, itemId = '') => {
  setCurrentView(view);
  if (itemId) setSelectedItemId(itemId);
  emit('trigger-event', {
    name: 'view-changed',
    event: { view, itemId },
  });
};

const handleBackToList = () => {
  editingItem.value = null;
  navigateTo('list');
};
```

### Exposed actions for WeWeb workflows

```javascript
expose({
  navigateToList: () => handleBackToList(),
  navigateToBuilder: (data) => { /* ... */ },
  navigateToDetail: (id) => { /* ... */ },
});
```

Register in `ww-config.js`:

```javascript
actions: [
  {
    name: 'navigateToList',
    label: { en: 'Navigate to List' },
    action: 'navigateToList',
  },
  {
    name: 'navigateToBuilder',
    label: { en: 'Navigate to Builder' },
    action: 'navigateToBuilder',
    args: [
      { name: 'itemData', type: 'Object', label: { en: 'Item Data (optional)' }, /* wwEditor:start */ bindable: true, /* wwEditor:end */ },
    ],
  },
  // ...
]
```

---

## Pattern 17: Inline Confirmation

Two approaches for confirming destructive actions without modal dialogs.

### Approach A: Table-row inline replacement

Replaces the action buttons in-place with confirmation text + Yes/Cancel. Best for list tables where actions happen per-row.

```vue
<td class="col-actions">
  <div v-if="confirmDeleteId === item?.id" class="confirm-inline">
    <PolarisText variant="bodySm" color="critical">
      Delete this item? This cannot be undone.
    </PolarisText>
    <PolarisButton variant="critical" size="slim" @click="confirmDelete(item)">
      Yes, delete
    </PolarisButton>
    <PolarisButton size="slim" @click="confirmDeleteId = null">Cancel</PolarisButton>
  </div>
  <div v-else class="action-buttons">
    <PolarisButton variant="plain" size="slim" @click="$emit('edit', item)">Edit</PolarisButton>
    <PolarisButton variant="plain" size="slim" @click="confirmDeleteId = item?.id">Delete</PolarisButton>
  </div>
</td>
```

### Styles

```scss
.confirm-inline {
  display: flex;
  align-items: center;
  gap: var(--p-space-200);
}

.action-buttons {
  display: flex;
  gap: var(--p-space-100);
}
```

### Approach B: Banner-based confirmation

Uses `PolarisBanner` below the action buttons. Best for detail views where there's more space.

```vue
<PolarisBlockStack gap="300">
  <PolarisInline gap="200">
    <PolarisButton
      v-if="!showDeleteConfirm"
      variant="critical"
      @click="showDeleteConfirm = true"
    >
      Delete
    </PolarisButton>
  </PolarisInline>

  <PolarisBanner
    v-if="showDeleteConfirm"
    variant="critical"
    dismissible
    @dismiss="showDeleteConfirm = false"
  >
    Are you sure? This cannot be undone.
    <template #actions>
      <PolarisButton variant="critical" size="slim" @click="handleDelete">
        Yes, delete
      </PolarisButton>
    </template>
  </PolarisBanner>
</PolarisBlockStack>
```

### Guard pattern: block deletion of active items

```vue
<PolarisBanner
  v-if="showDeleteConfirm && item?.is_active"
  variant="critical"
  dismissible
  @dismiss="showDeleteConfirm = false"
>
  Deactivate the item before deleting.
</PolarisBanner>
```

---

## Pattern 18: AND/OR Group Connector

Visual separator showing the logical operator between condition groups or rule blocks. Extends Pattern 5.

### Template

```vue
<template v-for="(group, idx) in groups" :key="group.id">
  <div v-if="idx > 0" class="group-connector">
    <div class="group-connector__line"></div>
    <span class="group-connector__badge">AND</span>
    <div class="group-connector__line"></div>
  </div>
  <ConditionGroupCard :group="group" ... />
</template>
```

### Styles

```scss
.group-connector {
  display: flex;
  align-items: center;
  gap: var(--p-space-200);
  padding: var(--p-space-200) 0;
}

.group-connector__line {
  flex: 1;
  height: 1px;
  background: var(--p-color-border);
}

.group-connector__badge {
  font-size: var(--p-font-size-300);
  font-weight: var(--p-font-weight-semibold);
  color: var(--p-color-text-secondary);
  background: var(--p-color-bg-surface);
  padding: var(--p-space-100) var(--p-space-300);
  border-radius: var(--p-border-radius-full);
  border: 1px solid var(--p-color-border);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  white-space: nowrap;
}
```

Within condition rows, a simpler connector label:

```scss
.condition-connector {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--p-space-100) 0;
}

.connector-label {
  font-size: var(--p-font-size-300);
  font-weight: var(--p-font-weight-semibold);
  color: var(--p-color-text-secondary);
  background: var(--p-color-bg-surface-secondary);
  padding: 2px var(--p-space-200);
  border-radius: var(--p-border-radius-100);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}
```

---

## Pattern 19: Form Validation Error Display

Dismissible error banner at the top of a form with a bulleted list of specific validation errors.

### Template

```vue
<PolarisBanner
  v-if="validationErrors.length"
  variant="critical"
  title="Please fix the following errors"
  dismissible
  @dismiss="validationErrors = []"
>
  <ul class="error-list">
    <li v-for="(err, idx) in validationErrors" :key="idx">{{ err }}</li>
  </ul>
</PolarisBanner>
```

### Styles

```scss
.error-list {
  margin: var(--p-space-100) 0 0;
  padding-left: var(--p-space-400);

  li {
    margin-bottom: var(--p-space-100);
  }

  li:last-child {
    margin-bottom: 0;
  }
}
```

### Validation logic pattern

```javascript
const validate = () => {
  const errors = [];
  if (!name.value?.trim()) {
    errors.push('Name is required');
  }
  items.value.forEach((item, idx) => {
    const label = `Item ${idx + 1}`;
    if (!item.field) errors.push(`${label}: Field is required`);
  });
  return errors;
};

const handleSave = () => {
  const errors = validate();
  validationErrors.value = errors;
  if (errors.length) return;
  // proceed with save...
};
```

### Inline field-level errors

Combine with Polaris component `error` props for per-field feedback:

```javascript
const nameError = computed(() => {
  if (validationErrors.value.length && !name.value?.trim()) {
    return 'Name is required';
  }
  return '';
});
```

```vue
<PolarisTextField label="Name" :error="nameError" ... />
```

---

## Pattern 20: Pagination / Load More

A "load more" button inside a bordered table card, showing progress as "X of Y".

### Template

```vue
<div class="table-wrap">
  <table class="data-table">
    <!-- thead + tbody -->
  </table>

  <div v-if="hasMore" class="table-pagination">
    <PolarisButton variant="plain" @click="loadMore">
      Load more ({{ items.length }} of {{ totalCount }})
    </PolarisButton>
  </div>
</div>
```

### Styles

```scss
.table-wrap {
  border: 1px solid var(--p-color-border);
  border-radius: var(--p-border-radius-300);
  background: var(--p-color-bg-surface);
}

.table-pagination {
  padding: var(--p-space-300);
  text-align: center;
  border-top: 1px solid var(--p-color-border);
}
```

### Logic

```javascript
const PAGE_SIZE = 50;
const currentOffset = ref(0);

const hasMore = computed(() => items.value.length < (totalCount.value || 0));

const loadMore = () => {
  currentOffset.value = items.value.length;
  emit('load-page', {
    limit: PAGE_SIZE,
    offset: items.value.length,
  });
};
```

---

## Pattern 21: Stats / Metrics Row

A horizontal row of label + value stat items inside a card. Common for detail views and dashboards.

### Template

```vue
<PolarisCard>
  <PolarisCardSection>
    <div class="stats-row">
      <div class="stat-item">
        <PolarisText variant="bodySm" color="subdued">Members</PolarisText>
        <PolarisText variant="headingMd">{{ formatNumber(count) }}</PolarisText>
      </div>
      <div class="stat-item">
        <PolarisText variant="bodySm" color="subdued">Created</PolarisText>
        <PolarisText variant="bodyMd">{{ formatDate(createdAt) }}</PolarisText>
      </div>
      <div class="stat-item">
        <PolarisText variant="bodySm" color="subdued">Updated</PolarisText>
        <PolarisText variant="bodyMd">{{ formatDate(updatedAt) }}</PolarisText>
      </div>
    </div>
  </PolarisCardSection>
</PolarisCard>
```

### Styles

```scss
.stats-row {
  display: flex;
  gap: var(--p-space-800);
  flex-wrap: wrap;
}

.stat-item {
  display: flex;
  flex-direction: column;
  gap: var(--p-space-100);
}
```

For numeric stats, use `variant="headingMd"`. For text/date stats, use `variant="bodyMd"`.

---

## Pattern 22: Empty State in Bordered Card

Wrap `PolarisEmptyState` in a bordered card that visually matches the table wrapper, so loading → data → empty states all share consistent chrome.

### Template

```vue
<!-- Data table when items exist -->
<div v-if="items.length" class="table-wrap">
  <table class="data-table">...</table>
</div>

<!-- Empty state in matching card -->
<div v-else class="empty-wrap">
  <PolarisEmptyState heading="No items yet" icon="📋">
    Create your first item to get started.
    <template #actions>
      <PolarisButton variant="primary" @click="$emit('create')">
        Create item
      </PolarisButton>
    </template>
  </PolarisEmptyState>
</div>
```

### Styles

```scss
.empty-wrap {
  min-height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--p-color-bg-surface);
  border: 1px solid var(--p-color-border);
  border-radius: var(--p-border-radius-300);
}
```

### Contextual empty state messaging

Vary the message based on component state for better UX:

```vue
<PolarisEmptyState heading="No members yet" icon="👤" compact>
  {{ isActive
    ? 'Members will appear here as users qualify.'
    : 'Activate this item to start evaluating users.' }}
</PolarisEmptyState>
```

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
| Draggable action picker sidebar | Node Palette (Pattern 11) |
| Mode selection (2–3 exclusive options) | Entry Mode Radio Cards (Pattern 12) |
| Side panel editing header/footer | Config Panel Chrome (Pattern 13) |
| Template variable reference | Variable Reference Panel (Pattern 14) |
| Toolbar status indicator (Live/Draft) | Status Badge (Pattern 15) |
| 3+ view CRUD navigation | Multi-View Navigation Shell (Pattern 16) |
| Destructive action confirmation (table) | Inline Confirmation — row replacement (Pattern 17A) |
| Destructive action confirmation (detail) | Inline Confirmation — banner (Pattern 17B) |
| Logical operator between groups | AND/OR Group Connector (Pattern 18) |
| Form validation errors | Validation Error Banner (Pattern 19) |
| Paginated table with load-more | Pagination / Load More (Pattern 20) |
| Key metrics display row | Stats / Metrics Row (Pattern 21) |
| Empty table / list placeholder | Empty State in Bordered Card (Pattern 22) |

## Rules

- **No hardcoded colors or pixel values** — always use `var(--p-color-*)`, `var(--p-space-*)`, etc.
- **No global CSS** — all styles must be `<style lang="scss" scoped>`
- **Scoped styles only** — `.vue` files are raw SFCs compiled by `@weweb/cli`
- **Use `?.` optional chaining** for all prop access (WeWeb requirement)
- **Data binding** — always use `:modelValue` + `@update:modelValue` instead of `v-model`
- **`accent-color`** — set once on the root element using `:deep()` so all child components inherit it:
  ```scss
  .component-root {
    :deep(input[type="radio"]),
    :deep(input[type="checkbox"]) {
      accent-color: var(--p-color-bg-fill-brand, #2C6ECB);
    }
  }
  ```
- **BEM naming** — use double-dash for modifiers (`.th--first`, `.item--active`) per BEM convention
- **Table typography** — use `--p-font-size-325` for table body text, `--p-font-size-275` for table headers. The smaller size reads better for dense data.
