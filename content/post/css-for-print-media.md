+++
date = "2014-11-25T14:06:28+05:30"
sticky = false
tags = ["css"]
title = "CSS for Print Media"

+++

I never really felt it was important to style web pages for the print medium till a few days ago when I was formatting my résumé. The theme that I've forked to use on this blog employs `font-kerning` and `anti-aliasing` to make my posts look like text on paper. This made me curious to see what my resume would look like if printed.

The *Print Preview* was embarrassing. The text was huge and there were gaps everywhere. Paragraphs were getting split between pages in all the wrong places. What was more, the page count stood at 6 A4 pages because of the oversized text. Much has been written about the importance of brevity when it comes to job applications. What if a prospective employer wanted to take a printout for their records, and then had their printer spew out pages after pages of poorly formatted text?

I decided to spend some time collecting best practices from all over the web. This blogpost is a choice collection of the essential ideas that one needs to know in order to write CSS for the printer.

#### Media Queries to the Rescue

Media queries have traditionally been used for making websites responsive -- to make sure your content looks good on the most common screen sizes. But, as the name suggests, they can be used to query not just screen sizes, but just about any detail about the medium. For example, you can add print styles in the following two ways:

{{< highlight html >}}
<!--Styles for screens-->  
<link rel="stylesheet" src="screen.css">
...
<!--Styles for printers-->
<link rel="stylesheet" media="print" src="print.css">
{{< /highlight >}}

...or, right inside screen.css, like so:

{{< highlight css >}}
@media (print) {  
    /*Styles for printers go in here*/
}
{{< /highlight >}}

Media queries can be used to achieve many more details, like whether the printer is monochrome or color. I encourage you to read more about them on [MDN](http://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Media_queries)

But what exactly does one need to keep in mind while writing printer styles?

#### Drop the Headers, Footers and Backgrounds

When a reader prints out your blogpost, they probably don't want any of the navigation, search boxes, or other header elements. Likewise, sitemaps and other links that go into your footer aren't usually required. The same background color that makes your website look nice might be superfluous when it comes to printing on a monochrome printer. Therefore, you'll likely need to add the following to `print.css`:

{{< highlight css >}}
header, footer {  
    display: none;
}

body {  
    background: white;
}
{{< /highlight >}}

#### Bring out the Page-breaks

In most situations, you don't want page-breaks to be inserted into your content arbitrarily. To control where your content is split across pages, you can use the `page-break-before` and `page-break-after` properties, like so:

{{< highlight html >}}
...  
<!-- inserts a page-break here -->
<div class="page-break"></div>
...
{{< /highlight >}}

{{< highlight css >}}
.page-break {  
    display: block;
    page-break-before: always;
}
{{< /highlight >}}

These properties support a bunch of useful options, like `avoid`, `left` and `right`. Read more about them [here](http://developer.mozilla.org/en-US/docs/Web/CSS/page-break-after).

#### The Ideal font-size

Failing to land at the perfect px number after many trials, I finally resorted to A List Apart for some help. They have a great article explaining their print stylesheet, and the author suggests the use of a point-size instead of a pixel-size:

{{< highlight css >}}
p {  
    font-size: 12pt;
}
{{< /highlight >}}

#### Links

Let's face it -- hyperlinks are mostly useless when it comes to the print medium. However, keep in mind that as long as they're important (and short) enough, your readers might actually be willing to make the effort to type them into a browser. So just make sure all of the links you *really* want your readers to see, are displayed on paper. For example,

{{< highlight html >}}
<a href="/resume">Résumé</a><span class="print-only">(www.blog.me/resume)</span>
{{< /highlight >}}

Your stylesheet would then look like:

{{< highlight css >}}
.print-only {  
    display: none;
}

@media (print) {
    .print-only {
        display: inline;
    }
}
{{< /highlight >}}

#### Further Reading

For more on writing print stylesheets, you can check out this [fantastic article at A List Apart](http://alistapart.com/article/goingtoprint).
