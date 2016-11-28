---
title: Static Assets
---

Besides dynamically generated content,
most websites also load other files which are static,
such as images, JavaScript and stylesheets.
These files need not be dynamically generated upon requests.
We call them "static assets" and Skygear provides two ways
to serve these static assets.

1. Simple approach: using the `public_html` directory in your cloud code.
2. More customized approach: using the `static_assets` decorator to
   have a function specifying the locations of the static assets.

## Put your static assets in a directory

The simplest way to serve static assets to your users is to put them into
a directory called `public_html` in your cloud code directory. The static
assets you put in this folder will be automatically available at
`http://<endpoint>/static/`.

For example, if your cloud code project contains a file saved at
`./public_html/images/hero.png`, your user will be able to access it
at `http://<endpoint>/static/images/hero.png`.

## Declare your static assets using decorator

If your cloud code project is more complex,
or if you cannot put your static assets in the `public_html` directory,
you can also declare your static assets using the `static_assets`
decorator so that Skygear knows where to find your static assets.

```python
import skygear
from skygear.utils.assets import directory_assets


@skygear.static_assets('/hello-world')
def set_hello_world_assets():
    return directory_assets('my-assets/files')
```

In the above example,
Skygear will find your static assets at
`./my-assets/files/hero.png` if the user makes a request to
`http://<endpoint>/static/hello-world/hero.png`.

<div class="advanced">

The `directory_assets` function returns the absolute path of the static assets
directory by specifying a path relative to the current directory,
or an absolute path.

</div>
