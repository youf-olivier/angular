# Angular compiler options

When you use [AoT compilation](guide/aot-compiler), you can control how your application is compiled by specifying *template* compiler options in the `tsconfig.json` [TypeScript configuration file](guide/typescript-configuration).

The template options object, `angularCompilerOptions`, is a sibling to the `compilerOptions` object that supplies standard options to the TypeScript compiler.

```json
    {
      "compilerOptions": {
        "experimentalDecorators": true,
                  ...
      },
      "angularCompilerOptions": {
        "fullTemplateTypeCheck": true,
        "preserveWhitespaces": true,
                  ...
      }
  }
  ```

{@a tsconfig-extends}
## Configuration inheritance with extends

Like the TypeScript compiler, The Angular AoT compiler also supports `extends` in the `angularCompilerOptions` section of the TypeScript configuration file, `tsconfig.json`.
The `extends` property is at the top level, parallel to `compilerOptions` and `angularCompilerOptions`.

A TypeScript configuration can inherit settings from another file using the `extends` property.
The configuration options from the base file are loaded first, then overridden by those in the inheriting `tsconfig` file.

For example:

```json
{
  "extends": "../tsconfig.base.json",
  "compilerOptions": {
    "experimentalDecorators": true,
    ...
  },
  "angularCompilerOptions": {
    "fullTemplateTypeCheck": true,
    "preserveWhitespaces": true,
    ...
  }
}
```

For more informaton, see the [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html).

## Template options

The following options are available for configuring the AoT template compiler.

### `allowEmptyCodegenFiles`

When true, generate all possible files even if they are empty. Default is false. Used by the Bazel build rules to simplify how Bazel rules track file dependencies. Do not use this option outside of the Bazel rules.

### `annotationsAs`

Modifies how Angular-specific annotations are emitted to improve tree-shaking. Non-Angular annotations are not affected. One of `static fields` (the default) or `decorators`.

* By default, the compiler replaces decorators with a static field in the class, which allows advanced tree-shakers like [Closure compiler](https://github.com/google/closure-compiler) to remove unused classes.

* The `decorators` value leaves the decorators in place, which makes compilation faster. TypeScript emits calls to the` __decorate` helper. Use `--emitDecoratorMetadata` for runtime reflection (but note taht the resulting code will not properly tree-shake.

### `annotateForClosureCompiler`

When true, use [Tsickle](https://github.com/angular/tsickle) to annotate the emitted JavaScript with [JSDoc](http://usejsdoc.org/) comments needed by the
[Closure Compiler](https://github.com/google/closure-compiler). Default is false.

### `disableExpressionLowering`

When true (the default), transforms code that is or could be used in an annotation, to allow it to be imported from template factory modules. See [metadata rewriting](guide/aot-compiler#metadata-rewriting) for more information.

When `false`, disables this rewriting, requiring the rewriting to be done manually.

### `disableTypeScriptVersionCheck`

When `true`, the compiler does not check the TypeScript version and does not report an error when an unsupported version of TypeScript is used. Not recommended, as unsupported versions of TypeScript might have undefined behavior. Default is false.

### `enableResourceInlining`

When true, replaces the `templateUrl` and `styleUrls` property in all `@Component` decorators with inlined contents in `template` and `styles` properties.

When enabled, the `.js` output of `ngc` does not include any lazy-loaded template or style URLs.


{@a enablelegacytemplate}

### `enableLegacyTemplate`

When true, enables use of the `<template>` element, which was deprecated in Angular 4.0, in favor of `<ng-template>` (to avoid colliding with the DOM's element of the same name). Default is false. Might be required by some third-party Angular libraries. |

### `flatModuleId`

The module ID to use for importing a flat module (when `flatModuleOutFile` is true). References generated by the template compiler use this module name when importing symbols
from the flat module. Ignored if `flatModuleOutFile` is false.

### `flatModuleOutFile`

When true, generates a flat module index of the given file name and the corresponding flat module metadata. Use to create flat modules that are packaged similarly to `@angular/core` and `@angular/common`. When this option is used, the `package.json` for the library should refer
to the generated flat module index instead of the library index file.

Produces only one `.metadata.json` file, which contains all the metadata necessary
for symbols exported from the library index. In the generated `.ngfactory.js` files, the flat
module index is used to import symbols that includes both the public API from the library index
as well as shrowded internal symbols.

By default the `.ts` file supplied in the `files` field is assumed to be the library index.
If more than one `.ts` file is specified, `libraryIndex` is used to select the file to use.
If more than one `.ts` file is supplied without a `libraryIndex`, an error is produced.

A flat module index `.d.ts` and `.js` is created with the given `flatModuleOutFile` name in the same location as the library index `.d.ts` file.

For example, if a library uses the `public_api.ts` file as the library index of the module, the `tsconfig.json` `files` field would be `["public_api.ts"]`.
The `flatModuleOutFile` options could then be set to (for example) `"index.js"`, which produces `index.d.ts` and  `index.metadata.json` files.
The `module` field of the library's `package.json` would be `"index.js"` and the `typings` field
would be `"index.d.ts"`.

### `fullTemplateTypeCheck`

When true (recommended), enables the [binding expression validation](guide/aot-compiler#binding-expression-validation) phase of the template compiler, which uses TypeScript to validate binding expressions.

Default is currently false.

### `generateCodeForLibraries`

When true (the default), generates factory files (`.ngfactory.js` and `.ngstyle.js`)
for `.d.ts` files with a corresponding `.metadata.json` file.

When false, factory files are generated only for `.ts` files. Do this when using factory summaries.


### `preserveWhitespaces`

When false (the default), removes blank text nodes from compiled templates, which results in smaller emitted template factory modules. Set to true to preserve blank text nodes.

### `skipMetadataEmit`

When true, does not to produce `.metadata.json` files. Default is `false`.

The `.metadata.json` files contain information needed by the template compiler from a `.ts`
file that is not included in the `.d.ts` file produced by the TypeScript compiler.
This information includes, for example, the content of annotations  (such as a component's template), which TypeScript emits to the `.js` file but not to the `.d.ts` file.

You can set to `true` when using factory summaries, because the factory summaries
include a copy of the information that is in the `.metadata.json` file.

Set to `true` if you are using TypeScript's `--outFile` option, because the metadata files
are not valid for this style of TypeScript output. However, we do not recommend using `--outFile` with Angular. Use a bundler, such as [webpack](https://webpack.js.org/), instead.

### `skipTemplateCodegen`

When true, does not emit `.ngfactory.js` and `.ngstyle.js` files. This turns off most of the template compiler and disables the reporting of template diagnostics.

Can be used to instruct the template compiler to produce `.metadata.json` files for distribution with an `npm` package while avoiding the production of `.ngfactory.js` and `.ngstyle.js` files that cannot be distributed to `npm`.

### `strictMetadataEmit`

When true, reports an error to the `.metadata.json` file if `"skipMetadataEmit"` is `false`.
Default is false. Use only when `"skipMetadataEmit"` is false and `"skipTemplateCodeGen"` is true.

This option is intended to validate the `.metadata.json` files emitted for bundling with an `npm` package. The validation is strict and can emit errors for metadata that would never produce an error when used by the template compiler. You can choose to suppress the error emitted by this option for an exported symbol by including `@dynamic` in the comment documenting the symbol.

It is valid for `.metadata.json` files to contain errors.
The template compiler reports these errors if the metadata is used to determine the contents of an annotation.
The metadata collector cannot predict the symbols that are designed for use in an annotation, so it preemptively includes error nodes in the metadata for the exported symbols.
The template compiler can then use the error nodes to report an error if these symbols are used.

If the client of a library intends to use a symbol in an annotation, the template compiler does not normally report this until the client uses the symbol.
This option allows detection of these errors during the build phase of
the library and is used, for example, in producing Angular libraries themselves.

### `strictInjectionParameters`

When true (recommended), reports an error for a supplied parameter whose injection type cannot be determined. When false (currently the default), constructor parameters of classes marked with `@Injectable` whose type cannot be resolved produce a warning.

### `trace`

When true, prints extra information while compiling templates. Default is false.