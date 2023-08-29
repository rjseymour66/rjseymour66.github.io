---
title: "Configuration"
linkTitle: "Configuration"
weight: 50
description: >
  How to configure environments.
---

## Environments

Web development requires that you have different development and production environments, with different settings for each. For example, dev environments should not use minification so you can debug easily, and testing environments should have more logging. A standard set of environments in a large project can include the following:
- Development: Where active development begins. Individual branches are merged to this environment.
- Testing: Where you can execute automated and manual tests.
- Staging/Alpha: Stable, for testing across teams in your organization.
- Beta: A public prerelease that is ready to go live.
- Production: Live environment that is served to users.

When you run `hugo serve[r]`, it defaults to the development environment. The `hugo build` command defaults to the production environment.

### Directory structure

Create a `config` directory, and then create a directory for each of your environments:

```shell
config/
├── _default
│   ├── author.yaml
│   ├── config.yaml
│   ├── markup.yaml
│   ├── menu.yaml
│   └── params.yaml
├── development
│   └── config.yaml
└── production
    └── config.yaml
```

Files in the `development` and `production` directories store values that override those defined in files in the `_default` directory only. The `_default` directory holds the primary configuration values.

#### _default directory

Hugo looks in the `_default/` directory for the defaults. You can store everything in the `_default/config.yaml` file, but you can also split it up into separate files for easier management. The filenames should share names with the configuration sections that they hold. For example, the `author:` configuration section is now in the `_default/author.yaml` file.

Because the name of the file identifies which configuration parameters the file contains, you do not need to add the top level value when you add values to the file. For example, below is the contents of the `menu.yaml` file:

```yaml
main:
  - identifier: about
    name: About
    url: /about
    weight: 100
  - identifier: contact
    name: Contact
    url: /contact
    weight: 200
```

If these settings were still in the `_default/config.yaml` file, it would require the `menu` key at the beginning:

```yaml
menu:
  main:
    - identifier: about
    ...
```

### Menus

Hugo provides menus that you can populate with the `menu` configuration field. You can define these menus in the main `_default/menu.yaml` file, or in the frontmatter of the content page that the menu link leads to.

If the menu item links to internal files, and you might delete the page in the future, you should place the menu configuration in the page frontmatter. Otherwise, maintain it in the `_default/menu.yaml` file.

The following example defines a `main` menu (across the top of the navbar) and `footer`:

```yaml
main:
  - name: Blog
    identifier: blog
    weight: 110
  - name: Community
    parent: blog
footer:
  name: Blog
  weight: 100
```
Notice that the `main` menu is an array of menu items, where the `Community` item is a child of the `Blog` item. This makes `Community` a subheading, which is available when you hover over `Blog`. Hugo uses the `identifier` field as a unique value that Hugo references when working with menu items.

## Page bundles

Page bundles group associated content files in an isolated directory structure. The isolation helps to make the content more easily maintained---you can add or remove content without affecting the rest of the site, and it makes the content more modular so that individuals can contribute without interfering with each other. There are three kinds of bundles:
- Branch
- Leaf
- Headless

### Branch bundles

A _page bundle_ is a collection of resources such as text files, images, and PDFs, that represent one or more pages of related content---a section of a website.

#### Create a branch bundle

To create a branch bundle, complete the following:
1. Create a folder in the directory location where you want the section.
2. In the directory, create a file named `_index.md` that contains the content.


### Leaf bundles

A _leaf bundle_ is also a collection of resources, but the resources represent a single web page.

#### Create a leaf bundle

To create a leaf bundle, complete the following:
1. Create a folder in the directory location where you want the page, and name the directory the page name.
2. In the directory, create a file named `index.md` that contains the content.

You can store multiple files and content pages in the leaf bundle directory, but Hugo only renders the `index.md` page.


> The `index.md` file represents a single web page. `_index.md` represents a section's root.

## Taxonomies

_Taxonomies_ group pages and describe relationships between the pages. They have two parts:
- _taxonomy list_: 

## Helpful settings

```yaml

# _default
enableGitInfo: true

# development
params:
  DebugMenu: true 
```