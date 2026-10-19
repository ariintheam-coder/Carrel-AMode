# Carrel-AMode

**A complete EPUB library and reader in one HTML file.**

*Carrel* is a small enclosed desk in a library stack, where one person reads alone.

## Why this exists

Carrel is a local-first EPUB library and reader: keep books on your device, organize a personal shelf, and read without an account or a required network connection.

## Use it

**Local file** — open `index.html` in a browser. It works offline. Browser support for persistent storage from `file://` varies, so a local or static web server is more reliable if you want your library to persist.

**Self-host** — copy `index.html` to any static host. No build step or dependencies are needed.

Use **Add books** to choose **Add files** or **Add folder**. Folder selection imports EPUB files from the chosen folder (including subfolders where supported by the browser). You can also drag and drop EPUB files onto the page.

## What it does

**Library** — stores imported books locally in IndexedDB, extracts metadata and cover art, and provides search, sorting by recently opened / added, title, author, or rating, and five-star ratings. Books without cover art get generated cloth bindings. You can also set a custom cover.

**Reader** — paginated reading with whole-book page numbers, a table of contents from EPUB 3 navigation or EPUB 2 NCX, six system typeface choices, Paper / Sepia / Night page themes, adjustable type size, line spacing and margins, keyboard and touch navigation, and saved reading progress.

**Notes** — create margin notes at your reading position and highlight selected passages with optional notes and tags. Page pins make margin notes easy to revisit. The library Notes panel can filter notes by book, tag, and note type, and jump back to their locations.

**Privacy** — books, covers, preferences, progress, and notes are stored locally in the browser; books are not uploaded to a service. No account is required.

## How it works

| Part | Approach |
|---|---|
| **Unzip** | Parses the ZIP central directory and inflates stored and deflated entries with `DecompressionStream('deflate-raw')`; includes a ZIP64 fallback. |
| **Parse** | Reads `META-INF/container.xml` and the OPF package for the manifest, spine, Dublin Core metadata, and cover. Reads the table of contents from an EPUB 3 nav document or, as a fallback, an EPUB 2 NCX. |
| **Store** | IndexedDB holds the original EPUB as a `Blob`, along with library data and reader state. |
| **Render** | Chapter XHTML is written into a same-origin iframe. Scripts and inline event handlers are stripped; book stylesheets are inlined and images are converted to data URLs for local rendering. |
| **Paginate** | CSS multi-column layout paginates each chapter. A hidden iframe counts pages in the background; counts are cached against the reading layout settings, while an estimate is shown until counting finishes. |
| **Annotate** | Highlights and margin notes, including their notes and tags, are stored in IndexedDB separately from the EPUB file. |

## Limitations

- **DRM-protected books won't open.** This project does not circumvent DRM.
- **Fixed-layout EPUBs** (such as comics and picture books) are reflowed rather than displayed as fixed pages.
- **Reading position** uses a spine index and fraction, not a proper [EPUB CFI](https://idpf.org/epub/linking/cfi/), so it may shift by about a page after layout changes.
- **EPUB only.** MOBI, AZW3, and PDF are not supported.
- **Nested tables of contents** are flattened to one level.
- **Typefaces are system font stacks**, so their appearance varies by operating system. The reader does not download fonts.
- **Folder selection depends on browser support.** If folder picking is unavailable, use **Add files** or drag and drop EPUBs instead.

## Browser support

The app requires `DecompressionStream`: Chrome / Edge 103+, Firefox 113+, or Safari 16.4+. It checks for this API and displays a message in unsupported browsers. Folder selection also depends on browser support for directory file pickers.

## Contributing

Issues and pull requests are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md). The project is intended to remain a single HTML file with no dependencies.

## License

[MIT](LICENSE).
