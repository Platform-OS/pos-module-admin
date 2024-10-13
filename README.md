# Admin module

This module provides an admin interface for your app and other modules. It implements the `/admin` route and provides a hook for other modules to register their own admin page implementations.

## Installation

The platformOS Admin Module is available on the [Partner Portal Modules Marketplace](https://partners.platformos.com/marketplace/pos_modules/138).

### Prerequisites

Before installing the module, ensure that you have [pos-cli](https://github.com/mdyd-dev/pos-cli#overview) installed. This tool is essential for managing and deploying platformOS projects.

The platformOS Admin Module is fully compatible with [platformOS Check](https://github.com/Platform-OS/platformos-lsp#platformos-check----a-linter-for-platformos), a linter and language server that supports any IDE with Language Server Protocol (LSP) integration. For Visual Studio Code users, you can enhance your development experience by installing the [VSCode platformOS Check Extension](https://marketplace.visualstudio.com/items?itemName=platformOS.platformos-check-vscode).

### Installation Steps

1. **Navigate to your project directory** where you want to install the Admin Module.

2. **Run the installation command**:

```bash
   pos-cli modules install admin
```

This command installs the Admin Module along with its dependencies ([pos-module-core](https://github.com/Platform-OS/pos-module-core), [pos-module-user](https://github.com/Platform-OS/pos-module-user), [pos-module-components](https://github.com/Platform-OS/pos-module-components)) and updates or creates the `app/pos-modules.json` file in your project directory to track module configurations.

### Setup

1. First, install the module using the [pos-cli](https://github.com/Platform-OS/pos-cli).

2. Configure the [components](https://github.com/Platform-OS/pos-module-components) theme paths by adding the following `theme_search_paths` property to the [app/config.yml](https://documentation.platformos.com/developer-guide/platformos-workflow/directory-structure/config) file:

```yaml
theme_search_paths:
  - ''
  - modules/user
  - modules/admin
  - modules/components
```

3. Configure  [app/user.yml](https://documentation.platformos.com/developer-guide/users/user#adding-properties-to-the-user) file:

```yaml
properties:
  - name: roles
    type: array

```

4. Overwrite user's module permission file and add [admin_pages.view](https://github.com/Platform-OS/pos-module-admin/blob/master/modules/admin/public/views/pages/admin/index.liquid#L18) (and [admin.users.manage](https://github.com/Platform-OS/pos-module-user/blob/master/modules/user/public/lib/hooks/hook_admin_page.liquid) to access [manage users admin page defined in the user module](https://github.com/Platform-OS/pos-module-user/blob/master/modules/user/public/views/partials/admin_pages/list.liquid)) to the admin role. The links point to the implementation, which should allow you to understand how everything is wired together. Example permissions file:

```
{% parse_json data %}
{
  {% if context.constants.USER_DEFAULT_ROLE != blank %}
  "{{ context.constants.USER_DEFAULT_ROLE }}": [],
  {% endif %}
  "admin": ["admin_pages.view", "admin.users.manage"],
  "anonymous": ["sessions.create"],
  "authenticated": ["sessions.destroy"]
}
{% endparse_json %}

{% return data %}
```

5. Add `admin` role to the user you would like to make admin (or add [`superadmin` role to ensure they have access to everything, always).

To manually set user's role, first register using the [/users/new endpoint provided but the user module](https://github.com/Platform-OS/pos-module-user?tab=readme-ov-file#endpoints-for-the-registration), and then use `pos-cli gui serve` GraphQL explorer to invoke the following query:

```
mutation () {
  user: user_update(id: 1, user: { properties: [{ name: "roles", value_array: ["admin"] }] }) {
    id
    email
    roles: property_array(name: roles)
  }
}
```

> [!NOTE] 
> The example GraphQL assumes you want to promote the first registered user to become an admin, and that's why the `id` is hardcoded to `1`. You can always use `pos-cli gui serve` -> [Users](http://localhost:3333/users) to quickly check the id of any user, or use `users` GraphQL query.

## Usage

Once you have this module installed you can access the admin UI at `/admin`. Your user needs the `admin_pages.view` permission to have access to the admin menu and the pages.  
For more details see the [Permission module](https://github.com/Platform-OS/pos-module-permission).

## Hooks

The Module provides the `hook_admin_page` and the `hook_admin_layout` hooks.

### hook_admin_page

If you want an admin page (or just a link) to show up in the admin navigation and the admin UI, you can implement `lib/hooks/hook_admin_page.liquid`. 

It should return an array of objects describing the page and/or navigation items:
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

The `admin_layout` hook simply returns the path of a liquid partial as a string. Example `lib/hooks/admin_layout_hook.liquid`:

```
{% return 'admin_layout' %}
```

Which will load `app/views/partials/admin_layout.liquid`.

## Versioning

```
git fetch origin --tags
npm version major | minor | patch
```
