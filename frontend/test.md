# 自动化集成测试

<!-- toc -->

## 什么是测试

### [为什么需要测试](http://javascript.ruanyifeng.com/tool/testing.html#toc0)


Web应用程序越来越复杂，这意味着有更多的可能出错。测试是帮助我们提高代码质量、降低错误的最好方法和工具之一。

- 测试可以确保得到预期结果。
- 加快开发速度。
- 方便维护。
- 提供用法的文档。

通过测试提供软件的质量，在开始的时候，可能会降低开发速度。但是从长期看，尤其是那种代码需要长期维护、不断开发的情况，测试会大大加快开发速度，减轻维护难度。

### [测试的方法](http://javascript.ruanyifeng.com/tool/testing.html#toc0)

测试中TDD与BDD是两个重要概念，但它们并不像名称看起来那样具有对比性，实际上它们是不同层面的两个概念。TDD告诉你什么时候写测试，BDD则告诉你怎么写测试。

> TDD是“测试驱动的开发”（Test-Driven Development）的简称，指的是先写好测试，然后再根据测试完成开发。使用这种开发方式，会有很高的测试覆盖率。

> BDD是“行为驱动的开发”（Behavior-Driven Development）的简称，指的是写出优秀测试的最佳实践的总称。

TDD的开发步骤如下：

- 先写一个测试。
- 写出最小数量的代码，使其能够通过测试。
- 优化代码。
- 重复前面三步。

有关如何写测试，BDD认为，不应该针对代码的实现细节写测试，而是要针对行为写测试。

### 注意的点

- 不要让TDD变成教条，凡事从实际出发。
- 只写必须的测试。
- 避免重复测试。
- 不断提高自己编码水平，写出优雅简洁的代码。

## JavaScript测试

测试框架是组织和运行测试用例的工具。

### 测试框架之Mocha

> Mocha is a feature-rich JavaScript test framework running on Node.js and in the browser, making asynchronous testing simple and fun. Mocha tests run serially, allowing for flexible and accurate reporting, while mapping uncaught exceptions to the correct test cases.

> Mocha是一个支持异步测试并运行在Node.js和浏览器的JavaScrit测试框架。它以串行方式运行，提供灵活准确的报告，同时将异常映射到正确的测试用例上。

#### 安装

```
项目安装：npm install mocha
全局安装：sudo npm install -g mocha
```
安装完成后便可以使用`mocha`命令。

#### 文件配置

Mocha默认运行根目录下`test`子目录第一层测试用例。若`test`子目录下还有子孙目录，并且你想运行`test`下全部测试用例，则需要给Mocha指定--recursive参数，表示递归读取子目录。

Mocha命令支持多参数，如`mocha --recursive --reporter tap --growl`。你可以将参数写在package.json，也可以在`test`目录下放置`mocha.opts`文件然后将参数写入其中，如：

```
// mocha.opts
--reporter tap
--recursive
--growl
--watch
--bail
--grep `用例名`
--invert
```

#### 书写测试脚本

书写测试脚本主要是书写测试用例，每个测试用例使用`it`定义，语法如`it(用例名， 执行的函数)`。`describe`将`it`分类包裹，如下：
```
// add.test.js
var add = require('./add.js');
var expect = require('chai').expect;

describe('加法函数的测试', function() {
  it('1 加 1 应该等于 2', function() {
    expect(add(1, 1)).to.be.equal(2);
  });
  it('.2 加 .1 应该等于 .3', function() {
    expect(add(.1, .2)).to.be.equal(.3);
  });
});
```

####  钩子函数

Mocha在describe块之中，提供测试用例的四个钩子：before()、after()、beforeEach()和afterEach()。它们会在指定时间执行。

```
describe('hooks', function() {

  before(function() {
    // 在本区块的所有测试用例之前执行
  });

  after(function() {
    // 在本区块的所有测试用例之后执行
  });

  beforeEach(function() {
    // 在本区块的每个测试用例之前执行
  });

  afterEach(function() {
    // 在本区块的每个测试用例之后执行
  });

  // test cases
});
```

#### 异步测试

Mocha默认每个测试用例最多执行2000毫秒，如果到时没有得到结果，就报错。对于涉及异步操作的测试用例，这个时间往往是不够的，需要用-t或--timeout参数指定超时门槛`mocha -t 5000 test/timeout.js`。

异步测试时，it块需要知道程序什么时候执行完成了，所以it块执行的时候，需传入一个done参数，当测试结束的时候，必须显式调用这个函数，告诉Mocha测试结束了。

```
it('测试应该5000毫秒后结束', function(done) {
  var x = true;
  var f = function() {
    x = false;
    expect(x).to.be.not.ok;
    done(); // 通知Mocha测试结束
  };
  setTimeout(f, 4000);
});
```

Mocha默认会高亮显示超过75毫秒的测试用例，可以用-s或--slow调整这个参数。`mocha -t 5000 -s 1000 test/timeout.js`


Mocha内置对Promise的支持，允许直接返回Promise，等到它的状态改变，再执行断言，而不用显式调用done方法。

```
it('异步请求应该返回一个对象', function() {
  return fetch('https://api.github.com')
    .then(function(res) {
      return res.json();
    }).then(function(json) {
      expect(json).to.be.an('object');
    });
});
```

除了Mocha以外，类似的测试框架还有Jasmine、jest等。

### 断言库(assertion library)

#### 什么是断言库

> Assertion libraries are tools to verify that things are correct.

> 断言库是用来验证对象是否正确的工具。

上述示例中`chai`就是断言库。Mocha本身不包含断言库。除了`chai`还有node.js的`assert`断言库、should.js断言库。由于实现的功能大体相同，它们的使用方式也大同小异。

常用断言：

- 判断是否相等
- 判断是否深度相等
- 判断是否为真
- 判断抛出错误/错误类型
- 判断是否没有抛出错误

由于`assert`是node.js内置的断言库，在debug方面具有优势。

#### 几个断言库
- [chai](https://www.chaijs.com/api/bdd/)
- [node.js assert](http://nodejs.cn/api/assert.html#assert_assert_deepequal_actual_expected_messag)
- [shoud.js](https://github.com/shouldjs/should.js)

**should.js检测函数内抛出异常**

```
var should = require('should');
describe('test', function() {
  it('throw', function() {
    function isPositive(n) {
      if(n <= 0) throw new Error('Given number is not positive')
    }
    isPositive.bind(null, 10).should.not.throw();
    isPositive.bind(null, -10).should.throw();
  });
});
```

### 测试运行器

#### Karma

> Karma is essentially a tool which spawns a web server that executes source code against test code for each of the browsers connected.

> Karma本质上是虚拟一个Web服务器的工具，该服务器针对连接的每个浏览器执行测试代码。

Karma能让我们批量在浏览器中测试代码。[支持的浏览器插件列表](https://www.npmjs.com/search?q=keywords:karma-launcher)。使用Karma需引入测试框架，`npm install -g Karma`后，运行`karma init`命令，在终端中会询问我们关于Karma配置的信息，包括选择什么测试框架、引入哪种浏览器测试插件、测试代码放在哪个文件夹、引入什么依赖等信息，最后会在根目录生成包含这些信息的[karma.conf.js](http://karma-runner.github.io/3.0/config/configuration-file.html)文件(大多数配置项也在cli中被支持)。karma.conf.js中有个singleRun配置项，适用于持续集成，如果设置成true，Karma将测试用例在各浏览器运行一次后就退出。

运行`karma start`后，karma会加载插件和配置文件，并在本地启动一个监听连接的服务。
任何等待服务器websockets的浏览器会立即重新连接。作为加载插件的一部分，测试报告会注册“浏览器”事件，以便准备好测试结果。我们可以在终端查看测试结果，也可以点击浏览器访问页的debug按钮在新打开页面的控制台查看测试信息。

##### 文件匹配

```
Examples:

**/*.js: All files with a "js" extension in all subdirectories
**/!(jquery).js: Same as previous, but excludes "jquery.js"
**/(foo|bar).js: In all subdirectories, all "foo.js" or "bar.js" files
```

### 持续集成(CI)

> Continuous Integration is the practice of merging in small code changes frequently - rather than merging in a large change at the end of a development cycle. The goal is to build healthier software by developing and testing in smaller increments. This is where Travis CI comes in.

> 持续集成是高频合并小改动的实践，区别于在开发周期快结束时才合并一次大改动。它的目标是通过小增量开发和测试来构建更健壮的软件。这也是Travis CI产生的目的。

#### Travis CI

> As a continuous integration platform, Travis CI supports your development process by automatically building and testing code changes, providing immediate feedback on the success of the change. Travis CI can also automate other parts of your development process by managing deployments and notifications.

> 作为一个持续集成平台，Travis CI 通过自动打包和测试代码改动，以及提供对结果的实时反馈来帮助你开发。通过管理部署和通知，Travis CI也能自动化你的其它开发过程。

##### 配置文件

在项目根目录新建[.travis.yml](https://docs.travis-ci.com/user/customizing-the-build/)文件，示例如下：

```
language: node_js

node_js:
  - "iojs"
  - "7"

before_script:
  - npm install -g gulp-cli

script: gulp

before_install:
  - npm i -g npm@version-number
  - curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version version-number
  - export PATH="$HOME/.yarn/bin:$PATH"

cache:
  yarn: true

addons:
  firefox: "latest"
```

`.travis.yml`文件包含了项目运行环境、依赖等信息，生命周期相关配置项如下：

```
OPTIONAL Install apt addons
OPTIONAL Install cache components
before_install
install
before_script
script
OPTIONAL before_cache (for cleaning up cache)
after_success or after_failure
OPTIONAL before_deploy
OPTIONAL deploy
OPTIONAL after_deploy
after_script
```

编辑`.travis.yml`文件后，将它推到你的GitHub账号，若你在Travis CI平台已经激活项目，Travis CI会检查配置设置环境自动打包测试脚本。若测试脚本全部通过则可将项目部署到web服务器或项目主机上。


##### 平台测试

使用github账号登陆[Travis CI免费平台](https://travis-ci.org/)后，Travis CI会同步你的GitHub项目信息。你可以在平台上添加并激活特定项目。项目激活后，若项目中存在`.travis.yml`文件，每次向github push代码后会触发平台的构建测试（Github提供了[webhooks](https://developer.github.com/webhooks/)让你订阅github上的特定事件），依照你的配置运行、构建、测试、部署代码。



## 资料

* [Test Driven Development: what it is, and what it is not](https://medium.freecodecamp.org/test-driven-development-what-it-is-and-what-it-is-not-41fa6bca02a2)
* [What’s the difference between Unit Testing, TDD and BDD?](https://codeutopia.net/blog/2015/03/01/unit-testing-tdd-and-bdd/)
* [测试的道理](http://www.yinwang.org/blog-cn/2016/09/14/tests)
* [Egg单元测试](https://eggjs.org/zh-cn/core/unittest.html)
* [What is the difference between a test runner, testing framwork, assertion library, and a testing plugin?](https://amzotti.github.io/testing/2015/03/16/what-is-the-difference-between-a-test-runner-testing-framework-assertion-library-and-a-testing-plugin/)
* [测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)
* [JavaScript 程序测试](http://javascript.ruanyifeng.com/tool/testing.html)
* [How karma works？](http://karma-runner.github.io/3.0/intro/how-it-works.html)
* [Travis CI Node](https://docs.travis-ci.com/user/languages/javascript-with-nodejs/)
* [持续集成服务 Travis CI 教程](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html)
