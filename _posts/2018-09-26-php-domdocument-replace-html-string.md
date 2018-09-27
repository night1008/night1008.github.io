---
layout: post
keywords: php, domdocument, replace, domelement, child, html string
description: php domdocument replace domelement child with html string
title: php domdocument替换html string
comments: true
---

laravel项目中，想要在后端进行html内容的替换，场景如下，

有一段html, 想替换其中的在p标签内的a链接为其他的块元素，

```php
use Symfony\Component\DomCrawler\Crawler;

$content = '<h1>h1</h1>'. '<p><a href="">test link</a></p>'. '<p><a href="">test link</a></p>';
$crawler = new Crawler($content);
$nodes = $crawler->filter('a');
foreach ($nodes as $node) {
    $fragment = $node->ownerDocument->createDocumentFragment();
    $fragment->appendXML('<div>replace<span>replace</span></div>');
    // $node->parentNode->replaceChild($fragment, $node);
    $parent = $node->parentNode;
    while ($parent->nodeName != 'p') {
        $parent = $parent->parentNode;
    }
    $parent->parentNode->replaceChild($fragment, $parent);
}
$html = $crawler->filter('body')->html();
dd($html);
```



