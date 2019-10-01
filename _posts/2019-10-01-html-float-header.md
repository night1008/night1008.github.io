---
layout: post
keywords: html float header, 页面顶栏滚动浮现
description: 实现页面顶栏滚动浮现
title: 实现页面顶栏滚动浮现
comments: true
---

页面滚动的时候，顶部的header能够替换成其他的内容，在 (Kevin Blog)[https://zhowkev.in/2017/02/28/ri-ben-de-ba-ge-shen-mei-yi-shi-chan-pin-she-ji-mei-xue-de-kua-jie-shu-dan/] 的博客里面看到不错的实现方式，
在此记录一下，留以后用。

```html
<header class="site-header outer">
    <div class="inner">
        <nav class="site-nav">
            <div class="site-nav-left">
                <a class="site-nav-logo" href="/">...</a>
                <ul class="nav" role="menu">
                    <li class="nav-producter-rang-chan-pin-cong-0-dao-1" role="menuitem"><a href="/">...</a></li>
                    <li class="nav-ios-kai-fa-shi-pin" role="menuitem"><a href="/">...</a></li>
                </ul>
            </div>
            <div class="site-nav-right">
                <div class="social-links">
                    <a class="social-link social-link-fb" href="/" title="Facebook" target="_blank" rel="noopener">...</a>
                    <a class="social-link social-link-tw" href="/" title="Twitter" target="_blank" rel="noopener">...</a>
                </div>
                <a class="rss-button" href="/" title="RSS" target="_blank" rel="noopener">...</a>
            </div>
        </nav>
    </div>
</header>

<div class="floating-header">
    <div class="floating-header-logo">
        <a href="/">
            <span>Blog</span>
        </a>
    </div>
    <span class="floating-header-divider">—</span>
    <div class="floating-header-title">...</div>
    <div class="floating-header-share">
        <div class="floating-header-share-label">...</div>
        <a class="floating-header-share-tw" href="/">...</a>
        <a class="floating-header-share-fb" href="/">...</a>
    </div>
    <progress id="reading-progress" class="progress" value="431" max="13924">
        <div class="progress-container">
            <span class="progress-bar"></span>
        </div>
    </progress>
</div>
```

```css
.floating-header {
    visibility: hidden;
    position: fixed;
    top: 0;
    right: 0;
    left: 0;
    z-index: 1000;
    display: flex;
    align-items: center;
    height: 60px;
    border-bottom: rgba(0,0,0,0.06) 1px solid;
    background: rgba(255,255,255,0.95);
    transition: all 500ms cubic-bezier(0.19, 1, 0.22, 1);
    transform: translate3d(0, -120%, 0);
}

.floating-active {
    visibility: visible;
    transition: all 500ms cubic-bezier(0.22, 1, 0.27, 1);
    transform: translate3d(0, 0, 0);
}

.floating-header-logo {
    overflow: hidden;
    margin: 0 0 0 20px;
    font-size: 1.6rem;
    line-height: 1em;
    letter-spacing: -1px;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.floating-header-logo a {
    display: flex;
    align-items: center;
    color: var(--darkgrey);
    line-height: 1.1em;
    font-weight: 700;
}

.floating-header-logo a:hover {
    text-decoration: none;
}

.floating-header-logo img {
    margin: 0 10px 0 0;
    max-height: 20px;
}

.floating-header-divider {
    margin: 0 5px;
    line-height: 1em;
}

.floating-header-title {
    flex: 1;
    overflow: hidden;
    margin: 0;
    color: #2e2e2e;
    font-size: 1.6rem;
    line-height: 1.3em;
    font-weight: bold;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.floating-header-share {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    padding-left: 2%;
    font-size: 1.3rem;
    line-height: 1;
}

.floating-header-share a {
    display: flex;
    justify-content: center;
    align-items: center;
}

.floating-header-share svg {
    width: auto;
    height: 16px;
    fill: #fff;
}

.floating-header-share-label {
    flex-shrink: 0;
    display: flex;
    align-items: center;
    margin-right: 10px;
    color: rgba(0,0,0,0.7);
    font-weight: 500;
}

.floating-header-share-label svg {
    margin: 0 5px 0 10px;
    width: 18px;
    height: 18px;
    stroke: rgba(0,0,0,0.7);
    transform: rotate(90deg);
}

.floating-header-share-tw,
.floating-header-share-fb {
    display: block;
    align-items: center;
    width: 60px;
    height: 60px;
    color: #fff;
    line-height: 48px;
    text-align: center;
    transition: all 500ms cubic-bezier(0.19, 1, 0.22, 1);
}

.floating-header-share-tw {
    background: #33b1ff;
}

.floating-header-share-fb {
    background: #005e99;
}

.progress {
    position: absolute;
    right: 0;
    bottom: -1px;
    left: 0;
    width: 100%;
    height: 2px;
    border: none;
    color: var(--blue);
    background: transparent;

    appearance: none;
}

.progress::-webkit-progress-bar {
    background-color: transparent;
}

.progress::-webkit-progress-value {
    background-color: var(--blue);
}

.progress::-moz-progress-bar {
    background-color: var(--blue);
}

.progress-container {
    position: absolute;
    top: 0;
    left: 0;
    display: block;
    width: 100%;
    height: 2px;
    background-color: transparent;
}

.progress-bar {
    display: block;
    width: 50%;
    height: inherit;
    background-color: var(--blue);
}

@media (max-width: 900px) {
    .floating-header {
        height: 40px;
    }
    .floating-header-title,
    .floating-header-logo {
        font-size: 1.5rem;
    }
    .floating-header-share-tw,
    .floating-header-share-fb {
        width: 40px;
        height: 40px;
        line-height: 38px;
    }
}

@media (max-width: 800px) {
    .floating-header-logo {
        margin-left: 10px;
    }
    .floating-header-logo a {
        color: #2e2e2e;
    }
    .floating-header-title,
    .floating-header-divider {
        visibility: hidden;
    }
}

@media (max-width: 450px) {
    .floating-header-share-label {
        display: none;
    }
}
```


```javascript
<script>
// NOTE: Scroll performance is poor in Safari
// - this appears to be due to the events firing much more slowly in Safari.
//   Dropping the scroll event and using only a raf loop results in smoother
//   scrolling but continuous processing even when not scrolling
$(document).ready(function () {
    // Start fitVids
    var $postContent = $(".post-full-content");
    $postContent.fitVids();
    // End fitVids

    var progressBar = document.querySelector('#reading-progress');
    var header = document.querySelector('.floating-header');
    var title = document.querySelector('.post-full-title');

    var lastScrollY = window.scrollY;
    var lastWindowHeight = window.innerHeight;
    var lastDocumentHeight = $(document).height();
    var ticking = false;

    function onScroll() {
        lastScrollY = window.scrollY;
        requestTick();
    }

    function onResize() {
        lastWindowHeight = window.innerHeight;
        lastDocumentHeight = $(document).height();
        requestTick();
    }

    function requestTick() {
        if (!ticking) {
            requestAnimationFrame(update);
        }
        ticking = true;
    }

    function update() {
        var trigger = title.getBoundingClientRect().top + window.scrollY;
        var triggerOffset = title.offsetHeight + 35;
        var progressMax = lastDocumentHeight - lastWindowHeight;

        // show/hide floating header
        if (lastScrollY >= trigger + triggerOffset) {
            header.classList.add('floating-active');
        } else {
            header.classList.remove('floating-active');
        }

        progressBar.setAttribute('max', progressMax);
        progressBar.setAttribute('value', lastScrollY);

        ticking = false;
    }

    window.addEventListener('scroll', onScroll, {passive: true});
    window.addEventListener('resize', onResize, false);

    update();

});
</script>
```