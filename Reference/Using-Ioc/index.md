---
versionFrom: 8.0.0
---

# IoC/Dependency injection

Since Umbraco 8.0 we now support dependency injection out of the box. This means that you no longer have to install an external package such as Autofac in order to register your dependencies - sweet!

## Registering dependencies

Umbraco 8 is using LightInject. To register your own dependencies to the container you need to do so in a composer ([Read more about composers and components](../../implementation/composing/index.md)). It looks like this:

```csharp
using Doccers.Core.Services;
using Doccers.Core.Services.Implement;
using Umbraco.Core;
using Umbraco.Core.Composing;

namespace Doccers.Core
{
    public class Composer : IUserComposer
    {
        public void Compose(Composition composition)
        {
            composition.Register<ITagService, TagService>();
        }
    }
}
```

It is not required to have an interface for your dependency. You can also do this:

```csharp
public void Compose(Composition composition)
{
    composition.Register<Foobar>();
}
```

During registration you have the ability to define the lifetime:

```csharp
composition.Register<TService, TImplementing>(Lifetime.Request);
```

The `Lifetime` supports:

```csharp
public enum Lifetime
{
    // Always get a new instance.
    // This is the default lifetime.
    Transient = 0,

    //One unique instance per request.
    Request = 1,

    // One unique instance per scope.
    Scope = 2,

    // One unique instance per container.
    Singleton = 3
}
```

## Injecting dependencies

Once you have registered your services, factories, helpers or whatever you need for you application, you can go ahead and inject them in a class constructor:

```csharp
using Doccers.Core.Services;
using System.Web.Mvc;
using Umbraco.Web.Models;
using Umbraco.Web.Mvc;

namespace Doccers.Core.Controllers
{
    public class HomeController : RenderMvcController
    {
        private readonly ITagService _tagService;
        private readonly Foobar _foobar;

        public HomeController(ITagService tagService, Foobar foobar)
        {
            _tagService = tagService;
            _foobar = foobar;
        }

        public ActionResult Home(ContentModel model)
        {
            var bar = _foobar.Foo();

            return CurrentTemplate(model);
        }
    }
}
```

If I place a breakpoint on `var bar = _foobar.Foo()` and inspect the variable in my `Locals` windows of Visual Studio I see that the value is `Bar`, which is what I expect, since all the `Foobar.Foo()` method does is to return `Bar` as a string:

```csharp
namespace Doccers.Core
{
    public class Foobar
    {
        public string Foo() => "Bar";
    }
}
```

Cool! Know you know how to register your own dependencies in your application. :)

## Other things you can inject

Most of (if not all) the Umbraco goodies you work with every day can be injected. Here are some examples.

### UmbracoHelper

[Read more about the UmbracoHelper](../querying/umbracohelper/index.md)

```csharp
using System.Globalization;
using System.Linq;
using Umbraco.Web;
using Umbraco.Web.PublishedModels;

namespace Doccers.Core.Services.Implement
{
    public class SiteService : ISiteService
    {
        private readonly UmbracoHelper _umbraco;

        public SiteService(UmbracoHelper umbraco)
        {
            _umbraco = umbraco;
        }

        public Site GetSiteByCulture(string culture)
        {
            return _umbraco
                .ContentAtRoot()
                .OfType<Site>()
                .FirstOrDefault(x => x.GetCultureFromDomains() == culture);
        }
    }
}
```

### ExamineManager

[Read more about examine](../Searching/Examine/index.md).

```csharp
using Examine;
using Examine.Providers;
using System;
using Umbraco.Core.Composing;

namespace Doccers.Core.Components
{

    public class ExamineComponent : IComponent
    {
        private readonly IExamineManager _examineManager;

        public ExamineComponent(IExamineManager examineManager)
        {
            _examineManager = examineManager;
        }

        public void Initialize()
        {
            if (_examineManager.TryGetIndex("ExternalIndex", out var index))
            {
                if (!(index is BaseIndexProvider indexProvider))
                    throw new InvalidOperationException("Could not cast");

                // Do stuff with the index
            }
        }

        public void Terminate() { }
    }
}
```

### ILogger

[Read more about logging](../../Getting-Started/Code/Debugging/Logging/index.md)

```csharp
using System;
using Umbraco.Core.Logging;

namespace Doccers.Core
{
    public class Foobar
    {
        private readonly ILogger _logger;

        public Foobar(ILogger logger)
        {
            _logger = logger;
        }

        public void Foo()
        {
            _logger.Info<Foobar>($"Method Foo called at {DateTime.UtcNow}");
        }
    }
}
```
