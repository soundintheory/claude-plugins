  ---
  name: piranha-archive-page-patterns
  description: Reference for building Piranha archive pages — SinglePage<T> pattern, post loading, routing, and query string filters
  version: 1.0.0.0
  ---

  # Piranha Archive Page Patterns — AI Reference Guide

  Covers the conventions for archive pages: PageModel structure, loading posts, routing, and query string filtering.

  ---

  ## Archive Page Model

  An archive page type needs `IsArchive = true` and `[PageTypeArchiveItem]` to associate the post type:

  ```csharp
  [ContentTypeRoute(Title = "Archive", Route = "/patientVideo/archive")]
  [PageType(Title = "Patient Video Archive", IsArchive = true, UseBlocks = true)]
  [PageTypeArchiveItem(typeof(PatientVideoPost))]
  public class PatientVideoArchive : Page<PatientVideoArchive> { }

  ---
  PageModel Structure

  Archive page PageModels inherit SinglePage<T> and override OnGet(Guid id, bool draft):

  public class PatientVideoArchivePageModel(IApi api, IModelLoader loader)
      : SinglePage<PatientVideoArchive>(api, loader)
  {
      public override async Task<IActionResult> OnGet(Guid id, bool draft = false)
      {
          Data = await _loader.GetPageAsync<PatientVideoArchive>(id, HttpContext.User, draft);
          if (Data == null) return NotFound();

          // ... load posts, build view data

          return Page();
      }
  }

  - Data is the typed page model — access page fields via Data.MyRegion.MyField
  - _api and _loader are provided by the base class constructor

  ---
  Loading Posts

  All posts (no pagination)

  var posts = await _api.Posts.GetAllAsync<PatientVideoPost>(Data.Id);

  Omitting page/pageSize returns all posts. Used when filtering or grouping in memory.

  Paginated posts

  // page index is 0-based; pageSize is the count per page
  var posts = await _api.Posts.GetAllAsync<BlogPost>(Data.Id, pageIndex, pageSize);
  var total  = await _api.Posts.GetCountAsync(Data.Id);

  Used when the page shows a fixed number of results with Load More / HTMX pagination (see BlogArchive).

  ---
  @page Directive vs ContentTypeRoute

  The @page directive in a Razor page must result in a URL that matches the ContentTypeRoute attribute on the page model. The default route is derived from
  the file path:

  ┌─────────────────────────────────┬──────────────────────────────────┐
  │            File path            │        Conventional route        │
  ├─────────────────────────────────┼──────────────────────────────────┤
  │ Pages/Blog/Archive.cshtml       │ /Blog/Archive                    │
  ├─────────────────────────────────┼──────────────────────────────────┤
  │ Pages/EventGallery/List.cshtml  │ /EventGallery/List               │
  ├─────────────────────────────────┼──────────────────────────────────┤
  │ Pages/PatientVideo/Index.cshtml │ /PatientVideo (Index is dropped) │
  └─────────────────────────────────┴──────────────────────────────────┘

  Rule: Only add an explicit path to @page when the conventional route doesn't match the ContentTypeRoute.

  Examples

  @* Pages/Blog/Archive.cshtml — ContentTypeRoute is /blog/archive — matches by convention *@
  @page

  @* Pages/EventGallery/List.cshtml — ContentTypeRoute is /eventGallery/list — needs slug param *@
  @page "/eventGallery/list/{slug?}"

  @* Pages/PatientVideo/Index.cshtml — ContentTypeRoute is /patientVideo/archive
     but Index convention gives /PatientVideo — no match, explicit path required *@
  @page "/patientVideo/archive"

  ASP.NET Core routing is case-insensitive, so /Blog/Archive matches /blog/archive.

  ---
  Query String Filter Binding

  Use [BindProperty(SupportsGet = true)] to bind query string parameters to PageModel properties. They are populated automatically before OnGet runs.

  [BindProperty(SupportsGet = true)]
  public Guid? CancerTypeId { get; set; }

  [BindProperty(SupportsGet = true)]
  public Guid? ServiceTypeId { get; set; }

  - Guid? — empty string in the query string binds as null
  - string? — use for text-based filters
  - int with default — use for page index: public int CurrentPage { get; set; } = 1;

  Filter in memory after loading all posts:

  var filtered = posts.AsEnumerable();

  if (ServiceTypeId.HasValue)
      filtered = filtered.Where(v => v.Info.ServiceType.HasValue && v.Info.ServiceType.Id == ServiceTypeId);
  if (CancerTypeId.HasValue)
      filtered = filtered.Where(v => v.Info.CancerType.HasValue && v.Info.CancerType.Id == CancerTypeId);

  ---
  Template Boilerplate

  @page "/patientVideo/archive"
  @using Web2.Pages.PatientVideo
  @model PatientVideoArchivePageModel

  <div class="container-fluid generic-wrapper">
      <div class="row">

          <div class="col-xs-24 col-sm-24 col-lg-5">
              <div class="generic-left">
                  <partial name="Shared/Navigation/_SecondaryNavigation"
                           model="(Model.Data.Id, Model.Data.ParentId)" />
              </div>
          </div>

          <div class="col-xs-24 col-sm-24 col-lg-17">
              <div class="generic-right">

                  <h1>@Model.Data.Title</h1>

                  @foreach (var block in Model.Data.Blocks)
                  {
                      @Html.DisplayFor(m => block, block.GetType().Name)
                  }

                  @* filter form, content grid etc *@

              </div>
          </div>

      </div>
  </div>

  - Secondary nav partial takes (pageId, parentId) tuple
  - Right column is col-lg-17 for pages with secondary nav (not col-lg-19)
  - Blocks loop before main content lets editors add intro content above filters

  ---
  Existing Archive Pages — Quick Reference

  ┌───────────────┬─────────────────────────────────┬────────────────────────────────────┬──────────────────┬───────────────────────────┐
  │     Page      │              File               │               Route                │    Posts type    │         Filtering         │
  ├───────────────┼─────────────────────────────────┼────────────────────────────────────┼──────────────────┼───────────────────────────┤
  │ Blog          │ Pages/Blog/Archive.cshtml       │ @page                              │ BlogPost         │ None — HTMX pagination    │
  ├───────────────┼─────────────────────────────────┼────────────────────────────────────┼──────────────────┼───────────────────────────┤
  │ Event Gallery │ Pages/EventGallery/List.cshtml  │ @page "/eventGallery/list/{slug?}" │ EventGalleryPost │ Slug: archive vs current  │
  ├───────────────┼─────────────────────────────────┼────────────────────────────────────┼──────────────────┼───────────────────────────┤
  │ Patient Video │ Pages/PatientVideo/Index.cshtml │ @page "/patientVideo/archive"      │ PatientVideoPost │ ContentField Guid filters │
  ├───────────────┼─────────────────────────────────┼────────────────────────────────────┼──────────────────┼───────────────────────────┤
  │ ```           │                                 │                                    │                  │                           │
  └───────────────┴─────────────────────────────────┴────────────────────────────────────┴──────────────────┴───────────────────────────┘
