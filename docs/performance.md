# Performance and indexing

How Grimoire indexes your library, what OCR costs, and the knobs worth turning on a small
device or a large collection.

---

## Indexing

On first startup Grimoire scans the library and indexes every PDF page for full-text search. This can take several minutes for large collections. The index is stored in the data volume and subsequent startups are fast.

Use the **Rescan** button in the sidebar to pick up newly added files, or configure a scheduled rescan in **Settings → Maintenance**.

## OCR

Some PDFs contain only scanned page images with no embedded text layer (common with older, scanned game books). These can't be full-text searched from their text layer alone and show an **Image Only** badge.

The default Grimoire image bundles the [Tesseract](https://github.com/tesseract-ocr/tesseract) OCR engine (English), so scanned image-only PDFs are run through OCR and their recognised text is added to the search index. Books indexed this way show an **OCR** badge. No extra container or service is required.

OCR runs quietly in the background, so it never holds up the rest of your library. A scan indexes all your regular (text-based) books, maps, tokens, and audio first - those are searchable right away - and then works through the scanned books afterward. Even if you add 100+ scanned books at once, the rest of your library stays available while they're being processed.

Scanned books are also processed **page by page**, and progress is saved as it goes. If the server restarts (or you stop and start a scan), OCR simply picks up where it left off instead of starting the book over - so even a very large scanned book will finish, however long it takes.

- **Progress:** the admin scan status shows an OCR phase with a progress bar and the book currently being processed.
- **Disable OCR:** set `OCR_ENABLED=false`. Scanned image-only PDFs are then left unindexed, the same as on the slim image.
- **Slim image:** the `-slim` tags (e.g. `hunterreadca/grimoire:v1.5.0-slim`, `:slim`) omit Tesseract for a smaller image. OCR is automatically disabled there.
- **Upgrading to OCR:** if you enable OCR later (or switch from a slim image), any books that were previously skipped as image-only are automatically queued for OCR on the next scan.
- **Additional languages:** set `OCR_LANGUAGES` to a `+`-joined list of Tesseract language codes (e.g. `eng+deu+fra`). The extra languages' data files must be present in the image - mount a directory of `.traineddata` files (or point `TESSDATA_PREFIX` at one) to add languages without rebuilding.

### Speeding up OCR

Scanning a large book takes a while, and it happens quietly in the background - you can browse and search the rest of your library the whole time, and OCR picks up where it left off if the server restarts. If you have a big collection of scanned books and want it to finish faster, two optional settings help:

- **`OCR_CONCURRENCY`** - how many scanned books to work on at once. The default is `1`, which is gentle on small devices. If you're running on a machine with several CPU cores and plenty of memory to spare, raising this (e.g. `2`–`8`) processes books in parallel and gets through the queue faster. On a small device like a Raspberry Pi, leave it at `1`. Set it to `0` to turn OCR off entirely - handy if OCR keeps failing or running your machine out of memory and you just want it to stop, without switching to the slim image.
  - Each parallel worker uses roughly 50–250 MB of RAM depending on the pdf page image size, so make sure you have that much to spare per unit, and don't set it higher than the number of CPU cores (virtual/hyper-threaded cores count) or the workers just compete for the same processors without going any faster.
- **`OCR_DPI`** - how sharp the scanned pages are rendered before reading them (default `150`). Lowering it (e.g. `120`) makes OCR faster and lighter; raising it (e.g. `200`–`300`) can improve results on faint or low-quality scans at the cost of speed. Note: OCR scanned books can be individually rescaned at a higher DPI if needed from the application.
- **`OCR_PAGE_TIMEOUT`** - how long a single page may take before Grimoire gives up on it and moves to the next one (default `120` seconds). This exists so one pathological page can't stall a book forever, but how long a page takes depends on how *dense and noisy* the scan is far more than how big it is: on a low-power CPU an ordinary page might read in 13 seconds while a cramped, speckled one needs four minutes. When that happens the slow pages are skipped and their text never becomes searchable. If your logs mention pages being skipped, or a book is badged **OCR 14/206**, raise this (e.g. `600`) and re-read the book. Raising it costs nothing when pages finish quickly - it is a ceiling, not a delay. Set it to `0` for no limit at all, if you would rather wait indefinitely than lose text.

A rough guide: a small always-on device (like a Pi) is happiest at the defaults; a typical NAS can handle `OCR_CONCURRENCY=2`; a powerful desktop or server can go higher. It's safe to start low and raise it later - the queue just continues faster.

### When a book is only partly read

A scanned book that OCR'd cleanly is badged **OCR**. If some of its pages were skipped - they took longer than `OCR_PAGE_TIMEOUT`, or reading them failed - it is badged **OCR 14/206** instead: amber, and showing how many pages were actually read. Those pages are not in the search index, so searching the book will quietly miss that text.

The fix is to give the slow pages more time: raise `OCR_PAGE_TIMEOUT` (see [Speeding up OCR](performance.md#speeding-up-ocr)), restart, then re-read the book from its actions menu (**⋮** → **Re-OCR…**). Skipped pages are also logged as warnings when the book finishes, so `docker logs grimoire` will name them.

### Re-OCR a single book at a higher DPI

`OCR_DPI` sets the resolution for the whole library, and `150` is usually plenty. But the occasional faint or low-quality scan reads better at a higher resolution. Rather than raise the global default (and re-OCR everything), you can re-OCR just that one book: on an OCR-badged book, open the actions menu (**⋮**) and choose **Re-OCR…**, optionally enter a DPI (e.g. `300`), and run it. The book is re-read in the background at that resolution while the rest of the library is untouched; leave the DPI blank to re-OCR at the global default. It appears only for scanned/OCR'd books and requires GM or admin.

### Re-scan &amp; re-index a single book

Edited a PDF in place (embedded encounter notes, added errata)? Its search index goes stale until the next full library rescan. Instead, on that book open the actions menu (**⋮**) and choose **Re-scan &amp; re-index** to re-read just that file: its page count and thumbnail refresh, and its text is re-extracted and re-indexed in the background (an image-only PDF is re-queued for OCR). Works for any PDF and requires GM or admin.

## Page rendering

PDFs are rendered page-by-page server-side as WebP images rather than streamed as raw files. This keeps the viewer fast on mobile and avoids loading large files into the browser. Switch to the native PDF viewer anytime via the toolbar.

## Caching

Rendered pages are cached to disk by default. Provide a `VALKEY_URL` to use an in-memory Redis-compatible cache instead for faster repeat loads.

Cache entries are keyed by a hash of the source file's **contents**, so replacing a book with a different file at the same path automatically supersedes everything cached from the old one - pages, cover, and search text. The next rescan notices the change, and the reader picks up the new pages without a restart or a manual cache purge.

The on-disk cache is trimmed oldest-first back under `PAGE_CACHE_MAX_MB` (default 2 GiB) at startup and after each library scan.

## Large map &amp; token libraries

The Maps and Tokens galleries are built for large collections (thousands of items, plus their variants):

- **Items load progressively.** The gallery fetches items in pages and shows the first batch as soon as it arrives, instead of waiting for the whole library. Search, tag filters, and folder grouping still apply across everything once loading settles.
- **Thumbnails are cached by the browser.** Map and token thumbnails now carry cache validators, so revisiting a gallery or scrolling back re-uses the images already downloaded rather than re-fetching every one.
- **Folders start collapsed**, so opening a large library doesn't render thousands of cards at once - expand just the folders you need.

If a gallery still feels slow with a very large collection, keeping folders collapsed and using search or tag filters to narrow the view is the fastest way to work.

---

## See also

- [Configuration](configuration.md) - the full environment variable reference
- [Library structure](library-structure.md) - what gets indexed, per format
