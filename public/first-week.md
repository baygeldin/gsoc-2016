# First week

So, the first week of GSoC has passed and here's my first weekly report. During this week the main goal was to get some `gdb-js` tests running. So, I've implemented some tests first and then started to raise functionality to match these tests. But as usual things have been changing along the way. 

## On tests

The first idea was to write tests this way: 
1. Mock the stdin/stdout/stderr streams
2. Pass them to the newly created GDB wrapper instance
3. Check the state consistency and the stdin output

For streams mocking I decided to create a command line util to run GDB and write the results of its execution to a json file (with streams splitting, of course). The problem here was that output of some programs when they are not attached to interactive device is line-buffered and debugging these programs becomes hard. The workaround I've come up with:

```javascript
let gdb = spawn('stdbuf', ['-i0', '-o0', '-e0',
  'gdb', '--interpreter=mi', './main'])
```

So far so good and it feels like it's a right approach to do unit tests. But there is one significant disadvantage to it: the tests are not clear. It's hard to reason about what the specific test should do by looking at the json file with recorded streams. So, I decided to take another approach — use the actual GDB instance. So the tests now are looking more like integration tests than unit tests. But it's fine... The only thing I don't like is that they're much slower for obvious reasons :) The integration tests are done this way:
* I've set up a [GitHub repo](https://github.com/baygeldin/gdb-examples) with Docker image and an [automation build for it](https://hub.docker.com/r/baygeldin/gdb-examples/) on DockerHub.
* I use `dockerode-promise` wrapper to run the Docker container with this image from Node.js
* And then execute tests within the container

The next challenge was to write the code that would match these tests. First, I created some abstract implementation of the GDB wrapper class. I choosed to use `highland.js` for stream handling since it has zero dependencies and awesome documentation. Then I started to think about parser.

## On parsers

GDB/MI output syntax plays well with JSON in fact. So the natural approach was to map it to JSON. I've considered several parser generators and decided to use `PEG.js` since it's very easy to use and it has a nice [online version](http://pegjs.org/online). There was one problem with symbols escaping in the grammar but I've handled it by taking an example of JSON parser for PEG.js:

```javascript
Const "c-string"
  = "\"" chars:Char* "\"" { return chars.join('') }

Char "char"
  = [\x20-\x21\x23-\x5B\x5D-\u10FFFF]
  / "\\" seq:Escaped { return seq }

Escaped "escaped"
  = ("\"" / "\\") / ("b" / "f" / "n" / "r" / "t") {
      return '\\' + text()
    }
```

By the way, seems like the official GDB/MI syntax has some mistakes according to their bugzilla but I hope it won't be a big problem.

## Results
* Parser for GDB/MI is working
* Test for state checking is passing

## P.S.
* During my parser generators investigation I've understood that my knowledge of syntax analysers is poor... So I planned to spend some time on them later on. On the other hand, I found an awesome and almost fresh article about [state-of-art analyzer](http://www.antlr.org/papers/allstar-techreport.pdf)!
* The next week I want to write even more tests and more methods for a GDB wrapper. I need to do it in pace in order to save some time for preparation to the exam which will be held on 9th of June.
