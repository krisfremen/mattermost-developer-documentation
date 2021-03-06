---
title: Quick Start
date: 2018-07-10T00:00:00-05:00
subsection: Web App Plugins
weight: -10
---

This tutorial will walk you through the basics of extending the Mattermost web app.

Note that the steps below are intentionally very manual to explain all of the pieces fitting together. In practice, we recommend referencing [mattermost-plugin-sample](https://github.com/mattermost/mattermost-plugin-sample) for helpful build scripts. Also, the plugin API changed in Mattermost 5.2. Consult the [migration](/extend/plugins/migration) document to upgrade older plugins.

## Prerequisites

Plugins, just like the Mattermost web app itself, are built using [ReactJS](https://reactjs.org/) with [Redux](https://redux.js.org/). Make sure to install [npm](https://www.npmjs.com/get-npm) to manage your JavaScript dependencies.

You'll also need a Mattermost server to install and test the plugin. This server must have [Enable](https://docs.mattermost.com/administration/config-settings.html#enable-plugins) set to true in the [PluginSettings](https://docs.mattermost.com/administration/config-settings.html#plugins-beta) section of its config file. If you want to upload plugins via the System Console or API, you'll also need to set [EnableUploads](https://docs.mattermost.com/administration/config-settings.html#enable-plugin-uploads) to true in the same section.

## Setting up the Workspace

Create a directory to act as your plugin workspace. With that directory, create and switch to a `webapp` directory:

```bash
mkdir webapp
cd webapp
```

Install the necessary NPM dependencies:

```bash
npm install --save-dev webpack webpack-cli babel-loader babel-core babel-preset-env babel-preset-react
npm install --save react
```

Configure Webpack by creating a `webpack.config.js` file:

```js
var path = require('path');

module.exports = {
    entry: [
        './src/index.jsx',
    ],
    resolve: {
        modules: [
            'src',
            'node_modules',
        ],
        extensions: ['*', '.js', '.jsx'],
    },
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['env', 'react'],
                    },
                },
            },
        ],
    },
    externals: {
        react: 'React',
    },
    output: {
        path: path.join(__dirname, '/dist'),
        publicPath: '/',
        filename: 'main.js',
    },
};
```

Observe that `react` is specified as an external library. This allows you to test your code locally (e.g. with [jest](https://jestjs.io/) and snapshots) but leverage the version of React shipped with Mattermost to avoid bloating your plugin.

Now create the entry point file and output directory:
```bash
mkdir src dist
touch src/index.jsx
```

Then populate `src/index.jsx` with the following:
```js
import React from 'react'

// Courtesy of https://feathericons.com/
const Icon = () => <i className='icon fa fa-plug'/>;

class HelloWorldPlugin {
    initialize(registry, store) {
        registry.registerChannelHeaderButtonAction(
            // icon - JSX element to use as the button's icon
            <Icon />,
            // action - a function called when the button is clicked, passed the channel and channel member as arguments
            // null,
            () => {
                alert("Hello World!");
            },
            // dropdown_text - string or JSX element shown for the dropdown button description
            "Hello World",
        );
    }
}

window.registerPlugin('com.mattermost.webapp-hello-world', new HelloWorldPlugin());
```

Generate a minified bundle ready to install as a web app plugin:

```bash
./node_modules/.bin/webpack --mode=production
```

Now, we'll need to define the required manifest describing your plugin's entry point. Create a file named `plugin.json` with the following contents:

```json
{
    "id": "com.mattermost.webapp-hello-world",
    "name": "helloworld",
    "description": "",
    "webapp": {
        "bundle_path": "main.js"
    }
}
```

This manifest gives the server the location of our web app  within our bundle. (Note that you may alternatively use `plugin.yaml`, as shown in [../../server/hello-world/](../../server/hello-world/).)

Bundle the manifest and entry point into a tar file:

```bash
mkdir -p com.mattermost.webapp-hello-world
cp -r dist/main.js com.mattermost.webapp-hello-world/
cp plugin.json com.mattermost.webapp-hello-world/
tar -czvf plugin.tar.gz com.mattermost.webapp-hello-world
```

You should now have a file named `plugin.tar.gz` in your workspace. Congratulations! This is your first web app plugin!

## Installing the Plugin

Install the plugin in one of the following ways:

1) Through System Console UI:

   - Log in to Mattermost as a System Admin.
   - Open the System Console at `/admin_console`
   - Navigate to **Plugins (Beta) > Management** and upload the `plugin.tar.gz` you generated above.
   - Click **Enable** under the plugin after it has uploaded.

2) Or, manually:

 - Extract `plugin.tar.gz` to a folder with the same name as the plugin id you specified in ``plugin.yaml``, in this case `com.mattermost.server-hello-world/`.
 - Add the plugin to the directory set by **PluginSettings > Directory** in your ``config.json`` file. If none is set, defaults to `./plugins` relative to your Mattermost installation directory. The resulting directory structure should look something like:

    ```
     mattermost/
        plugins/
            com.mattermost.webapp-hello-world/
                plugin.json
                main.js
     ```
 - Restart the Mattermost server.

Navigate to a regular Mattermost page and observe the new icon in the channel header. Click the icon and observe the alert dialog.
