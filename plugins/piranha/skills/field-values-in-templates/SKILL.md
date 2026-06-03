---
name: piranha-field-values-in-templates
description: How to access Piranha field values in Razor templates — .Value, Html.Raw, and field-type gotchas
version: 1.0.0
---

  # Piranha Field Values in Templates — AI Reference Guide

  Piranha fields are wrapper objects, not raw values. You must unwrap them to get the underlying value in Razor templates.

  ---

  ## The `.Value` Pattern

  Most Piranha field types expose a `.Value` property for the underlying primitive:

  ```razor
  @* StringField *@
  @item.Info.VideoUrl.Value

  @* TextField *@
  @item.Info.Description.Value

  @* NumberField *@
  @item.Info.SomeNumber.Value

  Common mistake: Writing @item.Info.VideoUrl outputs the field object's ToString(), which may render as empty or a type name rather than the actual string
  value.

  ---
  HtmlField — Use Html.Raw()

  HtmlField stores HTML markup. Render it with @Html.Raw() to avoid double-encoding:

  @Html.Raw(item.Info.Description.Value)

  Without Html.Raw, angle brackets are escaped and raw HTML tags appear as text on screen.

  ---
  Field Types Quick Reference

  ┌───────────────────┬─────────────────────────────────────────┬──────────────────────────────┐
  │    Field type     │             Access pattern              │            Notes             │
  ├───────────────────┼─────────────────────────────────────────┼──────────────────────────────┤
  │ StringField       │ .Value → string                         │ Plain text                   │
  ├───────────────────┼─────────────────────────────────────────┼──────────────────────────────┤
  │ TextField         │ .Value → string                         │ Multi-line plain text        │
  ├───────────────────┼─────────────────────────────────────────┼──────────────────────────────┤
  │ HtmlField         │ .Value → string                         │ Must use @Html.Raw()         │
  ├───────────────────┼─────────────────────────────────────────┼──────────────────────────────┤
  │ NumberField       │ .Value → int                            │                              │
  ├───────────────────┼─────────────────────────────────────────┼──────────────────────────────┤
  │ CheckBoxField     │ .Value → bool                           │                              │
  ├───────────────────┼─────────────────────────────────────────┼──────────────────────────────┤
  │ DateField         │ .Value → DateTime?                      │                              │
  ├───────────────────┼─────────────────────────────────────────┼──────────────────────────────┤
  │ ContentField      │ .HasValue + .Id                         │ No .Value string — see below │
  ├───────────────────┼─────────────────────────────────────────┼──────────────────────────────┤
  │ LinkField         │ .HasValue + .Url + .Text                │ See CLAUDE.md                │
  ├───────────────────┼─────────────────────────────────────────┼──────────────────────────────┤
  │ CroppedImageField │ See /skill piranha--cropped-image-field │                              │
  └───────────────────┴─────────────────────────────────────────┴──────────────────────────────┘

  ---
  ContentField — Different Pattern

  ContentField is a reference to a Content<T> item, not a string wrapper. It does not have a .Value string property:

  // Check if a content item has been selected
  v.Info.CancerType.HasValue   // bool

  // Get the ID of the referenced content item (for filtering/comparison)
  v.Info.CancerType.Id         // Guid?

  To display the title of the referenced item, load the content items separately and look up by ID — see the piranha-content-api skill.

  ---
  Checking Before Rendering

  Use .HasValue or null-checks before rendering optional fields:

  @* ContentField *@
  @if (item.Info.CancerType.HasValue) { ... }

  @* StringField — check the unwrapped value *@
  @if (!string.IsNullOrEmpty(item.Info.VideoUrl.Value)) { ... }

  The template-level .Where() guard pattern for collections:

  @foreach (var item in Model.Videos.Where(v => !string.IsNullOrEmpty(v.Info.VideoUrl.Value)))
  {
      <iframe src="@item.Info.VideoUrl.Value" ...></iframe>
  }