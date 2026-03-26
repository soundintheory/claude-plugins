---
name: custom-field-types
description: This skill should be used when the user asks to create a custom field, or is working on an existing field type in Piranha CMS project
version: 1.0.0
---

# Piranha CMS Custom Field Types

Piranha allows you to define your own custom field types which consist of a type, an optional serializer to deal with database 
serialization / deserialization and a vue component to use when rendering the field in the manager UI

## The FieldType attribute

Every custom field type must have the [FieldType] attribute (FQN: `Piranha.Extend.FieldTypeAttribute`). These are the properties:

- `Name`: The display name for the field type
- `Shorthand`: Optional shorthand name. Only needed if the field name is particularly long or complex
- `Component`: The name of the vue component to use when rendering the field in the manager UI

## Simple field types

When the underlying value of the field has a simple type, (eg. string, decimal, int), Piranha provides a generic base class
`Piranha.Extend.SimpleField<T>` containing a `Value` property.

The implicit operators are optional but recommended — they allow the field to be assigned and compared as if it were its underlying type (e.g. `field == "#ff0000"` rather than `field.Value == "#ff0000"`). For example:

```csharp
using Piranha.Extend;

[FieldType(Name = "Color", Shorthand = "Color", Component = "color-field")]
public class ColorField : SimpleField<string>
{
    /// <summary>
    /// Implicit operator for converting a string to a field.
    /// </summary>
    public static implicit operator ColorField(string str)
    {
        return new ColorField { Value = str };
    }

    /// <summary>
    /// Implicitly converts the color field to a string.
    /// </summary>
    public static implicit operator string(ColorField field)
    {
        return field.Value;
    }
}
```

## Complex field types

For fields that store non simple data types or multiple properties, you only need to implement `Piranha.Extend.IField`. For example:

```csharp
using Piranha.Extend;

[FieldType(Name = "User Review Field", Shorthand = "User Review", Component = "user-review-field")]
public class UserReviewField : IField
{
    public string Name { get; set; }

    public int Stars { get; set; }

    public string Comment { get; set; }

    /// <summary>
    /// Gets the label shown for this field when it appears as a row in a collection region
    /// (e.g. a repeatable list of items in the Piranha manager). Return a meaningful
    /// human-readable string so editors can identify individual entries at a glance.
    /// </summary>
    public virtual string GetTitle()
    {
        return Name;
    }
}
```

## Registering field types

All field types must be registered in Program.cs after the `App.Init()` call:

```csharp
Piranha.App.Fields.Register<UserReviewField>();
```

## Serializers

By default, the whole field object will get JSON serialized (and deserialized from) the database value. You
can customise this by defining a serializer for your field. Serializers are also useful if there is potentially
old, varied or corrupted data already stored in the database as it will allow you to correct it and prevent deserialization errors. For example:

```csharp
using Piranha.Extend;

public class DecimalFieldSerializer : ISerializer
{
    /// <summary>
    /// Serializes the given object.
    /// </summary>
    /// <param name="obj">The object</param>
    /// <returns>The serialized value</returns>
    public string Serialize(object obj)
    {
        if (obj is DecimalField field)
        {
            return field.Value.ToString();
        }
        throw new ArgumentException("The given object doesn't match the serialization type");
    }

    /// <summary>
    /// Deserializes the given string.
    /// </summary>
    /// <param name="str">The serialized value</param>
    /// <returns>The object</returns>
    public object Deserialize(string str)
    {
        var ret = Activator.CreateInstance<DecimalField>();

        if (decimal.TryParse(str, out var val))
        {
            ret.Value = val;
        }
        else
        {
            // Possible old data format, could add custom logic here to fix it
        }

        return ret;
    }
}
```

Then you need to register the serializer against your field type in your Program.cs after the `App.Init()` call:

```csharp
Piranha.App.Serializers.Register<DecimalField>(new DecimalFieldSerializer());
```

## Init methods

Piranha provides a way to execute logic when a field is initialized, both in the manager and on the front end. These methods
are defined on the field type and get executed via reflection. They can also accept DI registered services as parameters. For example:

```csharp
using Piranha.Extend;

[FieldType(Name = "Product Link Field", Shorthand = "Product Link", Component = "product-link-field")]
public class ProductLinkField : IField
{
    public int Id { get; set; }

    // A custom serializer will ensure that this runtime property doesn't get unintentionally serialized to the database
    public IProduct Product { get; set; }

    public string Url => Product?.Url ?? "";

    /// <summary>
    /// Gets the string representation of this field if used in collection regions.
    /// </summary>
    public virtual string GetTitle()
    {
        return Product?.Title ?? "(empty)";
    }

    // Init runs on both the front end and the manager. Use it to populate runtime
    // properties (like Product here) that shouldn't be stored in the database.
    public virtual async Task Init(IProductService productService)
    {
        if (Id > 0)
        {
            Product = productService.Get(Id);
        }
    }

    // ManagerInit runs only when the field is loaded in the Piranha manager UI.
    // Only add this if the manager needs different or additional data compared to
    // the front end (e.g. extra fields for the editor, unpublished data, etc.).
    public virtual async Task ManagerInit(IProductService productService)
    {
        if (Id > 0)
        {
            Product = productService.GetForManager(Id);
        }
    }
}
```

## Field Vue Component

When Piranha renders your field in the manager UI, it expects there to be a Vue 2 component registered with the name
you specified in the `FieldType` attribute. It is up to the user to add the required custom JS and CSS in the manager
to make this available.

We usually have a folder containing these vue components somewhere like `resources/assets/manager/js/components` that is
built by Vite and auto registers the vue components by their file name (eg. `resources/assets/manager/js/components/my-field.vue`
would be registered as `my-field`).

**If you cannot easily find the location for these manager vue components:** ask the user where they want you to put them
before searching the entire project.

The built assets are usually already registered in Program.cs like this:

```csharp
App.Modules.Manager().Styles.Add("~/assets/manager/css/app.css?v=2");
App.Modules.Manager().Scripts.Add("~/assets/manager/js/app.js?v=2");
```

**If you cannot find this registration in Program.cs:** ask the user whether they want you to set up the asset pipeline before writing the component. If they say no, write the Vue component in the most logical location you can find and note where you've put it so they can wire it up themselves.

Here is an example of the built in StringField Vue component for reference:

```
<template>
    <div>
        <div v-if="maxLength() > 0" class="input-group">
            <input class="form-control" type="text" :maxlength="maxLength()" :required="isRequired()" :placeholder="meta.placeholder" v-model="model.value" v-on:change="update()">
            <div class="input-group-append">
                <div class="input-group-text text-muted">
                    {{ piranha.utils.strLength(model.value) + "/" + maxLength() }}
                </div>
            </div>
        </div>
        <input v-else class="form-control" type="text" :maxlength="maxLength()" :required="isRequired()" :placeholder="meta.placeholder" v-model="model.value" v-on:change="update()">
    </div>
</template>

<script>
export default {
    // Note: the "model" prop is what will be sent to the server when the form is submitted
    props: ["uid", "model", "meta"],
    methods: {
        update: function () {
            // Tell parent that title has been updated
            if (this.meta.notifyChange) {
                this.$emit('update-title', {
                    uid: this.uid,
                    title: this.model.value
                });
            }
        },
        maxLength: function () {
            // Note: this.meta.settings is provided by field settings - see "Field Settings" section
            return this.meta.settings.MaxLength != null && this.meta.settings.MaxLength > 0 ?
                this.meta.settings.MaxLength : null;
        },
        isRequired: function () {
            return false;
        }
    }
}
</script>
```

## Field Settings

Custom field settings can be provided by defining a field settings attribute. For example:

```csharp
using Piranha.Extend;

public class UserReviewFieldSettingsAttribute : FieldSettingsAttribute
{
    public bool HideUserName { get; set; }
}
```

This would then be used on a Piranha model like this:

```csharp
[Field]
[UserReviewFieldSettings(HideUserName = false)]
public UserReviewField UserReview { get; set; }
```

## Summary Checklist

When adding a new custom field type:

- [ ] Create the field type class with the `[FieldType]` attribute
- [ ] Determine whether any init methods are required on load
- [ ] Register the field type in Program.cs
- [ ] Create and register a custom serializer if required
- [ ] Create a custom field settings attribute if required
- [ ] Create the vue component for the field
