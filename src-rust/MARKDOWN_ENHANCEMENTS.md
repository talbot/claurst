# Markdown Enhancements: Tables and Inline Formatting

## Overview

This document describes the enhanced markdown rendering features added to the Claude Code Rust port. The implementation includes:

1. **Markdown Table Rendering** with box-drawing characters
2. **Inline Formatting** support for bold, italic, and strikethrough
3. **Inline Code** highlighting
4. **Table Alignment** detection (left, center, right)

## Files Modified/Created

### New Files
- `crates/tui/src/messages/markdown_enhanced.rs` - Core implementation
- `crates/tui/tests/markdown_enhancements.rs` - Comprehensive test suite

### Modified Files
- `crates/tui/src/messages/markdown.rs` - Integrated table detection
- `crates/tui/src/messages/mod.rs` - Exported new public API

## Features

### 1. Markdown Table Rendering

Tables are detected and rendered using box-drawing characters for a polished appearance.

#### Detection

Tables must follow standard Markdown table format:
```markdown
| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |
| Cell 3   | Cell 4   |
```

The detector:
- Identifies table rows (lines starting and ending with `|`)
- Validates separator row (dashes with optional colons)
- Extracts headers and data rows
- Validates column count consistency

#### Rendering

Tables are rendered with:
- Top border: `┌─┬─┐`
- Header row with **bold** text
- Separator row: `├─┼─┤`
- Data rows with proper spacing
- Bottom border: `└─┴─┘`
- Proper column width calculation

Example rendered output:
```
  ┌──────────┬──────────┐
  │ Header 1 │ Header 2 │
  ├──────────┼──────────┤
  │ Cell 1   │ Cell 2   │
  │ Cell 3   │ Cell 4   │
  └──────────┴──────────┘
```

#### Alignment Support

Column alignment is detected from the separator row:

| Pattern | Alignment |
|---------|-----------|
| `:---` | Left |
| `:---:` | Center |
| `---:` | Right |
| `---` | Left (default) |

### 2. Inline Formatting

#### Bold

Syntax: `**text**` or `__text__`

Applied styling:
- `Modifier::BOLD` style
- Color: White (inherits from context)

Example:
```markdown
This is **bold** text and __also bold__ text.
```

#### Italic

Syntax: `*text*` or `_text_`

Applied styling:
- `Modifier::ITALIC` style
- Color: inherits from context

Example:
```markdown
This is *italic* text and _also italic_ text.
```

#### Strikethrough

Syntax: `~~text~~`

Applied styling:
- `Modifier::CROSSED_OUT` style
- Color: inherits from context

Example:
```markdown
This is ~~removed~~ text.
```

#### Inline Code

Syntax: `` `code` ``

Applied styling:
- `Color::Yellow`
- Monospace-like appearance

Example:
```markdown
Run `cargo check` to verify compilation.
```

### 3. Parsing Algorithm

The markdown renderer processes text in two stages:

#### Stage 1: Line-level Processing

1. Check for code block delimiters (```)
2. Check for table detection (pipes and dashes)
3. Check for blockquotes (> prefix)
4. Check for headings (#, ##, ###)
5. Process regular paragraphs with inline formatting

#### Stage 2: Inline Formatting

For regular text lines:
1. Scan for formatting markers: `, **, __, *, _, ~~
2. Extract content between markers
3. Apply corresponding styling
4. Continue parsing remainder

The parser uses:
- Character-by-character iteration
- Index-based tracking to avoid backtracking
- Maximum search limit (500 chars) to prevent runaway parsing
- No recursive nesting (for performance)

## Implementation Details

### Module Structure

```
messages/
├── mod.rs (module declaration, public API)
├── markdown.rs (main renderer with table integration)
└── markdown_enhanced.rs (table & formatting utilities)
```

### Public API

From `crate::messages`:
- `render_markdown(text: &str, width: u16) -> Vec<Line<'static>>`
- `detect_table(lines: &[&str], start_idx: usize) -> Option<(Table, usize)>`
- `render_table(table: &Table) -> Vec<Line<'static>>`
- `parse_inline_formatting(text: &str) -> Vec<Span<'static>>`

### Key Types

```rust
pub enum TableAlignment {
    Left,
    Center,
    Right,
}

pub struct Table {
    pub headers: Vec<String>,
    pub rows: Vec<Vec<String>>,
    pub alignments: Vec<TableAlignment>,
}
```

## Performance Considerations

### Optimization Strategies

1. **Lazy Regex Compilation**: Regex patterns compiled once at module load via `Lazy`
2. **Bounded Search**: Table detection and formatting search limited to prevent excessive scanning
3. **Non-Recursive Parsing**: Inline formatting doesn't support deep nesting to avoid stack overflow
4. **Early Returns**: Table detection fails fast if requirements not met

### Complexity

- Table detection: O(n) where n = number of lines
- Table rendering: O(m*c) where m = number of rows, c = columns
- Inline formatting: O(l) where l = line length (capped at 500 char lookahead)

## Edge Cases Handled

1. **Empty table cells**: Rendered with spacing preserved
2. **Tables with single column**: Detected and rendered correctly
3. **Incomplete formatting markers**: Treated as literal text
4. **Unclosed code blocks**: Trailing content rendered as code
5. **Multiple tables in document**: Each detected and rendered separately

## Limitations

1. **No nested formatting**: Nesting disabled for performance (e.g., `**bold *italic* bold**` renders flat)
2. **No format escaping**: Cannot escape formatting markers with backslash
3. **No table cell formatting**: Formatting markers within table cells not processed
4. **Table detection only in main flow**: Tables in code blocks/quotes not detected
5. **Width constraints**: Very narrow terminals (<20 chars) may not render well

## Testing

Comprehensive test suite in `crates/tui/tests/markdown_enhancements.rs`:

### Test Coverage

- ✅ Basic table detection
- ✅ Multi-row tables
- ✅ Alignment detection (left, center, right)
- ✅ Table rendering produces correct lines
- ✅ Tables in markdown documents
- ✅ Empty cells in tables
- ✅ Bold formatting (** and __)
- ✅ Italic formatting (* and _)
- ✅ Strikethrough formatting (~~)
- ✅ Inline code formatting (`)
- ✅ Multiple formatting in one line
- ✅ Markdown with mixed features

### Running Tests

```bash
# Run all markdown enhancement tests
cargo test --test markdown_enhancements

# Run specific test
cargo test --test markdown_enhancements table_basic_detection
```

## Examples

### Example 1: Simple Table

Input:
```markdown
| Name | Age | City |
|------|-----|------|
| Alice | 30 | NYC  |
| Bob   | 25 | LA   |
```

Rendered as:
```
  ┌───────┬─────┬───────┐
  │ Name  │ Age │ City  │
  ├───────┼─────┼───────┤
  │ Alice │ 30  │ NYC   │
  │ Bob   │ 25  │ LA    │
  └───────┴─────┴───────┘
```

### Example 2: Formatting Mix

Input:
```markdown
This is **bold**, *italic*, and ~~strikethrough~~ text with `code`.
```

Rendered with:
- "bold" in **bold**
- "italic" in *italic*
- "strikethrough" with ~~strikethrough~~
- "code" in yellow

### Example 3: Aligned Table

Input:
```markdown
| Left | Center | Right |
|:-----|:------:|------:|
| L1   |   C1   |    R1 |
| L2   |   C2   |    R2 |
```

Rendered with:
- Left column: left-aligned
- Center column: centered
- Right column: right-aligned

## Integration Notes

### Compatibility

- Fully compatible with existing markdown features (headings, blockquotes, code blocks)
- Does not break any existing functionality
- Tables are processed before inline formatting (correct priority)

### Terminal Requirements

- UTF-8 support for box-drawing characters
- Minimum width: 20 characters recommended
- Works on all modern terminals (Windows 10+, macOS, Linux)

### Style Inheritance

Formatting styles are applied independently:
- Colors set in formatting override defaults
- Modifiers (BOLD, ITALIC, CROSSED_OUT) stack
- Code blocks use fixed Color::Yellow regardless of context

## Future Enhancements

Potential improvements for future versions:

1. **Nested Formatting**: Safe implementation with depth limits
2. **Escape Sequences**: Support for `\*`, `\*\*`, etc.
3. **Cell Formatting**: Process markdown within table cells
4. **Table Features**:
   - Colspan/rowspan support
   - Colored cell backgrounds
   - Custom cell alignment
5. **Extended Syntax**: GFM table extensions
6. **Performance**: SIMD-optimized parsing for large documents

## Contributing

When modifying markdown rendering:

1. Add tests to `markdown_enhancements.rs`
2. Run `cargo test --test markdown_enhancements`
3. Verify no performance regression with `cargo test --release`
4. Document changes in this file
5. Update examples if behavior changes
