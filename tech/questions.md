# 问题库

## 什么是Source Map?

A source map is a file that maps from the transformed source to the original source, enabling the browser to reconstruct the original source and present the reconstructed original in the debugger.

To enable the debugger to work with a source map, you must:
- generate the source map
- include a comment in the transformed file, that points to the source map. The comment's syntax is like this:
```
//# sourceMappingURL=http://example.com/path/to/your/sourcemap.map
```

生产环境下，我们会通过压缩JS、CSS等文件来优化程序，通常压缩后的代码没有换行符。那么如果程序生产环境下出错，比如，打开浏览器，我们看到一个JS报错信息，又如何定位错误所在的程序行呢？source map是它的解决方案。source map不仅限于压缩文件中使用，它是通用的将最终文件代码行映射到源文件代码行的工具文件，例如将CoffeeScript生成的JavaScript文件映射到CoffeeScript源码。

Source map是后缀名通常是.map的JSON文件，包含了最终代码对应源码位置的信息，帮助我们既能优化源码还能顺利debug。也就是说，source maps是可以供浏览器使用的帮助debug的文件。

Chrome和Firefox开发者工具都对source map文件提供了内部支持。在Chrome控制台的setting中可以设置是否支持js/css source map。

通过在最终代码行末添加`//# sourceMappingURL=http://example.com/path/to/your/sourcemap.map`，在浏览器控制台打开的情况下，浏览器看到这行代码就会加载url指向的map文件。

**资料**

* [WTF is a Source Map](https://www.schneems.com/2017/11/14/wtf-is-a-source-map/)
* [Use a source map(MDN)](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map)

## 什么是sitemaps？

A sitemap is a file where you **provide information about the pages, videos, and other files on your site, and the relationships between them. Search engines like Google read this file to more intelligently crawl your site.**

A sitemap tells the crawler which files you think are important in your site, and also provides valuable information about these files: for example, for pages, when the page was last updated, how often the page is changed, and any alternate language versions of a page.

You can use a sitemap to provide information about specific types of content on your pages, including video and image content. For example:

- A sitemap video entry can specify the video running time, category, and age appropriateness rating.
- A sitemap image entry can include the image subject matter, type, and license.

**Do I need a sitemap?**

If your site’s pages are properly linked, our web crawlers can usually discover most of your site. Even so, a sitemap can improve the crawling of your site, particularly if your site meets one of the following criteria:

- Your site is really large. As a result, it’s more likely Google web crawlers might overlook crawling some of your new or recently updated pages.
- Your site has a large archive of content pages that are isolated or not well linked to each other. If your site pages do not naturally reference each other, you can list them in a sitemap to ensure that Google does not overlook some of your pages.
- Your site is new and has few external links to it. Googlebot and other web crawlers crawl the web by following links from one page to another. As a result, Google might not discover your pages if no other sites link to them.
- Your site uses rich media content, is shown in Google News, or uses other sitemaps-compatible annotations. Google can take additional information from sitemaps into account for search, where appropriate.

Using a sitemap doesn't guarantee that all the items in your sitemap will be crawled and indexed, as Google processes rely on complex algorithms to schedule crawling. However, in most cases, your site will benefit from having a sitemap, and you'll never be penalized for having one.

**资料**

* [Learn about sitemaps(Google Support)](https://support.google.com/webmasters/answer/156184?hl=en)

## 什么是robots.txt?

A robots.txt file tells search engine crawlers which pages or files the crawler can or can't request from your site. This is used mainly to avoid overloading your site with requests; **it is not a mechanism for keeping a web page out of Google.** To keep a web page out of Google, you should use noindex tags or directives, or password-protect your page.robots.txt is used primarily to **manage crawler traffic to your site**, and occasionally to keep a page off Google, depending on the file type。

**资料**
* [robots.txt示例](https://stackoverflow.com/robots.txt)
* [Learn about robots.txt files(Google Support)](https://support.google.com/webmasters/answer/6062608?hl=en&ref_topic=6061961)