* Create a new polymerx project: `polymerx create default name-of-your-project`
* Copy your src files (not the index.html) into the new project
  * Migrate the changes in your index.html manually, because what we need to keep is the injection of the bundle.js and how the webcomponents.js is loaded from polymerx
  * Update the lines in the index.html that load the app (in there it's called `sk-app`, but yours might be different)
* `npm run dev`
* If you get `cyclic dependency` error, which comes from toposort and can be resolved by https://github.com/facebook/create-react-app/issues/4667
* You should create `polymerx.config.js` file to overwrite Webpack configuration like this:

```js
/**
 * Function that mutates original webpack config.
 * Supports asynchronous changes when promise is returned.
 *
 * @param {object} config - original webpack config.
 * @param {object} env - options passed to CLI.
 * @param {WebpackConfigHelpers} helpers - object with useful helpers when working with config.
 **/
export default function (config, env, helpers) {
  let { plugin } = helpers.getPluginsByName(config, "HtmlWebpackPlugin")[0];
  plugin.options.chunksSortMode = 'none'
}

```

* This will make sure that you actually see all the cyclic dependencies and can fix them
* Issue: https://github.com/PolymerX/polymerx-cli/issues/179