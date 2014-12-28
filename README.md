React HN
===
This tutorial will show you how to build [the Hacker News front page in React](https://mking.github.io/react-hn). We are going to build small, self-contained components, and then we will compose them (like Lego bricks) to get the final result.

Technologies learned: React, Browserify

Background required: basic HTML/CSS/JS

There are four parts to this tutorial:

 1. [Build the NewsItem](#newsitem)

    <img src="img/NewsItem.png" width="532">
 2. [Build the NewsHeader](#newsheader)

    <img src="img/NewsHeader.png" width="532">
 3. [Build the NewsList](#newslist)

    <img src="img/NewsList.png" width="532">
 4. [Display live data](#hacker-news-api)

    During development, we use static data from the /json directory.

NewsItem
---
> Note: You will be building the app from scratch, but you can check your work against the solutions in this repo.

 1. Create the initial project.

     1. Create the directory for your project and cd into it. You will be doing your work out of this directory.
        ```
        mkdir hn
        cd hn
        ```

     2. Create the project directory structure.
        ```
        mkdir -p {build/js,css,html,img,js,json}
        ```

     3. [Download the sample data](https://raw.githubusercontent.com/mking/react-hn/master/json/items.json) into /json.

     4. Download [y18.gif](https://news.ycombinator.com/y18.gif) and [grayarrow2x.gif](https://news.ycombinator.com/grayarrow2x.gif) into /img.

 2. Create the initial NewsItem component.

     1. Create the demo page (/html/NewsItem.html). This way, we can preview the NewsItem component by itself, without requiring the full app.
        ```
        <!DOCTYPE html>
        <html>
          <head>
            <meta charset="utf-8">
            <title>NewsItem</title>
            <link href="/css/NewsItem.css" rel="stylesheet">
          </head>
          <body>
            <div id="content"></div>
            <script src="/build/js/NewsItem.js"></script>
          </body>
        </html>
        ```

     2. Create the JS file (/js/NewsItem.js). This is an empty component to make sure all our tools are hooked up correctly.
        ```
        var $ = require('jquery');
        var React = require('react');

        var NewsItem = React.createClass({
          render: function () {
            return (
              <div className="newsItem">
                NewsItem test
              </div>
            );
          }
        });

        React.render(<NewsItem/>, $('#content')[0]);
        ```

     3. Create CSS file (/css/NewsItem.css): an empty file.

 3. Install, configure, and run Browserify.

     1. Create /package.json.
        ```
        {
          "name": "hn",
          "version": "0.1.0",
          "private": true
        }
        ```

     2. Install Browserify, React, and tools.
        ```
        # These dependencies are required for the running app.
        npm install --save react jquery lodash moment

        # These dependencies are required for building the app.
        npm install --save-dev browserify watchify reactify

        # These dependencies are globally installed command line tools.
        npm install -g browserify watchify
        ```

     3. Configure Browserify in /package.json.
        ```
        {
          ...
          "browserify": {
            "transform": [
              ["reactify"]
            ]
          }
        }
        ```

     4. Run Watchify. We need Watchify in order to recompile React components whenever they change. I normally run this in a separate terminal tab.
        ```
        watchify -v -o build/js/NewsItem.js js/NewsItem.js
        ```

 4. Run the demo.
    ```
    python -m SimpleHTTPServer 8888
    ```

    Visit [http://localhost:8888/html/NewsItem.html](http://localhost:8888/html/NewsItem.html). You should see "NewsItem test".

    If you change the text, save the JS file, and refresh the browser, you should be able to see this change reflected in the page.

 5. Display the item title.

     1. Load data from the JSON file. I normally log the data so I can inspect the data structure in the developer console.
        ```
        var NewsItem = React.createClass({
          ...
          componentWillMount: function () {
            $.ajax({
              url: '/json/items.json'
            }).then(function (items) {
              console.log('items', items);
              this.setState({item: items[0]});
            }.bind(this));
          },
          ...
        ```

        <img src="img/DeveloperConsole.png" width="274">

     2. Initialize the component state.
         ```
        var NewsItem = React.createClass({
          ...
          getInitialState: function () {
            return {};
          },
          ...
        ```

     3. Display the data. If the state is not yet loaded, render nothing.
        ```
        var NewsItem = React.createClass({
          ...
          render: function () {
            if (!this.state.item) {
              return null;
            }

            return (
              <div className="newsItem">
                {this.state.item.title}
              </div>
            );
          }
          ...
        ```

     4. Refresh the browser. You should see the following.

        <img src="img/Title.png" width="107">

 6. Display the title link and domain. Note: We are following the [CSS style guide](https://medium.com/@fat/mediums-css-is-actually-pretty-fucking-good-b8e2a6c78b06) from Jacob Thornton (creator of Bootstrap).

     1. Update the JS.
        ```
        var url = require('url');
        ...
        var NewsItem = React.createClass({
          ...
          getDomain: function () {
            return url.parse(this.state.item.url).hostname;
          },
          ...
          render: function () {
            ...
            return (
              <div className="newsItem">
                <a className="newsItem-titleLink" href={this.state.item.url}>{this.state.item.title}</a>
                <div className="newsItem-domain">
                  ({this.getDomain()})
                </div>
              </div>
            );
          }
          ...
        ```
     2. Update the CSS.
        ```
        body {
          font-family: Verdana, sans-serif;
        }

        .newsItem {
          color: #828282;
        }

        .newsItem-domain {
          font-size: 8pt;
          margin-left: 5px;
        }

        .newsItem-titleLink {
          color: black;
          font-size: 10pt;
          text-decoration: none;
        }
        ```
     3. Refresh the browser. You should see the following.

        <img src="img/TitleDomain.png" width="213">

 7. Display the subtext.

     1. Update the JS. We factor out the title part into its own method.
        ```
        var moment = require('moment');
        ...
        var NewsItem = React.createClass({
          ...
          getCommentLink: function () {
            var commentText = 'discuss';
            if (this.state.item.kids.length) {
              commentText = this.state.item.kids.length + ' comments';
            }

            return (
              <a href={'https://news.ycombinator.com/item?id=' + this.state.item.id}>{commentText}</a>
            );
          },
          ...
          getSubtext: function () {
            return (
              <div className="newsItem-subtext">
                {this.state.item.score} points by <a href={'https://news.ycombinator.com/user?id=' + this.state.item.by}>{this.state.item.by}</a> {moment.utc(this.state.item.time * 1000).fromNow()} | {this.getCommentLink()}
              </div>
            );
          },
          ...
          getTitle: function () {
            return (
              <div className="newsItem-title">
                ...
              </div>
            );
          },
          ...
          render: function () {
            ...
            return (
              <div className="newsItem">
                {this.getTitle()}
                {this.getSubtext()}
              </div>
            );
          }
        ```

     2. Update the CSS.
        ```
        .newsItem-subtext {
          font-size: 7pt;
        }

        .newsItem-subtext > a {
          color: #828282;
          text-decoration: none;
        }

        .newsItem-subtext > a:hover {
          text-decoration: underline;
        }
        ```

     3. Refresh the browser. You should see the following.

        <img src="img/Subtext.png" width="268">

 8. Display the rank and vote.

     1. Update the JS. We use a fake rank for now.
         ```
        var NewsItem = React.createClass({
          ...
          getRank: function () {
            return (
              <div className="newsItem-rank">
                1.
              </div>
            );
          },
          ...
          getVote: function () {
            return (
              <div className="newsItem-vote">
                <a href={'https://news.ycombinator.com/vote?for=' + this.state.item.id + '&dir=up&whence=news'}>
                  <img src="/img/grayarrow2x.gif" width="10"/>
                </a>
              </div>
            );
          },
          ...
          render: function () {
            ...
            return (
              <div className="newsItem">
                {this.getRank()}
                {this.getVote()}
                <div className="newsItem-itemText">
                  {this.getTitle()}
                  {this.getSubtext()}
                </div>
              </div>
            );
          }
        ```

     2. Update the CSS.
        ```
        .newsItem {
          ...
          align-items: baseline;
          display: flex;  
        }
        ...
        .newsItem-itemText {
          flex-grow: 1;
        }
        ...
        .newsItem-rank {
          flex-basis: 25px;
          font-size: 10pt;
          text-align: right;
        }
        ...
        .newsItem-vote {
          flex-basis: 15px;
          text-align: center;
        }
        ```

     3. Refresh the browser. You should see the following.

        <img src="img/RankVote.png" width="297">


NewsHeader
---
<img src="img/NewsList.png" width="532" height="1000">

NewsList
---
<img src="img/NewsList.png" width="532" height="1000">

Hacker News API
---
<img src="img/NewsList.png" width="532" height="1000">

python -m SimpleHTTPServer 8888
watchify -v -o build/js/app.js js/app.js
scss --watch scss:build/css

NOT using ReactFire in order to illustrate JSON access.


Step 1: Show html
Step 2: Hook up react: Show js component (NewsList)
Step 3: Hook up css: Style the test div
Step 4: Get the data from local and print the one title.
Step 5: Print all titles. Factor out NewsItem.
- Fix the key missing warning.
Step 6: Hook up the urls.
- Hold open the data structure in the console.
Step 7: Let's develop NewsItem independently of NewsList.

title=>subtext=>

Style
---
JS
- One file per component
- Alphabetize methods and requires
- Use `get*` for render helpers. A component should have one main method (`render()`) that breaks out into smaller render helpers.
- Use `handle*` for event handlers.
- Follow the React team's JS style.

CSS
- One file per component
- Alphabetize rules and declarations
- Follow [Jacob Thornton's CSS style](https://medium.com/@fat/mediums-css-is-actually-pretty-fucking-good-b8e2a6c78b06) (components and variables only)
