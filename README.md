# AMP ⚡ converter library

[![Build Status](https://travis-ci.com/magyarandras/amp-converter.svg?branch=master)](https://travis-ci.com/magyarandras/amp-converter)

A library to convert HTML articles, blog posts or similar content to [AMP (Accelerated Mobile Pages)](https://amp.dev).

**Note**: This library is not intended to convert entire HTML documents if you want to convert an entire page you should use a more advanced library, for example: [Lullabot/amp-library](https://github.com/Lullabot/amp-library/)

## Installation:

```
composer require magyarandras/amp-converter
```

## Currently supported elments:
* [x] amp-img
* [x] amp-img in amp-pan-zoom
* [x] amp-img with lightbox
* [x] amp-video
* [x] amp-audio
* [x] amp-iframe(A placeholder is automatically added to the iframes, so you can [embed iframes even above the fold](https://www.youtube.com/watch?v=TqfYmlkHVCs).)
* [x] amp-youtube
* [x] amp-facebook
* [x] amp-instagram
* [x] amp-twitter
* [x] amp-pinterest
* [x] amp-playbuzz
* [x] amp-gist(Github gist embed)
* [x] amp-vimeo
* [ ] amp-soundcloud(You can use amp-iframe instead)
* [x] amp-vk
* [x] amp-imgur
* [x] amp-dailymotion
* [x] amp-gfycat

## Usage:

Simple example:

**Make sure your HTML code doesn't contain tags or attributes invalid in HTML5 otherwise, the generated AMP will be invalid too.**

```php
<?php

use magyarandras\AMPConverter\Converter;

/*
If you have images with unknown dimension in your HTML code and you use relative URLs, you have to pass the base URL of the images to the constructor.

Examples:
$converter = new Converter('https://example.com');
$converter = new Converter('https://cdn.example.com');

*/
$converter = new Converter();

//Load built-in converters
$converter->loadDefaultConverters();

///HTML to convert

$html = '
<p><strong>This is a sample HTML code generated by TinyMCE.</strong></p>
<p><strong><img src="https://cdn.example.com/photo/example_960_720.jpg" alt="" width="960" height="640" /></strong></p>
<p><span style="font-family: \'times new roman\', times;">Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</span></p>
<p style="text-align: center;"><iframe src="https://www.youtube.com/embed/gYJ03GyrSrM" width="560" height="314" allowfullscreen="allowfullscreen"></iframe></p>
<p style="text-align: center;">&nbsp;</p>
<blockquote class="twitter-tweet">
<p dir="ltr" lang="en">Check out the latest post on the AMP Blog to learn how <a href="https://twitter.com/AdobeExpCloud?ref_src=twsrc%5Etfw">@AdobeExpCloud</a> has been working to seamless integrate AMP support into its applications ⚡<br /><br />Learn more here 👉 <a href="https://t.co/hX3QmJ707x">https://t.co/hX3QmJ707x</a></p>
&mdash; AMP Project (@AMPhtml) <a href="https://twitter.com/AMPhtml/status/1248666798901194753?ref_src=twsrc%5Etfw">April 10, 2020</a></blockquote>
';

//Convert html to amp html
$amphtml = $converter->convert($html);

//Get the necessary amp components
$amp_scripts = $converter->getScripts();

print_r($amphtml);
echo PHP_EOL;
print_r($amp_scripts);

```
The output:
```html
<p><strong>This is a sample HTML code generated by TinyMCE.</strong></p>
<p><strong><amp-img src="https://cdn.example.com/photo/example_960_720.jpg" width="960" height="640" alt layout="responsive"></amp-img></strong></p>
<p><span>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</span></p>
<p><amp-youtube data-videoid="gYJ03GyrSrM" width="480" height="270" layout="responsive"></amp-youtube></p>
<p>&nbsp;</p>
<amp-twitter width="375" height="472" layout="responsive" data-tweetid="1248666798901194753"></amp-twitter>
```

```
Array
(
    [0] => <script async custom-element="amp-youtube" src="https://cdn.ampproject.org/v0/amp-youtube-0.1.js"></script>
    [1] => <script async custom-element="amp-twitter" src="https://cdn.ampproject.org/v0/amp-twitter-0.1.js"></script>
)
```

You can specify which converters to use by loading them manually:

```php
<?php

use magyarandras\AMPConverter\Converter;

use magyarandras\AMPConverter\TagConverter\AMPImgZoom;
use magyarandras\AMPConverter\TagConverter\AMPYoutube;

//PHP 7+ syntax:
//use magyarandras\AMPConverter\TagConverter\{AMPImgZoom,AMPYoutube}

$converter = new Converter();

$converter->addConverter(new AMPImgZoom());
$converter->addConverter(new AMPYoutube());

$amphtml = $converter->convert($html);

$amp_scripts = $converter->getScripts();

print_r($amphtml);
print_r($amp_scripts);

```
## Writing your own converters:

The library can't support everything out of the box, but you can extend it with your own converters(or you can replace existing ones if you need).

For example, consider the following: You use a jQuery countdown library in some of your articles/blog posts and you want to convert the following code to AMP.

```html
<!-- The following line should be replaced with amp-date-countdown component -->
<div data-countdown="2038/01/19"></div>

<!-- Countdown library: http://hilios.github.io/jQuery.countdown/examples/multiple-instances.html -->
<script>
$('[data-countdown]').each(function() {
  var $this = $(this), finalDate = $(this).data('countdown');
  $this.countdown(finalDate, function(event) {
    $this.html(event.strftime('%D days %H:%M:%S'));
  });
});
</script>
```

You can create a custom converter class that implements the TagConverterInterface.

```php
<?php

class CountdownConverter implements \magyarandras\AMPConverter\TagConverterInterface
{

    private $necessary_scripts = [];

    private $extension_scripts = [
    '<script async custom-element="amp-date-countdown" src="https://cdn.ampproject.org/v0/amp-date-countdown-0.1.js"></script>',
    '<script async custom-template="amp-mustache" src="https://cdn.ampproject.org/v0/amp-mustache-0.2.js"></script>'
    ];

    public function convert(\DOMDocument $doc)
    {

        $query = '//div[@data-countdown]';

        $xpath = new \DOMXPath($doc);

        $entries = $xpath->query($query);

        if ($entries->length > 0) {
            $this->necessary_scripts = $this->extension_scripts;
        }

        foreach ($entries as $tag) {

            //Although in this example there isn't any validation, you definitely should check if the date is valid.
            $timestamp = strtotime($tag->getAttribute('data-countdown'));

            $countdown = $doc->createElement('amp-date-countdown');

            $countdown->setAttribute('timestamp-seconds', $timestamp);
            $countdown->setAttribute('layout', 'fixed-height');
            $countdown->setAttribute('height', '50');

            $template = $doc->createElement('template');
            $template->setAttribute('type', 'amp-mustache');

            $paragraph = $doc->createElement('p');
            $paragraph->setAttribute('class', 'p1');

            $text = $doc->createTextNode('{{d}} days, {{h}} hours, {{m}} minutes and {{s}} seconds');

            $paragraph->appendChild($text);
            
            $template->appendChild($paragraph);
            $countdown->appendChild($template);

            $tag->parentNode->replaceChild($countdown, $tag);
        }
                  


        return $doc;
    }

    public function getNecessaryScripts() 
    {
        return $this->necessary_scripts;
    }

}

```

Using the custom converter:

```php
<?php

require_once 'vendor/autoload.php';
require_once 'CountdownConverter.php';

use magyarandras\AMPConverter\Converter;

$converter = new Converter();

//Load built-in converters
$converter->loadDefaultConverters();

$converter->addConverter(new CountdownConverter());

$html = '<div><h1>Hello!</h1><div data-countdown="2038/01/19"></div></div>';

//Convert html to amp html
$amphtml = $converter->convert($html);

//Get the necessary amp components
$amp_scripts = $converter->getScripts();

print_r($amphtml);
print_r($amp_scripts);

```