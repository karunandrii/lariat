# Lariat

[![Build](https://github.com/Widen/lariat/actions/workflows/build.yml/badge.svg)](https://github.com/Widen/lariat/actions/workflows/build.yml)

Page object framework for end-to-end testing in Playwright.

## Installation

Lariat comes in a number of variants based on the browser automation framework you are using. For example, to install the Playwright version of Lariat, you can run:

```sh
npm install @lariat/playwright
```

While the remainder of the docs reference `@lariat/playwright`, you can swap `@lariat/playwright` with any of the variants listed below:

- Puppeteer - Coming soon!
- Cypress - Coming soon!

## Usage

At the core of Lariat is the `Collection` class. This class is used to represent a collection of elements in a page or section of a page and can include associated utility methods for interacting with those elements.

To create your own collections, simply create a class which extends the `Collection` class from Lariat. You can then define elements using the `Collection.el()` method which we will explore more in a moment.

```ts
import { Collection } from '@lariat/playwright'

class TodoPage extends Collection {
  input = this.el('#todo-input')
}
```

With your collection defined, you can instantiate it in your test to access the elements.

```ts
test('create a todo', async () => {
  const todoPage = new TodoPage(page)
  const input = await todoPage.input()
  await input.fill('Finish the website')
})
```

### Elements

Elements are defined in collections using the `Collection.el()` method.

```ts
class TodoPage extends Collection {
  saveButton = this.el('#save-button')
}
```

Elements can be used in two ways. First, you can call the element as a function which will wait for the element to be visible and return it. This is what you would typically find in a page object model.

```ts
const todoPage = new TodoPage(page)
const saveButton = await todoPage.saveButton()
await saveButton.click()
```

However, Lariat elements are more powerful than that, as you can access the underlying element selector by accessing the `$` property on the element. This can reduce boilerplate or allow you to wait for an element to become hidden.

```ts
const todoPage = new TodoPage(page)
await page.click(todoPage.saveButton.$)
await page.waitForSelector(todoPage.saveButton.$, { state: 'hidden' })
```

#### Dynamic selectors

Because collections in Lariat are plain JavaScript classes, you can easily create elements with dynamic selectors. Consider a todo list where we find an item based on it's name. Our collection might look something like this:

```ts
class TodoPage extends Collection {
  item = (name: string) => this.el(`#todo-item[data-name="${name}"]`)
}

const todoPage = new TodoPage(page)
const item = await todoPage.item('Finish the website')()
```

### Utility methods

Because collections in Lariat are plain JavaScript classes, you can easily add utility methods to your collections.

```ts
class TodoPage extends Collection {
  input = this.el('#todo-input')
  saveButton = this.el('#save-button')

  async create(name: string) {
    await this.page.fill(this.input.$, name)
    await this.page.click(this.input.$)
  }
}

const todoPage = new TodoPage(page)
await todoPage.create('Finish the website')
```

### Collection root

When defining element collections, you may find yourself prefixing each element's selector with a common ancestor. This can be required when you are targeting elements from a component library where there may not be an attribute that uniquely identifies each element.

For example, consider a `TextField` collection which contains an `input` and `error` element.

```ts
class TextField extends Collection {
  input = this.el('#my-text-field .text-field-input')
  error = this.el('#my-text-field .text-field-error')
}
```

As you can see, we are forced to prefix both elements with `#my-text-field` to fully qualify the selectors. However, Lariat provides a better way!

```ts
class TextField extends Collection {
  root = this.el('#my-text-field')
  input = this.el('.text-field-input')
  error = this.el('.text-field-error')
}
```

By adding the `root` property, Lariat will automatically prefix the selectors for all elements in the collection. Not only that, but you can still use the `root` property like any other element!

```ts
const textField = new TextField(page)
console.log(textField.root.$) // #my-text-field
console.log(textField.input.$) // #my-text-field >> .text-field-input
```

_Note: Selector chaining is framework dependent. The Playwright package will use Playwright's chaining syntax (`>>`) to allow mixing selector engines. This technique is not used in the packages for other frameworks._

While this works well in many cases, your may want your collection to be generic where the `root` can be specified during instantiation. Good news, just pass a second argument with the value for `root` when instantiating your collection and you are good to go!

```ts
class TextField extends Collection {
  input = this.el('.text-field-input')
  error = this.el('.text-field-error')
}

new TextField(page, '#my-text-field')
```

### Nested collections

TODO: Documentation coming soon

#### Can I escape nesting?

TODO: Documentation coming soon
