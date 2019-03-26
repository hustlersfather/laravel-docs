# Upgrade Guide

- [Upgrading To 5.9.0 From 5.9](#upgrade-5.9.0)

<a name="upgrade-5.9.0"></a>
## Upgrading To 5.9.0 From 5.8

#### Estimated Upgrade Time: TODO

> {note} We attempt to document every possible breaking change. Since some of these breaking changes are in obscure parts of the framework only a portion of these changes may actually affect your application.

### Updating Dependencies

Update your `laravel/framework` dependency to `5.9.*` in your `composer.json` file.

Of course, don't forget to examine any 3rd party packages consumed by your application and verify you are using the proper version for Laravel 5.9 support.

### Miscellaneous

We also encourage you to view the changes in the `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). While many of these changes are not required, you may wish to keep these files in sync with your application. Some of these changes will be covered in this upgrade guide, but others, such as changes to configuration files or comments, will not be. You can easily view the changes with the [GitHub comparison tool](https://github.com/laravel/laravel/compare/5.8...master) and choose which updates are important to you.
