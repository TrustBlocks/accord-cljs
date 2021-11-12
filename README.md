# Accord + CLJS example


## Setup

### Install Shadow-cljs
2. Installation

2.1. Standalone via npm

You will need:

node.js (v6.0.0+, most recent version preferred)

npm or yarn

Any Java SDK (Version 8 or higher). OpenJDK or Oracle

In your project directory you’ll need a package.json. If you do not have one yet you can create one by running npm init -y. If you don’t have a project directory yet consider creating it by running

$ npx create-cljs-project my-project
This will create all the necessary basic files and you can skip the following commands.

If you have a package.json already and just want to add shadow-cljs run

NPM
$ npm install --save-dev shadow-cljs
Yarn
$ yarn add --dev shadow-cljs
For convenience you can run npm install -g shadow-cljs or yarn global add shadow-cljs. This will let you run the shadow-cljs command directly later. There should always be a shadow-cljs version installed in your project, the global install is optional.

### Setting up accord-cljs

```bash
npm install
```

## Dev

```bash

npx shadow-cljs watch frontend

```


#### Shadow-cljs Compiler option discussion

```
Option #2: :js-provider :external
shadow-cljs has the concept of a :js-provider built-in. This controls who is actually in charge of providing JS dependencies. For node builds this is just :require which maps all JS requires in your ns forms to regular node require calls. For :browser builds it defaults to :shadow which means shadow-cljs will provide all JS dependencies and bundle them for you. An additional :js-provider I added not too long ago is :external. It is similar to what :bundle from CLJS provides but with few different design choices.

{:builds
 {:app
  {:target :browser
   :output-dir "public/js"
   :modules
   {:main {:init-fn my.app/init}}}
   :js-options
   {:js-provider :external
    :external-index "target/index.js"}}}
In this example shadow-cljs will generate all the regular CLJS output and write it into the public/js directory. It will however not process any JS dependencies itself and instead just generates the additional :external-index file at the specified location. That file is just a regular JS file and will contain all the JS require calls that your CLJS code used and a bit of glue code that exposes them so that the CLJS code can find them at runtime.

Supposed you have

(ns my.app
  (:require ["react" :as react]))
The generated index file will contain require("react") which JS tools understand. You are responsible for further processing that file and making sure that the output of that is loaded before the CLJS output.

So you could for example run

npx webpack --entry target/index.js --output public/js/libs.js
and then include the generated libs.js from webpack and the generated main.js from shadow-cljs in your HTML.

<script defer src="/js/libs.js"></script>
<script defer src="/js/main.js"></script>
This is basically an automated version of the double-bundle approach that a few people have been using for a while.

However this is different in that the output is intended to stay separate. JS code lives in one and CLJS code in the other. JS code can’t interact with the CLJS code but CLJS code can access the provided JS dependencies. This does give you a very basic code-splitting out of the box which is a good default IMHO. However as mentiond in my previous post this kind of code-splitting is very limited and not as fine grained as what :js-provider :shadow will give you. You can still use :modules for your CLJS code but your external JS file might get unwieldly large and not fit your :modules properly as JS dependencies won’t be split at all.



### References 



  * [Shadow-cljs Installation and Docs](https://shadow-cljs.github.io/docs/UsersGuide.html#_installation)
  * [AWS Amplify with Clojurescript and Storybook](https://davidvujic.blogspot.com/2021/09/clojurescript-amplified.html) 
  * [Reframe Reagent and Slate.js](https://github.com/jhund/re-frame-and-reagent-and-slatejs/blob/master/src/cljs/rrs/ui/slatejs/core.cljs)
  * [Thomas Heller's explanation of why he did not use Webpack](https://code.thheller.com/blog/shadow-cljs/2018/06/15/why-not-webpack.html)
