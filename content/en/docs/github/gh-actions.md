---
title: "GitHub actions"
weight: 30
description: >
  How to use GitHub Actions.
---

## Hugo smoke test

```yaml
name: build test

on: [push, pull_request]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v2

      - name: Hugo setup
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.99.1'
          extended: true

      - name: Node setup 
        uses: actions/setup-node@v3
        with:
          node-version: 16
          cache: 'npm'
          cache-dependency-path: '${{github.workspace}}/package-lock.json'
      - run: npm ci

      - name: Clean public directory
        run: rm -rf public

      - name: Build Hugo 
        run: hugo --minify
```

## Publish Hugo to GH pages

```yaml
name: github pages 

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v2

      - name: Hugo setup
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.99.1'
          extended: true

      - name: Node setup 
        uses: actions/setup-node@v3
        with:
          node-version: 16
          cache: 'npm'
          cache-dependency-path: '${{github.workspace}}/package-lock.json'
      - run: npm ci

      - name: Clean public directory
        run: rm -rf public

      - name: Build Hugo 
        run: hugo --minify

      # uncomment when cname is complete
      # - name: Create cname file
      #   run: echo 'www.example.com' > public/CNAME

      # push to gh-pages branch
      - name: Deploy 
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```