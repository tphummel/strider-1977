---
title: Trail Journal
date: 2023-08-02T14:10:00.000-08:00
toc: false
---

# Intro

Load trail journal entries

# Usage

```
VCR_MODE=cache ./node_modules/.bin/zx content/t/trail-journal.md
```

# Code

```js
import fetch from 'fetch-vcr'
import cheerio from 'cheerio'
import url from 'node:url'

const cssQueries = {
  content: 'body > div.container.body-content > div.flex-row.row > div.col-sm-8.site-main > div.panel.panel-default.panel-entry > div.panel-body > div > div',
  date: 'body > div.container.body-content > div.flex-row.row > div.col-sm-8.site-main > div.panel.panel-default.panel-entry > div.panel-heading > div > div.col-sm-12.entry-text-center.entry-date',
  destination: 'body > div.container.body-content > div.flex-row.row > div.col-sm-8.site-main > div.panel.panel-default.panel-entry > div.panel-heading > div.visible-xs > div:nth-child(1) > div:nth-child(1) > span.entry-text-detail',
  start: 'body > div.container.body-content > div.flex-row.row > div.col-sm-8.site-main > div.panel.panel-default.panel-entry > div.panel-heading > div.visible-xs > div:nth-child(1) > div:nth-child(2) > span.entry-text-detail',
  todayMiles: 'body > div.container.body-content > div.flex-row.row > div.col-sm-8.site-main > div.panel.panel-default.panel-entry > div.panel-heading > div.visible-xs > div:nth-child(2) > div:nth-child(1) > span.entry-text-detail',
  tripMiles: 'body > div.container.body-content > div.flex-row.row > div.col-sm-8.site-main > div.panel.panel-default.panel-entry > div.panel-heading > div.visible-xs > div:nth-child(2) > div:nth-child(2) > span.entry-text-detail'
}

async function fetchEntry(title, url) {
  try {
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error(`Network response was not ok ${response.statusText}`)
    }
    const html = await response.text()
    let $ = cheerio.load(html)

    const result = {}

    for (let [field, query] of Object.entries(cssQueries)) {
      result[field] = $(query).text().trim()
    }

    return result
  } catch (error) {
    console.error(`Fetch operation failed: ${error}`)
    return null
  }
}

const convertDateFormat = (inputDate) => {
  const date = new Date(inputDate)
  const year = date.getFullYear()

  // JavaScript's getMonth() function starts with 0 (0 = January), we add 1 to get a human-readable month
  const month = ("0" + (date.getMonth() + 1)).slice(-2)

  // Ensure the day is always two digits
  const day = ("0" + date.getDate()).slice(-2)

  return `${year}-${month}-${day}`
}

const allEntries = await fs.readJson('./entries.json')

for (const {location, url} of allEntries) {
  console.log(location, url)
  const entry = await fetchEntry(location, url)

  const parsedUrl = new URL(url)
  const splitPathname = parsedUrl.pathname.split('/')
  const entryId = splitPathname.pop()

  const yaml = `---
title: ${location}
date: ${convertDateFormat(entry.date)}
originalDate: ${entry.date}
trailJournalUrl: ${url}
trailJournalId: ${entryId}
dayStart: ${entry.start}
dayDestination: ${entry.destination}
todayMiles: ${entry.todayMiles}
tripMiles: ${entry.tripMiles}
---
${entry.content}
`
  fs.writeFileSync(`content/j/${entryId}.md`, yaml)
}
```
