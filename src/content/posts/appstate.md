---
title: App State
published: 2025-03-08
updated: 2025-03-08
description: 'Core state management with the AppState struct'
tags: [docs, code]
category: 'code'
draft: false
---

# App State

The `AppState` struct is the heart of pola, managing all application state including skins, search results, UI selections, and user preferences.

## Definition

```rust
struct AppState {
    input: String,
    skins: Vec<Skin>,
    results: Vec<Skin>,
    table_state: TableState,
    all_terms: HashMap<String, TermInfo>,
    suggestion: Option<String>,
    suggestion_list: Vec<String>,
    suggestion_index: usize,
    name_map: HashMap<String, usize>,
    input_history: Vec<String>,
    history_index: usize,
    scroll_offset: usize,
    sort_field: SortField,
    sort_descending: bool,
    show_detail: bool,
    current_suggestion_terms: HashMap<String, TermInfo>,
    current_page: usize,
    items_per_page: usize,
    favorites: HashSet<String>,
    key_bindings: HashMap<String, (KeyCode, KeyModifiers)>,
    should_exit: bool,
    help_state: ListState,
    editing_keybinds: bool,
    awaiting_key: Option<String>,
}
```

### Key Fields

- **`input`**: Current search query.
- **`skins`**: Full list of loaded skins.
- **`results`**: Filtered and sorted search results.
- **`table_state`**: Tracks the selected item in the results table.
- **`favorites`**: Set of favorited skin names.
- **`key_bindings`**: Customizable key mappings.

### Initialization

```rust
impl AppState {
    fn new() -> Self {
        let skins = load_skins();
        let name_map: HashMap<_, _> = skins.iter().enumerate().map(|(i, s)| (s.name_lower.clone(), i)).collect();
        let all_terms = load_all_terms(&skins);
        let mut results = skins.clone();
        results.sort_by(|a, b| a.name_lower.cmp(&b.name_lower));
        let favorites = load_favorites().unwrap_or_default();
        let key_bindings = load_key_bindings().unwrap_or_else(|_| default_key_bindings());
        // ... (other field initializations)
    }
}
```

## Core Methods

### `update_search()`

Filters and sorts the `results` based on the current `input`.

```rust
fn update_search(&mut self) {
    if self.input.trim().is_empty() {
        self.results = self.skins.clone();
        self.sort_field = SortField::Name;
        self.sort_descending = false;
    } else {
        self.results = search_skins(&self.skins, &self.name_map, &self.input, &self.favorites);
    }
    self.sort_results();
    // Adjust pagination and selection
}
```

### `toggle_favorite()`

Adds or removes a `favorite` tag from a skin.

```rust
fn toggle_favorite(&mut self) {
    if let Some(selected) = self.table_state.selected() {
        let absolute_index = self.current_page * self.items_per_page + selected;
        if let Some(skin) = self.results.get(absolute_index) {
            if self.favorites.contains(&skin.name) {
                self.favorites.remove(&skin.name);
            } else {
                self.favorites.insert(skin.name.clone());
            }
            save_favorites(&self.favorites).expect("Failed to save favorites");
            self.update_search();
        }
    }
}
```

### `sort_results()`

Sorts results based on sort_field and sort_descending.

```rust
fn sort_results(&mut self) {
    match self.sort_field {
        SortField::Name => {
            if self.sort_descending {
                self.results.sort_by(|a, b| b.name_lower.cmp(&a.name_lower));
            } else {
                self.results.sort_by(|a, b| a.name_lower.cmp(&b.name_lower));
            }
        },
        // ... (Rarity, Event cases)
    }
}
```

:::tip
The name_map HashMap enables quick lookups by skin name, optimizing search performance.
:::
