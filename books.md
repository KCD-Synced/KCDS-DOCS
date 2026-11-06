# Books and letters

The game's book and letter screen shows your texts to this player:

```js
// client
kcds.ui.Book.open({ style: 'letter', pages: ['To the burgrave,\nthe grain did not arrive.'] });
```

```ts
// client
kcds.ui.Book.open({ style: 'letter', pages: ['To the burgrave,\nthe grain did not arrive.'] });
```

```lua
-- client
kcds.ui.Book.open({ style = "letter", pages = { "To the burgrave,\nthe grain did not arrive." } })
```

```cs
// client
Book.Open(new { style = "letter", pages = new[] { "To the burgrave,\nthe grain did not arrive." } });
```

`style` is `'book'`, `'plainBook'` or `'letter'`. `Book.write(true)` lets the player write on a page, firing
`bookTextChanged`, and `Book.image(path)` puts a picture your resource ships on a page.

`Book.setPages(pages)` and `Book.setReadable(value)` change an open book, and `Book.currentBook()` and
`Book.lastFailure()` read it.

Reference: [Book](/docs/api/book).
