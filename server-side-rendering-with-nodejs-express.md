## Server-side Rendering with Node.js / Express

### Files

The basic structure of a React+Flux application (see [other examples](https://github.com/facebook/flux/tree/master/examples))

```
 - /src/actions/AppActions.js     - Action creators (Flux)
 - /src/components/Application.js - The top-level React component
 - /src/constants/ActionTypes.js  - Action types (Flux)
 - /src/core/Dispatcher.js        - Dispatcher (Flux)
 - /src/stores/AppStore.js        - The main store (Flux)
 - /src/app.js                    - Client-side startup script
 - /src/server.js                 - Server-side startup script
```

### The Top-level React Component

```javascript
// src/components/Application.js

var React = require('react');

var Application = React.createClass({

  propTypes: {
    path: React.PropTypes.string.isRequired,
    onSetTitle: React.PropTypes.func.isRequired
  },

  render() {
    var page = AppStore.getPage(this.props.path);
    this.props.onSetTitle(page.title);
    
    return (
      <div className="container">
        <h1>{page.title}</h1>
        <div>{page.content}</div>
      </div>
    );
  }
});

module.exports = Application;
```

### Client-side Startup Script (aka Bootstrap)

```javascript
// src/app.js

var React = require('react');
var Dispatcher = require('./core/Dispatcher');
var Application = require('./components/Application');

var app = React.createElement(Application, {
  path: window.location.pathname,
  onSetTitle: (title) => document.title = title
}));

app = React.render(app, document.body);

Dispatcher.register((payload) => {
  if (payload.action.actionType === ActionTypes.CHANGE_LOCATION) {
    app.setProps({path: payload.action.path});
  }
});
```

### Server-side Startup Script (Node.js/Express)

```javascript
// src/server.js

var _ = require('lodash');
var express = require('express');
var React = require('react');

// The top-level React component + HTML template for it
var Application = React.createFactory(require('./components/Application'));
var template = fs.readFileSync(path.join(__dirname, 'index.html'), 'utf8');

var server = express();

server.set('port', (process.env.PORT || 5000));

// Server-side rendering (SSR)
server.get('*', function(req, res) {
  var data = {};
  var component = Application({
    path: req.path,
    onSetTitle: (title) => data.title = title,
    onPageNotFound: () => res.status(404)
  });
  data.body = React.renderToString(component);
  var html = _.template(template, data);
  res.send(html);
});

server.listen(server.get('port'), function() {
  console.log('HTTP server is running at http://localhost:' + server.get('port'));
});
```

### HTML Template for Server-side Rendering

```html
// src/index.html

<!doctype html>
<html class="no-js" lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title><%- title %></title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="css/app.css">
  </head>
  <body>
    <!--[if lt IE 8]>
      <p class="browsehappy">
        You are using an <strong>outdated</strong> browser.
        Please <a href="http://browsehappy.com/">upgrade your browser</a>
        to improve your experience.
      </p>
    <![endif]-->
    <%= body %>
    <script src="app.js"></script>
  </body>
</html>
```

<hr>
Back to [React and Flux Recipes](./README.md)