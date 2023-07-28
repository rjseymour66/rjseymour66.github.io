---
title: "Menus"
weight: 1
description: >
  Standard menus in Inkscape.
---

# File Menu

File > New From Template to create an image with default sizing.

File > Open--you should only open vector documents:
- .svg
- .eps
- .ai 
- .pdf (sometimes)

Opened files are often grouped together by default. To ungroup, select the object, Obect > Ungroup as many times as you ned to.

Import means that you are importing a file into an open file. 
- File > Import > select file

Clipart with File > Import Web Image
- Select a source and choose a clipart image that you want to import 

### Saving
When you save a file, save it as an Inkscape SVG 

### Exporting to png or jpeg 

File > Export 

You can do single exports or batch exports. There is a preview at the bottom.

#### Single export 

Export by:
- Document: Anything on or outside the page content
- Page: only what is on the page.
- Selection: Whatever you have selected. Click Export Selected only to export jsut what you selected. Otherwise, it will export what you selected and anything else behind the selections.
- Custom: export anything within the X and Y coordinates that you set.

Each tab has image size settings, like DPI. (Only worry about DPI when you are printing something or you are opening it up in another application. The DPI is 96. Standard print DPI is 300.)

Select the format at the bottom. PNG preserves transparency.

#### Batch export

Export multiple graphics at one time. Name of the file is automatically named. You can select the graphic and change the name in the top toolbar.

After you have your batch export ready, you can click Add Export. Then, you can create another export in a different format and export them in the same operation.

Export by:
- Selection: Batch exports anything that you select with the Selection tool.
- Layers: 
- Pages: 

After you click Export, accept the defaults in the popup.

## Document properties

File > Document Properties or CTRL + SHIFT + D

Document is the piece-of-paper-looking white space. It is 8.5 x 11 letter size by default.

- Display: Change the white space attributes.
  - Format: Select a template 
  - Width, height, orientation 
  - Resize to content: will update the workspace to fit any objects that you have created.
  - Display--maybe set it to px? defaults to mm 
  - Page, border, dekt. Change the bottom background color (alpha) to use a new background in your design
  - checkerboard to check transparency
  - always on top: border is always on top
  - antialiasing: smooths out the rigid edges, like in text or raster images.
- Guides: Changes colors of the guides that you can pull down from the top and side rulers 
- Grids: place grid lines all over the workspace.
  - Click New then select the grid. 
- Color: A color profile that you downloaded from the internet. Not the same as the color pallete below 
- Scripting: For scripts 
- Metadata: Metadata for the file that you created.
- License: Use an Open* license to make it free, use proprietary to protect it.

## Creating templates 

File > Document Properties

Allow you to save your document so you can open and launch from it without having to input the size dimensions, etc. each time.

File > Document Properties to change the sizing, etc 
File > Save Template. Then add name and set as default, if you want to.

Create a new file from a template: File > New From Template 

## Documents Tools

Version 1.2

Create multi-page documents. This toolbar is second from the top, directly below the standard New file, open directory toolbar. 

Click Create and Edit document Page on bottom of left toolbar. This activates nodes on the page. 
- You can click and drag to create a new page in the same workspace. 
- Click Create a New Page above select tool icon to create a copy of the currently selected page.

All pages have an order. You can select the page and change order (or name) in the second toolbar from top.

- `Move overlapping objects as the page is moved` will move any objects on the page along with the page, if you click and drag.
- `Fit the page to the current selection or drawing if there is no selection` trims the page down to the size of the object.

### Save a multipage document 

File > Save as > <format> (ex: pdf)

# Edit menu

## Inkscape preferences

Allows you to control settings and behaviors.

Edit > Preferences or CTRL + SHIFT + P 

### Tools 

These are all the tools on the left hand toolbar.
- Shapes > <shape> > last used style. This is why Inkscape creates an object just like the previous one. Use This tool's own style to change that.
- Text: Inkscape does not have fonts installed, it just references fonts that are installed on your OS.

### Interface

Zoom correction factor: makes sure what you create is true to size.

Toolbars: Change color, size of toolbar icons
Keyboard: Keyboard shortcuts

### Input/Output 

Autosave: Configure where and how often work is saved automatically

## Bitmap copies 

Select the object
Edit > Make a bitmap copy
Click and drag the new object where you want it

## Creating Clones 

Clones is a way to create duplicate copies of objects that are grouped together and behave together.

Edit > Clone > Create Clone or ALT + D. Then click and drag the new object over the existing one.
- THis lets you change properties of the original object and automatically apply them to the clone.

Edit > Clone > Unlink clone lets you make changes to clones individually. 

Edit > Clone > Unlink clones recursively lets you unlink clones that are grouped together.
- you can't group clones and then unlink them 

Edit > Clone > Relink to copy 

Edit > Clone > Select Original if you cannot find the original when there are many clones

## Creating Tiled Clones 

How to generate clones, in bulk.

Edit > Clone > Create tiled clones 

> All the tabs in this tool work together, so settings on one affect settings on another. Use the reset to get back to the defaults.

### Symmetry tab
After you create clones, the clones are created on top of the original. Raise the original to the top and then change as needed.

If you want to generate clones that cover a specific amount of space, you can set the width and height

Use the dropdown to select clone formatting. You can rotate them all X degrees, or create reflection.

### Shift
You can shift by row or by column.

### Color 

Before you can edit the color, you have to remove the fill color.
1. Select the object.
2. Go to Fill and Stroke, and select the question mark.
3. Go back to Create Tiled Clones and select the initial color. 

Use HSL:
H: Hue--the color that is being chosen 
S: Saturation--the levle of color that is applied
L: Lightness--gradient of the color, with black and white on either side.

### Trace 

Lets you trace the drawing under the clones with the top object. You create two objects, then select the one that you would like to use to trace.
- Opacity and Size looks pretty cool. 


# View menu

## View and display mode 

View > Zoom 1:1
- Lots of self-explanatory options 

View > Display Mode (CTRL + 5 to cycle through them)
- Normal is default 
- Outline is a wire-frame view of paths in design 
- No filters: Removes any filters from graphic or selected object. Good to view PDFs
- Visible hairline: Display any stroke, no matter how thin 
- Outline overlay: same as Outline, just leaves some color 

## Rotating the Canvas 

Not rotating the design, just the view of the design.

View > Orientation > Lock Rotation -- uncheck to rotate it
- To rotate, CTRL + Shift + 2 fingers on touchpad
- Also use options in View > Orientation > <option-name>

To reset, View > Orientation > Reset Rotation

## X-ray and Split view mode 

View > Split Mode > Split
- Double-click the arrow in the circle cursor to select which side displays the normal mode
View > Split Mode > X-Ray
- Gives you a large cursor to view everything in X-Ray mode.

# Layers and Objects 

Layering is how you control the stacking of objects in your workspace 

Layers > Layers and Objects opens up the side menu
- eyeball makes it visible/invisible 
- lock icon locks object into place 

You can rename the layer

Add layer button
- name it
- position it relative to existing layer 

When you delete a layer, it deletes the contents of the layer. 

# Object menu

## Grouping 

Select the objects that you want to group, then Object > Group or use the toolbar shortcut 

If you want to select individual paths or objects in a group, click the Edit Paths by Nodes in the left hand toolbar, below the Select tool. After it is selected, you can use the Select tool and edit it.

To remove a selected object from the group, select the object, then Object > Pop Selected Object out of group.

## Guides 

View > Show/Hide > Rulers needs to have a check beside it.

Hover cursor over ruler on top, corder, or left, then drag.

Double-click to open up guide properties. Change Angle to change the degree/slant

To disable some guides if they are getting in the way, View > Guides or SHIFT + | (vertical bar)
To lock guides, click the Lock in the top-left corner.

Use a shape to create guides:
1. Select object 
2. Object > Object to guides
Object disappears and there are guides that outline the shape.

## Raising and Lowering objects 

Object > Raise/Lower/*, but you can do this with toolbar icons. Or, keyboard shortcuts:
Fn + Home/PgUp/PgDn/End

## Fill and Stroke menu 

Object > Fill and Stroke 

### Fill tab 

Bottom colors are pre-defined colors, but fill menu allows you to create your own.

Create negative space, or remove color from overlapping sections of combined objects:

1. Path > Combine 
2. Fill and Stroke menu > Any path...holes in the fill icon on the right 

`hexcolorff`--`ff` is what makes it opaque.

#### Create blurred text
1. Write text
2. Make the text white 
3. Duplicate the text 
4. Make the duplicate black 
5. Push the dup text on the lower level 
6. Change th blur of the text 


Opacity slider in the HSLA color (the A part) applies to the fill of the object. Opacity at the bottom applies to the entire object. So, the HSLA value doesn't impact the stroke. 

The Blend dropdown changes the way that objects interact with each other.

## Working with strokes 

Stroke is an outline that gets applied to an object.

### Stroke style tab

Join:
Cap: Butt cap ends at the end of the node path exactly
Order: Changes how the stroke overlaps or interacts with the path. Put stroke on top of the fill or behind the fill.

4 icons to the right of the unit dropdown in the toolbar let you select if the stroke scales with the object.

## Working with Gradients 

Open the Fill and stroke tab CTRL + SHIFT + F, then click on the gradient tool in the left hand toolbar to view the stops.

To change the color of a gradient, click the stop and then select a new color.

There is a reverse gradient button in the toolbar beside the lock icon.

### Linear gradient
Click and drag a stop to change the gradient direction. 
Double-click on the gradient to add another stop.

### Radial gradient

CTRL + SHIFT to scale gradients evenly.

### Mesh gradient 

Select the Mesh tool in the left hand toolbar, then select the number of rows and cols for the mesh tool. Then select the mesh gradient from the Fill and Stroke toolbar.

You can change the color only where the mesh gradients intersect.

### Conical gradient

Provides a circular, top-down view of the object.

1. Select the mesh tool
2. Select Conical gradient from the top toolbar.
3. Click and drag over the image.

## Swatches 

View > Swatches, bottom of the screen with the color picker 

Use the up and down arrows on the far right to cycle through the color palette.

To save a custom color, select the object that has the color, then click the Swatch icon in the Fill and Stroke menu. Then, you can view the color in the Swatches tab.