---
title: Razor syntax reference for ASP.NET Core
author: guardrex
description: Learn about Razor markup syntax for embedding server-based code into webpages.
keywords: ASP.NET Core,Razor,Razor directives
ms.author: riande
manager: wpickett
ms.date: 09/29/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/views/razor
---
# Razor syntax for ASP.NET Core

By [Rick Anderson](https://twitter.com/RickAndMSFT), [Luke Latham](https://github.com/guardrex), and [Taylor Mullen](https://twitter.com/ntaylormullen)

Razor is a markup syntax for embedding server-based code into webpages. The Razor syntax consists of Razor markup, C#, and HTML. Files containing Razor generally have a *.cshtml* file extension.

## Rendering HTML

The default Razor language is HTML. Rendering HTML from Razor markup is no different than rendering HTML from an HTML file. If you place HTML markup into a *.cshtml* Razor file, it's rendered by the server unchanged.

## Razor syntax

Razor supports C# and uses the `@` symbol to transition from HTML to C#. Razor evaluates C# expressions and renders them in the HTML output.

When an `@` symbol is followed by a [Razor reserved keyword](#razor-reserved-keywords), it transitions into Razor-specific markup. Otherwise, it transitions into plain C#.

To escape an `@` symbol in Razor markup, use a second `@` symbol:

```cshtml
<p>@@Username</p>
```

The code is rendered in HTML with a single `@` symbol:

```html
<p>@Username</p>
```

HTML attributes and content containing email addresses don't treat the `@` symbol as a transition character. The email addresses in the following example are untouched by Razor parsing:

```cshtml
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

## Implicit Razor expressions

Implicit Razor expressions start with `@` followed by C# code:

```cshtml
<p>@DateTime.Now</p>
<p>@DateTime.IsLeapYear(2016)</p>
```

With the exception of the C# `await` keyword, implicit expressions must not contain spaces. You can intermingle spaces if the C# statement has a clear ending:

```cshtml
<p>@await DoSomething("hello", "world")</p>
```

## Explicit Razor expressions

Explicit Razor expressions consist of an `@` symbol with balanced parenthesis. To render last week's time, the following Razor markup is used:

```cshtml
<p>Last week this time: @(DateTime.Now - TimeSpan.FromDays(7))</p>
```

Any content within the `@()` parenthesis is evaluated and rendered to the output.

Implicit expressions, described in the previous section, generally can't contain spaces. In the following code, one week isn't subtracted from the current time:

[!code-cshtml[Main](razor/sample/Views/Home/Contact.cshtml?range=17)]

The code renders the following HTML:

```html
<p>Last week: 7/7/2016 4:39:52 PM - TimeSpan.FromDays(7)</p>
```

You can use an explicit expression to concatenate text with an expression result:

```cshtml
@{
    var joe = new Person("Joe", 33);
}

<p>Age@(joe.Age)</p>
```

Without the explicit expression, `<p>Age@joe.Age</p>` is treated as an email address, and `<p>Age@joe.Age</p>` is rendered. When written as an explicit expression, `<p>Age33</p>` is rendered.

## Expression encoding

C# expressions that evaluate to a string are HTML encoded. C# expressions that evaluate to `IHtmlContent` are rendered directly through `IHtmlContent.WriteTo`. C# expressions that don't evaluate to `IHtmlContent` are converted to a string by `ToString` and encoded before they're rendered.

```cshtml
@("<span>Hello World</span>")
```

The code renders the following HTML:

```html
&lt;span&gt;Hello World&lt;/span&gt;
```

The HTML is shown in the browser as:

```
<span>Hello World</span>
```

`HtmlHelper.Raw` output isn't encoded but rendered as HTML markup.

> [!WARNING]
> Using `HtmlHelper.Raw` on unsanitized user input is a security risk. User input might contain malicious JavaScript or other exploits. Sanitizing user input is difficult. Avoid using `HtmlHelper.Raw` with user input.

```cshtml
@Html.Raw("<span>Hello World</span>")
```

The code renders the following HTML:

```html
<span>Hello World</span>
```

## Razor code blocks

Razor code blocks start with `@` and are enclosed by `{}`. Unlike expressions, C# code inside code blocks isn't rendered. Code blocks and expressions in a view share the same scope and are defined in order:

```cshtml
@{
    var quote = "The future depends on what you do today. - Mahatma Gandhi";
}

<p>@quote</p>

@{
    quote = "Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.";
}

<p>@quote</p>
```

The code renders the following HTML:

```html
<p>The future depends on what you do today. - Mahatma Gandhi</p>
<p>Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.</p>
```

### Implicit transitions

The default language in a code block is C#, but you can transition back to HTML:

```cshtml
@{
    var inCSharp = true;
    <p>Now in HTML, was in C# @inCSharp</p>
}
```

### Explicit delimited transition

To define a sub-section of a code block that should render HTML, surround the characters for rendering with the Razor **\<text>** tag:

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <text>Name: @person.Name</text>
}
```

Use this approach when you want to render HTML that isn't surrounded by an HTML tag. Without an HTML or Razor tag, you receive a Razor runtime error.

The **\<text>** tag is also useful to control whitespace when rendering content. Only the content between the **\<text>** tags is rendered, and no whitespace before or after the **\<text>** tags appears in the HTML output.

### Explicit Line Transition with @:

To render the rest of an entire line as HTML inside a code block, use the `@:` syntax:

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    @:Name: @person.Name
}
```

Without the `@:` in the code, you recieve a Razor runtime error.

## Control Structures

Control structures are an extension of code blocks. All aspects of code blocks (transitioning to markup, inline C#) also apply to the following structures.

### Conditionals @if, else if, else, and @switch

`@if` controls when code runs:

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
```

`else` and `else if` don't require the `@` symbol:

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
else if (value >= 1337)
{
    <p>The value is large.</p>
}
else
{
    <p>The value is odd and small.</p>
}
```

You can use a switch statement like this:

```cshtml
@switch (value)
{
    case 1:
        <p>The value is 1!</p>
        break;
    case 1337:
        <p>Your number is 1337!</p>
        break;
    default:
        <p>Your number wasn't 1 or 1337.</p>
        break;
}
```

### Looping @for, @foreach, @while, and @do while

You can render templated HTML with looping control statements. To render a list of people:

```cshtml
@{
    var people = new Person[]
    {
          new Person("Weston", 33),
          new Person("Johnathon", 41),
          ...
    };
}
```

You can use any of the following looping statements:

`@for`

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@foreach`

```cshtml
@foreach (var person in people)
{
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@while`

```cshtml
@{ var i = 0; }
@while (i < people.Length)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
}
```

`@do while`

```cshtml
@{ var i = 0; }
@do
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
} while (i < people.Length);
```

### Compound @using

In C#, a `using` statement is used to ensure an object is disposed. In Razor, the same mechanism is used to create HTML Helpers that contain additional content. For instance, you can utilize HTML Helpers to render a form tag with the `@using` statement:

```cshtml
@using (Html.BeginForm())
{
    <div>
        email:
        <input type="email" id="Email" value="">
        <button>Register</button>
    </div>
}
```

You can also perform scope-level actions with [Tag Helpers](xref:mvc/views/tag-helpers/intro).

### @try, catch, finally

Exception handling is similar to C#:

[!code-cshtml[Main](razor/sample/Views/Home/Contact7.cshtml)]

### @lock

Razor has the capability to protect critical sections with lock statements:

```cshtml
@lock (SomeLock)
{
    // Do critical section work
}
```

### Comments

Razor supports C# and HTML comments:

```cshtml
@{
    /* C# comment */
    // Another C# comment
}
<!-- HTML comment -->
```

The code renders the following HTML:

```html
<!-- HTML comment -->
```

Razor comments are removed by the server before the webpage is rendered. Razor uses `@*  *@` to delimit comments. The following code is commented out, so the server doesn't render any markup:

```cshtml
@*
    @{
        /* C# comment */
        // Another C# comment
    }
    <!-- HTML comment -->
*@
```

## Directives

Razor directives are represented by implicit expressions with reserved keywords following the `@` symbol. A directive typically changes the way a view is parsed or enables different functionality.

Understanding how Razor generates code for a view makes it easier to understand how directives work.

[!code-html[Main](razor/sample/Views/Home/Contact8.cshtml)]

The code generates a class similar to the following:

```csharp
public class _Views_Something_cshtml : RazorPage<dynamic>
{
    public override async Task ExecuteAsync()
    {
        var output = "Getting old ain't for wimps! - Anonymous";

        WriteLiteral("/r/n<div>Quote of the Day: ");
        Write(output);
        WriteLiteral("</div>");
    }
}
```

Later in this article, the section [Viewing the Razor C# class generated for a view](#viewing-the-razor-c-class-generated-for-a-view) explains how to view this generated class.

### @using

The `@using` directive adds the C# `using` directive to the generated view:

[!code-cshtml[Main](razor/sample/Views/Home/Contact9.cshtml)]

### @model

The `@model` directive specifies the type of the model passed to a view:

```cshtml
@model TypeNameOfModel
```

If you create an ASP.NET Core MVC app with individual user accounts, the *Views/Account/Login.cshtml* view contains the following model declaration:

```cshtml
@model LoginViewModel
```

The class generated inherits from `RazorPage<dynamic>`:

```csharp
public class _Views_Account_Login_cshtml : RazorPage<LoginViewModel>
```

Razor exposes a `Model` property for accessing the model passed to the view:

```cshtml
<div>The Login Email: @Model.Email</div>
```

The `@model` directive specifies the type of this property. The directive specifies the `T` in `RazorPage<T>` that the generated class that your view derives from. If you don't specify the `@model` directive, the `Model` property is of type `dynamic`. The value of the model is passed from the controller to the view. See [Strongly typed models and the @model keyword](xref:tutorials/first-mvc-app/adding-model#strongly-typed-models-keyword-label) for more information.

### @inherits

The `@inherits` directive gives you full control of the class your view inherits:

```cshtml
@inherits TypeNameOfClassToInheritFrom
```

The following is a custom Razor page type:

[!code-csharp[Main](razor/sample/Classes/CustomRazorPage.cs)]

The `CustomText` is displayed in a view:

[!code-cshtml[Main](razor/sample/Views/Home/Contact10.cshtml)]

The code renders the following HTML:

```html
<div>Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping a slop bucket on the street below.</div>
```

You can't use `@model` and `@inherits` in the same view. You can have `@inherits` in a *_ViewImports.cshtml* file that the view imports:

[!code-cshtml[Main](razor/sample/Views/_ViewImportsModel.cshtml)]

The following is an example of a strongly-typed view:

[!code-cshtml[Main](razor/sample/Views/Home/Login1.cshtml)]

If "rick@contoso.com" is passed in the model, the view generates the following HTML markup:

```html
<div>The Login Email: rick@contoso.com</div>
<div>Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping a slop bucket on the street below.</div>
```

### @inject

The `@inject` directive enables you to inject a service from your [service container](xref:fundamentals/dependency-injection) into your view. See [Dependency injection into views](xref:mvc/views/dependency-injection) for more information.

### @functions

The `@functions` directive enables you to add function-level content to a view:

```cshtml
@functions { // C# Code }
```

For example:

[!code-cshtml[Main](razor/sample/Views/Home/Contact6.cshtml)]

The code generates the following HTML markup:

```html
<div>From method: Hello</div>
```

The following code is the generated Razor C# class:

[!code-csharp[Main](razor/sample/Classes/Views_Home_Test_cshtml.cs?range=1-19)]

### @section

The `@section` directive is used in conjunction with the [layout](xref:mvc/views/layout) to enable views to render content in different parts of the HTML page. See [Sections](xref:mvc/views/layout#layout-sections-label) for more information.

## Tag Helpers

There are three directives that pertain to [Tag Helpers](xref:mvc/views/tag-helpers/intro).

| Directive | Function |
| --------- | -------- |
| [@addTagHelper](xref:mvc/views/tag-helpers/intro#add-helper-label) | Makes Tag Helpers available to a view. |
| [@removeTagHelper](xref:mvc/views/tag-helpers/intro#remove-razor-directives-label) | Removes Tag Helpers previously added from a view. |
| [@tagHelperPrefix](xref:mvc/views/tag-helpers/intro#prefix-razor-directives-label) | Specifies a tag prefix to enable Tag Helper support and to make Tag Helper usage explicit. |

## Razor reserved keywords

### Razor keywords

* page (Requires ASP.NET Core 2.0 and later)
* functions
* inherits
* model
* section
* helper (Not currently supported by ASP.NET Core)

Razor keywords are escaped with `@(Razor Keyword)` (for example, `@(functions)`).

### C# Razor keywords

* case
* do
* default
* for
* foreach
* if
* else
* lock
* switch
* try
* catch
* finally
* using
* while

C# Razor keywords must be double-escaped with `@(@C# Razor Keyword)` (for example, `@(@case)`). The first `@` escapes the Razor parser. The second `@` escapes the C# parser.

### Reserved keywords not used by Razor

* namespace
* class

## Viewing the Razor C# class generated for a view

Add the following class to your ASP.NET Core MVC project:

[!code-csharp[Main](razor/sample/Utilities/CustomTemplateEngine.cs)]

Override the `RazorTemplateEngine` added by MVC with the `CustomTemplateEngine` class:

[!code-csharp[Main](razor/sample/Startup.cs?highlight=4&range=10-14)]

Set a break point on the `return csharpDocument` statement of `CustomTemplateEngine`. When program execution stops at the break point, view the value of `generatedCode`.

![Text Visualizer view of generatedCode](razor/_static/tvr.png)

## View lookups and case sensitivity

The Razor view engine performs case-sensitive lookups for views. However, the actual lookup is determined by the underlying file system:

* File based source: 
  * On operating systems with case insensitive file systems (for example, Windows), physical file provider lookups are case insensitive. For example, `return View("Test")` results in matches for */Views/Home/Test.cshtml*, */Views/home/test.cshtml*, and any other casing variant.
  * On case sensitive file systems (for example, Linux, OSX, and with `EmbeddedFileProvider`), lookups are case sensitive. For example, `return View("Test")` specifically matches */Views/Home/Test.cshtml*.
* Precompiled views: With ASP.Net Core 2.0 and later, looking up precompiled views is case insensitive on all operating systems. The behavior is identical to physical file provider's behavior on Windows. If two precompiled views differ only in case, the result of lookup is non-deterministic.

Developers are encouraged to match the casing of file and directory names to the casing of area, controller, and action names. This ensures your deployments will find their views regardless of the underlying file system.
