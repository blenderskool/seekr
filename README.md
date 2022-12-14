<div align="center">
  <br />
  <br />
  <h1>Seekr ๐ญ</h1>
  <p>
    Powerful data searching with a terse syntax.
  </p>
  <br />
  <br />
  <br />
</div>

Seekr is a data searching library which supports a wide variety of search algorithms along with support for Google-inspired search operators. It is intended to be used with _moderately_ sized datasets in environments where more sophisticated and powerful search engines / databases may not be feasible to use.

### Features

- โ **Search operators**
- โ **Boolean searching**
- โ **Fuzzy searching**
- โ **Prefix searching**
- โ **Less than 5KB JS** (minified + gzipped)
- <span title="Experimental, still under development and testing">๐งช</span> **Search indexes**

### Inspiration

Inspiration to build Seekr arose from the fact that data searching libraries in JavaScript had very basic support for search operators in a search query. Google search for example has a **[lot of nifty search operators](https://support.google.com/websearch/answer/2466433)** to include / exclude terms, search specific websites and so on. While most databases that support full-text search did have these search operators, they are **hard and infeasible to use** in a **browser environment** with moderately sized dataset in-memory. This is what Seekr aims to solve - to bring support for _most_ of the search operators in a JS library so that it can be used anywhere for powerful data searching! All packaged in an easy to learn and terse syntax.

### Usage

_Coming Soon_

### Syntax

This is an exhaustive list of Seekr's syntax going over each operator. It might seem complicated to understand at first, but they naturally make sense when you start using them.

- โ๏ธ **Search term**:  
  Single search terms. They represent the simplest query to be searched in the dataset.

  - **Simple**: `apple`  
    Searches in all the shallow fields in each object for the term `apple`.  
    They are treated as case-insensitive in currently supported search algorithms of Seekr.

  - **Scoped to a field**: `name:apple`  
    Searches **in the string value of `name` field** in each object for term `apple`.

    - **Deep fields**: `name.scientific:malus`  
      Seekr by default only searches on shallow fields of object. For deeper fields, they have to be explicitly scoped using dot-path syntax.
      The above query searches in each object's `name` field's `scientific` field's string value for term `malus`.

  - **Exclude**: `!apple`  
    Searches in all the shallow fields in each object and **excludes them** from the result if _some_ field has the term `apple`.

  - **Exclude scoped to a field**: `name:!apple`  
    Searches each object and excludes it if **the string value of `name` field** has the term `apple`.
  - **Comparisons**:  
    Comparisons use standard comparison operators `>`, `<`, `>=`, `<=`.

    - **Numeral comparisons**: `>5`, `<10`, `>=2e3`, `<=40.4`  
      Searches in all the shallow fields in each object if it's value is **greater than 5**, **less than 10**, **greater than or equal to 2e3(2000)**, **less than or equal to 40.4** respectively.

    - **String comparisons**: `>apple`, `<orange`, `>=zzz`, `<=3M`  
      Similar to previous example but uses [native string comparison](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#comparison_operators).

    Note: If the value in the field being searched has a different type than `string` or `number`(eg. booleans, nested objects, etc.), it's converted to its string representation and then exact matching is used instead of comparison.

    <details>
      <summary><strong>Comparisons with scoped fields</strong></summary>
      
      - `age:>18`, `duration:<=3500`, `area:<3.14`, etc.
      - `name:>apple`, `description:<orange`, etc.

    </details>

- ๐งซ **Compound search**:  
  These queries _combine multiple search terms_ specified above.

  - **`AND` operator**: `apple AND orange`  
    Searches in each object which includes the term `apple` **and** `orange` in some of its shallow fields.

  - **`OR` operator**: `apple OR orange`  
    Searches in each object which includes the term `apple` **or** `orange` in its shallow fields.

  - **Implicit `AND`**: `apple orange`  
    `AND` can be eliminated and the above query is treated the same as `apple AND orange`.

    <details>
      <summary>Few more examples:</summary>

    - `name:apple OR description:orange` - `name` field with `apple` **or** `description` field with `orange`.
    - `name:apple AND orange` - `name` field with `apple` **and** _some_ field with `orange`.
    - `name:apple type:fruit` - `name` field with `apple` **and** `type` field with `fruit`.
    - `name:apple age:<3` - `name` field with `apple` **and** `age` field less than `3`.
    - `name:!apple OR name:!orange` - `name` field without `apple` **or** `name` field without `orange`.

    </details>

- ๐งฐ **Nesting**: `(apple OR orange)`  
  Nesting groups multiple terms to a single search term. They are **not for representing priority order** of search expressions. The above query is same as `apple OR orange`. Nesting is useful when used with other operators in Seekr.

  - **Compound search within a field**: `name:(apple OR orange)`  
    Same as `name:apple OR name:orange` but shorter.

  - **Nested scoped fields**: `name:(apple scientific:malus)`  
    Same as `name:apple AND name.scientific:malus`.
  - **Exclude compound search terms**: `!(apple AND orange)`  
    Uses **[De-Morgan's law](https://en.wikipedia.org/wiki/De_Morgan%27s_laws)** to expand the nesting. Searches for all objects without both apple and orange terms. Thus, it's treated as `!apple OR !orange`. Can also be written as `!(apple orange)`.
  - **Nested compound search**: `apple AND (orange OR grape)`  
    Searches for LHS of `AND`, then RHS. Remember, brackets are not for specifying priority order.
  - `!(name:!apple OR age:<5)`: You guess what this is ๐ (Hint: try expanding the nesting)
    <details>
      <summary>Answer</summary>

    Same as `name:apple AND age:>=5`. Expand the nest using De-Morgan's law.
    Double exclusions cancel out, exclusion of `<` yields `>=` results.

    </details>

### ๐ Comparators - Search algorithms

Seekr **separates** the search algorithm from the syntax and its associated query parsing algorithm. Each search algorithm is called a **"Comparator"**. The benefit of having this separation is that it enables various search algorithms and approaches on an already defined and understood syntax without building a completely new library.

Each comparator(the search algorithm) defines its fundamental operations for the search operators supported by Seekr. Then it can be simply used with the same syntax with almost no code change for the consumer. Seekr currently supports following 3 comparators which have their own use-cases along with advantages and drawbacks:

- **`BinaryCmp`**: Boolean search algorithm. It checks if a single search term is exactly present in the string being compared to. If it is present, the record is included for further checks, otherwise it is excluded.

- **`FuzzyCmp`**: Fuzzy search algorithm. Instead of hard inclusion / exclusion of records in the result, it assigns a score to each record indicating how close it matches the search query. Higher the score, higher the match.

- **`PrefixCmp`**: Prefix search algorithm. Similar to `BinaryCmp` but only checks if single search term is the **prefix** of a word in the string being compared to. It internally uses optimized data structures when used with search indexes and is extremely fast compared to `BinaryCmp` for equivalent queries.

**Some operators may not be defined for some comparators**. Eg. comparison operators(`>`, `<`, `>=`, `<=`) has no meaning when used in fuzzy search / prefix search. Hence, `FuzzyCmp` and `PrefixCmp` both fallback to `BinaryCmp`'s logic for performing comparisons which only includes records that satisfy the comparison.

### ๐ Search Indexes

_This section is in TODO_
