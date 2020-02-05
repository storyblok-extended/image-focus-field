# Storyblok Extended: Number range

> A Storyblok field to create inclusive and exclusive number ranges.

## Input options

Input options to customize the plugin.

#### slider

Type: `'true' | false`

Enable the HTML5 `range` input type to create a native slider.
Valid only if the input is a valid range (`min` and `max` are provided).

#### step

Type: `number | 1`

Sets the range step using the HTML5 `step` attribute.

#### min

Type: `number | null`

Sets the range minimum using the HTML5 `min` attribute.

#### max

Type: `number | null`

Sets the range maximum using the HTML5 `max` attribute.

#### value

Type: `number | <valueof min> | null`

Sets the range initial value using the HTML `value` attribute.

#### placeholder

Type: `string | null`

Sets the input placeholder using the HTML `placeholder` attribute.

## Project setup

```
npm install
```

### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Lints and fixes files

```
npm run lint
```
