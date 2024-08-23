<script lang="ts" setup>
import type { Examples } from 'libphonenumber-js'
import type { CountryCode } from 'libphonenumber-js'
import {
    type ComponentPublicInstance,
    computed,
    nextTick,
    onMounted,
    provide,
    ref,
    watch,
} from 'vue'
import type { InjectedData, Results, Translations } from '../helpers/types'

import { defaultLocales } from '../helpers/default-locales'

import { useLibphonenumber } from '../helpers/use-libphonenumber'
import { usePhoneInput } from '../helpers/use-phone-input'
import { useSlots } from 'vue'
const slots = useSlots()

const props = withDefaults(defineProps<Props>(), {
    modelValue: undefined,
    countryCode: undefined,
    placeholder: undefined,
    label: undefined,
    preferredCountries: undefined,
    ignoredCountries: undefined,
    onlyCountries: undefined,
    translations: undefined,
    customCountriesList: undefined,
    countryLocale: undefined,
    noFormattingAsYouType: false,
    autoFormat: true,
    excludeSelectors: undefined,
})

const emits = defineEmits<{
    /**
     * emitted when country or phone number changes
     * @property {string} phoneNumber - phoneNumber formatted
     */
    'update:model-value': [value: string]
    /**
     * emitted when selected country changes
     * @property {CountryCode} countryCode - Country code
     */
    'country-code': [countryCode?: CountryCode]
    /**
     * emitted when country changes
     * @property {CountryCode} countryCode - Country code
     */
    'update:country-code': [countryCode?: CountryCode]
    /**
     * emitted when country or phone number changes
     * @property {Results} results - metadata of current phone number
     */
    update: [results: Results]
    /**
     * emitted when country or phone number changes
     * @property {Results} results - metadata of current phone number
     */
    data: [results: Results]
}>()

export interface Props {
    /** @model Country calling code + telephone number in international format */
    modelValue?: string
    /** @model Country code selected - Ex: "FR" */
    countryCode?: CountryCode
    /** Placeholder of the input */
    placeholder?: string
    /** List of country codes to place first in the select list - Ex: ['FR', 'BE', 'GE'] */
    preferredCountries?: CountryCode[]
    /** List of country codes to be removed from the select list - Ex: ['FR', 'BE', 'GE'] */
    ignoredCountries?: CountryCode[]
    /** List of country codes to only have the countries selected in the select list - Ex: ['FR', 'BE', 'GE'] */
    onlyCountries?: CountryCode[]
    /** Locale strings of the component */
    translations?: Partial<Translations>
    /** By default the component use the browser locale to set the default country code if not country code is provided */
    noUseBrowserLocale?: boolean
    /** The component will make a request (https://ipwho.is) to get the location of the user and use it to set the default country code */
    fetchCountry?: boolean
    /** No show the country selector */
    noCountrySelector?: boolean
    /** Replace country names */
    customCountriesList?: Record<CountryCode, string>
    /**
     * Disabled auto-format when phone is valid
     * @default true
     */
    autoFormat?: boolean
    /**
     * Disabled auto-format as you type
     * @default false
     */
    noFormattingAsYouType?: boolean
    /**
     * locale of country list - Ex: "fr-FR"
     * @default {string} browser locale
     */
    countryLocale?: string
    /** Exclude selectors to close country selector list - usefull when you using custom flag */
    excludeSelectors?: string[]
}

const { fetchCountryCode, sanitizePhoneNumber, getBrowserLocale } =
    usePhoneInput()
const { isCountryAvailable, getPhoneNumberResults, getAsYouTypeFormat } =
    useLibphonenumber()
const { getPhoneNumberExamplesFile, getPhoneNumberExample } =
    useLibphonenumber()

const locales = computed(() => ({
    ...defaultLocales,
    ...props.translations,
}))

const examples = ref<Examples>()

async function loadExamples() {
    try {
        if (examples.value) return

        examples.value = await getPhoneNumberExamplesFile()
    } catch (error) {
        console.error(' while loading phone number examples file', error)
    }
}

/**
 * State
 */

const { getCountriesList } = usePhoneInput()

const countries = computed(() =>
    getCountriesList(props.countryLocale, props.customCountriesList)
)

const countriesList = computed(() =>
    countries.value?.filter(
        (item) => !props.ignoredCountries?.includes(item.iso2)
    )
)

const countriesFiltered = computed(() => {
    const countries = props.onlyCountries || props.preferredCountries
    return countries?.map((country) =>
        countriesList.value?.find((item) => item.iso2.includes(country))
    )
})

const otherCountries = computed(() =>
    countriesList.value?.filter(
        (item) => !props.preferredCountries?.includes(item.iso2)
    )
)

const countriesSorted = computed(() =>
    props.preferredCountries
        ? [...(countriesFiltered.value ?? []), ...(otherCountries.value ?? [])]
        : props.onlyCountries
        ? countriesFiltered.value
        : countriesList.value
)

const countryOptions = computed(() =>
    countriesSorted.value
        ?.map((country) =>
            country
                ? {
                      ...country,
                      dialCode: `+${country.dialCode}`,
                  }
                : undefined
        )
        .filter((value) => !!value)
)

const phoneNumber = ref<string>('')
const selectedCountry = ref<CountryCode>()
const results = ref<Results>({
    isValid: false,
    countryCode: undefined,
})

/** Inject */

provide<InjectedData>('data', {
    selectedCountry,
    phoneNumber,
    results,
})

onMounted(async () => {
    setSelectedCountry(props.countryCode)

    if (props.fetchCountry && !selectedCountry.value) {
        const countryCode = await fetchCountryCode()
        setSelectedCountry(countryCode)
    }

    if (!props.noUseBrowserLocale && !selectedCountry.value) {
        const countryCode = getBrowserLocale()?.locale
        setSelectedCountry(countryCode)
    }

    loadExamples()
})

const PhoneInputRef = ref<ComponentPublicInstance>()
function getPhoneNumberInput() {
    return PhoneInputRef.value?.$el.querySelector('input') as
        | HTMLInputElement
        | undefined
}
async function selectPhoneNumberInput() {
    await nextTick()
    getPhoneNumberInput()?.select()
}

function countryChanged(countryCode?: CountryCode) {
    onCountryChanged({
        countryCode,
        autoFormat: props.autoFormat,
        noFormattingAsYouType: props.noFormattingAsYouType,
    })
    selectPhoneNumberInput()
}

function setSelectedCountry(countryCode?: string) {
    if (!countryCode) {
        return
    }

    if (!isCountryAvailable(countryCode)) {
        selectedCountry.value = undefined
        return
    }

    selectedCountry.value = countryCode as CountryCode
}

function handlePhoneNumberUpdate(
    eventOrData:
        | Event
        | {
              newPhoneNumber?: string
              autoFormat: boolean
              noFormattingAsYouType: boolean
              updateResults?: boolean
          }
) {
    // Determine if the input is an event or data object
    let newPhoneNumber: string | undefined
    let autoFormat: boolean = true
    let noFormattingAsYouType: boolean = false
    let updateResults: boolean = true

    if (eventOrData instanceof Event) {
        // Event case
        newPhoneNumber = (eventOrData.target as HTMLInputElement).value
    } else {
        // Data object case
        newPhoneNumber = eventOrData.newPhoneNumber
        autoFormat = eventOrData.autoFormat
        noFormattingAsYouType = eventOrData.noFormattingAsYouType
        updateResults = eventOrData.updateResults ?? true
    }

    // Sanitize the phone number
    const sanitizedPhoneNumber = sanitizePhoneNumber(newPhoneNumber)

    // Update the phone number in the component state
    phoneNumber.value = sanitizedPhoneNumber

    // Perform results update and formatting if needed
    if (updateResults) {
        results.value = getPhoneNumberResults({
            phoneNumber: sanitizedPhoneNumber,
            countryCode: selectedCountry.value,
        })
    }

    if (results.value.isValid && results.value.formatNational && autoFormat) {
        phoneNumber.value = results.value.formatNational
    } else if (!noFormattingAsYouType) {
        const asYouTypeFormatted = getAsYouTypeFormat(
            selectedCountry.value,
            sanitizedPhoneNumber
        )
        phoneNumber.value = asYouTypeFormatted
    } else {
        phoneNumber.value = sanitizedPhoneNumber
    }

    // Handle country changes if needed
    if (
        results.value.countryCode &&
        results.value.countryCode !== selectedCountry.value
    ) {
        onCountryChanged({
            countryCode: results.value.countryCode,
            autoFormat,
            noFormattingAsYouType,
            updateResults: false,
        })
    }

    // Emit the updated value
    emits('update:model-value', phoneNumber.value)
}

function updatePhoneNumber(value: string) {
    //@ts-ignore
    handlePhoneNumberUpdate(value)
}

function onPhoneNumberChanged({
    newPhoneNumber,
    autoFormat,
    noFormattingAsYouType,
    updateResults = true,
}: {
    newPhoneNumber?: string
    autoFormat: boolean
    noFormattingAsYouType: boolean
    updateResults?: boolean
}) {
    handlePhoneNumberUpdate({
        newPhoneNumber,
        autoFormat,
        noFormattingAsYouType,
        updateResults,
    })
}

function getCountryPhoneNumberExample(
    examples: Examples,
    selectedCountry?: CountryCode
) {
    const example = getPhoneNumberExample(examples, selectedCountry)
    return example ? `${locales.phoneInput.example} ${example}` : undefined
}

const inputLabelOrPlaceholder = computed(() => {
    if (props.placeholder) {
        return props.placeholder
    }

    const defaultPlaceholder = locales.phoneInput.placeholder
    if (!examples.value) {
        return defaultPlaceholder
    } else {
        const phoneExample = getCountryPhoneNumberExample(
            examples.value,
            selectedCountry.value
        )
        return results.value?.isValid || !phoneExample
            ? defaultPlaceholder
            : phoneExample
    }
})

function onCountryChanged({
    countryCode,
    autoFormat,
    noFormattingAsYouType,
    updateResults = true,
}: {
    countryCode?: CountryCode
    autoFormat: boolean
    noFormattingAsYouType: boolean
    updateResults?: boolean
}) {
    if (!countryCode) {
        selectedCountry.value = undefined
        return
    }

    if (countryCode !== selectedCountry.value) {
        setSelectedCountry(countryCode)
    }

    if (updateResults) {
        results.value = getPhoneNumberResults({
            phoneNumber: phoneNumber.value,
            countryCode,
        })
    }

    onPhoneNumberChanged({
        newPhoneNumber: phoneNumber.value,
        autoFormat,
        noFormattingAsYouType,
        updateResults: false,
    })
}

watch(
    () => props.modelValue,
    (value, oldValue) => {
        if (value !== oldValue && value !== phoneNumber.value) {
            onPhoneNumberChanged({
                newPhoneNumber: value,
                autoFormat: props.autoFormat,
                noFormattingAsYouType: props.noFormattingAsYouType,
            })
        }
    },
    {
        immediate: true,
    }
)

watch(
    () => props.countryCode,
    (value, oldValue) => {
        if (value && value !== oldValue && value !== selectedCountry.value) {
            onCountryChanged({
                countryCode: value,
                autoFormat: props.autoFormat,
                noFormattingAsYouType: props.noFormattingAsYouType,
            })
        }
    },
    {
        immediate: true,
    }
)

watch(
    results,
    (value) => {
        emits('update', value)
        emits('data', value)

        if (value.e164 && value.isValid) {
            emits('update:model-value', value.e164)
        } else {
            emits('update:model-value', phoneNumber.value)
        }

        emits('country-code', selectedCountry.value)
        emits('update:country-code', selectedCountry.value)
    },
    {
        immediate: true,
    }
)
</script>

<template>
    <div class="base-phone-input">
        <slot
            name="selector"
            :input-value="selectedCountry"
            :update-input-value="countryChanged"
            :countries="countryOptions"></slot>
        <div v-if="!slots.selector">
            <select
                class="base-select"
                name="selector"
                id="selector"
                :value="selectedCountry"
                @change="countryChanged($event.target?.value)">
                >
                <option
                    v-for="country in countryOptions"
                    :value="country.iso2"
                    :key="country.iso2">
                    {{ country.name }}
                </option>
            </select>
        </div>
        <slot
            name="input"
            :input-value="phoneNumber"
            :update-input-value="updatePhoneNumber"
            :placeholder="inputLabelOrPlaceholder">
        </slot>
        <div v-if="!slots.input">
            <input
                type="text"
                label="phone"
                :value="phoneNumber"
                @input="updatePhoneNumber"
                :placeholder="inputLabelOrPlaceholder" />
        </div>
    </div>
</template>

<style scoped>
.base-phone-input {
    display: flex;
    align-items: center;
}
</style>
