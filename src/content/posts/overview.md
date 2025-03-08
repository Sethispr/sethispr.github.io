---
title: pola
published: 2025-03-08
description: "overview"
image: ""
tags: ["docs", "main"]
category: overview
draft: false
---

# Code Overview

pola is a TUI written in Rust, using `ratatui` for UI rendering and `crossterm` for terminal control and event handling. This page outlines the overall architecture and the entry point of the application.

## Architecture

The application follows a single-threaded, event-driven model:

- **Dependencies**: Uses `ratatui` for TUI widgets, `crossterm` for raw terminal mode and input, and `fuzzy_matcher` for search functionality.
- **State Management**: Centralizes state in the `AppState` struct, which tracks skins, search results, UI state, and user preferences.
- **Main Loop**: Runs in `main()`, handling events, updating state, and rendering the UI in a continuous loop until exit.

## Entry Point: `main()`

The `main` function initializes the terminal, sets up the application state, and runs the main event loop.

```rust
fn main() -> io::Result<()> {
    enable_raw_mode()?;
    execute!(
        io::stdout(),
        EnterAlternateScreen,
        EnableMouseCapture,
        crossterm::terminal::DisableLineWrap,
    )?;

    let backend = CrosstermBackend::new(io::stdout());
    let mut terminal = Terminal::new(backend)?;

    let mut app = AppState::new();
    app.update_search();

    while !app.should_exit {
        terminal.draw(|f| ui(f, &mut app))?;
        if event::poll(Duration::from_millis(50))? {
            match event::read()? {
                Event::Key(key) => { /* Handle key events */ },
                Event::Mouse(mouse_event) => { /* Handle mouse events */ },
                _ => {},
            }
        }
    }

    disable_raw_mode()?;
    execute!(
        io::stdout(),
        LeaveAlternateScreen,
        DisableMouseCapture,
        crossterm::terminal::EnableLineWrap,
    )?;
    terminal.show_cursor()?;
    Ok(())
}
```

### Key Steps

The `main()` function serves as the application's lifecycle through the following steps:

1. **Terminal Setup**  
   - Enables raw mode with `enable_raw_mode()` to capture raw input.  
   - Executes terminal commands to enter alternate screen and enable mouse capture.

2. **Initialization**  
   - Creates a `CrosstermBackend` and a `Terminal` instance.  
   - Initializes `AppState` with `AppState::new()`.

3. **Main Loop**  
   - Renders the UI with `terminal.draw()`.  
   - Polls for events every 50ms using `event::poll()`.  
   - Processes key and mouse events to update `AppState`.

4. **Cleanup**  
   - Disables raw mode and restores the terminal state on exit.

:::note
The 50ms polling interval balances responsiveness and CPU usage. Adjust this value based on performance needs.
:::
