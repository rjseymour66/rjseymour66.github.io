---
title: "Layout and composition"
# linkTitle: ""
weight: 50
# description:
---

[Gestalt design principles](https://uxdesign.cc/how-to-use-powerful-gestalt-principles-in-design-with-infographic-4a10772eadbb)


Creating fluid layouts that smoothly collapse from a desktop screen to a mobile device is an important part of a site's strategy.

## Website structure

Before you begin, you should already have your website's goals and purposes in mind and have a clear idea of what content is going to be on each page.


First, come up with the basic sections of your site - header, main content, footer, etc\
- Header: global nav, logo, name, search bar, social media links, ...
- Main content: largest part and where users find value
- Footer: copyright, contact information, high-level sitemap, privacy policy, terms of use
- Sidebar: less important info, secondary set of navigation

Next, come up with what the main section content should contain:
- product image at top with CTA
- product desc
- product highlights with links to prod info pages
- testimonials
- Product returns and exchange information

## Grid

Before you start organizing your content, you need to establish a grid structure:
- focus on cols and how they align to create invisible, hard edges down the page that give a sense of organization
  - Even pages that don't have the same content share column dimensions and ratios
- rows depend on content

### Grid dimensions

The more columns you have, the more flexible your grid:
- remember that column widths =/= columns of content. A col of content can span multiple grid columns

1. Define the max width of the website content
   - most common widths for desktop screens are 960px and 1200px
2. Choose the number of cols for your grid system, and the gap between the cols. Most common numbers:
   - 12 for desktop
   - 8 for tablet
   - 4 for mobile

Rarely will you design rows for the grid--your content defines the rows.


## Layouts

Different pages will have unique content and goals
- elements like header and footer are the same throughout the site
- main content is unique and determined by how you want to engage your users

### One-column patterns

Good for blogs and articles--pages with lots of text content:
- decide how many grid cols will make up your max-width
- select an even number of cols to span so the left and right page margins are equal
- experiment with the size of elements that you want to emphasize. for example, have an image at the top span more grid cols than the text

### Multi-column patterns

On the homepage, mixing how many columns each subsection has is more common than on subpages:
- sections on a homepage is like a snippet that provides a preview of what the site has to offer before going to the longform subpage
- different col numbers for the homepage sections is a way to divide up sections and establish a relationship between content
- each type of subpage should have the same structure throughtout the site. For example, don't have multiple layouts for products
- header and footer should be consistent throughout the site

### Modular and masonry grid layouts

Modular layout: Has strict row height sizes

Masonry layout: like pintrest--multiple columns and rows, but the rows are not a fixed height
- fills up the vertical space between each block of content
- good for displaying cards, like lists of blog posts

## Reading patterns

Most common are the Z- and F-patterns

Nielsen Norman Group (NNG) has done a lot of research on [eye-tracking and reading patterns](https://www.nngroup.com/topic/eyetracking/)
- can help you make design decisions based on how users typically scan a page
- think about your layout in terms of how the culture reads: left to right, top to bottom

### Z-Pattern

This is how a user scans a page with minimal text content:
- uses an actual Z shape from top-left to bottom-right
- common for home pages and landing pages
- if you are struggling to define the page hierarchy, overlay a Z on the page
- use this recursively for each section on the page

### F-Pattern

The most commonly-observed reading pattern:
- how users scan pages with lots of text
  - trying to figure out what the content is about and get the important bits and pieces
- This pattern is not desirable, and it can help you determine if you chunked your content appropriately
  - can you add links?
  - bold things?
  - create a list?
  - create a subsection?
- each scan gets shorter as they go down the page

Used on pages or sections with these characteristics:
- text heavy without visual treatments that break up the wall of text, like subsections or subheadings
- users scan this way when they are being efficient
- when they aren't engaged enough to read every word

## Space

Whitespace (or negative space) makes the content more consumable and user friendly:
- creates clear focal points
- defines relationships between content on a page with proximity
- keeps UI clutter free
- Directs users through the page and is not overwhelming
- Can create a feeling of elegance. Think of high-end stores where items are displayed to showcase each piece, not crammed together on a shelf or rack

## Responsive design

Much of today's web browsing happens on mobile devices.

### Mobile design

The goals of mobile users is different than desktop--mobile users want answers quickly:
- keep interface minimal and focused--not a lot of clutter
  - hide elemnts or create a different way to serve them up
  - show only most important info here
  - info should be scannable
  - ex: hide nav behind hamburger menu
- think about how resources load on a mobile, like embedded videos. Can you replace the video with an image?

#### Design for touch

Users touch their screens:
- touch targets need to be much bigger with ample space between them
- buttons should be larger
- consider how people hold the phone with one hand--think about 
  - thumb zones--what is natural, stretch, and hard to reach
  - ex: put a "back to top" button at the bottom for easy reach
- DO NOT change the visual styles between devices (colors, type, etc). This messes up your brand

Chunk content to make it more manageable:
- break payment forms and shipping info into smaller chunks