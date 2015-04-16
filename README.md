# Ho Grid

> Modular grid system using less mixins

Uses an outdented grid methodology, stolen shamelessly (almost entirely and almost verbatim) from bootstrap.

## Mixins

```
.Component {
  @gutter: 20px;

  margin: 0 auto;
  width: 100%;
  max-width: 1140px;

  .Component-row {
    .m-ho-makeRow();
  }

  .Component-column {
    .m-ho-makeColumn( 1, 4 );
  }
}
```

```
<body>
  <div class="Component">
    <div class="Component-row">
      <div class="Component-column"><!-- Content --></div>
      <div class="Component-column"><!-- Content --></div>
      <div class="Component-column"><!-- Content --></div>
      <div class="Component-column"><!-- Content --></div>
    </div>
  </div>
</body>
```

The grid system uses the same outdented grid method employed by bootstrap which outdents columns to maintain vertical alignment of content within columns.

The mixins expect a `@gutter` variable to be supplied, which should be scoped to the component using the mixin (_global leak_ scoping is a bad thing here), meaning that each component would need a _parent_ selector wrapper.

Creating a grid which becomes a standard stacked list of content at a certain width is relatively straight forward:

```
.m-makeCell( @column-width, @columns ) {
  .m-ho-makeColumn( @column-width, @columns );

  @media (max-width: @breakpoint) {
    float: none;
    width: 100%;
  }
}

.Component {
  @gutter: 16px;
  @breakpoint: 540px;

  margin: 0 auto;
  width: 100%;
  max-width: 1140px;

  .Component-row {
    .m-ho-makeRow();
  }

  .Component-column {
    .m-makeCell( 1, 4 );
  }
}
```

## API

Some of the mixins require that a variable be defined somewhere, this should normally be scoped using nested selectors. There are many reasons why nesting is bad in preprocessors but due to _global leak_ issues with variables nesting here is required, although it could be avoided by scoping locally to each mixin.

### .m-ho-makeRow()

_Requires_
`@gutter`

_Description_

Creates the outdents necessary to contain the columns.

### .m-ho-makeColumns( @column-width, @num-columns )

_Requires_ `@gutter`

_Parameter_ `@column-width`

Specifies the number of columns this column stretches across.

_Parameter_ `@num-columns`

Specifies the total number of columns in this row.

_Example_

```
/* @Example */

/**
 * Stretches across 2 columns of a 3 column row.
 */

.m-ho-makeColumn( 2, 3 );
```

_Description_

Creates a column within the row. A defined column (the selector the mixin applies to) may stretch across multiple columns in the row. Adding more defined columns than the maximum number of columns in the row will simply cause them to overflow which is useful for creating grids of similarly heighted elements.

### .m-ho-beforeColumn( @column-width, @num-columns )

_Parameter_ `@column-width`

The number of columns this offset covers.

_Parameter_ `@num-columns`

The total number of columns in this row.

_Example_

```
/* @Example */

/**
 * Creates a 5 column offset before the selector it is applied to.
 */

.m-ho-beforeColumn( 5, 12 );
```

### .m-ho-afterColumn( @column-width, @num-columns )

_Parameter_ `@column-width`

The number of columns this offset covers.

_Parameter_ `@num-columns`

The total number of columns in this row.

_Example_

```
/* @Example */

/**
 * Creates a 5 column offset after the selector it is applied to.
 */

.m-ho-afterColumn( 5, 12 );
```



## Scoping

Variables used by the mixins need to be scoped inside a selector, otherwise globals will take precedence. This is an issue with less and mixin scoping and I’m not convinced there is a way around this, which is cumbersome and makes nesting, at least shallow nesting, necessary.


## Bug Finding

Picking up _global leak bugs_ is pretty difficult <sup>[1](https://en.bem.info/articles/side-effects-in-css/#the-hardest-problem-in-css)</sup>, particularly when the variable causing the leak is in a different file.

```
/* component.less */

/* WARNING -- ANTI-PATTERN */
@gutter: 20px;

.Component {
  .m-ho-makeRow();
}
.Component-column {
  .m-ho-makeColumn();
}
```

```
/* main.less */

.Component {
  @gutter: 10px;

  .Component-row {
    .m-ho-makeRow();
  }

  .Component-column {
    .m-ho-makeColumn();
  }
}
```

In this case the global `@gutter: 20px` found in `component.less` will be favoured by the mixin and result in all grids using a gutter size of 20. This may be desirable to keep consistent grids across your application but its likely more of a hindrance and certainly doesn’t help keep variables scoped where you might expect.

The main problem is that this is very difficult to diagnose, particularly as the number of components (and individual files) grows.

This, again, is to do with how less structures finding variables. A mixin will look locally for a variable, and then visit parent scope but the global scope takes precedence and as imported files don’t maintain their own scope we’re stuck with having to namespace our css, which is inefficient:

```
/* component.less */

.Component {
  @gutter: 20px;
  .fl-make-row();

  .Component-column {
    .m-ho-makeColumn( 1, 3 );
  }
}
```

If this inefficiency bothers you then consider if [rework](https://github.com/reworkcss/rework) or [suitcss](https://suitcss.github.io/) is a better fit (suit uses rework and allows for per-file scoping using the `:root` selector).

The alternative is to scope variables locally to each mixin.

```
.Component {
  @gutter: 20px;
  .fl-make-row();
}
.Component-column {
  @gutter: 20px;
  .m-ho-makeColumn( 1, 3 );
}
```

It would also be relatively straight forward to create a task to scan your less files and flag any globals<sup>[2](https://github.com/hocss/ho-conformance)</sup>.

---

## Refs

<sup>[1]</sup> [Side effects in css / Philip Walton](https://en.bem.info/articles/side-effects-in-css/#the-hardest-problem-in-css)

<sup>[2]</sup> [Conformance tool](https://github.com/hocss/ho-conformance)
