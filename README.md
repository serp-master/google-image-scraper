<h1 align="center">
Google Image Scraper: How To Scrape Google Images With JavaScript
</h1>

## Table of Contents 

- [Installation](#installation)
- [Example](#example)
  - [Results](#results)
- [Usage](#usage)
  - [Result](#result)
- [Contributions](#contributions) 

You may find yourself needing to learn or train a machine learning model for image classification, clustering, or detection algorithms. To test these models, you’ll need data. That may lead to searching for particular images on Google. But you’ll need hundreds of these data to train a machine learning model. A good solution will be to use an image scraper to scrape the Google images.

In this tutorial, you’ll learn how to scrape and download Google images using JavaScript.

## Installation 

To set up your project environments and dependencies, first, you need to create a folder to hold the Google image scraper project. Once that’s done, let’s initialize npm by running the command.

```npm init```

After this, install the Google scraper tool--[images-scraper](https://www.npmjs.com/package/images-scraper) and all other required node packages for this project.

```$ yarn add images-scraper node-fetch```

## Example

After installing the scraper package, let’s take a look at some examples. Here, we will attempt to scrape Google images for the first 10 results that match the keyword “Joe Biden”. 

```
var Scraper = require('images-scraper');

const google = new Scraper({
  puppeteer: {
    headless: false,
  },
});

(async () => {
  const results = await google.scrape('Joe Biden', 5);
  console.log('results', results);
})();
``` 

Let’s break down what’s happening here. To start with, import the images scraper. The images-scraper uses Puppeteer to scrape Google images.
 
```var Scraper = require('images-scraper');```

Then, we create an instance of the Scraper package with the next block of code. 
Finally, we run the scrape function with the scrape() method. The two arguments in these methods are keyword & expected number of image search results. 

NOTE: You may run into a timeout error on this package. Don’t worry, you’re not the first. The issue has been raised [on the GitHub repo](https://github.com/pevers/images-scraper/issues/84) but remains open. Till then, you can temporarily fix this by digging into the npm package and locating the file: node_modules/images-scraper/src/google/scraper.js. 

Then, go to the “_scrapePage function” and add “page.setDefaultNavigationTimeout(0);” after “const page = await this.browser.newPage();”. This action will help turn off the timeout error completely.

## Results

```
[
    {
        "url": "https://www.whitehouse.gov/wp-content/uploads/2021/04/P20210303AS-1901-cropped.jpg",
        "source": "https://www.whitehouse.gov/administration/president-biden/",
        "title": "Joe Biden: The President | The White House"
    },
    {
        "url": "https://upload.wikimedia.org/wikipedia/commons/6/68/Joe_Biden_presidential_portrait.jpg",
        "source": "https://en.wikipedia.org/wiki/Joe_Biden",
        "title": "Joe Biden - Wikipedia"
    },
    {
        "url": "https://www.biography.com/.image/ar_1:1%2Cc_fill%2Ccs_srgb%2Cfl_progressive%2Cq_auto:good%2Cw_1200/MTc0NjMzNDczMzM0NTg1MzM0/gettyimages-465470375.jpg",
        "source": "https://www.biography.com/us-president/joe-biden",
        "title": "Joe Biden - Age, Politics & Family - Biography"
    },
    {
        "url": "https://www.history.com/.image/ar_4:3%2Cc_fill%2Ccs_srgb%2Cfl_progressive%2Cq_auto:good%2Cw_1200/MTc2MzAyNDY4NjM0NzgwODQ1/joe-biden-gettyimages-1267438366.jpg",
        "source": "https://www.history.com/topics/us-presidents/joe-biden",
        "title": "Joe Biden: Age, Presidency, Family - HISTORY"
    },
    {
        "url": "https://media-cldnry.s-nbcnews.com/image/upload/t_fit-1500w,f_auto,q_auto:best/newscms/2020_45/3425385/201103-joe-biden-cs-124p.jpg",
        "source": "https://www.nbcnews.com/politics/2020-election/biden-defeats-trump-win-white-house-nbc-news-projects-n1246912",
        "title": "Biden defeats Trump to win White House, NBC News projects"
    }
]
```

## Usage 

You may search Google Images as a way to obtain access to a host of image data for training an image recognition algorithm or some other particular purpose. Because of this, we’ll attempt to save the image results in a local folder. 

The SaveImage function acts as a Google image downloader. Then, it gets all the image results, indexes and stores them individually in the folder.

```
/**
 * This function saves individual images to the images folder
 * @param {string} url image url
 * @param {string} filename name to save the image with
 */
async function saveImage(url, fileName) {
    try {
        console.log(`extracting and saving ${fileName}.jpg to file...`);
        const res = await fetch(url);
        const filePath = `./images/${trimText(fileName)}.jpg`;
        const dest = fs.createWriteStream(filePath);
        res.body.pipe(dest)

        console.log(`image ${trimText(fileName)}.jpg saved!`)
    } catch (error) {
        console.log(`image ${fileName} fetched from ${url} - failed!`)
        console.log(error.message);
    }
}
```

In addition to this, you can set up the images scraper to parse arguments from the CLI directly as the keyword. However, you still want to have the opportunity to make changes to the keyword within the code. You can achieve that with the following line of command.

```const keyword = process.argv[2]||'apple';```

The complete code for this project is available below.
 
```
import fs from 'fs';
import Scraper from 'images-scraper';
import fetch from 'node-fetch';

if (!globalThis.fetch) {
    globalThis.fetch = fetch;
}

const google = new Scraper({ // initialize
    puppeteer: {
        timeout: 120000
    }
})

const keyword = process.argv[2]||'apple';
const expectedResultCount = 5;

/**
 * Scrape for images link
 * @param {string} keyword image keyword to search for
 * @returns void
 */
async function ScrapeImg(keyword) {
    try {
        const result = await google.scrape(keyword, expectedResultCount);
        let data = JSON.stringify(result, null, 4);
        fs.writeFileSync(`imagesLink/${trimText(keyword)}.json`, data);
        result.map((img, idx) => {
            saveImage(img.url, `${keyword}-${idx}`); // joining the keyword with the image index (idx) to avoid overwrite when is getting saved
        })
        console.log(`Result saved to imagesLink/${trimText(keyword)}.json`);
        return;
    } catch (error) {
        console.log(error);
        process.exitCode = 1;
        process.exit();
    }
}

/**
 * This function saves individual images to the images folder
 * @param {string} url image url
 * @param {string} filename name to save the image with
 */
async function saveImage(url, fileName) {
    try {
        console.log(`extracting and saving ${fileName}.jpg to file...`);
        const res = await fetch(url);
        const filePath = `./images/${trimText(fileName)}.jpg`;
        const dest = fs.createWriteStream(filePath);
        res.body.pipe(dest)
        console.log(`image ${trimText(fileName)}.jpg saved!`)
    } catch (error) {
        console.log(`image ${fileName} fetched from ${url} - failed!`)
        console.log(error.message);
    }
}

function trimText(text) {
    return text.replace(/\s/g, '').toLowerCase();
}

ScrapeImg(keyword);
```

The image scraper begins with the puppeteer timeout set to give you as much time as possible for the Google image download. Then you have the option to log the arguments in the command line or search for the keyword "apple". 


The ScrapeImg function takes in the keyword argument. The result object from the keyword search is converted into a JSON readable format and saved in a folder inside the project. The saveImg function downloads a JPG format of each scraped image and stores it by joining the keyword with the image index (idx) to avoid overwriting.

## Result

The final result from the image scraper saves:
- A JSON format containing the image URL, image source, and page title
- JPG versions of each Google image scraped

## Contributions
If you’re looking to contribute to this tutorial on Google images downloader, you can locate the examples in this GitHub repo.

