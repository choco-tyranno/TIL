# Auto sizing text

> ​	Three ways provided(by The Support Library 26.0).
>
> + Default
> + Granularity
> + Preset Sizes

> ​	:warning: "wrap_content" for layout_width or layout_height attribute is not recommended.
>
> ​	It may produce unexpected results.



## Default

> The default dimensions for uniform scaling are 
> minTextSize = 12sp, 
> maxTextSize = 112sp, and 
> granularity = 1px.

### XML expression

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:autoSizeTextType="uniform" />
```

### Programmatical expression

`````java
TextViewCompat.setAutoSizeTextTypeWithDefaults(TextView textview, int autoSizeTextType)
`````





## Granularity

> It defining a range of minimum and maximum text sizes 
> and a dimension that specifies the size of each step.

### XML expression

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:autoSizeTextType="uniform"
    android:autoSizeMinTextSize="12sp"
    android:autoSizeMaxTextSize="100sp"
    android:autoSizeStepGranularity="2sp" />
```

### Programmatical expression

`````java
setAutoSizeTextTypeUniformWithConfiguration(int autoSizeMinTextSize, int autoSizeMaxTextSize, int autoSizeStepGranularity, int unit)
`````





## Preset Sizes

> Specifying all the values.

### Add array resource

```xml
<resources>
  <array name="autosize_text_sizes">
    <item>10sp</item>
    <item>12sp</item>
    <item>20sp</item>
    <item>40sp</item>
    <item>100sp</item>
  </array>
</resources>

//in res/values/arrays.xml
```

### XML expression

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView
    android:layout_width="match_parent"
    android:layout_height="200dp"
    android:autoSizeTextType="uniform"
    android:autoSizePresetSizes="@array/autosize_text_sizes" />
```

### Programmatical expression

`````java
setAutoSizeTextTypeUniformWithPresetSizes(int[] presetSizes, int unit)
`````

