# JetTheme

<p>
  <img src="https://github.com/lcdsmao/JetTheme/workflows/Android%20CI/badge.svg"/>
  <a href="https://bintray.com/lcdsmao/maven/jettheme/_latestVersion">
    <img src="https://api.bintray.com/packages/lcdsmao/maven/jettheme/images/download.svg"/>
  </a>
</p>

JetTheme is a flexible theme provider for Jetpack Compose.

- Change the theme and recompose the UI dynamically.
- Save theme preference to local storage.
- Support custom design system.

## Installation

```gradle
dependencies {

  // Use this if you want material design support (recommended)
  implementation "dev.lcdsmao.jettheme:jettheme-material:$latestVersion"

  // Use this if you want to build custom design system
  implementation "dev.lcdsmao.jettheme:jettheme:$latestVersion"
}
```

## Basic Usage

Material Design out of the box.

1. Define your app theme using `buildMaterialTheme`:

```kotlin
val AppTheme = buildMaterialThemePack {
  defaultMaterialTheme(
    colors = lightColors(...),
    typography = Typography(...),
    shapes = Shapes(...),
  )
  materialTheme(
    id = darkId,
    colors = darkColors(...),
  )
  materialTheme(
    id = "other_theme",
    colors = otherColors(...),
  )
}
```

2. Wrap your app using `ProvideAppMaterialTheme`:

```kotlin
@Composable
fun App() {
  ProvideAppMaterialTheme(appTheme) {
    // children
  }
}
```

3. Change theme using `ThemeControllerAmbient`:

```kotlin
val themeController = ThemeControllerAmbient.current
themeController.setDarkTheme()
themeController.setThemeId("other_theme")
```

4. Access current theme values using `MaterialTheme` (from `androidx.compose.material`):

```kotlin
Surface(
  color = MaterialTheme.colors.primary,
)
```

## Advanced Usage

### Custom Design System

You can use JetTheme to implement your own custom design system.

For example, we want to create a Simple Design System which has different color scheme from Material Design.
The color scheme contains two colors, the background color and text color.

1. Define the `ThemeSpec` of our Simple Design System:

```kotlin
data class SimpleThemeSpec(
  override val id: String,
  val colors: SimpleColors,
  val typography: Typography = Typography(),
  val shapes: Shapes = Shapes(),
) : ThemeSpec

class SimpleColors(
  background: Color,
  text: Color,
  isDark: Boolean,
) {
  var background by mutableStateOf(background)
    private set
  var text by mutableStateOf(text)
    private set
  var isDark by mutableStateOf(isDark)
    private set

  fun update(other: SimpleColors) {
    background = other.background
    text = other.text
    isDark = other.isDark
  }
}
```

2. Construct a `ThemePack` with the `SimpleThemeSpec`:

```kotlin
val AppTheme = buildThemePack {
  theme(
    SimpleThemeSpec(
      id = defaultId,
      colors = LightColorPalette,
    )
  )
  theme(
    SimpleThemeSpec(
      id = darkId,
      colors = DarkColorPalette,
    )
  )
}
```

3. Create a `SimpleTheme` which used to retrieve current theme colors:

```kotlin
private val AmbientSimpleColors = staticAmbientOf<SimpleColors>()

object SimpleTheme {
  @Composable
  val colors: SimpleColors
    get() = AmbientSimpleColors.current
}
```

4. Create a Simple Design System provider by the `ProvideAppTheme`:

```kotlin
@Composable
fun ProvideSimpleTheme(
  content: @Composable () -> Unit,
) {
  ProvideAppTheme<SimpleThemeSpec>(
    themePack = AppTheme,
  ) { theme ->
    val colorPalette = remember { theme.colors }
    colorPalette.update(theme.colors)
    Providers(AmbientSimpleColors provides colorPalette) {
      MaterialTheme(
        colors = debugColors(theme.colors.isDark),
        typography = theme.typography,
        shapes = theme.shapes,
        content = content,
      )
    }
  }
}

// Sets all colors to [debugColor] to discourage usage of [MaterialTheme.colors]
// in preference to [SimpleTheme.colors].
private fun debugColors(
    darkTheme: Boolean,
    debugColor: Color = Color.Magenta
) = Colors(
    primary = debugColor,
    primaryVariant = debugColor,
    secondary = debugColor,
    secondaryVariant = debugColor,
    background = debugColor,
    surface = debugColor,
    error = debugColor,
    onPrimary = debugColor,
    onSecondary = debugColor,
    onBackground = debugColor,
    onSurface = debugColor,
    onError = debugColor,
    isLight = !darkTheme
)
```

5. Theme our component with `SimpleTheme`:

```kotlin
@Composable
fun CustomDesignSystemApp() {
  ProvideSimpleTheme {
    Box {
      val (themeId, setThemeId) = ThemeControllerAmbient.current
      Text(
        "Custom Design System",
        modifier = Modifier.clickable(onClick = { setThemeId(AppTheme.nextThemeId(themeId)) }),
        color = SimpleTheme.colors.text
      )
    }
  }
}
```

## Contributing

Feel free to open a issue or submit a pull request for any bugs/improvements.
