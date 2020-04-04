---
layout: post
title: "Webpack errors in my Rails app"
tags:
- webpack
- rails
---

While spinning up my client's Rails project, I was getting a weird webpack error:
`configuration has an unknown property 'mode'`.

Full error:

```
Invalid configuration object. Webpack has been initialised using a configuration object that does not match the API schema.
 - configuration has an unknown property 'mode'. These properties are valid:
   object { amd?, bail?, cache?, context?, dependencies?, devServer?, devtool?, entry, externals?, loader?, module?, name?, node?, output?, parallelism?, performance?, plugins?, profile?, recordsInputPath?, recordsOutputPath?, recordsPath?, resolve?, resolveLoader?, stats?, target?, watch?, watchOptions? }
   For typos: please correct them.
   For loader options: webpack 2 no longer allows custom properties in configuration.
     Loaders should be updated to allow passing options via loader options in module.rules.
     Until loaders are updated one can use the LoaderOptionsPlugin to pass these options to the loader:
     plugins: [
       new webpack.LoaderOptionsPlugin({
         // test: /\.xxx$/, // may apply this only for some modules
         options: {
           mode: ...
         }
       })
     ]
```

After spending some time this error, I found a GitHub 
**[issue](https://github.com/JeffreyWay/laravel-mix/issues/1825#issuecomment-441420136)**
where it has mentioned:

> I think mode was introduced in Webpack 4

I opened `Gemfile.lock` file, I saw that I had webpacker gem version `3.6.0`. I
followed this GitHub **[issue](https://github.com/rails/webpacker/issues/1923)**
and upgraded webpacker gem version to `4.2.2`. Ran `yarn add @rails/webpacker@next` 
command to update `package.json` file.

If you have `.babelrc` file in your directory, you might get this error:
```
Cannot find module 'babel-plugin-syntax-dynamic-import'
```
Please remove that `.babelrc` file and it will resolve the above issue.

After all these changes, finally, I was able to run my webpack server 😄.
Sometimes Node.js issues are too cumbersome.

Please give me your feedback on these fixes.

Happy Hacking!
