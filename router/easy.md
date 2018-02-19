### easy way to customize child router based on parent route param

The problem we want to resolve:

1. in top level router, we have user page defined as `{route: "user/:id", ...}`.
2. in user page, we build 2nd level child router. We want to show different routes for different user type, for instance, show “admin” tab for admin user.

We have a problem here. The User component `configureRouter(config, router)` is processed before `canActivate`/`activate(params)` callbacks. This means you don’t know the user id when building the routes table in `configureRouter(config, router)`.

Well, there is an undocumented feature of child router. The child router `configureRouter(config, router)` actually gets extra arguments passed in from `navigationInstruction`, the first of those extra arguments is the `params` of parent route!

**This means we can do this:**

```javascript
export class User() {
configureRouter(config, router, params) {
this.router = router;
this.id = params.id;

return ajaxToGetUser(this.id).then(user => {
let routes = [
{
route: '', name: 'details', title: 'Details',
nav: true, moduleId: './user/details'
}
];

if (user.isAdmin) {
routes.push({
route: 'admin', name: 'admin', title: 'Admin',
nav: true, moduleId: './user/admin'
});
}

config.map(routes);
});
}
}
```

In addition, now you have user instance while build the routes table, which means you can put it in settings on all sub-routes if you want `{route: '', settings: {user}, ...}`.

> Note: this undocumented feature is safe to use, [it is here to stay](https://github.com/aurelia/router/issues/547).

> Note: please don't abuse this feature. Dynamic routes are not encouraged by @davismj (the current aurelia-router maintainer), [read more about his opinion here](https://discourse.aurelia.io/t/the-easy-way-to-customize-child-router-based-on-parent-router-param/768/11).

***

Written by @huochunpeng, 02/19/2018
