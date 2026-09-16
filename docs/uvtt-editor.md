# Universal VTT editor

Any image map in your library can be exported as a **Universal VTT** (`.uvtt`) file - the
format Foundry, Roll20, and other virtual tabletops read for dynamic lighting. The export
carries the map image and its grid, so the map drops into your VTT already lined up
instead of you scaling the image by hand. Export it from the **Download** menu on a map's
detail page. Maps that already have a real `.uvtt` linked to them do not offer the option,
since the file you already have carries walls and lighting of its own.

**Edit VTT** on the same page goes further, opening a full-screen editor for the
vision-blocking walls, doors, windows, and lights a virtual tabletop uses - the parts of
the format that otherwise take a separate tool like Dungeondraft to produce.

Nothing you do in the editor touches your library. The walls and lights are saved against the map inside Grimoire, and the `.uvtt` is built fresh each time you export it: your original image is never modified, and no extra file appears next to it. That also means an edited map can be added to a campaign like any other, and exported from there whenever you need the file.

## Confirm the grid first

Everything you draw is measured in grid squares, so the editor opens on the grid rather than the drawing tools: your map appears with the grid Grimoire detected drawn over it, and you can see at a glance whether it lines up. Zoom in to check it closely - at fit-to-window on a large map, one screen pixel covers several of the image's. If it does not line up, the panel beside the map gives you two ways to fix it: say how many squares the map is across and down, which is how most people already know their grid and redraws the overlay as you type, or set the cell size and the grid's offset directly, with nudge buttons for walking an almost-right grid into place a pixel at a time.

## Drawing walls, doors, and lights

Then draw:

- **Walls** block line of sight. Click to place each corner and double-click (or press Enter) to finish the run; close a room by ending where you started. Snapping is a three-way choice - grid intersections, half squares for a diagonal or a split doorway, or free-form for an irregular cave wall.
- **Object walls** are a separate layer for furniture, pillars, and other scatter. They are kept apart from ordinary walls because virtual tabletops treat them differently - Roll20, for instance, turns them into transparent barriers rather than solid walls.
- **Doors** and **windows** have a button each, and are drawn as a two-click line across an opening. A door blocks sight until it is opened at the table; a window can be seen through but not walked through.
- **Lights** are placed with a single click, and open for editing straight away - the panel at the top of the sidebar shows the new light's settings without you having to select it again. Start from a **preset** (candle, sconce, torch, brazier, campfire, moonlight, and a dozen more) picked from a menu or a row of colour swatches, then adjust the range in grid squares, the colour, the intensity, and whether it casts shadows. The preset values are taken from what Dungeondraft itself writes, so a torch starts out looking like one; tune any of them and the picker simply reads as Custom.

Whichever tool you pick, the panel on the right tells you how to drive it - how to finish a wall, how to abandon a half-drawn shape, which keys do what. Holding **Alt** places a single point off the grid without leaving your snap setting, for a room that is square apart from one canted corner.

The sidebar is a stack of collapsible sections. What you have selected sits at the top, since that is what you are looking at after a click; the tool help, layer counts and grid come next, and the player view and map-wide lighting settings anchor the bottom. Fold away anything you have stopped needing - the layer counts keep showing their total while collapsed - and selecting something on the map always reopens the selection panel.

Map-wide, you can set how bright an area with no light of its own looks, with a swatch showing the result, and mark a map whose lighting is **already painted into the artwork**. That last one matters: when it is set, a virtual tabletop may ignore or dim the lights you place, so Grimoire warns you if you have done both.

## Checking it from a player's chair

**Show player view** drops a token on the map and darkens everything that token could not see - walls cast real shadows, a closed door hides the room behind it, and a window does not. It answers "can my players see around that corner?" in place, instead of by exporting, importing, and moving a token in another program to find out.

Drag the token, or walk it with the arrow keys (hold Shift for half a square). Its sight is described the way a virtual tabletop describes one, with three separate switches:

- **Vision** is the master switch. Turn it off and the token sees nothing at all - which is what an object or scenery token is set to.
- **Night vision** lets the token see without any light, out to a distance you set. This is darkvision, and it is the only one of the three that genuinely has a range. It is tinted a cool blue in the preview, so you can tell ground that is merely visible from ground that is actually lit.
- **Token light** is the torch the token carries, which lights the ground around it.

Sight itself is not capped by a number of squares: what stops you seeing is walls and darkness. A token with vision but no night vision and no torch sees only where a placed light reaches, and Grimoire says so rather than leaving you with a black screen.

Lights you have placed light the room they are in, not just the square they sit on - stand anywhere within a lamp's range and you see by it, and you see the ground it falls on from across the room. Walls still apply: a lamp behind a shut door stays behind it, and light spilling around a corner reveals only the floor you can actually see. The **light level with no lights nearby** you set for the map drives how dark the unlit area looks here too, so a map authored with moonlight previews as moonlight rather than as pitch black.

It is a toggle, and off by default, because a darkened map is in the way while you are tracing walls. The token and its vision settings are **preview only**: a `.uvtt` has no concept of a player token, so none of it is saved or written to the exported file.

## Editing a `.uvtt` you already have

A Universal VTT file is not just viewable - it opens in the editor too, on the walls, doors, and lights it already carries, so you can fix a wall that is in the wrong place rather than redrawing the map from scratch. The file on disk is never rewritten; your changes are saved against the map in Grimoire, and exporting gives you a new `.uvtt` carrying the original's own image alongside your edits. When a map image and a `.uvtt` are linked as versions of each other, **Edit VTT** asks which one you mean - drawing fresh geometry over the picture, or changing what the linked file already holds. Only PDFs and videos are left out, having no single image to draw on.

## A map that is not in your library yet

The maps page has its own **VTT Editor** button, the same way the tokens page has a [token editor](token-editor.md). Drop in a map image or an existing `.uvtt` - from your desktop, a purchase you have not filed yet, anywhere - and edit it without adding anything to your library. When you are done, download the `.uvtt`, or send it straight into one of your campaigns as a linked resource, choosing the category it should be filed under. Nothing is written to your library either way, so this works exactly as well on a read-only mount.

## What the format cannot carry

Undo and redo cover everything, Delete removes whatever is selected, and Escape abandons a shape midway. The editor deliberately offers only what the format can actually carry, so nothing you set is quietly lost on export - which is why there is no wall thickness, secret or locked door, one-way wall, or light animation here. None of them exist in a `.uvtt`; Roll20 and Foundry let you add things like secret doors after importing.

## See also

- [Library structure - maps](library-structure.md#maps---organize-by-creator-or-collection) - where map files live, and how Grimoire detects the grid
- [Token editor](token-editor.md) - the same idea, for character tokens
