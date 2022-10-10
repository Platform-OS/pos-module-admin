# Module name

Admin module  

This module provides an admin interface for your app and other modules. It implements the `/admin` route and provides a hook for other modules to register their own admin page implementations.

## Usage
Once you have this module installed you can access the admin UI at `/admin`. Your user needs the `admin_pages.view` permission to have access to the admin menu and the pages.  
For more details see the [Permission module](https://github.com/Platform-OS/pos-module-permission).

## Hooks

The Module provides the `hook_admin_page` and the `hook_admin_layout` hooks.

### hook_admin_page
If you want an admin page (or just a link) to show up in the admin navigation and the admin UI, you can implement `hook_admin_page.liquid`.  
It should return a json object describing the page and/or navigation items:
```
{% parse_json admin_pages %}
[
  {
    "relative_path": "/admin/path_you_want_to_imeplement" | null,
    "partial": "path_to_your_admin_page_partial" (optional),
    "menu": {
      "title": "Title of your admin page"
      "link_attributes": {
        "attribute1": "attribute to add to the admin menu link (`link_attributes` is optional)"
        "attribute2": "attribute value"
        ...
      }
    },
    "permission": "name of the permission needed to have access your admin page"
  }
]
{% endparse_json %}

{% return admin_pages %}
```
From the example above you can see that the `relative_path` (which is the admin route) and the `partial` is optional. In case you don't want to register your admin page partial, but you still want a page to show up in the admin menu you can implement your own page and add the link to the admin menu using the `link_attributes` object, by specifying a `href` attribute. This is how the [Component Library Module](https://github.com/Platform-OS/pos-module-components) shows the link to the styleguide page in the admin.

### hook_admin_layout
You can register your own admin layout file to provide a custom admin experience in your app.
This is how the [Theme Manager Module](https://github.com/Platform-OS/pos-module-theme-manager) implements admin themes, by looking for available admin theme implementations and registering the admin layout from the currently selected active theme.
The `admin_layou` hook simply returns the path of a liquid partial as a string.
