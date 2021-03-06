![BracketPipe Icon](https://raw.githubusercontent.com/erdomke/BracketPipe/master/icon.png)

# BracketPipe

BracketPipe is a .NET library for building parsing and processing piplines for web languages like HTML, CSS, Javscript, SVG, and MathML. The parser is built upon the official W3C specification.  It differentiates itself from other libraries such as [AngleSharp](https://github.com/AngleSharp/AngleSharp) (which it is based on) and [HTML Agility Pack](http://htmlagilitypack.codeplex.com/) in that it does not build an in-memory representation of the DOM.  Rather, it focuses on providing a convenient streaming interface for fast processing of HTML documents.  This makes the library ideal for 

* minifying HTML
* sanitizing HTML to prevent XSS attacks
* converting HTML to text
* crawling hyperlinks from HTML documents
* cleaning up MS Word HTML
* ... and any other task that only requires a single traversal of the HTML document

It can also be viewed as a modern update to previous projects such as the [SgmlReader](https://github.com/MindTouch/SGMLReader) and [Majestic-12 HTML Parser](http://www.majestic12.co.uk/projects/html_parser.php)

## Usage

Use a pipeline to parse, minify, and sanitize HTML;

```csharp
var html = @"<div>  <script>alert('xss');</script>
              <a href=""http://www.google.com/"">Google</a>
              <a href=""http://www.yahoo.com/"">Yahoo</a>  </div>";
using (var reader = new HtmlReader(html))
{
  var result = (string)reader.Sanitize().Minify().ToHtml();
  Assert.AreEqual(@"<div><a href=""http://www.google.com/"">Google</a> <a href=""http://www.yahoo.com/"">Yahoo</a></div>"
    , result);
}
```

Parse the links from an HTML string

```csharp
var html = @"<html>  <body>  <script>alert('xss');</script>
              <a href=""http://www.google.com/"">Google</a>
              <a href=""http://www.yahoo.com/"">Yahoo</a>  </body>  </html>";
using (var reader = new HtmlReader(html))
{
  var urls = reader
    .OfType<HtmlStartTag>()
    .Where(t => t.Value == "a")
    .Select(t => t["href"])
    .ToArray();
  CollectionAssert.AreEqual(new string[]
  {
    "http://www.google.com/",
    "http://www.yahoo.com/"
  }
  , urls);
}
```

Sanitize, minify, or convert HTML

```csharp
var html = "Something to <strong>convert</strong>";
var str = Html.Sanitize(html);
str = Html.Minify(html);
str = Html.ToMarkdown(html)
```

Write HTML

```csharp
using (var s = new StringWriter())
using (var w = new HtmlTextWriter(s))
{
  w["html"]
    ["body"]
      ["div", "style", "color:red", "id", "1234"]
        .Text("A > value")
      ["/div"]
      ["input", "type", "text", "value", "start"]
      ["p"].Text("para & more")["/p"]
    ["/body"]
   ["/html"].Flush();
  var str = s.ToString();
}
```

XML to HTML

```csharp
using (var s = new StringWriter())
using (var w = new HtmlTextWriter(s))
{
  var xml = new XElement("body",
    new XElement("div", new XAttribute("style", "color:red"), "A > value"),
    new XElement("input", new XAttribute("type", "text"), new XAttribute("value", "start")),
    new XElement("p", "para & more"));
  xml.WriteTo(w);
  w.Flush();

  var str = s.ToString();
}
```

## Installing via NuGet

[![NuGet version](https://badge.fury.io/nu/BracketPipe.svg)](https://www.nuget.org/packages/BracketPipe)

    Install-Package BracketPipe

## Performance

Using only the stripped down core of [AngleSharp](https://github.com/AngleSharp/AngleSharp), BracketPipe
achives incredibly fast performance.  Comparison charts each showing the average time over thousands of operations:

**Conversion to Markdown**

Comparison to [ReverseMarkdown](https://github.com/mysticmind/reversemarkdown-net/) and [HTML2Markdown](https://github.com/baynezy/Html2Markdown/);

![Markdown conversion comparsion chart](http://chart.googleapis.com/chart?cht=bhg&chs=400x120&chd=t:1203.8,1148.8,5255.2&chds=0,5300&chxl=1:|Html2Markdown(5255ms)|ReverseMarkdown(1149ms)|BracketPipe(1204ms)&chxt=x,y&chxr=0,0,5300&chco=a347bb)

**Minifying HTML**

Comparison to [WebMarkupMin](https://github.com/Taritsyn/WebMarkupMin);

![Minification comparsion chart](http://chart.googleapis.com/chart?cht=bhg&chs=400x80&chd=t:853.3,1203&chds=0,1300&chxl=1:|WebMarkupMin(1203ms)|BracketPipe(853ms)&chxt=x,y&chxr=0,0,1300&chco=a347bb)

**Sanitizing HTML**

Comparison to [HtmlSanitizer](https://github.com/mganss/HtmlSanitizer)

![Sanitization comparsion chart](http://chart.googleapis.com/chart?cht=bhg&chs=400x80&chd=t:410.8,1983.6&chds=0,2000&chxl=1:|HtmlSanitizer(1983ms)|BracketPipe(410ms)&chxt=x,y&chxr=0,0,2000&chco=a347bb)

## Standards Conformance

The parser uses the [HTML 5.1 specification](https://dev.w3.org/html5/spec-preview/tokenization.html), which defines error handling and element correction.  As 
a result, it works the same as all modern browsers.

## Portable

It is designed as a .Net Standard library targeting [.Net Standard 1.0](https://docs.microsoft.com/en-us/dotnet/articles/standard/library).  
That means it supports the following platforms:

* .NET Core 1.0+
* .NET Framework 4.5+
* Xamarin vNext+
* Universal Windows Platform 10.0+
* Windows 8.0+
* Windows Phone 8.1+
* Windows Phone Silverlight 8.0+

The NuGet package build also supports .Net 3.5 and .Net 4.0.

## License

The MIT License (MIT)

Copyright (c) 2013 - 2016 BracketPipe

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
