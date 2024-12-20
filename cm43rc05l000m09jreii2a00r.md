---
title: "使用github action定时同步obsidian内容到astro博客仓库"
seoTitle: "Automate Obsidian to Astro Blog Sync"
seoDescription: "自动同步Obsidian内容至Astro博客仓库的GitHub Action指南。"
datePublished: Sat Nov 30 2024 05:54:07 GMT+0000 (Coordinated Universal Time)
cuid: cm43rc05l000m09jreii2a00r
slug: synchronize-profiles-to-remote-and-local-on-a-regular-basis
canonical: https://blog.kazoottt.top/blog/synchronize-profiles-to-remote-and-local-on-a-regular-basis/
tags: github-actions-1, astro, obsidian

---

## github action

目标仓库以及具体目录可以根据自己的项目来定

```yaml
name: Update Docs

on:
  push:
    paths:
      - "content/**"
      - "scripts/build_docs.cjs"
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source repository
        uses: actions/checkout@v2
        with:
          token: ${{ secrets.PAT_TOKEN }}

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: "20"

      - name: Install dependencies
        run: |
          echo "Installing dependencies in source repository..."
          npm install
          echo "Dependencies installed successfully"

      - name: Build docs
        run: |
          echo "Starting docs build..."
          npm run build:docs:server
          echo "Docs built successfully"

      - name: Checkout target repository
        uses: actions/checkout@v2
        with:
          repository: kazoottt/kazoottt-blog
          path: kazoottt-blog
          token: ${{ secrets.PAT_TOKEN }}

      - name: Copy built docs to target repository
        run: |
          echo "Starting copy process..."
          echo "Content of astroContent before copy:"
          cp -rv astroContent/* kazoottt-blog/src/content/post/
          echo "Content of astroContent copy done"

      - name: Setup target repository
        run: |
          cd kazoottt-blog
          echo "Configuring git..."
          git config --global user.name 'github-actions[bot]'
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'
          echo "Installing dependencies in target repository..."
          npm install
          echo "Running sort..."
          npm run sort

      - name: Check for changes and commit
        run: |
          cd kazoottt-blog
          if [[ -n $(git status -s) ]]; then
            git add .
            git commit -m "Update docs and sort content"
            git push
          else
            echo "No changes to commit"
          fi
        env:
          GITHUB_TOKEN: ${{ secrets.PAT_TOKEN }}
```

PAT\_TOKEN申请  
[Sign in to GitHub · GitHub](https://github.com/settings/tokens)

需要权限：repo, workflow

![image.png](https://pictures.kazoottt.top/2024/11/20241129-d27ae43fa5ad1344d66dff2e9d79c70e.png align="left")

## obsidian模板

published是true且notAstro不为true的时候，会被发布到另外一个仓库

```plaintext
---
title: {{title}}
date: {{date}}
author: KazooTTT
type: Post
status: Published
tags: []
finished: false
published: false
category: 
slug:
description: 
notAstro:
---
```

## 脚本 scripts/build\_docs.cjs

依赖安装

```plaintext
npm install gray-matter
```

可以配置一些不想同步的文件夹

```js
const fs = require("fs").promises
const fsSync = require("fs")
const path = require("path")
const matter = require("gray-matter")
const { execSync } = require("child_process")

const CONFIG = {
  outputDir: "astroContent",
  inputDir: "./content",
  ignoreList: [
    ".github",
    ".obsidian",
    "草稿箱",
    "模板",
    "attachment",
    "记录",
    "导航用",
    "微信读书",
  ],
  validExtensions: [".md"],
  // Local specific config
  localDestinationRoot: "", // to edit
  localDestination: "", // to edit
}

/**
 * Ensures output directory exists
 */
function initializeOutputDir() {
  if (!fsSync.existsSync(CONFIG.outputDir)) {
    fsSync.mkdirSync(CONFIG.outputDir, { recursive: true })
  }
}

/**
 * Validates if a file should be processed based on its metadata
 */
function shouldProcessFile(frontMatter) {
  return frontMatter.published === true && !frontMatter.notAstro
}

/**
 * Processes a single markdown file
 */
async function processMarkdownFile(fullPath, outputPath) {
  try {
    const fileContent = await fs.readFile(fullPath, "utf8")
    const { data } = matter(fileContent)

    if (shouldProcessFile(data)) {
      await fs.copyFile(fullPath, outputPath)
      console.log(`✓ Copied: ${path.relative(CONFIG.inputDir, fullPath)}`)
    }
  } catch (error) {
    console.error(`Error processing file ${fullPath}:`, error.message)
  }
}

/**
 * Recursively processes directories and files
 */
async function processDirectory(dir) {
  try {
    const files = await fs.readdir(dir)

    await Promise.all(
      files.map(async (file) => {
        const fullPath = path.join(dir, file)
        const relativePath = path.relative(CONFIG.inputDir, fullPath)
        const outputPath = path.join(CONFIG.outputDir, relativePath)

        const stats = await fs.stat(fullPath)

        if (stats.isDirectory()) {
          if (CONFIG.ignoreList.includes(file)) return
          await fs.mkdir(outputPath, { recursive: true })
          await processDirectory(fullPath)
        } else if (path.extname(file) === ".md" && file !== "index.md") {
          await processMarkdownFile(fullPath, outputPath)
        }
      }),
    )
  } catch (error) {
    console.error(`Error processing directory ${dir}:`, error.message)
  }
}

/**
 * Copies processed files to final destination and commits changes
 */
async function copyToLocalDestination() {
  try {
    // Remove existing content
    if (fsSync.existsSync(CONFIG.localDestination)) {
      await fs.rm(CONFIG.localDestination, { recursive: true })
    }

    // Create destination directory
    await fs.mkdir(CONFIG.localDestination, { recursive: true })

    // Copy files
    const files = await fs.readdir(CONFIG.outputDir)
    await Promise.all(
      files.map(async (file) => {
        const sourcePath = path.join(CONFIG.outputDir, file)
        const destPath = path.join(CONFIG.localDestination, file)
        await fs.rename(sourcePath, destPath)
      }),
    )

    console.log("Content has been successfully copied to the local destination.")

    // Commit changes
    execSync(`cd ${path.dirname(CONFIG.localDestination)} && npm run sort`)
    execSync(`git add ${CONFIG.localDestination} && git commit -m "update content"`)
    // push changes
    // execSync(`git push`)
  } catch (error) {
    console.error("Error copying to destination:", error.message)
  }
}

/**
 * Cleans up the temporary output directory
 */
async function cleanupOutputDir() {
  try {
    if (fsSync.existsSync(CONFIG.outputDir)) {
      await fs.rm(CONFIG.outputDir, { recursive: true })
    }
    console.log("Cleaned up temporary directory.")
  } catch (error) {
    console.error("Error cleaning up:", error.message)
  }
}

/**
 * Main execution
 */
async function main() {
  try {
    const isLocalBuild = process.argv.includes("--local")
    console.log(`Starting document processing... (${isLocalBuild ? "local" : "server"} build)`)

    initializeOutputDir()
    await processDirectory(CONFIG.inputDir)

    if (isLocalBuild) {
      await copyToLocalDestination()
      await cleanupOutputDir()
    }

    console.log("Document processing completed successfully!")
  } catch (error) {
    console.error("Fatal error:", error.message)
  }
}

main()
```