---
title: "Web Development"
linkTitle: "Web Dev"
# weight: 30
description: >
  Notes on web development, including HTML, CSS, and Javascript.
---

```bash
web-app/
│── src/
│   ├── assets/
│   │   ├── images/          # Store image assets
│   │   ├── fonts/           # Custom fonts
│   │   ├── icons/           # SVG or icon fonts
│   ├── styles/
│   │   ├── main.scss        # Main Sass entry point
│   │   ├── components/      # Sass partials for UI components
│   │   ├── layouts/         # Sass files for layouts
│   │   ├── variables.scss   # Global Sass variables
│   │   ├── mixins.scss      # Reusable mixins
│   ├── scripts/
│   │   ├── main.js          # Main JavaScript entry point
│   │   ├── components/      # JS modules for UI components
│   │   ├── utils/           # Helper functions
│   ├── pages/
│   │   ├── index.html       # Main HTML file
│   │   ├── about.html       # Example subpage
│── dist/                    # Compiled and optimized files (after build)
│── public/                   # Static assets like favicons
│── .gitignore                # Ignore node_modules, build files, etc.
│── package.json              # Dependencies and scripts
│── webpack.config.js         # Webpack configuration (if using Webpack)
│── README.md                 # Project documentation
```

- `src`/: The main source directory where development files reside.
- `dist`/: The output folder for built files after running the build process.
- `styles`/: Contains the Sass files, organized into separate folders for structure.
- `scripts`/: Organized with separate folders for better maintainability.
- `pages`/: Houses HTML files for different pages of the site.
- `public`/: Stores static assets that don’t need processing.