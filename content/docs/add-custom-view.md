---
title: "Add a Custom View"
description: "How to add a custom view for specific steps."
draft: false
date: "2022-01-10"
tags:
- Custom View
categories:
- Development
- Extension
- Frontend

---

Kaoto's frontend is extendable, allowing you to create custom views that you
can configure to show for specific steps. We call these custom views **Step
Extensions**.

![Example of a Step Extension](/images/step-extension.png)

You can use the Step Extension API to interact with Kaoto's internal state,
whether that be to control deployments remotely, or to do some kind of data
transformation with the output of a particular step.

## Step Extensions

Each Step Extension is, in essence, a web application that uses Webpack 4.x
or later. This can even be in the form of built static files. In fact, this is
how we are able to host our [example](https://step-extension.netlify.app/) on Netlify. You can use this [example extension](https://github.com/KaotoIO/step-extension) as a guide.

Or maybe you already have an external application that you'd like to have
embedded within Kaoto. All you would have to do is point to it from your View
Definition catalog. We'll take a look at how you can do that in just a moment.

## Under the Hood

Because we are using Webpack 5's [module federation](https://webpack.js.org/concepts/module-federation/) under the hood, it means that your
application would need to use Webpack 4.x or later to take advantage of step
extensions. Webpack can be used in tandem with other tools like Rollup, and
you can even use it just for module federation if you choose to.

If you're not familiar with the concept of module federation, you don't need
to be. The important thing to know is that it gives you the flexibility to
deploy your application separately, and Kaoto wouldn't even need to be
restarted. Dependency alignment is a thing of the past with module
federation, as you can even specify whether you want to share a
particular version of a framework like React, or if you want to use your own.

## Enabling Module Federation in Your App

One of the biggest benefits of using module federation is that it works just
like any other `import`. This means **you don't have to make any alterations
to an existing code base**. It's simply a matter of configuring it.

When you enable module federation, your `webpack.config.js` file will look 
something like this:

```js
const HtmlWebPackPlugin = require("html-webpack-plugin");
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
const deps = require("./package.json").dependencies;

module.exports = {
  devServer: {
    port: 3000,
    historyApiFallback: true,
  },
  output: {
    publicPath: "http://localhost:3000/",
  },
  resolve: {
    extensions: [".tsx", ".ts", ".jsx", ".js", ".json"],
  },
  plugins: [
    new ModuleFederationPlugin({
      name: "funapp",
      filename: "remoteEntry.js",
      exposes: {
        "./FunComponent": "./src/FunComponent",
      },
      shared: {
        ...deps,
        react: {
          singleton: true,
          requiredVersion: deps.react,
        },
        "react-dom": {
          singleton: true,
          requiredVersion: deps["react-dom"],
        }, },
    }),
    new HtmlWebPackPlugin({
      template: "./src/index.html",
    }),
  ],
};
```

Some important things to note is the `ModuleFederationPlugin`, which comes 
built into Webpack. In it, you have a few properties that are relevant:

- `name`: The name of your application. This cannot conflict with another Step Extension `name`.
- `filename`: This is the name of your remote entry file, which is usually 
  `remoteEntry.js`.
- `exposes`: The files this application will expose to Kaoto. Typically, 
  this will be a single component. In this example, we are sharing a component 
  called `FunComponent`.
- `shared`: Any libraries you want to share with Kaoto.

There are other options we won't go into, but these are the basics.

## Add View Definition to repository

To make Kaoto aware of your Step Extension, you need to point to its
`remoteEntry.js` file (generated by Webpack), from your View Definition 
Catalog. You can use one of the [default View Definition catalogs](https://github.com/KaotoIO/kaoto-viewdefinition-catalog).

This is what that would look like:

```yaml {linenos=inline,hl_lines=[4,5,6],linenostart=1}
name: Fun Component
id: detail-step
type: step
url: http://localhost:8000
module: './FunComponent'
scope: 'funapp'
constraints: 
   -
      mandatory: true
      operation: CONTAINS_STEP_NAME
      parameter: twitter-search-source      
```

Most of these properties are simply mapping to the `ModuleFederationPlugin` 
properties we've just defined above:

- `name` is what will appear as the label of the new tab of the component
- `url` is where your application is running
- `module` here corresponds to the `exposes` object, or the components 
  you're exposing.
- `scope` MUST match the `name` of your application that we defined earlier

It is important that the `url` points to where you have deployed the extension
implemented in the previous step.

The `constraints` will define when this extension will be shown. In this 
example, the extension will be shown when configuring the 
`twitter-search-source` step. It will appear as a new "Fun Component" 
tab when you click on `twitter-search-source` in Kaoto.

Stay turned for details on what you can do with your Step Extension, as 
we've just released a brand-new API for interacting with Kaoto's state. 


