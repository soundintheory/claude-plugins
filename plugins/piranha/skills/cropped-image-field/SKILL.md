---
name: cropped-image-field
description: This skill provides comprehensive guidance on using the `CroppedImageField` from SoundInTheory.Piranha.MediaExtensions.Images
version: 1.0.0
---

# Piranha CMS CroppedImageField Usage Guide

This skill provides comprehensive guidance on using the `CroppedImageField` from SoundInTheory.Piranha.MediaExtensions.Images

## Package & Imports

```csharp
using SoundInTheory.Piranha.MediaExtensions.Images;
using SoundInTheory.Piranha.MediaExtensions.Images.Fields;
```

For templates, also import:
```cshtml
@using SoundInTheory.Piranha.MediaExtensions.Images.Model
```

## Field Configuration Patterns

### 1. Single Aspect Ratio (Most Common)

Use when an image should maintain a specific aspect ratio:

```csharp
[Field]
[CroppedImageFieldSettings(AspectRatio = 1520d / 380d)]
public CroppedImageField Image { get; set; }
```

**Examples from codebase:**
- `AspectRatio = 1520d / 380d` - Wide banner (HeroBannerImage.cs:12)
- `AspectRatio = 595d / 255d` - Card image (Card.cs:26)
- `AspectRatio = 115d / 96d` - Photo slider (PhotoSlider.cs:13)
- `AspectRatio = 833d / 360d` - Promotional banner (LocationHeroRegion.cs:58)
- `AspectRatio = 1d / 1d` - Square image (SupportingImages.cs:29)
- `AspectRatio = 1400d / 933d` - Card image (LocationAttractionRegion.cs:16)
- `AspectRatio = 1520d / 836d` - Main attraction image (LocationAttractionRegion.cs:20)

**Always use `d` suffix for double precision** in aspect ratio calculations.

### 2. Multiple Aspect Ratios with Named Crops

Use when different crops are needed for different screen sizes/contexts:

```csharp
[Field]
[CroppedImageFieldSettings(
    AspectRatios = new double[] { 1520d / 552d, 2d / 1d, 480d / 542d },
    Crops = new string[] { "Large Letterbox", "Smaller Monitor", "Mobile" }
)]
public CroppedImageField Image { get; set; }
```

**From LocationHeroRegion.cs:11** - The arrays must have the same length. Each crop name corresponds to its aspect ratio by index.

### 3. No Settings (Flexible Cropping)

Use when any aspect ratio is acceptable:

```csharp
[Field]
public CroppedImageField Image { get; set; }
```

**Example:** PromotionalBanner.cs:11

## Template Usage with ImageCrop.CropImage()

The `ImageCrop.CropImage()` method is the primary way to render cropped images in templates.

### Method Signatures

```csharp
// With named crop and width
ImageCrop.CropImage(CroppedImageField field, string cropName, int width)

// With width and height
ImageCrop.CropImage(CroppedImageField field, int width, int height)

// Width only
ImageCrop.CropImage(CroppedImageField field, int width)

// Height only (width = null)
ImageCrop.CropImage(CroppedImageField field, null, int height)
```

### Using Named Crops

When the field has named crops defined:

```cshtml
@* Named crop with width *@
<source srcset="@Url.Content(ImageCrop.CropImage(Model.Hero.Image, "Large Letterbox", 1520))"
        media="(min-width: 1200px)" />
<source srcset="@Url.Content(ImageCrop.CropImage(Model.Hero.Image, "Smaller Monitor", 1199))"
        media="(min-width: 481px)" />
<source srcset="@Url.Content(ImageCrop.CropImage(Model.Hero.Image, "Mobile", 480))"
        media="(min-width: 1px)" />
```

**From LocationHeroBanner.cshtml:10-12**

### Using Width and Height

For single aspect ratio fields:

```cshtml
@* Width and height *@
<img src="@Url.Content(ImageCrop.CropImage(Model.Image, 1520, 380))" />
```

**From HeroBannerImage.cshtml:7**

### Width Only

Most common pattern:

```cshtml
@* Maintains aspect ratio, scales to width *@
<img src="@Url.Content(ImageCrop.CropImage(card.Image, 595))" />
```

**From ThreeCards.cshtml:37**

### Height Only

Use null for width parameter:

```cshtml
@* Maintains aspect ratio, scales to height *@
<source srcset="@Url.Content(ImageCrop.CropImage(slider.Image, null, 189))"
        media="(min-width: 1700px)" />
```

**From PhotoSliderGallery.cshtml:34**

## Responsive Images Pattern

Standard pattern using `<picture>` element for responsive images:

```cshtml
<picture>
    <source srcset="@Url.Content(ImageCrop.CropImage(Model.Image, 1520, 380))"
            media="(min-width: 992px)" />
    <source srcset="@Url.Content(ImageCrop.CropImage(Model.Image, 991, 380))"
            media="(min-width: 768px)" />
    <source srcset="@Url.Content(ImageCrop.CropImage(Model.Image, 767, 380))"
            media="(min-width: 480px)" />
    <img class="img-fit"
         alt="@Model.Image?.Media?.AltText"
         src="@Url.Content(ImageCrop.CropImage(Model.Image, 479, 380))"
         asp-append-version="true" />
</picture>
```

**From HeroBannerImage.cshtml:6-12**

### High DPI (Retina) Support

Use srcset with pixel density descriptors:

```cshtml
<source srcset="@Url.Content(ImageCrop.CropImage(Model.Hero.Image, "Mobile", 480)) 1x,
                @Url.Content(ImageCrop.CropImage(Model.Hero.Image, "Mobile", 960)) 2x"
        media="(min-width: 1px)" />
```

**From LocationHeroBanner.cshtml:12** - 2x provides double resolution for retina displays.

## Checking for Values

Always check if the field has a value before using:

```cshtml
@if (Model.Hero?.Image?.HasValue == true)
{
    <picture>
        <img src="@Url.Content(ImageCrop.CropImage(Model.Hero.Image, 1520))" />
    </picture>
}
```

**From LocationHeroBanner.cshtml:7** and **HeroBannerImage.cshtml:7**

### Ternary Pattern

```cshtml
<img src="@(Model.Image?.HasValue == true ?
            Url.Content(ImageCrop.CropImage(Model.Image, 1520, 380)) :
            "")" />
```

**From HeroBannerImage.cshtml:10**

Or for optional rendering:

```cshtml
src="@(slider.Image.HasValue ? Url.Content(ImageCrop.CropImage(slider.Image, 1200)) : "")"
```

**From PhotoSliderGallery.cshtml:29**

## Accessing Image Metadata

The CroppedImageField exposes the underlying Media object:

```cshtml
@* Alt text *@
alt="@(Model.Image?.Media?.AltText)"

@* With fallback *@
alt="@(Model.Hero?.Image?.Media?.AltText ?? Model.Hero.Title)"

@* Length check *@
@(card.Image.Media?.AltText?.Length > 0 ?
  card.Image.Media?.AltText :
  card.Title + " at Company Name")
```

**Examples from:**
- LocationHeroBanner.cshtml:13
- ThreeCards.cshtml:38
- LocationAttractions.cshtml:48

## Background Image Pattern

For CSS background images:

```cshtml
<div role="img"
     aria-label="@Model.Image.Media.AltText"
     loading="lazy"
     data-style="background-color: #fff; background-image: url('@Url.Content(ImageCrop.CropImage(Model.Image, 982))')">
</div>
```

**From PromotionalBanner.cshtml:29-34** - Note the use of `role="img"` and `aria-label` for accessibility.

## Calculated/Conditional Image Properties

You can create computed properties that return a CroppedImageField:

```csharp
public CroppedImageField CalculatedMainImageToUse
{
    get
    {
        if (this.MainImage.HasValue)
            return this.MainImage;
        else if (this.Attraction.Value.Model != null &&
                 this.Attraction.Value.Model.AttractionInfo.MainImage.HasValue)
            return this.Attraction.Value.Model.AttractionInfo.MainImage;
        else
            return new CroppedImageField();
    }
}
```

**From LocationAttractionRegion.cs:26-41** - Useful for override patterns.

## Common Pitfalls

1. **Forgetting the `d` suffix** in aspect ratio calculations
   - ❌ `AspectRatio = 1520 / 380`
   - ✅ `AspectRatio = 1520d / 380d`

2. **Not checking HasValue** before rendering
   - Always use `?.HasValue == true` or null-coalescing

3. **Mismatched crop arrays**
   - `AspectRatios` and `Crops` arrays must be the same length

4. **Not wrapping with Url.Content()**
   - Always use `@Url.Content(ImageCrop.CropImage(...))`

5. **Missing alt text accessibility**
   - Always provide alt text from `Media.AltText` or a meaningful fallback

## Field Attributes

Common field attributes used with CroppedImageField:

```csharp
[Field(Title = "Custom Title")]
[Field(Title = "Hero Image", Description = "This appears in the banner")]
[CroppedImageFieldSettings(AspectRatio = 16d / 9d)]
public CroppedImageField Image { get; set; }
```

**From various model files** - Title and Description help content editors understand the field's purpose.

## Summary Checklist

When adding a new CroppedImageField:

- [ ] Add using statements for SoundInTheory.Piranha.MediaExtensions.Images
- [ ] Decorate with `[Field]` attribute
- [ ] Add `[CroppedImageFieldSettings]` if aspect ratio constraints are needed
- [ ] Use `d` suffix in aspect ratio calculations
- [ ] In templates, check `HasValue` before rendering
- [ ] Wrap ImageCrop.CropImage() calls with Url.Content()
- [ ] Provide alt text from Media.AltText with meaningful fallback
- [ ] Consider responsive images with `<picture>` element
- [ ] Consider high DPI displays with 2x srcset where appropriate
