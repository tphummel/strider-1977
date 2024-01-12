---
title: Analyze All Journals
date: 2024-01-12T14:10:00.000-08:00
toc: false
---

# Intro

Analyze all journal entries and enrich each entry metadata.

# Usage

```
./node_modules/.bin/zx content/t/analyze-enrich.md
```

# Code

```js
import fs from 'fs/promises'
import path from 'path'
import matter from 'gray-matter'
import yaml from 'js-yaml'
import Sentiment from 'sentiment'
import { franc } from 'franc-min'
import sw from 'stopword'
import natural from 'natural'
const { TfIdf } = natural
const sentiment = new Sentiment()

const processTextIntoMeta = (text) => {

  const wordCount = text.split(' ').length

  const sentimentResult = sentiment.analyze(text)

  // Guess the language of the text and remove the stop words of that language
  const textLang = franc(text)
  const stopWords = sw[textLang]
  const textWithoutStopWords = sw.removeStopwords(text.split(' '), stopWords).join(' ')

  const TfIdf = natural.TfIdf
  const tfidf = new TfIdf()

  tfidf.addDocument(textWithoutStopWords)

  const keywords = []

  tfidf.listTerms(0 /* in document index */).forEach(function (item) {
    if (item.tfidf > 1) {
      keywords.push(item.term)
    }
  })

  return {
    wordCount,
    keywords,
    sentiment: {
      score: sentimentResult.score,
      comparative: sentimentResult.comparative
    },
  }
}

async function processFiles() {
  // directory path
  const directoryPath = path.join(process.cwd(), 'content/j')
  
  const dirFiles = await fs.readdir(directoryPath)

  for (const file of dirFiles) {
    if(path.extname(file) !== '.md') {
      const filePath = path.join(directoryPath, file)

      // read markdown file as string
      const markdownContent = await fs.readFile(filePath, 'utf8')

      // process existing frontmatter and content
      const doc = matter(markdownContent)

      // process markdown text content
      const meta = processTextIntoMeta(doc.content)

      // merge with existing frontMatter
      const mergedData = {
        ...doc.data,
        ...meta
      }

      const newFrontMatter = '---\n' + yaml.dump(mergedData) + '---\n'

      // write the new frontmatter and the original content back into the markdown file
      await fs.writeFile(filePath, newFrontMatter + doc.content)

      console.log(`File processed: ${file}`)
    }
  }
}

processFiles()
```
