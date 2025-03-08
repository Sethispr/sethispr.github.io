---
title: UI Rendering
published: 2025-03-08
updated: 2025-03-08
description: 'Rendering the TUI with Ratatui'
tags: [docs, code]
category: 'code'
draft: false
---

# UI Rendering

Pola uses `ratatui` to render its terminal-based user interface. The `ui()` function does the layout and widget rendering.

## Main UI Function

```rust
fn ui<B: Backend>(f: &mut Frame<B>, app: &mut AppState) {
    let chunks = Layout::default()
        .direction(Direction::Vertical)
        .margin(1)
        .constraints([
            Constraint::Length(3),  // Search bar
            Constraint::Length(5),  // Suggestions
            Constraint::Min(3),     // Results table + details
            Constraint::Length(1),  // Status bar
        ])
        .split(f.size());
    
    // Render search input
    let search_input = Paragraph::new(/* input text */).block(/* ... */);
    f.render_widget(search_input, chunks[0]);
    f.set_cursor(/* cursor position */);

    // Render suggestions
    let suggestion_list = List::new(/* suggestions */).block(/* ... */);
    f.render_stateful_widget(suggestion_list, chunks[1], &mut list_state);

    // Render table and details
    render_table_view(f, app, table_area);
    if let Some(detail_area) = detail_area {
        render_detail_panel(f, app, detail_area);
    }

    // Render status bar
    let status_bar = Paragraph::new(/* status text */).alignment(Alignment::Center);
    f.render_widget(status_bar, chunks[3]);
}
```

## Layout

The UI is divided into the following sections:

- **Search Bar**: Top section for input, styled with a rounded border.
- **Suggestions**: List of autocomplete suggestions below the search bar.
- **Results Table**: Main area showing filtered skins, with an optional detail panel.
- **Status Bar**: Bottom row with keybinding hints.

## Table Rendering

```rust
fn render_table_view<B: Backend>(f: &mut Frame<B>, app: &mut AppState, area: Rect) {
    let rows: Vec<Row> = app.results[start..end].iter().map(|skin| {
        Row::new(vec![
            Line::from(Span::styled(&skin.name, Style::default().fg(D_CYAN))),
            // ... (rarity, event, year, tags)
        ])
    }).collect();

    let table = Table::new(rows)
        .header(/* header row */)
        .block(/* title with pagination info */)
        .widths(&[/* column widths */])
        .highlight_style(/* selected row style */);
    f.render_stateful_widget(table, table_area, &mut app.table_state);

    // Render scrollbar
    let scrollbar = Scrollbar::new(ScrollbarOrientation::VerticalRight);
    f.render_stateful_widget(scrollbar, scrollbar_area, &mut scrollbar_state);
}
```

## Detail Panel

```rust
fn render_detail_panel<B: Backend>(f: &mut Frame<B>, app: &AppState, area: Rect) {
    if let Some(skin) = app.results.get(absolute_index) {
        let details = vec![
            Line::from(vec![Span::styled("Name: ", /* style */), Span::styled(&skin.name, /* style */)]),
            // ... (rarity, event, year, tags)
        ];
        let details_paragraph = Paragraph::new(details).wrap(Wrap { trim: true });
        f.render_widget(details_paragraph, inner_area);
    }
}
```

:::note
The Passion Fruit color scheme (inspired by MonkeyType) uses custom RGB values for consistency across widgets.
:::
