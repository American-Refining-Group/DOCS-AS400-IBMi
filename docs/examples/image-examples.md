# Image Usage Examples

This page demonstrates how to embed and reference images in your MkDocs documentation.

## Basic Image Syntax

The standard Markdown syntax for images is:

```markdown
![Alt text](path/to/image.png)
```

## Image Paths

Images should be placed in the `docs/images/` directory. Paths are relative to the markdown file location.

### From Root of docs/

If your markdown file is in `docs/`, reference images like this:

```markdown
![Diagram](images/diagrams/process-flow.png)
![Screenshot](images/screenshots/app-screen.png)
```

### From Subdirectories

If your markdown file is in a subdirectory like `docs/accounts-payable/`, use relative paths:

```markdown
![Diagram](../../images/diagrams/process-flow.png)
```

Or use absolute paths from the docs root:

```markdown
![Diagram](/images/diagrams/process-flow.png)
```

## Image Organization

Recommended directory structure:

```
docs/
  images/
    diagrams/          # Flowcharts, architecture diagrams
    screenshots/       # Application screenshots
    icons/            # Small icons and logos
    logos/            # Company/product logos
```

## Examples

### Live Example

Here's an actual image rendered in this documentation:

![Example Diagram](../images/diagrams/example-diagram.svg)

This SVG diagram demonstrates how images appear in your documentation. SVG format is ideal for diagrams as it scales perfectly at any size.

### Inline Images

Images can be placed inline with text:

Here is a diagram showing the process: ![Process Flow](../images/diagrams/example-diagram.svg)

### Standalone Images

Images can be displayed on their own line:

![Example Diagram](../images/diagrams/example-diagram.svg)

### Images with Titles

You can add titles using HTML:

```html
<img src="../images/diagrams/example-diagram.svg" alt="Process Flow" title="Order Processing Flow">
```

Example with title:

<img src="../images/diagrams/example-diagram.svg" alt="Example Diagram" title="This is an example diagram showing Start → Process → End flow">

### Responsive Images

For responsive images that scale properly:

```html
<img src="../images/diagrams/example-diagram.svg" alt="Process Flow" style="max-width: 100%; height: auto;">
```

Example of responsive image:

<img src="../images/diagrams/example-diagram.svg" alt="Example Diagram" style="max-width: 100%; height: auto;">

## Supported Image Formats

MkDocs supports common image formats:

- PNG (`.png`) - Best for screenshots and diagrams with text
- JPEG/JPG (`.jpg`, `.jpeg`) - Best for photographs
- GIF (`.gif`) - For animated images
- SVG (`.svg`) - Vector graphics, scales perfectly
- WebP (`.webp`) - Modern format with good compression

## Best Practices

1. **File Naming**: Use descriptive, lowercase filenames with hyphens:
   - ✅ `order-processing-flow.png`
   - ❌ `IMG_1234.PNG`

2. **Alt Text**: Always provide meaningful alt text for accessibility:
   - ✅ `![Order Processing Flowchart](images/diagrams/order-flow.png)`
   - ❌ `![image](images/img1.png)`

3. **File Size**: Optimize images to keep file sizes reasonable (< 1MB when possible)

4. **Organization**: Group related images in subdirectories

5. **Version Control**: Commit images to git rather than hosting externally

## Linking Images

You can make images clickable by wrapping them in a link:

```markdown
[![Thumbnail](images/thumbnails/large-diagram-thumb.png)](images/diagrams/large-diagram.png)
```

## Image Alignment

Using HTML, you can align images:

```html
<p align="center">
  <img src="../images/diagrams/example-diagram.svg" alt="Centered Image">
</p>
```

Example of centered image:

<p align="center">
  <img src="../images/diagrams/example-diagram.svg" alt="Centered Example Diagram" style="max-width: 400px;">
</p>

