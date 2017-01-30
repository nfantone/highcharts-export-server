# ![highcharts](https://www.dropbox.com/s/3se8pnb23b4csay/highcharts.png?raw=1)

[![Greenkeeper badge](https://badges.greenkeeper.io/nfantone/highcharts-export-server.svg)](https://greenkeeper.io/)

[![Build Status](https://travis-ci.org/nfantone/highcharts-export-server.svg?branch=develop)](https://travis-ci.org/nfantone/highcharts-export-server)

The file `lib/highcharts-convert.js` is a [PhantomJS](http://phantomjs.org/) script to convert SVG or Highcharts JSON options objects to chart images. It is ideal for batch processing Highcharts configurations for attaching to emails or reports. An online demo with a GUI can be viewed at [export.highcharts.com/demo](http://export.highcharts.com/demo).

> **Note**  The Highcharts files are subjected to the Highcharts License.

## Installation

- You can just install the module globally (preferred) or locally, and run it as a service.

```sh
# Global
sudo npm install -g highcharts-phantomjs
highcharts-phantomjs start
info: [highcharts-convert]    Started converter on 127.0.0.1:3030 (PID: 9351)

# Local
npm install highcharts-phantomjs
cd node_modules/highcharts-phantomjs
npm start
info: [highcharts-convert]    Started converter on 127.0.0.1:3030 (PID: 8342)
```

- Alternatively, you could clone this repository and run `phantomjs` manually.

```sh
git clone https://github.com/nfantone/highcharts-export-server
cd highcharts-export-server
npm install --production --no-optional
phantomjs lib/highcharts-convert.js -host 127.0.0.1 -port 3030
```

> For this option to work, you need to have `phantomjs` binary in your PATH (see http://phantomjs.org/download.html).

## Usage

```sh
Usage: highcharts-phantomjs start [-h HOST][-p PORT] -- [options]

Commands:
  start  Starts highcharts-convert HTTP server

Options:
  -h, --hostname, --host  Hostname of the server          [default: "127.0.0.1"]
  -p, --port              Port the server will listen on         [default: 3030]
  --help                  Show help                                    [boolean]
  --version               Show version number                          [boolean]
```


### Options

Besides `-h` (hostname) and `-p` (port number), any other arguments specified after `--` will be passed as is to the `phantomjs` script on `lib/highcharts-convert.js`.

The following are all the available options.

##### `-infile`
The file to convert, assumes it's either a JSON file, the script checks for the input file to have the extension '.json', or otherwise it assumes it to be an svg file.

##### `-outfile`
The file to output. Must be a filename with the extension .jpg, .png .pdf or .svg.

##### `-scale`
To set the zoomFactor of the page rendered by PhantomJS. For example, if the _chart.width_ option in the chart configuration is set to 600 and the scale is set to 2, the output raster image will have a pixel width of 1200. So this is a convenient way of increasing the resolution without decreasing the font size and line widths in the chart. This is ignored if the _-width_ parameter is set.

##### `-width`
Set the exact pixel width of the exported image or pdf. This overrides the _-scale_ parameter.

##### `-constr`
The constructor name. Can be one of _Chart_ or _StockChart_. This depends on whether you want to generate Highstock or basic Highcharts.

##### `-callback`
Filename of the _callback_. The _callback_ is a function which will be called in the constructor of Highcharts to be executed on chart load. All code of the callback must be enclosed by a function. See this example of contents of the callback file:

```javascript
function(chart) {
    chart.renderer.arc(200, 150, 100, 50, -Math.PI, 0).attr({
        fill : '#FCFFC5',
        stroke : 'black',
        'stroke-width' : 1
     }).add();
}
```

### API
After starting the converter server, it will listen for `POST` request. You can use the same parameters as for command line usage, but wrap them in a JSON structure.

Here is an example of a valid request, given a JSON body in some file:

```javascript
// example-request.json
{
    "infile": "{xAxis: {categories: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']},series: [{data: [29.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5, 216.4, 194.1, 95.6, 54.4]}]};",
    "callback": "function(chart) {chart.renderer.arc(200, 150, 100, 50, -Math.PI, 0).attr({fill : '#FCFFC5',stroke : 'black','stroke-width' : 1}).add();}",
    "constr": "Chart"
}
```

```sh
highcharts-phantomjs start -p 3003
info: [highcharts-convert]    Started converter on 127.0.0.1:3030 (PID: 1350)

# On another terminal
curl -XPOST http://localhost:3003 -H 'Content-Type: application/json' -d @example-request.json
```
