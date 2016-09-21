# read vinyl file stream

Turns out that reading all the files in a vinyl stream is cumbersome, and supporting all of the options is a littel bit annoying. I decided that I don't want to write that code more than once. So here is a library that does that. This is most useful for gulp plugins that need to transform all the files in a stream, though I am sure you can figure out other ways to use it too.

## Install

```bash
npm install read-vinyl-file-stream
```

## API

The module is a function that creates a transform stream. It will read the vinyl file, whether it is a buffer or a stream internally. It takes the following parameters, in order:

- **iterator** _{Function}_ Required - the function that will process the files.
- **encoding** _{String}_ Optional - the encoding to use for the content provided to the iterator function. By default, this is a UTF-8 string. The following options are supported:
  - `'utf8'` - provide the content in a UTF-8 string.
  - `'buffer'` - provide the content in a raw buffer. This is useful if you are processing binary files, for example.

The function that you provide to it has the following parameters, in order:

- **content** - the content of the file.
- **file** - the vinyl file itself.
- **stream** - the transform stream that is being iterated.
- **cb** - a callback to call once you are done processing the file.

## Examples

Observe all of the files:

```javascript
var readFiles = require('read-vinyl-file-stream');

var input = getVinylStream();

var hashOfFiles = {};

input.pipe(readFiles(function (content, file, stream, cb) {
    hashOfFiles[file.path] = content;

    cb();
}));
```

Transform the content of the file and output it back to the stream:

```javascript
var readFiles = require('read-vinyl-file-stream');

var input = getVinylStream();

input.pipe(readFiles(function (content, file, stream, cb) {
    var newContent = doWorkToTheContent(content);

    cb(null, newContent);
}));
```

Split the file into multiple files and output all of them to the stream:

```javascript
var readFiles = require('read-vinyl-file-stream');
var File = require('vinyl');

var input = getVinylStream();

input.pipe(readFiles(function (content, file, stream, cb) {
    var lines = content.split('\n');

    lines.forEach(function (line, idx) {
        stream.push(new File({
            contents: new Buffer(line),
            path: file.path + 'line' + idx
        }));
    });

    cb();
}));
```

Use inside gulp (to create a filter):

```javascript
var gulp = require('gulp');
var readFiles = require('read-vinyl-file-stream');

gulp.task('mytask', function() {
    return gulp.src('*.ext')
        .pipe(readFiles(content, file, stream, cb) {
            if (/^n/.test(content)) {
                return cb(null, content);
            }

            cb();
        })
        .pipe(gulp.dest('filesThatStartWithN'));
});
