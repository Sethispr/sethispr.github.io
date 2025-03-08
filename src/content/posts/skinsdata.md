---
title: Skins Data
published: 2025-03-08
updated: 2025-03-08
description: 'Loading skins and managing favorites'
tags: ["docs", "code"]
category: 'code'
draft: false
---

# Skins Data

Pola manages skin data and user favorites using in-memory structures and file I/O.

## Loading Skins: `load_skins()`

Loads a hardcoded list of skins (could be extended to read from a file):

```rust
fn load_skins() -> Vec<Skin> {
    let mut skins = vec![
        Skin {
            name: "Cupid".to_string(),
            name_lower: "cupid".to_string(),
            rarity: "Pink".to_string(),
            rarity_lower: "pink".to_string(),
            event: "Valentine Case".to_string(),
            event_lower: "valentine case".to_string(),
            year: None,
            year_str: "".to_string(),
            tags: vec!["case".to_string()],
            tags_lower: vec!["case".to_string()].into_iter().map(|t| t.to_lowercase()).collect(),
        },
        // ... (many more skins)
    ];
    for skin in &mut skins {
        skin.name_lower = skin.name.to_lowercase();
        // ... (populate derived fields)
    }
    skins
}
```

## Favorites Management

Favorites are stored in `favorites.txt`:

```rust
fn load_favorites() -> io::Result<HashSet<String>> {
    if let Ok(content) = fs::read_to_string("favorites.txt") {
        Ok(content.lines().map(|s| s.trim().to_string()).filter(|s| !s.is_empty()).collect())
    } else {
        Ok(HashSet::new())
    }
}

fn save_favorites(favorites: &HashSet<String>) -> io::Result<()> {
    let content = favorites.iter().map(|s| s.as_str()).collect::<Vec<_>>().join("\n");
    fs::write("favorites.txt", content)
}
```

:::tip
Consider replacing the hardcoded load_skins() with a JSON or TOML file for easier updates.
:::
