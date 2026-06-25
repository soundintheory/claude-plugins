 ---
  name: navigation-module
  description: Reference for the SoundInTheory.Piranha.Navigation package — LinkField, ApplicationServiceAccessor, menus, and service registration
  version: 1.0.0
  ---

  # SoundInTheory.Piranha.Navigation

  Two NuGet packages provide navigation features in this project:

  | Package | Purpose |
  |---|---|
  | `SoundInTheory.Piranha.Navigation.Links` v2.2.2 | `LinkField`, `ApplicationServiceAccessor`, URL helpers |
  | `SoundInTheory.Piranha.Navigation.Menus` v2.3.1 | Menu definitions, `NavigationHooks` |

  ## Registration in Program.cs

  ```csharp
  // Inside the Piranha service builder callback:
  options.UseLinks();   // registers LinkField, ApplicationServiceAccessor, link API
  // Menus are configured via App.Modules.Navigation().Menus.Register(...)

  UseLinks() calls AddLinks() → AddLinkServices() which runs
  TryAddScoped<ApplicationServiceAccessor>(). No manual registration needed.

  LinkField

  This handles internal pages, external URLs, text, and anchor/target attributes in one field.

  Model

  using SoundInTheory.Piranha.Navigation.Models;

  [Field]
  public LinkField MyLink { get; set; }

  Template

  @* Check before rendering *@
  @if (Model.MyLink.HasValue)
  {
      <a link="@Model.MyLink">@Model.MyLink.Text</a>
  }

  @* Or use individual properties *@
  <a href="@Model.MyLink.Url" target="@Model.MyLink.Attributes?.Target">
      @Model.MyLink.Text
  </a>

  The link= attribute is a tag helper provided by the navigation package that sets
  href, target, rel, and other attributes automatically.

  Properties

  ┌───────────────────┬─────────┬──────────────────────────────────────┐
  │     Property      │  Type   │                Notes                 │
  ├───────────────────┼─────────┼──────────────────────────────────────┤
  │ HasValue          │ bool    │ True if a URL or content link is set │
  ├───────────────────┼─────────┼──────────────────────────────────────┤
  │ Url               │ string  │ Final resolved URL                   │
  ├───────────────────┼─────────┼──────────────────────────────────────┤
  │ Text              │ string  │ Display text                         │
  ├───────────────────┼─────────┼──────────────────────────────────────┤
  │ Type              │ string  │ "Custom", "Page", or "Post"          │
  ├───────────────────┼─────────┼──────────────────────────────────────┤
  │ Attributes.Target │ string? │ e.g. "_blank"                        │
  ├───────────────────┼─────────┼──────────────────────────────────────┤
  │ Attributes.Rel    │ string? │ e.g. "nofollow"                      │
  └───────────────────┴─────────┴──────────────────────────────────────┘

  ApplicationServiceAccessor

  Exposes IApplicationService outside of Razor templates (where WebApp is already
  available). Primarily used to generate site-prefixed URLs in C# code.

  using SoundInTheory.Piranha.Navigation;

  public class MyService(ApplicationServiceAccessor appAccessor)
  {
      public string GetUrl(string permalink) =>
          appAccessor.ApplicationService.UrlWithPrefix(
              permalink,
              appAccessor.ApplicationService.Site.Id);
  }

  See the piranha:site-aware-urls skill for full usage patterns.

  Menu Registration

  Menus are registered by slug at startup and then populated in the Piranha Manager UI:

  App.Modules.Navigation().Menus.Register("main-header", "Main Site Header", maxDepth: 3);
  App.Modules.Navigation().Menus.Register("main-footer-columns", "Footer Columns", maxDepth: 2);

  Custom menu item types:

  App.Modules.Navigation().MenuItems.Register<Link>();
  App.Modules.Navigation().MenuItems.Register<DropdownMenu>();

  Sitemap helpers

  PiranhaSitemapExtensions (in Web2/Extensions/) adds helpers on Sitemap and
  SitemapItem:

  sitemap.GetParent(id)          // → SitemapItem? parent
  sitemap.Next(id)               // → SitemapItem? next sibling
  sitemap.Previous(id)           // → SitemapItem? previous sibling
  item.ToLink(appAccessor)       // → Link (with UrlWithPrefix applied)
  sitemap.AsFlat<T, TItem>()     // → IEnumerable<TItem> depth-first flat list

  Namespaces Quick Reference

  using SoundInTheory.Piranha.Navigation;           // ApplicationServiceAccessor
  using SoundInTheory.Piranha.Navigation.Models;    // LinkField, Link, LinkType
  using SoundInTheory.Piranha.Navigation.Links;     // (assembly namespace)

  ---