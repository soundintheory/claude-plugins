  ---
  name: piranha-content-api
  description: Reference for querying Piranha Content<T> items and using ContentField
  version: 1.0.0.0
  ---

  # Piranha Content API — AI Reference Guide

  Covers querying content group items (`Content<T>`) via `IApi` and working with `ContentField` references on posts/pages.

  ---

  ## Content Types vs Posts vs Pages

  Piranha has three distinct model hierarchies:

  | Base class | Queried via | Used for |
  |---|---|---|
  | `Page<T>` | `_api.Pages` | Site pages |
  | `Post<T>` | `_api.Posts` | Archive post items |
  | `Content<T>` | `_api.Content` | Standalone content items (taxonomy, references) |

  `Content<T>` items are not tied to any URL or archive. They live in named content groups and are referenced from posts/pages via `ContentField`.

  ---

  ## Defining a Content Type

  ```csharp
  [ContentType(Title = "Cancer Type", UseExcerpt = false, UsePrimaryImage = false)]
  [ContentGroup(Title = "Cancer Type", Icon = "fas fa-disease")]
  public class CancerType : Content<CancerType>
  {
      [Field(Title = "Title", Description = "Required")]
      public string Title { get; set; }
  }

  - [ContentGroup] registers the admin navigation group
  - Title on Content<T> is a plain string, not a StringField
  - Id (Guid) is inherited from the base class

  ---
  Referencing Content from a Post/Page

  Use ContentField with [ContentFieldSettings(Group = "...")] to tie the field to a specific content type group:

  [Field(Title = "Cancer Type")]
  [ContentFieldSettings(Group = "CancerType")]
  public ContentField CancerType { get; set; }

  The Group string must match the class name of the target content type.

  ContentField properties

  ┌───────────┬───────┬─────────────────────────────────────────────────────┐
  │ Property  │ Type  │                        Notes                        │
  ├───────────┼───────┼─────────────────────────────────────────────────────┤
  │ .HasValue │ bool  │ True if a content item has been selected            │
  ├───────────┼───────┼─────────────────────────────────────────────────────┤
  │ .Id       │ Guid? │ ID of the referenced content item (null if not set) │
  └───────────┴───────┴─────────────────────────────────────────────────────┘

  ---
  Querying Content Items

  // Returns all items of a given content type, ordered by title
  var cancerTypes = (await _api.Content.GetAllAsync<CancerType>())
      .OrderBy(x => x.Title)
      .ToList();

  - Returns IEnumerable<T>
  - No pagination parameters — always returns all items
  - Items have .Id (Guid) and .Title (string) from the base class

  ---
  Archive Page Filter Pattern

  Full pattern for a Piranha archive page with content type dropdowns, filtering posts in memory.

  PageModel

  public class PatientVideoArchivePageModel(IApi api, IModelLoader loader)
      : SinglePage<PatientVideoArchive>(api, loader)
  {
      [BindProperty(SupportsGet = true)]
      public Guid? CancerTypeId { get; set; }

      [BindProperty(SupportsGet = true)]
      public Guid? ServiceTypeId { get; set; }

      public List<PatientVideoPost> Videos { get; set; } = [];
      public List<CancerType> CancerTypes { get; set; } = [];
      public List<ServiceType> ServiceTypes { get; set; } = [];
      public string FilterTitle { get; set; } = "";

      public override async Task<IActionResult> OnGet(Guid id, bool draft = false)
      {
          Data = await _loader.GetPageAsync<PatientVideoArchive>(id, HttpContext.User, draft);
          if (Data == null) return NotFound();

          CancerTypes = (await _api.Content.GetAllAsync<CancerType>()).OrderBy(x => x.Title).ToList();
          ServiceTypes = (await _api.Content.GetAllAsync<ServiceType>()).OrderBy(x => x.Title).ToList();

          var allVideos = await _api.Posts.GetAllAsync<PatientVideoPost>(Data.Id);
          var filtered = allVideos.AsEnumerable();

          if (ServiceTypeId.HasValue)
              filtered = filtered.Where(v => v.Info.ServiceType.HasValue && v.Info.ServiceType.Id == ServiceTypeId);
          if (CancerTypeId.HasValue)
              filtered = filtered.Where(v => v.Info.CancerType.HasValue && v.Info.CancerType.Id == CancerTypeId);

          Videos = filtered.ToList();

          var selectedService = ServiceTypeId.HasValue ? ServiceTypes.FirstOrDefault(s => s.Id == ServiceTypeId) : null;
          var selectedCancer  = CancerTypeId.HasValue  ? CancerTypes.FirstOrDefault(c => c.Id == CancerTypeId)  : null;

          if (selectedService != null && selectedCancer != null)
              FilterTitle = $" - {selectedService.Title} for {selectedCancer.Title}";
          else if (selectedService != null)
              FilterTitle = $" - {selectedService.Title}";
          else if (selectedCancer != null)
              FilterTitle = $" - {selectedCancer.Title}";

          return Page();
      }
  }

  Razor template dropdowns

  <select class="form-control styled-select bordered" name="serviceTypeId">
      <option value="">All</option>
      @foreach (var svc in Model.ServiceTypes)
      {
          <option value="@svc.Id" selected="@(Model.ServiceTypeId == svc.Id)">@svc.Title</option>
      }
  </select>

  - value="" on the All option — Guid? binding treats empty string as null
  - selected="@(bool)" — Razor renders selected="True" / omits attribute correctly
  - Filter params survive across both dropdowns on each GET submission

  ---
  Notes

  - _api.Content.GetAllAsync<T>() confirmed available in Piranha 12.0.0 (verified via DLL inspection — Piranha.Services.ContentService+<GetAllAsync>d__81`)
  - In-memory filtering is appropriate for content group items (typically small lists) and post archives (no Piranha API for field-level post filtering)
  - Content type registration is automatic — ContentTypeBuilder.AddAssembly() in Program.cs picks up all [ContentType] classes