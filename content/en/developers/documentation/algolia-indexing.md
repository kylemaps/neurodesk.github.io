---
title: "Indexing with Algolia"
linkTitle: "Indexing with Algolia"
weight: 1
description: >
  How to index the Neurodesk documentation with Algolia
---

## 1. Prerequisites
- [Algolia account](https://www.algolia.com/)

## 2. Create a crawler to index the documentation
1. In the Algolia Overview page, click on "Go To Crawlers".
2. Click on "Add new crawler".
3. Name your crawler (e.g., `neurodesk-docs`).
4. Set the "Start URL" to your documentation URL (e.g., `https://neurodesk.org/`).
5. Click "Create".

## 3. Configure the crawler
1. In the crawler settings, go to the "Editor" section under "SETUP".
3. You can specify how the content should be indexed. It can override the default settings.
{{< alert color="warning" >}}Make sure to configure `startUrls`, `discoveryPatterns`, `pathsToMatch` and other relevant settings to match the base URL for indexing.{{< /alert >}}
4. Run the crawler by clicking on "Run Test" in the top right corner.
5. Once the test is successful, click on "Review and Publish" to start indexing the documentation.

## 4. Create a new Algolia index
1. Log in to your Algolia account.
2. Click on "Indices" in the left sidebar.
3. Click on "Create index".
4. Name your index (e.g., `neurodesk-docs`).
5. Click "Create".

## 5. Configure the Algolia index
1. In the index settings, go to the "Searchable attributes" section.
2. Add attributes you want to be searchable, eg. `title`, `content`.
3. In the "Ranking" section, set the ranking criteria according to your needs.
4. Save the settings.

