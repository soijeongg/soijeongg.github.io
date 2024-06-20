# Post

## Creating a post

You can use the `initpost.sh` script to generate new posts when you clone the repo (the script isn't available in the `gem`).

To do so, in your project directory, just run:

```
./initpost.sh -c "Your Post Title"
```

The new file will be created in `_posts` with the format `YYYY-MM-DD-your-post-title.md`.

## Front Matter properties

If you don't know what these are, check the Jekyll [documentation](https://jekyllrb.com/docs/front-matter/).

A *Jekflix* post file looks like:

```yaml
# _posts/2010-01-01-welcome-to-the-desert-of-the-real.md
---
date: 2019-05-16 23:48:05
layout: post
title: Welcome to the desert of the real
subtitle: Lorem ipsum dolor sit amet, consectetur adipisicing elit.
description: >-
  Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
  tempor incididunt ut labore et dolore magna aliqua.
image: https://res.cloudinary.com/dm7h7e8xj/image/upload/v1559821647/theme6_qeeojf.jpg
optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559821647/theme6_qeeojf.jpg
category: blog
tags:
  - welcome
  - blog
author: thiagorossener
paginate: true
---

bla bla bla
```

Below is a full list of the template Front Matter properties explained:

#### `date`

Type: *datetime*

The post publishing date. Format: `YYYY-MM-DD hh:mm:ss`

Example:

```yaml
# _posts/2019-08-22-example.md
---
...
date: 2019-05-16 23:48:05
...
---
```

#### `layout`

Type: *string*

The layout file that will be used. The template has only one valid layout for posts, which is `post`.

```yaml
# _posts/2019-08-22-example.md
---
...
layout: post
...
---
```

#### `title`

Type: *string*

The post title.

Example:

```yaml
# _posts/2019-08-22-example.md
---
...
title: Welcome to the desert of the real
...
---
```

#### `subtitle`

*(Optional)*

Type: *string*

The post subtitle. It appears below the title.

Example:

```yaml
# _posts/2019-08-22-example.md
---
...
subtitle: Lorem ipsum dolor sit amet, consectetur adipisicing elit.
...
---
```

#### `description`

Type: *string*

The post description. It's used in the home and category pages, in meta description tag for SEO purposes and for social media sharing.

Example:

```yaml
# _posts/2019-08-22-example.md
---
...
description: >-
  Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
  tempor incididunt ut labore et dolore magna aliqua.
...
---
```

#### `image`

Type: *url*

The featured image. It's used in the home and category pages and for social media sharing.

**Tip:** Use a media server to provide images for your website, like [Cloudinary](https://cloudinary.com)

*Obs: The recommended image resolution is 760x399*

Example:

```yaml
# _posts/2019-08-22-example.md
---
...
image: https://res.cloudinary.com/dm7h7e8xj/image/upload/v1559821647/theme6_qeeojf.jpg
...
---
```

#### `optimized_image`

*(Optional*)

Type: *url*

The optimized featured image. Set a smaller image to appear in the home and category pages, they will load faster.

In case there is no `optimized_image` set, the pages show the one in the `image` property.

*Obs: The recommended image resolution is 380x200*

Example:

```yaml
# _posts/2019-08-22-example.md
---
...
optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559821647/theme6_qeeojf.jpg
...
---
```

#### `category`

Type: *string*

Only one category is allowed. Make sure to create a `<category>.md` file in `category` directory if it's a new one.

Example:

```yaml
# _posts/2019-08-22-example.md
---
...
category: blog
...
---
```

#### `tags`

Type: *list*

A list of the post keywords. It's used in the home, category and tags pages, and as meta keywords for SEO purposes.

Example:

```yaml
# _posts/2019-08-22-example.md
---
...
tags:
  - welcome
  - blog
...
---
```

#### `author`

*(Optional)*

Type: *string*

The post author. Set the author filename used in the `_authors` folder.

Every time you create a new author, make sure to create a file in there too.

Leave it blank if there is no author.

Example:

```yaml
# _posts/2019-08-22-example.md
---
...
author: thiagorossener
...
---
```

#### `paginate`

*(Optional)*

Type: *boolean*

To break your post into pages, set the property `paginate` to `true` and use the divider `--page-break--` where you want to break it.

**Note:** The template uses the `jekyll-paginate-content`, which is [not supported for GitHub Pages](https://pages.github.com/versions/). If you need that feature, please deploy somewhere else like Netlify.

Example:

```yaml
# 2019-08-20-ten-skills-you-need-to-have-to-become-a-good-developer.md
---
...
paginate: true
...
---

Skill 1

--page-break--

Skill 2
```

It would look like:

![Paginated Page Screenshot](https://res.cloudinary.com/dm7h7e8xj/image/upload/v1566430021/paginated-page-screenshot_zx4xjn.jpg)

- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Thomas A. Anderson</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

# Heading 1

## Heading 2

### Heading 3

#### Heading 4

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

## Code

Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

```js
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
```

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

## Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

* Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
* Donec id elit non mi porta gravida at eget metus.
* Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

## Images

Quisque consequat sapien eget quam rhoncus, sit amet laoreet diam tempus. Aliquam aliquam metus erat, a pulvinar turpis suscipit at.

![placeholder](https://placehold.it/800x400 "Large example image")
![placeholder](https://placehold.it/400x200 "Medium example image")
![placeholder](https://placehold.it/200x200 "Small example image")

## Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>
