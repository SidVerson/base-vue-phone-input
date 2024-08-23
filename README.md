# Base vue phone input

Basic phone input deeply customizable with slots

## Installation

using npm

```bash
  npm install basic-vue-phone-input
```

## Examples

[Demo with shadcn/vue](https://shadcn-vue-phone-input.vercel.app/)

## Usage

```javascript
<script setup lang="ts">
import { ref } from 'vue'
import PhoneInput from './index'

const tel = ref('')
const res = ref()
</script>

<template>
    <PhoneInput
        v-model="tel"// for phone number
        @update="res = $event"// for phone number object />
    {{ res }}
    {{ tel }}
</template>

```

But this input is meant to be customized

## Features

-   You can set preferred-countries, ignored-countries or have only-countries
-   Multi options to getting country code : By default the component get the country code via the browser (disable it with no-use-browser-locale) or you can use fetch-country to get the country code via https://ip2c.org/s (network needed) - you can use default-country-code option instead to set one
-   Phone number formatting while typing
-   You can search your country in list (open countries list & type your country name)
-   Keyboard accessibility (Arrow down, Arrow up: Countries list navigation - Escape: Close countries list)
-   Phone number example for each country in placeholder
-   Fully cusstomizable with slots

## Props API

| Prop name             | Description                                                                                                                                         | Type        | Values | Default   |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ------ | --------- |
| modelValue            | `@model` Country calling code + telephone number in international format                                                                            | string      | \-     | undefined |
| countryCode           | `@model` Country code selected - Ex: "FR"                                                                                                           | CountryCode | \-     | undefined |
| placeholder           | Placeholder of the input                                                                                                                            | string      | \-     | undefined |
| label                 | label of the input                                                                                                                                  | string      | \-     | undefined |
| preferredCountries    | List of country codes to place first in the select list - Ex: \['FR', 'BE', 'GE'\]                                                                  | Array       | \-     | undefined |
| ignoredCountries      | List of country codes to be removed from the select list - Ex: \['FR', 'BE', 'GE'\]                                                                 | Array       | \-     | undefined |
| onlyCountries         | List of country codes to only have the countries selected in the select list - Ex: \['FR', 'BE', 'GE'\]                                             | Array       | \-     | undefined |
| translations          | Locale strings of the component                                                                                                                     | Partial     | \-     | undefined |
| noUseBrowserLocale    | By default the component use the browser locale to set the default country code if not country code is provided                                     | boolean     | \-     |           |
| fetchCountry          | The component will make a request ([https://ipwho.is](https://ipwho.is)) to get the location of the user and use it to set the default country code | boolean     | \-     |           |
| customCountriesList   | Replace country names                                                                                                                               | Record      | \-     | undefined |
| autoFormat            | Disabled auto-format when phone is valid <br>`@default` true                                                                                        | boolean     | \-     | true      |
| noFormattingAsYouType | Disabled auto-format as you type <br>`@default` false                                                                                               | boolean     | \-     | false     |
| countryLocale         | locale of country list - Ex: "fr-FR" <br>`@default` {string} browser locale                                                                         | string      | \-     | undefined |
| excludeSelectors      | Exclude selectors to close country selector list - usefull when you using custom flag                                                               | Array       | \-     | undefined |

## Events API

| Event | Return |
| --- | --- |
| input | [AsYouType value](https://github.com/catamphetamine/libphonenumber-js#as-you-type-formatter) (emit when new value is enter on phone number input && when a country is choosed) |
| update | All values (Result type) |

## Keyboard accessibility

| Props     | Action                            |
| --------- | --------------------------------- |
| ArrowDown | Navigation down in countries list |
| ArrowUp   | Navigation up in countries list   |
| Escape    | Close countries list              |

## Named slots

| Name     | Description                           | Bindings                                                                                                                                                                                     |
| -------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| selector | Change the country selector component | **input-value** `String` - current selected country code - Ex: `"FR"` **updateInputValue** - action for changing country code by passing country code **countries** - array of all countries |
| input    | Change the input component            | **input-value** `String` - phone number value **updateInputValue** - action for updating phone value **placeholder** - placeholder                                                           |

## Translations

### Labels & placeholders

```javascript
<PhoneInput
  :translations="{
    phoneInput: {
      placeholder: 'Phone number',
      example: 'Example:',
    },
  }"
/>

```

### Country list

Two ways to translate the country list:

#### First solution - set country locale

```javascript
<PhoneInput country-locale='fr-FR' />
```

#### Second solution - custom list

```javascript
<PhoneInput
  :custom-countries-list="{
    FR: 'France',
    BE: 'Belgique',
    DE: 'Allemagne',
    US: 'Etats-Unis',
  }"
/>

```

## Types

Results emitted by @update or @data event

```javascript
export type Results = {
  isValid: boolean
  isPossible?: boolean
  countryCode?: CountryCode
  countryCallingCode?: CountryCallingCode
  nationalNumber?: NationalNumber
  type?: NumberType
  formatInternational?: string
  formatNational?: string
  uri?: string
  e164?: string
  rfc3966?: string
}

```

Country type

```javascript
export interface Country {
    iso2: CountryCode
    dialCode: CountryCallingCode
    name: string
}

```
