## Release Notes

Generate release note pages from git commit history.

### Installation

It's preferable to install it globally through [`npm`](https://www.npmjs.com/package/git-release-notes)

    npm install -g git-release-notes

### Usage

The basic usage is

    cd <your_git_project>
    git-release-notes <since>..<until> <template>

Where

* `<since>..<until>` specifies the range of commits as in `git log`, see [gitrevisions(7)](http://www.kernel.org/pub/software/scm/git/docs/gitrevisions.html)
* `<template>` is an [ejs](https://github.com/visionmedia/ejs) template file used to generate the release notes

Three sample templates are included as a reference in the `templates` folder

 * `markdown` [(sample)](https://github.com/ariatemplates/git-release-notes/blob/master/samples/output-markdown.md)
 * `html` [(sample)](http://htmlpreview.github.io/?https://github.com/ariatemplates/git-release-notes/blob/master/samples/output-html.html)
 * `html-bootstrap` [(sample)](http://htmlpreview.github.io/?https://github.com/ariatemplates/git-release-notes/blob/master/samples/output-html-bootstrap.html)

This for example is the release notes generated for `joyent/node` by running

    git-release-notes v0.9.8..v0.9.9 html > changelog.html

[<img src="https://github.com/ariatemplates/git-release-notes/raw/master/samples/node_thumb.png" alt="Node's release notes">](https://github.com/ariatemplates/git-release-notes/raw/master/samples/node.png)


#### Custom template

The second parameter of `git-release-notes` can be any path to a valid ejs template files.

##### Template Variables

Several template variables are made available to the script running inside the template.

`commits` is an array of commits, each containing

* `sha1` commit hash (%H)
* `authorName` author name (%an)
* `authorEmail` author email (%ae)
* `authorDate` author date (%aD)
* `committerName` committer name (%cn)
* `committerEmail` committer email (%ce)
* `committerDate` committer date (%cD)
* `title` subject (%s)
* `messageLines` array of body lines (%b)

`dateFnsFormat` is the date-fns [format](https://date-fns.org/docs/format) function. See the [html-bootstrap](https://github.com/ariatemplates/git-release-notes/blob/master/templates/html-bootstrap.ejs) for sample usage.

`range` is the commits range as passed to the command line

### Options

More advanced options are

* `p` or `path` Git project path, defaults to the current working path
* `b` or `branch` Git branch, defaults to `master`
* `t` or `title` Regular expression to parse the commit title (see next chapter)
* `m` or `meaning` Meaning of capturing block in title's regular expression
* `f` or `file` JSON Configuration file, better option when you don't want to pass all parameters to the command line, for an example see [options.json](https://github.com/ariatemplates/git-release-notes/blob/master/options.json)
* `s` or `script` External script for post-processing commits

#### Title Parsing

Some projects might have special naming conventions for the commit title.

The options `t` and `m` allow to specify this logic and extract additional information from the title.

For instance, [Aria Templates](https://github.com/ariatemplates/ariatemplates) has the following convention

    fix #123 Title of a bug fix commit
    feat #234 Title of a cool new feature

In this case using

```
git-release-notes -t "^([a-z]+) #(\d+) (.*)$" -m type -m issue -m title v1.3.6..HEAD html
```

generates the additional fields on the commit object

* `type` first capturing block
* `issue` second capturing block
* `title` third capturing block (redefines the title)


Another project using similar conventions is [AngularJs](https://github.com/angular/angular.js), [commit message conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#).

```
git-release-notes -t "^(\w*)(?:\(([\w\$\.]*)\))?\: (.*)$" -m type -m scope -m title v1.1.2..v1.1.3 markdown
```

#### Post Processing

The advanced options cover the most basic use cases, however sometimes you might need some additional processing, for instance to get commit metadata from external sources (Jira, GitHub, Waffle...)

Using `-s script_file.js` you can invoke any arbitrary node script with the following signature:

```js
module.exports = function(data, callback) {
  /**
   * Here `data` contains exactly the same values your template will normally receive. e.g.
   *
   * {
   *   commits: [], // the array of commits as described above
   *   range: '<since>..<until>',
   *   dateFnsFormat: function () {},
   * }
   *
   * Do all the processing you need and when ready call the callback passing the new data structure
   */
  callback({
    commits: data.commits.map(doSomething),
    extra: { additional: 'data' },
  });
  //
};
```

The object passed to the callback will be merged with the input data and passed back to the template.

For an example check `samples/post-processing.js`

### Debug

If the output is not what you expect, set the DEBUG environment variable:

#### Linux
    DEBUG=release-notes:* git-release-notes ...

#### Windows

    SET DEBUG=release-notes:*
    git-release-notes ...
