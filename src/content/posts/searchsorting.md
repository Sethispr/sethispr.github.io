---
title: Search and Sorting
published: 2025-03-08
updated: 2025-03-08
description: 'Search mechanics and sorting logic'
tags: ["docs", "code"]
category: 'code'
draft: false
---

# Search and Sorting

pola provides a powerful search system with fuzzy matching and sorting capabilities.

## Search: `search_skins()`

Uses `fuzzy_matcher::skim::SkimMatcherV2` for fuzzy matching:

```rust
fn search_skins(
    skins: &[Skin],
    _name_map: &HashMap<String, usize>,
    query: &str,
    favorites: &HashSet<String>,
) -> Vec<Skin> {
    let matcher = fuzzy_matcher::skim::SkimMatcherV2::default();
    let mut scored_skins: Vec<(i64, &Skin)> = skins.iter().filter_map(|skin| {
        let mut score = 0;
        let mut matched = false;
        for term in &terms {
            if term.contains("fav") && favorites.contains(&skin.name) { score += 1000; matched = true; }
            if let Some(s) = matcher.fuzzy_match(&skin.name_lower, term) { score += s; matched = true; }
            // ... (event, rarity, tags, year)
        }
        if matched { Some((score, skin)) } else { None }
    }).collect();
    scored_skins.sort_by(|a, b| b.0.cmp(&a.0));
    scored_skins.into_iter().map(|(_, s)| s.clone()).collect()
}
```

## Features

- Filters: Supports `field:value` syntax (e.g., `rarity:pink`). OR just directly saying the rarity/tag/name!
- Regex: Allows regex patterns with `/pattern/`.
- Fuzzy Matching: Scores matches based on name, event, rarity, etc.

## Sorting: `sort_results()`

Sorts results based on `sort_field` and `sort_descending`:

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
        // ... (Rarity, Event)
    }
}
```

:::note
Clicking table headers toggles sorting direction via `toggle_sort()`.
:::
