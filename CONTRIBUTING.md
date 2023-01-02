# Contributing Guide

Hi! We are really excited that you are interested in contributing to Elk. Before submitting your contribution, please make sure to take a moment and read through the following guide.

Refer also to https://github.com/antfu/contribute.

## Set up your local development environment

The package manager used to install and link dependencies must be [pnpm](https://pnpm.io/) (Note: on Linux in a standard Node 16+ environment, you should follow the instructions to install via Node's `corepack` rather than using the `curl` command).

To develop and test the Elk package:

1. Fork the Elk repository to your own GitHub account and then clone it to your local device.

2. Ensure using the latest Node.js (16.x)

3. Elk uses pnpm v7, you must enable [Corepack](https://github.com/nodejs/corepack) by running `corepack enable`.

4. Check out a branch where you can work and commit your changes:
```shell
git checkout -b my-new-branch
```

5. Run `pnpm i` in Elk's root folder

6. Run `pnpm nuxi prepare` in Elk's root folder

7. Run `pnpm dev` in Elk's root folder to start dev server or `pnpm dev:mocked` to start dev server with `@elkdev@universeodon.com` user.

## CI errors

Sometimes when you push your changes, the CI can fail, but we cannot check the logs to see what went wrong, run the following commands on your local environment:
- `pnpm test:unit` to run unit tests, maybe you also need to update snapshots
- `pnpm test:typecheck` to run TypeScript checks run on CI

## RTL Support

Elk supports `right-to-left` languages, we need to make sure that the UI is working correctly in both directions.

Simple approach used by most websites of relying on direction set in HTML element does not work because direction for various items, such as timeline, does not always match direction set in HTML.

We've added some `UnoCSS` utilities styles to help you with that:
- Do not use `left/right` padding and margin: for example `pl-1`. Use `padding-inline-start/end` instead. So `pl-1` should be `ps-1`, `pr-1` should be `pe-1`. Same rules applies for margin.
- Do not use `rtl-` classes, such as `rtl-left-0`.
- For icons that should be rotated for RTL, add `class="rtl-flip"`. This can only be used for icons outside of elements with `dir="auto"`, such as timeline, and is the only exception from rule above. For icons inside timeline it might not work as expected.
- For absolute positioned elements, don't use `left/right`: for example `left-0`. Use `inset-inline-start/end` instead. `UnoCSS` shortcuts are `inset-is` for `inset-inline-start` and `inset-ie` for `inset-inline-end`. Example: `left-0` should be replaced with `inset-is-0`.
- If you need to change border radius for an entire left or right side, use `border-inline-start/end`. `UnoCSS` shortcuts are `rounded-is` for left side, `rounded-ie` for right side. Example: `rounded-l-5` should be replaced with `rounded-ie-5`.
- If you need to change border radius for one corner, use `border-start-end-radius` and similar rules. `UnoCSS` shortcuts are `rounded` + top/bottom as either `-bs` (top) or `-be` (bottom) + left/right as either `-is` (left) or `-ie` (right). Example: `rounded-tl-0` should be replaced with `rounded-bs-is-0`.

## Internalization

We are using [vue-i18n](https://vue-i18n.intlify.dev/) via [nuxt-i18n](https://i18n.nuxtjs.org/) to handle internalization.

### Adding a new language

1. Add a new file in [locales](../locales) folder with the language code as the filename.
2. Copy [en-US](../locales/en-US.json) and translate the strings.
3. Add the language to the `locales` array in [config/i18n.ts](../config/i18n.ts#L13)
4. If the language is `right-to-left`, add `dir` option with `rtl` value, for example, for [ar-EG](../config/i18n.ts#L63)
5. If the language requires special pluralization rules, add `pluralRule` callback option, for example, for [ar-EG](../config/i18n.ts#L64)

Check [Pluralization rule callback](https://vue-i18n.intlify.dev/guide/essentials/pluralization.html#custom-pluralization) for more info.

### Messages interpolation

Most of the messages used in Elk do not require any interpolation, however, there are some messages that require interpolation: check [Message Format Syntax](https://vue-i18n.intlify.dev/guide/essentials/syntax.html) for more info.

We're using these types of interpolation:
- [List interpolation](https://vue-i18n.intlify.dev/guide/essentials/syntax.html#list-interpolation)
- [Named interpolation](https://vue-i18n.intlify.dev/guide/essentials/syntax.html#interpolations)
- [Linked messages](https://vue-i18n.intlify.dev/guide/essentials/syntax.html#linked-messages)
- [Literal interpolation](https://vue-i18n.intlify.dev/guide/essentials/syntax.html#literal-interpolation)

#### List interpolation

You can access the elements of the list using the object notation using the index: for example, `{0}` for the first element, `{1}` for the second, `{2}` for the third and so on.

#### Named interpolation

Elk will use named interpolation only to handle plurals for number formatting. We have 2 scenarios for this:
- using `plural` **with** `i18n-t` component
- using `plural` **without** `i18n-t` component

Check [Custom Plural Number Formatting Entries](#custom-plural-number-formatting-entries) for custom plural entries in Elk with available values for interpolation.

When using plural number formatting, we'll have always `{n}` available in the message, for example, `You have {n} new notifications|You have {n} new notification|You have {n} new notifications` or `You have no new notifications|You have 1 new notification|You have {n} new notifications`.

We've included `v` named parameter, it will be used to pass the formatted number using [Intl.NumberFormat::format](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat/format): will be the number with separators symbols. The exception to previous rule is when we're using `plural` **with** `i18n-t` component, in this case, we'll need to use `{0}` instead `{v}` to access the number.

Additionally, Elk will use [compact notation for numbers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat/NumberFormat#parameters) for some entries, check `notation` and `compactDisplay` options: for example, `1K` for `1000`, `1M` for `1000000`, `1B` for `1000000000` and so on. That entry will be available in the message using `{v}` named parameter (or `{0}` if using the message **with** `i18n-t` component).

You can run this code in your browser console to see how it works:
```ts
[1, 12, 123, 1234, 12345, 123456, 1234567].forEach((n) => {
  const acc = {}

  Array.from(['en-US', 'en-GB', 'de-DE', 'zh-CN', 'ja-JP', 'es-ES', 'fr-FR', 'cs-CZ', 'ar-EG']).forEach((l) => {
    const nf = new Intl.NumberFormat(l, {
      style: 'decimal',
      maximumFractionDigits: 0,
    })
    const nf2 = new Intl.NumberFormat(l, {
      notation: 'compact',
      compactDisplay: 'short',
      maximumFractionDigits: 1,
    })
    acc[l] = {
      number: n,
      format: nf.format(n),
      compact: nf2.format(n),
    }
  })
  console.table(acc)
})
```

#### Custom Plural Number Formatting Entries

**Warning**:
Either **{0}**, **{v}** or **{followers}** should be used with the exception being custom plurals entries using the `{n}` placeholder.

This is the full list of entries that will be available for number formatting in Elk:
- `action.boost_count` (no need to be included, we should use always `en-US` entry): `{0}` for formatted number and `{n}` for raw number - **{0} should be use**
- `action.favourite_count` (no need to be included, we should use always `en-US` entry): `{0}` for formatted number and `{n}` for raw number - **{0} should be use**
- `action.reply_count` (no need to be included, we should use always `en-US` entry): `{0}` for formatted number and `{n}` for raw number - **{0} should be use**
- `account.followers_count`: `{0}` for formatted number and `{n}` for raw number - **{0} should be use**
- `account.following_count`: `{0}` for formatted number and `{n}` for raw number - **{0} should be use**
- `account.posts_count`: `{0}` for formatted number and `{n}` for raw number - **{0} should be use**
- `notification.followed_you_count`: `{followers}` for formatted number and `{n}` for raw number - **{followers} should be use**
- `status.poll.count`: `{0}` for formatted number and `{n}` for raw number - **{0} should be use**
- `time_ago_options.*`: `{0}` for formatted number and `{n}` for raw number - **{0} should be use**: since numbers will be always small, we can also use `{n}`
- `timeline.show_new_items`: `{v}` for formatted number and `{n}` for raw number - **{v} should be use**