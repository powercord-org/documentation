<!--
  Copyright (c) 2020-2021 aetheryx & Cynthia K. Rey
  This work is licensed under a Creative Commons Attribution-NoDerivatives 4.0 International License.
  https://creativecommons.org/licenses/by-nd/4.0
-->

# Settings
Plugins are nice, but giving users a way to customize how the plugin work is better. Powercord provides a module
dedicated to allowing users customize the plugin's behavior.

## The basics
### Limits & constrains
Each plugin has a maximum quota of 8MB for its settings storage. This is to ensure plugins cannot go wild and eat
user's disk space. If you attempt to do a set operation that'd exceed the quota, an `Error` will be thrown. The size
is computed based on the compressed JSON representation of all the settings.

Settings are also limited to types that can be stored as JSON. Symbols, functions, bigints, sets, maps aren't supported.
If you attempt to store an unsupported data structure, an `Error` will be thrown.

### Key naming and complex structures
Setting keys can only be strings, but it doesn't mean you have to give up complex structures. Just like in pure
JavaScript, you can dig (or build) structures used our beloved `.` character. For example, take the following
structure:

###### Example settings data
```json
{
  "test": true,
  "badges": {
    "enabled": true,
    "color": "#7289da",
    "types": {
      "a": 0
    }
  }
}
```

Getting `test` or `badges` seem easy enough, but getting `a` may not be as pretty. You may be tempted to get `badges`,
then grab it in code, the classic way. While it works, it causes 2 problems:
 - When you'll use it within a React component, you'll get unnecessary updates due to fields you don't use
 - To set, you need to pass the entire object

Instead, you can use the `badges.types.a` key, and Powercord will automatically work its way out. It'll also do
optional chaining for you, or when doing a set operation generate the whole tree if needed.

>warn
> If you try to use an invalid key (Starting or ending with a dot or with multiple chained dots (`..`)), an `Error`
> will be thrown.

## Registering settings
This step is not strictly required, but strongly recommended. It allows for Powercord to know what can be configured
in your plugin and it's used internally to automatically build a settings UI, as well as adding shortcuts in the
context menu of the setting wheel (the thing you click to open settings. Didn't know it existed? You're welcome).
It also lets you specify default values for all your settings in a single, central place instead of spread around in
the getters.

###### Quick example for registering settings
```js
import { Plugin } from '@powercord/core'
import { registerSettings } from '@powercord/settings'
// Note: @powercord/settings is unavailable in DevTools

class HelloWorld extends Plugin {
  pluginStart () {
    registerSettings({
      entries: [
        {
          key: 'mySetting',
          ...
        },
        ...
      ]
    })
  }
}
```

>info
> Each plugin can only have a single settings UI. Attempting to register twice will just override previously set data.

Because the registration of all the settings also includes setting up the logic for validation, the documentation
will look a bit verbose but it's not very complicated.

### Basic entry properties
 - `label`: Label shown in settings and other places. **Required**.
 - `type`: The [type](#types) of the value. **Required**.
 - `note`: The note (description if you prefer) shown in settings.
 - `default`: The default value for the setting. Affects the settings UI and calls to `getSetting`.

### Types
Each entry must have a valid type to it, so Powercord can generate the appropriate setting field for it and other
internal purposes.

Some types allows you to customize how the user can input the value through additional properties you can set to the
setting entry object. They are documented below for each concerned datatype.

<!-- ideas for the future: string[], string[n], ... -->

#### Strings
All of the string types allow you to specify an `options` properties to the setting entry, turning it into a `select`
or a `set` field. Otherwise, the user will be prompted with a classic text input. Each option is an object that with a
`name` property and a `value` property, that both are self explanatory.

If you want a radio field, you can set the `radio` property to `true` in the settings entry object. For radios, the
options accept 2 additional properties: `color` and `description`.

##### `string`
A basic string, just like all of the others. You can specify a minimum length and/or a maximum length by setting a
`min` and/or a `max` to the setting entry object.

##### `url`
A glorified `string` input that only accepts valid URLs. Prevents you from validating it manually.

#### Numbers
All of the numerical types allow you to specify a set of properties to the setting entry to customize the basic
validation rules and the way the input displays. By default, the field looks like a classic text input.

 - `min` and `max`: Allows you to set boundaries to the value.
 - `slider`: A boolean that indicates if the field should be presented as a slider. **Requires having boundaries set**.
 - `markers`: An array of numbers to add markers to the slider. Ignored if `slider` isn't set to true.
 - `stick`: Only allow values specified in `markers`. **Requires at least 2 markers**. Ignored if `slider` isn't set to true.

##### `integer`
A basic integer. Nothing more to say about it.

>warn
> `min`, `max` and entries in `markers` must all be integers.

##### `float`
A basic IEEE 754 64-bit floating point number with a 53 bits mathematical significand. Also known simply as a float.

#### Miscellaneous
##### `boolean`
A basic boolean. Classic, nothing more to say.

##### `color`
A color field, which will always return a color in the `rgb(a)` form. You can add `alpha: true` to the setting entry
object to get a picker that allows for setting the opacity.

### Data validation
While the built-in types bring some degree of validation, you sometimes want to also have a more specific validation.
For this purpose, you can specify an array or rules in `validationRules` that Powercord will use to validate user input.

Each rule consists of 2 properties: a `rule` property that receives the user input and must return a boolean (keeping
in mind behavior for [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) and
[falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) values), and a `message` property specifying the error
message to display to the user if the check fails.

#### Categories
The type `category` is a specific type of field that allows you to have a collapsible category in your settings UI,
which can help de-clutter your page and make the settings less scary for users.

For this type, the `default` key is unused. You must specify an `entries` properties, that just behaves the same as
the root `entries` property of `registerSettings`.

###### Example of setting entry with data validation
```js
import { Plugin } from '@powercord/core'
import { registerSettings } from '@powercord/settings'

class HelloWorld extends Plugin {
  pluginStart () {
    registerSettings({
      entries: [
        {
          key: 'mySetting',
          type: 'url',
          validationRules: [
            {
              rule: (value) => new URL(value).hostname === 'powercord.dev',
              message: 'URL must be on powercord.dev'
            },
            {
              rule: (value) => !new URL(value).pathname.startsWith('/backoffice'),
              message: 'Admin endpoints are protected'
            }
          ]
        },
        ...
      ]
    })
  }
}
```

>info
> For plugins doing [internationalization](##advanced-plugins/i18n), if you set the `message` field to
> `Messages.MY_STRING`, it will not update when the locale is changed. To handle this case, `message` can be
> a function returning a `string`, so for example `() => Messages.MY_STRING`.

### Dynamically enable/disable, show/hide settings
You can specify for each entry an `isDisabled` function, and an `isHidden` function. For convenience, `isEnabled` is
also available as well as `isVisible` for fields that are by default disabled/hidden, however you **must not** provide
both, or an `Error` will be raised (As Powercord will be unable to decide which one to use).

Functions must return a boolean (keep in mind behavior for [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy)
and [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) values). **The functions cannot be
asynchronous, or return a Promise**. If they do, the result will always be truthy.

>info
> Make sure the footprint of those functions is minimal. Having methods that take a while to execute will result in
> a laggy settings panel.

## Basic read & write
Reading and writing is so easy that you'll just get the code for it:

###### Example read & write
```js
import { getSetting, setSetting } from '@powercord/settings'

// We assume here no default value has been defined

console.log(getSetting('setting.key')) // ~> undefined
console.log(getSetting('setting.key', 'uwu')) // ~> "uwu"
setSetting('setting.key', 'owo')
console.log(getSetting('setting.key', 'uwu')) // ~> "owo"

// If you need to update setting based on its previous value
// (e.g: toggling a boolean), you can pass a function to
// setSetting, to avoid race conditions
setSetting('test.key', (v) => !v)

// You can even supply a default value if necessary, just like in get:
setSetting('test.key', (v) => !v, true) // ~> sets to false
```

### Within React components
Within components, the use of the classic `getSetting` and/or `setSetting` is discouraged. Instead, you should use the
`useSetting` hook. This will allow your component to immediately update when a change occurs, without having to craft
the whole logic around it. The hook behaves similarly to [`useState`](https://reactjs.org/docs/hooks-state.html),
except it takes the setting key as parameter.

###### Example use of useSetting hook
```js
import React from 'react'
import { useSetting } from '@powercord/settings'

export default function MyComponent () {
  // You can specify a default value, just like in classic get.
  const [ value, setValue ] = useSetting('key')

  return (
    <p>The value of the <code>key</code> setting is {value}!</p>
  )
}
```

## Subscribe to changes
Plugins can listen to changes using the `SettingsDispatcher` object exported by `@powercord/settings`. It's useful
to handle cases where you need to execute a piece of code, or change some bits that aren't React components where
`useSetting` would handle everything for you. Also, fun fact `useSetting` actually uses the `SettingsDispatcher`
internally. Crazy!

>warn
> Unlike injections, this does **not** clean up automatically. You must unsubscribe yourself!

###### Subscribe & unsubscribe to changes
```js
import { SettingsDispatcher } from '@powercord/settings'

function handler (key, previous, next) {
  console.log(`Setting ${key} was updated from ${previous} to ${next}!`)
}

SettingsDispatcher.subscribe('settingKey', handler)

...

SettingsDispatcher.unsubscribe('settingKey', handler)
```

You can also subscribe to `*`, which will be fired whenever any setting is updated. The `key` value will still be
the name of the setting node that has been updated.

For cases where a structure is updated, the event will bubble up, so parent nodes will receive the update too. This
is useful for when you want to observe an entire structure. Just like for the `*` event, the `key` value will still be
the name of the setting node that has been updated.

>node
> While the events bubble up, they don't bubble down to children. Refer to the examples below.

###### Example deep structure
```json
{
  "owo": 1,
  "a": {
    "uwu": 2,
    "b": {
      "c": 5
    }
  }
}
```

###### Example deep structure update 1
```js
import { setSetting, SettingsDispatcher } from '@powercord/settings'

setSetting('a.b.c', 1337)
// "a.b.c" emitted with: key = a.b.c, previous = 5, next = 1337
// "a.b" emitted with: key = a.b.c, previous = 5, next = 1337
// "a" emitted with: key = a.b.c, previous = 5, next = 1337
// "*" emitted with: key = a.b.c, previous = 5, next = 1337
```

###### Example deep structure update 2
```js
import { setSetting, SettingsDispatcher } from '@powercord/settings'

setSetting('a.b', { c: 1337 })
// "a.b" emitted with: key = a.b, previous = { c: 5 }, next = { c: 1337 }
// "a" emitted with: key = a.b, previous = { c: 5 }, next = { c: 1337 }
// "*" emitted with: key = a.b, previous = { c: 5 }, next = { c: 1337 }
```

## Create a UI
You can decide to use a custom UI for your settings page, and build it from scratch to have more control over it,
or add additional decoration like a preview. This is done in `registerSettings`:

###### Use a custom UI
```js
import { registerSettings } from '@powercord/settings'
import Settings from './Settings'

registerSettings({
  ui: Settings,
  entries: [ ... ]
})
```

The component will replace the one Powercord would have used. However, if you just want to add a preview no worries!
You can still get the UI Powercord would have inserted using `children`. To ensure Powercord stays speedy and don't
render useless things, the children is a function you need to call (so it's only rendering when needed).

###### Example re-use of the automatic settings UI
```js
import React from 'react'

function Preview () {
  return (
    <div className='my-amazing-preview'>
      ...
    </div>
  )
}

export default function Settings ({ children }) {
  return (
    <div>
      <Preview/>
      {children()}
    </div>
  )
}
```
