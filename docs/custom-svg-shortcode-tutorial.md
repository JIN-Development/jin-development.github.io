# Tutorial: Creating a Custom Hugo Shortcode for SVG Images

## Overview

This tutorial teaches you how to create a reusable Hugo shortcode for displaying SVG images in your markdown files. You'll learn how to create a parameterized shortcode that provides a clean, maintainable way to include SVG graphics across your Hugo site.

**What you'll learn:**
- How to create custom Hugo shortcodes
- How to use shortcode parameters
- How to organize SVG assets in a Hugo project
- How to use your custom shortcode in markdown files

**Prerequisites:**
- Basic understanding of Hugo static site generator
- Familiarity with HTML and markdown
- A Hugo project already set up

---

## Table of Contents

1. [Understanding Hugo Shortcodes](#understanding-hugo-shortcodes)
2. [Project Structure](#project-structure)
3. [Creating the SVG Shortcode](#creating-the-svg-shortcode)
4. [Adding SVG Files](#adding-svg-files)
5. [Using the Shortcode in Markdown](#using-the-shortcode-in-markdown)
6. [Customization Options](#customization-options)
7. [Common Pitfalls](#common-pitfalls)
8. [Advanced Usage](#advanced-usage)

---

## Understanding Hugo Shortcodes

### What Are Shortcodes?

Hugo shortcodes are simple snippets that you can use in your markdown files to call built-in or custom templates. They allow you to embed complex HTML structures without cluttering your markdown content.

### Why Use Shortcodes for SVG Images?

**Benefits:**
- **Consistency**: Same styling and structure across all SVG images
- **Maintainability**: Update display logic in one place
- **Clean Markdown**: Keep your content files readable and focused
- **Parameterization**: Customize each instance (title, alt text, class, etc.)
- **Reusability**: One shortcode works for unlimited SVG files

---

## Project Structure

Your Hugo project should have this structure:

```
your-hugo-site/
├── layouts/
│   └── shortcodes/
│       └── svg-image.html          # Your custom shortcode
├── static/
│   └── images/
│       └── diagrams/                # SVG files location
│           ├── diagram1.svg
│           ├── diagram2.svg
│           └── ...
└── content/
    └── your-content.md              # Where you use the shortcode
```

---

## Creating the SVG Shortcode

### Step 1: Create the Shortcode Directory

If it doesn't exist already:

```bash
mkdir -p layouts/shortcodes
```

### Step 2: Create the Shortcode File

Create a new file at `layouts/shortcodes/svg-image.html`:

```bash
touch layouts/shortcodes/svg-image.html
```

### Step 3: Add the Shortcode Template Code

Open `layouts/shortcodes/svg-image.html` and add this content:

```html
<figure{{ with .Get "class" }} class="{{ . }}"{{ else }} class="center"{{ end }}>
  <img src="/images/diagrams/{{ .Get "src" }}"
       alt="{{ if .Get "alt" }}{{ .Get "alt" }}{{ else }}{{ .Get "title" }}{{ end }}" />
  {{ with .Get "title" }}
  <figcaption>
    <h4>{{ . }}</h4>
  </figcaption>
  {{ end }}
</figure>
```

### Step 4: Understanding the Template Code

Let's break down what this code does:

#### Line 1: Figure Element with Class
```html
<figure{{ with .Get "class" }} class="{{ . }}"{{ else }} class="center"{{ end }}>
```
- Creates an HTML `<figure>` element
- Uses the `class` parameter if provided, otherwise defaults to `"center"`
- `{{ with .Get "class" }}` checks if the parameter exists

#### Line 2-3: Image Element
```html
<img src="/images/diagrams/{{ .Get "src" }}"
     alt="{{ if .Get "alt" }}{{ .Get "alt" }}{{ else }}{{ .Get "title" }}{{ end }}" />
```
- Gets the `src` parameter and prepends the path `/images/diagrams/`
- For alt text: uses `alt` parameter if provided, otherwise falls back to `title`

#### Line 4-8: Optional Caption
```html
{{ with .Get "title" }}
<figcaption>
  <h4>{{ . }}</h4>
</figcaption>
{{ end }}
```
- Only renders the `<figcaption>` if a `title` parameter is provided
- `{{ with }}` conditionally includes content only if the variable exists

---

## Adding SVG Files

### Step 1: Create the Images Directory

```bash
mkdir -p static/images/diagrams
```

### Step 2: Add Your SVG Files

Place your SVG files in `static/images/diagrams/`. For example:

**File: `static/images/diagrams/reverse-proxy-architecture.svg`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<svg viewBox="0 0 900 600" xmlns="http://www.w3.org/2000/svg">
  <style type="text/css">
  <![CDATA[
    /* Your CSS animations here */
    @keyframes flow {
      0% { stroke-dashoffset: 20; }
      100% { stroke-dashoffset: 0; }
    }
  ]]>
  </style>

  <!-- Your SVG content here -->
  <rect x="50" y="50" width="800" height="500" fill="#f0f0f0" />
  <text x="450" y="300" text-anchor="middle">Your Diagram</text>
</svg>
```

**Important Notes:**
- Include the XML declaration: `<?xml version="1.0" encoding="UTF-8"?>`
- Wrap CSS in `<![CDATA[` blocks within `<style>` tags
- Use `viewBox` for responsive scaling
- Keep animations and styles inside the SVG file for portability

---

## Using the Shortcode in Markdown

### Basic Usage (Required Parameters Only)

The minimum required parameter is `src`:

```markdown
{{< svg-image src="reverse-proxy-architecture.svg" >}}
```

This will:
- Display the SVG from `/images/diagrams/reverse-proxy-architecture.svg`
- Apply the default `center` class
- No caption (title not provided)

### With Title (Recommended)

Add a title to show a caption:

```markdown
{{< svg-image src="reverse-proxy-architecture.svg"
    title="Reverse Proxy Architecture" >}}
```

This will:
- Display the SVG
- Show "Reverse Proxy Architecture" as a caption
- Use the title as alt text (for accessibility)

### Full Example with All Parameters

```markdown
{{< svg-image
    src="reverse-proxy-architecture.svg"
    title="Reverse Proxy Architecture"
    alt="Diagram showing Azure VM with Nginx reverse proxy, Tomcat application server, bastion host, and traffic flows including HTTP to HTTPS redirect"
    class="center" >}}
```

This provides:
- Custom alt text for screen readers (different from title)
- Explicit CSS class
- Full accessibility compliance

### Multiple SVG Images in One Page

```markdown
## Network Architecture

{{< svg-image src="network-topology.svg" title="Network Topology" >}}

## Authentication Flow

{{< svg-image src="auth-flow.svg" title="OAuth 2.0 Flow" >}}

## Database Schema

{{< svg-image src="database-schema.svg" title="Entity Relationships" >}}
```

---

## Customization Options

### Available Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `src` | ✓ Yes | N/A | Filename of the SVG (e.g., "diagram.svg") |
| `title` | No | None | Caption text displayed below the image |
| `alt` | No | Uses `title` | Alt text for accessibility (screen readers) |
| `class` | No | `"center"` | CSS class for styling the figure element |

### CSS Styling

Add custom styles in your theme's CSS:

```css
/* Center alignment (default) */
figure.center {
  text-align: center;
  margin: 2rem auto;
}

/* Full width */
figure.full-width {
  width: 100%;
  max-width: 100%;
}

/* Float left */
figure.float-left {
  float: left;
  margin-right: 1rem;
  max-width: 50%;
}

/* Custom styling for captions */
figure figcaption {
  margin-top: 0.5rem;
  font-style: italic;
  color: #666;
}

figure figcaption h4 {
  margin: 0;
  font-size: 1rem;
  font-weight: normal;
}
```

### Example Usage with Custom Classes

```markdown
{{< svg-image src="small-icon.svg" class="float-left" >}}

{{< svg-image src="banner.svg" class="full-width" >}}
```

---

## Common Pitfalls

### 1. Nested Shortcodes Don't Work

❌ **Wrong:** You cannot use a Hugo shortcode inside another shortcode:

```html
<!-- This will cause a parse error -->
{{< figure src="..." >}}
```

✓ **Correct:** Use HTML with Hugo template variables:

```html
<figure class="{{ .Get "class" }}">
  <img src="/images/{{ .Get "src" }}" />
</figure>
```

### 2. Forgetting the Path Prefix

The shortcode automatically prepends `/images/diagrams/`, so:

❌ **Wrong:**
```markdown
{{< svg-image src="/images/diagrams/diagram.svg" >}}
```

✓ **Correct:**
```markdown
{{< svg-image src="diagram.svg" >}}
```

### 3. SVG File Not Found

**Error:** SVG doesn't display or shows a broken image icon.

**Solution:**
- Verify the file exists: `ls static/images/diagrams/`
- Check the filename matches exactly (case-sensitive)
- Ensure the file is in `static/`, not `content/`
- Restart Hugo server after adding new files

### 4. Missing Alt Text for Accessibility

❌ **Not recommended:**
```markdown
{{< svg-image src="diagram.svg" >}}
```

✓ **Better:**
```markdown
{{< svg-image src="diagram.svg"
    title="My Diagram"
    alt="Detailed description for screen readers" >}}
```

---

## Advanced Usage

### Conditional Rendering

You can modify the shortcode to add conditional logic:

```html
{{ $src := .Get "src" }}
{{ if not $src }}
  <p style="color: red;">Error: src parameter is required</p>
{{ else }}
  <figure{{ with .Get "class" }} class="{{ . }}"{{ else }} class="center"{{ end }}>
    <img src="/images/diagrams/{{ $src }}"
         alt="{{ if .Get "alt" }}{{ .Get "alt" }}{{ else }}{{ .Get "title" }}{{ end }}" />
    {{ with .Get "title" }}
    <figcaption>
      <h4>{{ . }}</h4>
    </figcaption>
    {{ end }}
  </figure>
{{ end }}
```

### Adding Width/Height Parameters

Extend the shortcode to accept dimensions:

```html
<figure{{ with .Get "class" }} class="{{ . }}"{{ else }} class="center"{{ end }}>
  <img src="/images/diagrams/{{ .Get "src" }}"
       alt="{{ if .Get "alt" }}{{ .Get "alt" }}{{ else }}{{ .Get "title" }}{{ end }}"
       {{ with .Get "width" }}width="{{ . }}"{{ end }}
       {{ with .Get "height" }}height="{{ . }}"{{ end }} />
  {{ with .Get "title" }}
  <figcaption>
    <h4>{{ . }}</h4>
  </figcaption>
  {{ end }}
</figure>
```

Usage:
```markdown
{{< svg-image src="diagram.svg" width="800" height="600" >}}
```

### Multiple Asset Directories

If you want to support SVGs from different directories, add a `path` parameter:

```html
<figure{{ with .Get "class" }} class="{{ . }}"{{ else }} class="center"{{ end }}>
  <img src="{{ if .Get "path" }}{{ .Get "path" }}{{ else }}/images/diagrams{{ end }}/{{ .Get "src" }}"
       alt="{{ if .Get "alt" }}{{ .Get "alt" }}{{ else }}{{ .Get "title" }}{{ end }}" />
  {{ with .Get "title" }}
  <figcaption>
    <h4>{{ . }}</h4>
  </figcaption>
  {{ end }}
</figure>
```

Usage:
```markdown
{{< svg-image src="icon.svg" path="/images/icons" >}}
{{< svg-image src="diagram.svg" path="/images/diagrams" >}}
{{< svg-image src="default.svg" >}}  <!-- Uses /images/diagrams -->
```

---

## Testing Your Shortcode

### Step 1: Build and Serve

```bash
hugo server --buildDrafts
```

### Step 2: Check for Errors

Look for template parsing errors in the console:

```
Error: parse of template failed: template: _shortcodes/svg-image.html:1: ...
```

### Step 3: Verify in Browser

1. Navigate to the page with your shortcode
2. Open browser DevTools (F12)
3. Inspect the rendered HTML:

```html
<figure class="center">
  <img src="/images/diagrams/reverse-proxy-architecture.svg" alt="Reverse Proxy Architecture">
  <figcaption>
    <h4>Reverse Proxy Architecture</h4>
  </figcaption>
</figure>
```

### Step 4: Check Accessibility

Use browser tools to verify:
- Alt text is present
- Image loads correctly
- Caption displays properly

---

## Complete Example

Here's a complete working example from setup to usage:

### 1. Create the Shortcode

**File: `layouts/shortcodes/svg-image.html`**

```html
<figure{{ with .Get "class" }} class="{{ . }}"{{ else }} class="center"{{ end }}>
  <img src="/images/diagrams/{{ .Get "src" }}"
       alt="{{ if .Get "alt" }}{{ .Get "alt" }}{{ else }}{{ .Get "title" }}{{ end }}" />
  {{ with .Get "title" }}
  <figcaption>
    <h4>{{ . }}</h4>
  </figcaption>
  {{ end }}
</figure>
```

### 2. Add Your SVG

**File: `static/images/diagrams/reverse-proxy-architecture.svg`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<svg viewBox="0 0 900 600" xmlns="http://www.w3.org/2000/svg">
  <!-- Your SVG content -->
</svg>
```

### 3. Use in Markdown

**File: `content/tutorials/my-tutorial.md`**

```markdown
---
title: "My Tutorial"
date: 2025-01-15
---

## Architecture Overview

Here's how our reverse proxy architecture works:

{{< svg-image src="reverse-proxy-architecture.svg"
    title="Reverse Proxy Architecture"
    alt="Diagram showing Azure VM with Nginx reverse proxy, Tomcat application server, and bastion host" >}}

The diagram above shows the complete architecture with all components.

## Next Steps

Continue reading to understand each component...
```

### 4. Result

When Hugo builds your site, the shortcode expands to:

```html
<figure class="center">
  <img src="/images/diagrams/reverse-proxy-architecture.svg"
       alt="Diagram showing Azure VM with Nginx reverse proxy, Tomcat application server, and bastion host" />
  <figcaption>
    <h4>Reverse Proxy Architecture</h4>
  </figcaption>
</figure>
```

---

## Summary

You've learned how to:

✓ Create a custom Hugo shortcode for SVG images
✓ Structure your project to organize SVG assets
✓ Use parameters to customize shortcode behavior
✓ Implement the shortcode in markdown files
✓ Handle accessibility with alt text and titles
✓ Avoid common pitfalls with Hugo templates

**Benefits of this approach:**
- **One shortcode** works for unlimited SVG files
- **Clean markdown** - no HTML clutter
- **Maintainable** - update styles/structure in one place
- **Professional** - follows Hugo best practices
- **Accessible** - proper semantic HTML with alt text

---

## Further Reading

- [Hugo Shortcodes Documentation](https://gohugo.io/content-management/shortcodes/)
- [Hugo Template Functions](https://gohugo.io/functions/)
- [HTML Figure Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/figure)
- [SVG Accessibility](https://www.w3.org/WAI/tutorials/images/complex/)

---

**Created:** 2025-01-15
**Last Updated:** 2025-01-15
**Version:** 1.0
