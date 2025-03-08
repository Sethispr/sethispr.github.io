---
title: Input Handling
published: 2025-03-08
description: "Event handling and keybindings with crossterm"
image: ""
tags: ["docs", "code"]
category: code
draft: false
---

# Input Handling

Pola uses `crossterm` to handle keyboard and mouse events, with a customizable keybinding system.

## Event Loop

Events are processed in the `main()` loop:

```rust
if event::poll(Duration::from_millis(50))? {
    match event::read()? {
        Event::Key(key) => {
            if let Some(action) = app.get_action_for_key(&key) {
                if action == "show_help" {
                    app.show_help(&mut terminal)?;
                } else {
                    app.handle_action(&action);
                }
            } else {
                match key.code {
                    KeyCode::Char(c) => { app.input.push(c); app.update_search(); },
                    KeyCode::Backspace => { app.input.pop(); app.update_search(); },
                    _ => {},
                }
            }
        },
        Event::Mouse(mouse_event) => { /* Handle scrolling and clicks */ },
        _ => {},
    }
}
```

## Keybinds

Keybinds are stored in `AppState::key_bindings` and loaded from `~/.pola/key_bindings.json`.

```rust
fn default_key_bindings() -> HashMap<String, (KeyCode, KeyModifiers)> {
    let mut bindings = HashMap::new();
    bindings.insert("clear_search".to_string(), (KeyCode::Char('l'), KeyModifiers::CONTROL));
    bindings.insert("show_help".to_string(), (KeyCode::Char('h'), KeyModifiers::CONTROL));
    // ... (other defaults)
    bindings
}
```

## Help Menu

The `show_help()` method displays and allows editing of keybindings:

```rust
fn show_help<B: Backend>(&mut self, terminal: &mut Terminal<B>) -> io::Result<()> {
    loop {
        terminal.draw(|f| {
            let items: Vec<ListItem> = self.get_help_actions().iter().map(/* ... */).collect();
            let list = List::new(items).block(/* ... */);
            f.render_stateful_widget(list, modal_area, &mut self.help_state);
        })?;
        // Handle key/mouse events for navigation and editing
    }
}
```

:::tip
Select a keybind you want to edit and Press `E` in the help menu to enter edit mode, rebind keys by pressing your selected keybind and pressing `Enter` to confirm.
:::
