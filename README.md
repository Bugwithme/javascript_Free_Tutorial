# JavaScript异步编程指南

本章内容将讨论 JavaScript 中非常重要的一个概念：如何处理异步代码。这是 JavaScript 最具特色的特性，而理解异步代码，离不开一个最基础、也是最重要的概念——回调函数（Callback ）。

我们首先在异步编程语境下简单介绍回调函数。然后学习Promise，讲解Promise是什么以及如何使用Promise；接着我们会介绍 async/await，并演示一些常见模式，以及如何用async/await和Promise来处理异步代码。

1.回调函数概念

在这一节，我们首先了解回调函数。所有开发者或至少大部分开发者——应该已经熟悉“回调函数”这个术语。回调函数本质上是将一个函数作为参数传递给另一个函数，并在某个合适的时机被调用的函数。

1.1 函数式编程模式中回调函数

在 JavaScript 中，我们经常使用回调函数，我们也肯定在其他开发这编写的代码中看到过，甚至也编写过。一个简单的例子是JavaScript的函数式编程模式，以及很多内置数组方法，比如 forEach()、filter()、map()、reduce()。这些方法都要求我们提供一个函数作为参数。

假设有一个数字数组num，我们需要过滤出其中的偶数元素，我们可以使用filter()方法。代码如下：
```
const nums=[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15];
const evenNums=nums.filter(num => num % 2 === 0);
console.log(evenNums);
```
在这个示例中，我们给filter()方法传递了一个箭头函数，但是我们没有直接调用该函数，而是将其作为参数传给 filter方法。这个函数会接收一个 num参数，如果传入的数字是偶数就返回，也就是数字取模2等于0。如果运行这段代码，浏览器控制输出结果为：2、4、6、8、10、12、14。这正是使用回调函数的经典场景——我们也一直在这样使用。

1.2 事件驱动编程中的回调函数

另一种使用回调函数的常见情况是事件驱动编程，尤其是在处理DOM事件时。比如当用户点击表单提交按钮提交表单时，调用事件函数将数据发送到服务器，或者当用户按钮点击时，我们执行定义的某个操作。也就是说，我们给DOM监听事件传递一个函数，在特定事件被触发时调用该函数。这也是回调函数的一种使用场景。代码如下所示。
```
button.addEventListener('click', sayHi);

在这个例子中， sayHi就是一个回调函数，该函数会在用户点击按钮是被调用。
```
因此，从本质上来说，回调函数就是将函数作为参数传入给另一个函数，并在适当时机执行的函数。

1.3 setTimeout中的回调函数
第三种使用回调函数的场景是在处理异步代码。一个典型例子是 setTimeout：
```
setTimeout(sayHi, 3000);
```
我们给setTimeout传入函数sayHi，告诉JavaScript在3秒后调用sayHi 函数。我们也可以使用 setInterval设置重复执行 sayHi 函数的时间间隔。

1.4 异步操作中的回调函数

在过去，回调函数也常用于获取数据、发送 HTTP 请求等异步操作。例如，早期的Ajax库通常会要求开发中传入一个回调函数，在请求完成后执行。我们无法预知请求要花多长时间——可能很快，也可能很慢，甚至可能出错——于是我们传入一个回调函数，该回调函数通常接收 data参数，用于处理返回的结果。代码如下所示
```
makeRequest("/apl/users/1", function (data) {
    data.user.email
})
```
当然这是过去常用的模式。在下一节，我们会讨论为什么这种模式可能令人困扰，以及如今是否有更好的方式来处理异步操作。

1.5 小结
在本节，我们了解了回调函数的几种常用场景：

•	函数式编程模式（如 map、filter）；
•	事件驱动编程（如点击、提交等事件）；
•	异步操作（如 setTimeout、HTTP 请求）。

使用回调函数本身并不一定是坏事，我们在很多场景中仍会继续使用回调函数。但在处理异步代码时，我们还有更清晰、更优雅的替代方案——也就是 Promise 和 async/await，它们能显著减轻异步编程的精神负担。我们将在下一个节详细展开。

2.回调地狱

在上一节中，我们已经了解了什么是回调函数：回调函数就是作为参数传递给另一个函数，并由另一个个函数在合适的时候调用的函数。在很多场景下，回调函数都非常有用。

但现在，本节将专门聊一下用于异步代码的回调函数。在我刚开始编写 JavaScript 代码的时候，回调函数我处理异步代码唯一的方式——我需要传递大量的回调函数。

2.1 JavaScript是单线程语言

现在我们来谈谈一个问题：为什么在 JavaScript 中，我们必须以不同的方式来处理异步代码？请阅读以下代码。
```
stopHere(1000);
var response=ajaxLibrary.get("/page");
```
为什么上面这种写法的代码是行不通的？在上面这段代码中，我们假设有一个叫 stop 的函数，让JavaScript代码暂停运行 1000 毫秒。然后使用 ajaxLibrary的 get 方法，向某个 URL 发送 HTTP 请求，然后等待获取返回结果。

为什么这样不行？为什么这必须是一个需要回调函数的异步操作？或者，后来我们为什么要使用 async/await 或 Promise？

答案是：JavaScript 是一门单线程语言，在同一时间只能执行一段代码。

这意味着什么？如果JavaScript暂停 1000 毫秒，或者在非常慢的网络环境下向 /page 发起 HTTP 请求，需要 4 秒、5 秒才能返回结果，那么整个页面就会被卡住，无法做其他操作。

当然，这只是一个假设，因为 JavaScript 实际上并不是这样工作的，不会在某一行代码上等待返回结果。因为如果JavaScript真的会暂停在某一行代码，那 JavaScript 就无法同时执行其他操作，比如响应点击事件、拖拽事件、窗口大小变化事件，也无法重新绘制 DOM、更新页面。整个页面就会完全被卡住，这肯定会给用户带来极其糟糕的体验。

由于JavaScript 是单线程语言的，在任意时刻都只能执行一段代码。当然，这是一种简化的解释，但从根本上来说，这就是上面代码问题的核心。我们必须以特殊方式处理异步代码，因为大多数时候还需要处理其它操作。或者至少，我们希望JavaScript在任何时刻都能处理其他操作。

理想情况下，开发者当然希望能够像同步代码那样编写JavaScript 代码：向某个 URL 发送请求，等请求完成之后，再使用 console.log 打印响应结果。代码如下所示：
```
var response=ajaxLibrary,get(“/page”);
console.log(reponse);
```
上面这段代码当然这是不可能执行的的。因为会阻塞 JavaScript，导致 JavaScript 无法处理后台的其他操作，比如监听事件、更新 DOM、重绘浏览器窗口等等，以及 JavaScript 一直在持续执行的所有其他操作。

因此，我们过去的做法是：传入一个回调函数，在异步操作完成时再执行该函数。

在早期的Ajax库中，当我刚开始学编程的时候，我们会这样编写异步代码：先等请求返回响应结果，再使用 console.log 打印响应结果。也就是说，我们提供一个函数，这个库会在合适的时机，也就是获取响应结果后调用该函数打印响应结果。代码如下所示。
```
var response = ajaxLibreary.get("/page", function (response) {
    console.log(response);
    something();
});
```
获取响应结果的时间可能需要 5 秒，也可能只要 500 毫秒，但一旦请求完成，ajaxLibreary函数中回调函数就会被执行，我们就可以在这个回调函数中编写所有“请求完成之后”要执行的代码，这段中就是something指代的操作。

这就是过去异步代码的工作方式，而且直到今天，很多库仍然支持这种回调函数风格。这样做的好处是：JavaScript 在获取响应结果的同时，可以继续执行其他任务，而不被阻塞住。

但是这种方式也存在一个非常严重的问题：大量使用回调函数会让代码变得极其混乱，难以阅读和管理。这正是开发者常说的回调地狱。

2.2 回调地狱详解

假设我们需要先请求 /page1，在/page1请求完成之后再请求/page2，代码可能会变成下面这样：
```
var response = ajaxLibreary.get("/page", function (response) {
    console.log(response)
    ajaxLibreary.get("/page2", function (respone2) {
        console.log(response2)
    })
});
```
在这段代码中，我们在第一个请求的回调函数中发起第二个请求，然后中调用一个新的回调函数。

如果我们需要等第一个请求完成之后才能发起第二个请求，我们就需要将回调函数一层一层地嵌套起来。如果有大量请求，比如还需要请求/page3、/page4，我们最终会得到下面的请求代码。
```
var response = ajaxLibreary.get  ("/page", function (response) {
    console.log(response)
    ajaxLibreary.get("/page2", function (respone2) {
        console.log(response2)
        ajaxLibreary.get("/page3", function (respone3) {
            console.log(respons3)
            ajaxLibreary.get("/page4", function (respone4) {
                console.log(response4)
                ......
            })
        })
    })
});
```
在这段请求代码中，我们还没有处理错误。如果加上处理错误的代码，将会使用请求回调函数变得更加混乱。这就是我们前面前一节提及的“回调地狱”。有时也被称之为末日金字塔，因为代码会形成一种三角形结构，回调函数会不断地一层层的嵌套下去。

当然，这种问题只在我们需要按顺序执行操作时才会出现。也就是说，我们需要先发起第一个请求，等第一个请求完成之后，再发起第二个请求，等第二个请求完成之后，再发起第三个请求。以此类推，不断发起下一个请求。

下面就是一个回调地狱的示例。这是一个来自 Node.js 的例子，这段代码使用了 Node.js 的文件系统（fs）模块读取一个文件file1.txt。
```
fs.readFile("file1.txt", "utf8", function (err, data) {
    if (err) {
        console.log(err);
    } else {
        fs.readFile("file12.txt", "utf8", function (err, data) {
            if (err) {
                console.log(err);
            } else {
                fs.readFile("file3.txt", "utf8", function (err, data) {
                    if (err) {
                        console.log(err);
                    } else {
                        console.log(data);
                    }
                })
            }
        })
    }
})
```
在这段代码中，readFiles()方法接收一个回调函数。也就是说，这一整段代码，本质上就是将一个函数作为参数传递给readFiles()方法。这段代码简化以下形式，不断将下面的函数作为参数重复传入。
```
fs.readFile("file.txt", "utf8", function () {
})
```
在这段代码中，我们给fs.readFile()方法传递一个匿名函数参数，该函数会在文件读取完成之后执行，也就是读取下一个文件file2.txt、file3.txt。读取文件是一个异步操作，是需要时间的。所以，如果我们先读取文件file1.txt，然后在读取成功之后再读取文件file2.txt"，那么我们就需要在fs.readFile()方法的一回调函数重复调用fs.readFile()方法。如果我们还需要顺序地读取第三个文件，甚至更多文件，那我们就只能继续嵌套。这就产生了上面的大量的嵌套代码。这简直是一场噩梦，非常非常丑。

很多以前 JavaScript 代码就是这样编写的。当时几乎没有办法避免，或者说没有什么简单的替代方案。我们只能一层一层地编写调用回调函数，回调函数套回调函数，代码嵌套级别变得越来越深。

我们再来看下面这个例子。下面这段代码是以前使用Ajax发送请求的写法。
```
function makeRequest(url1, callback1) {
  const xhr1 = new XMLHttpRequest();
  xhr1.open("GET", url1, true);
  xhr1.onload = function () {
    if (xhr1.status === 200) {
      const response1 = JSON.parse(xhr1.responseText);
      const url2 = response1.nextUrl;

      const xhr2 = new XMLHttpRequest();
      xhr2.open("GET", url2, true);
      xhr2.onload = function () {
        if (xhr2.status === 200) {
          const response2 = JSON.parse(xhr2.responseText);
          const url3 = response2.nextUrl;

          const xhr3 = new XMLHttpRequest();
          xhr3.open("GET", url3, true);
          xhr3.onload = function () {
            if (xhr3.status === 200) {
              const response3 = JSON.parse(xhr3.responseText);
              callback1(response3.data);
            } else {
              console.error("Request 3 failed");
            }
          };
          xhr3.send();
        } else {
          console.error("Request 2 failed");
        }
      };
      xhr2.send();
    } else {
      console.error("Request 1 failed");
    }
  };
  xhr1.send();
}

makeRequest("https://api.example.com/endpoint1", function (result) {
  console.log(result);
});
```
在 fetch API 出现之前，我们必须使用 XMLHttpRequest（XHR）获取，而传统的 XHR 依赖于回调函数。在这段代码中，我们创建了一个新的请求对象xhr1，然后调用 open 方法，接着再调用一个 onload 回调函数。于是这接下来的整块代码都是 onload 回调函数重复嵌套定义onload回调函数。当第一个请求完成时，onload 回调函数就会执行第二个请求xhr2，以此类推。如果后续增加更多请求，嵌套层级会持续加深，代码会呈现向右延伸的金字塔形状，代码可读性会越来越差。

再回调函数中处理错误又是另一场噩梦。每一个异步操作都需要单独编写一套错误处理逻辑，错误处理代码会和业务逻辑交织在一起，且分散在各个嵌套层级中，无法实现统一捕获和处理。回调函数模式没有内置的「错误冒泡」机制，如果内层回调函数发生错误，想要将错误传递到外层（或全局）进行处理，需要手动编写大量的错误传递代码，过程繁琐且容易出错。在回调函数中下，错误类型多样，且没有强制的错误处理约束，很容易遗漏一些边缘场景的错误，导致程序出现不可预期的行为。
当然我们可以使用命名函数来让回调函数变得扁平一些，代码如下所示。
```
ajaxLib.get("/step-1", doStep2);

function doStep2(resp) {
    ajaxLib.get("/step-2", { resp.body }, doStep3);
}

function doStep3(resp) {
    ajaxLib.get("/sttep-3", { resp.body }, giveAnswer);
}

function giveAnswer(resp) {
    console.log("got final answer", resp.body);
}
```

在这段代码示例中，我们使用 Ajax 库先发送第一个请求，传入一个叫 doStepTwo 的函数作为回调函数。doStep2 也是一个具名函数，而它里面又接收一个回调函数 doStep3。这样确实可以解决回调函数层次潜逃问题，但这会让错误处理变得非常麻烦。因为每一个函数都必须要知道“接下来该做什么”，使得代码之间高度耦合，纠缠在一起，而且错误处理依然非常痛苦，仍然很难让我们写出真正独立的函数。

值得庆祝的的是，在今天，我们已经不需要再依赖大量回调函数来管理异步代码了，因为我们有了 Promise 这把异步代码处理利器。下一节，我们将正式开始学习 Promise。

2.3 小结

JavaScript 是一门单线程语言，在同一时间只能执行一段代码。因此，它不能在某一行代码上“停住”，等待一个耗时操作（比如网络请求或文件读取）完成，否则整个程序都会被阻塞，无法响应用户事件、更新 DOM 或执行其他逻辑。
正因为如此，JavaScript 必须以特殊的方式处理异步操作。当异步任务正在进行时，JavaScript 会继续执行其他操作。在任务最终完成后，再通过回调函数或者我们后面学习 的Promise 或 async / await 来指定后续要执行的代码。这个“完成”的时间是不可预测的，可能是几毫秒，也可能是几秒钟甚至更久。但无论何时完成，只要完成了，就会触发对应的处理逻辑。
这正是异步编程的核心问题。过去，大量依赖回调函数往往会导致代码结构混乱，形成所谓的“回调地狱”。虽然本文中的示例还比较基础，但在真实项目中，这曾经是一个非常普遍且严重的问题。好在现代 JavaScript 已经提供了更好的工具，可以有效缓解甚至彻底解决“回调地狱”问题。
3.Promise 的基础
我们已经在上一节了解了什么是“回调地狱”，以及为什么在处理异步代码时要尽量避免使用回调函数。从本节开始，我们将学习现在JavaScript 中“回调地狱”的真正解决方案—— Promise。
Promise 是一种用于处理异步操作的机制，相比回调函数，它能让代码变得更易管理、更有结构，也不用担心“回调地狱”问题。Promise 提供了一种替代回调函数来编写异步代码的方式。它是在 ES6 中引入的，也就是我们现在所说的 ES2015。其核心思想是：Promise 是对“未来某个值”的一种承诺。Promise 用来表示一个异步操作最终会成功完成，或者可能会失败。通过Promise ，我们能够写出更干净、更优雅的异步代码。我们可以将多个异步操作组合、串联在一起，让错误处理更简单，也让我们更容易管理复杂的异步流程。Promise 是现代 JavaScript 开发中非常基础、非常重要的一部分。无论你是编写在浏览器中运行的代码，还是在 Node.js，或者在其他 JavaScript 运行环境中工作，Promise 都是一个必须理解的概念。
3.1 Fetch()方法
我们有一个fetch()方法，本节我们就将大量使用这个方法。fetch()方法已经是现代浏览器原生支持的功能。 它本质上就是一个叫做 fetch 的函数，我们可以用它来调用 API ，也就是发起 Ajax 请求。所以fetch方法的最核心、最简单的用法就是传入一个要请求的URL。如下所示：
fetch(https://www.google.com)
上面这行代码会向google.com 发起一个 GET 请求，当然，出于演示 fetch() 方法的具体用法，我没有给出具体请求的网页。如果你以前从来没见过 fetch() 方法，也没关系。 Fetch 其实还能做很多事情，比如可以设置请求头（headers）、发起不同类型的 HTTP 请求、携带请求体（body），也可以发送 JSON 数据或表单数据等。不过暂时无需深入关注这些内容，在上面的示例中仅使用了最简单的 GET 请求。
在本节内容将采用 Pokemon API（宝可梦 API）进行实战演示，该 API 的地址为 pokeapi.co/api/v2/pokemon。访问这个API，页面显示内容如下所示。 
 如果在这个 URL的后面加上 /1， 就会得到一系列关于 妙蛙种子（Bulbasaur） 的信息。
 
 这是是第一只宝可梦。如果你访问 /2、/3， 就会得到相应的宝可梦的信息。如果你不了解宝可梦，其实也完全没关系。这只是为告诉你，你可以通过这个 API 获取某一只宝可梦的相关信息的JSON数据。 
以上就是你目前需要掌握 fetch API 的全部内容。
即便你此前从未接触过 fetch() 也无妨，现在首先要明确的是：当我们调用 fetch() 方法时,如果直接传入API的 URL，不会获取到任何数据。
const BASE_URL= "https://pokeapi.co/api/v2/pokemon";
const url=`{BASE_URL}/1`;
fetch(url);
运行上面这段代码，页面不会显示任何内容。如果在浏览器控制台中执行这段代码，会返回结果是一个名为 Promis的对象，如下所示。
 
Promise 可理解为「对未来结果的承诺」，它代表着一个尚未揭晓的未来结果。这个「承诺」并非凭空存在，而是对应的一个异步操作的最终走向，在异步操作完成之前，Promise 无法给出确切的结果，只能以「占位符」的形式存在，等待异步操作的最终反馈。在这个宝可梦示例中，这个结果只有两种非此即彼的可能：要么是从 API 成功获取的结构化数据（如 JSON 格式的宝可梦信息），要么是请求过程中出现的各类错误（如网络中断、接口 404 未找到、服务器 500 内部错误等），不存在第三种中间状态。
获取数据是一个异步操作，而 fetch 返回的 Promise 对象正是用于标识该异步操作的最终成败，并且它通过自身的状态流转来清晰呈现异步操作的全过程。
此外，Promise 是 JavaScript 的原生特性，自 2015 年 ES6（ECMAScript 2015）规范发布起，便已被纳入语言标准，无需引入任何第三方库即可直接使用，彻底解决了传统回调函数嵌套带来的「回调地狱」问题。
3.2 Promise的三种状态
你需要知道关于 Promise 的第二件事是：Promise 有三种截然不同且不可逆转的状态，任何一个 Promise 对象在其生命周期内，一定会处于这三种状态中的某一种，且状态一旦发生流转，就无法再回退或更改，这是 Promise 保证异步结果可靠性的核心特性之一。
第一种状态是 pending（进行中 / 等待中），这是 Promise 被创建时的初始状态。当我们调用 fetch() 发起 API 请求的瞬间，返回的 Promise 就处于 pending 状态，它对应着异步操作正在执行但尚未完成的阶段。也就是正在传输请求数据、服务器正在处理用户的查询并准备响应结果的过程。pending 的核心含义就是：当前的 Promise 还没有拿到一个最终的、确定的值，它只是一个等待结果的「占位符」，此时我们无法从中提取有效数据，也无法获取错误信息，只能等待异步操作给出最终反馈。
第二种状态是 resolved（已成功，也常称为 fulfilled / 已兑现），需要说明的是，在 Promise 的官方规范中，fulfilled 是更精准的表述，resolved 更多是日常开发中的通俗叫法，二者都指向「异步操作顺利完成」这一结果。这意味着对应的异步操作已经毫无异常地执行完毕，Promise 已经成功地获取到了一个合法的、可用的最终值 —— 在前面示例中，就是 API 成功返回的 JSON 格式数据（如宝可梦的名称、属性、技能等结构化信息），这个值会被妥善存储在 Promise 中，等待我们通过 .then() 等方法提取并进行后续处理。一旦 Promise 的状态从 pending 流转为 fulfilled（resolved），就会固定在这个状态，永远不会再转回 pending，也不会再转为 rejected。
第三种状态是 rejected（已拒绝 / 已失败），也就是说，对应的异步操作在执行过程中因为某些意外原因发生了异常，最终失败了，没有能够获取到我们期望的有效值。这些失败原因在实际开发中十分常见，可能包括：网络中断导致请求无法送达服务器、API 地址错误出现 404 未找到状态码、服务器内部出错返回 500 状态码、没有访问权限返回 403 状态码等。当异步操作失败时，Promise 会捕获到对应的错误信息（如错误类型、错误描述、状态码等），并将自身状态从 pending 不可逆地流转为 rejected。和 fulfilled 状态一样，rejected 状态也是固定的，一旦进入该状态，就无法再转回其他两种状态，我们可以通过 .catch() 方法捕获这个状态对应的错误信息，进行错误提示、重试等后续处理。
Promise 并不是 fetch API 所特有的特性，它本质上是 Promise 异步编程规范的通用表现。fetch 仅仅是一个最直观、最容易上手的演示 Promise 核心概念的例子而已 —— 因为它是浏览器内置的常用异步 API，无需额外配置，调用后直接返回 Promise 对象，能快速帮我们直观感受 Promise 的状态、返回值和异步流转逻辑，非常适合入门阶段的演示和学习。
3.3 小结
获取取 API 数据是前端开发中一个非常常见且典型的异步操作，我们此前也一直以 fetch 调用 API 为例来讲解 Promise，但实际上，Promise 的应用范围远不止于此 —— 市面上众多的 JavaScript 库、工具以及内置 API，都已经全面支持 Promise 规范，比如处理文件读写的 fs.promises（Node.js 环境）、处理定时器的 util.promisify(setTimeout)、第三方请求库 Axios，甚至是处理异步数据流的相关工具，均以 Promise 作为异步处理的核心载体。
所以，在这里需要特别强调，请不要误以为 Promise 只和 fetch 相关，更不要将二者划等号。本文之所以选择 fetch 作为演示案例，仅仅是因为它足够简洁直观、无需额外引入第三方依赖，且是浏览器原生支持的异步 API，调用后直接返回标准的 Promise 对象，能够帮助大家快速聚焦于 Promise 的核心概念，而不会被复杂的工具配置所干扰，从而更轻松地理解 Promise 到底是什么以及 Promise 的工作模式。
到目前为止，我们已经了解fetch 会返回一个Promise对象，这个对象的初始状态时pending，最终状态要么是 fulfilled，要么是 rejected。即使是在请求失败的情况下，Promise对象的初始状态也仍然是 pending。那么在 Promise 被成功解决或被拒绝之后，该如何处理返回的结果呢？这正是我们接下来要学习的内容。
4. Promise的then与catch方法：捕获异步结果
在上一节，我们们已明确知道：Promise 是一种具备状态管理能力的 JavaScript 对象，其生命周期的最终状态仅有两种可能——要么被解析（resolved/fulfilled，已兑现），要么被拒绝（rejected，已失败）。这两种最终状态会分别携带对应的数据：成功时的有效结果与失败时的错误信息。
要访问Promise承载的数据，我们无需复杂操作，只需在 Promise 对象上链式调用 then 与 catch 两个方法即可。这两个方法均接收回调函数作为参数，分别对应 Promise 成功与失败状态的处理逻辑。
4.1 then方法：处理Promise成功结果
then 方法是 Promise 成功状态的专属处理器，其核心特性的是：无论传入何种回调函数，该函数仅会在 Promise 状态转为 resolved（已兑现）时触发执行，且其回调函数参数可直接访问 Promise 成功时返回的结果数据。我们来看看下面这个例子。
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";
const url = `${BASE_URL}/1`;
fetch(url).then(function (data) {
    console.log(data);
});
在上面示例中，我们使用 fetch 方法向 Pokemon API发起请求，在 fetch 返回的 Promise 后直接链式调用 then() 方法，并在 then 方法中定义一个回调函数。该回调函数的参数 data 会自动接收 Promise 解析后的有效数据 —— 无论数据格式是 JSON、文本还是其他类型，都可以通过这个参数获取。在这个示例中，我们直接使用 console.log(data) 打印获取到的数据，打开浏览器控制台便可以看到来自 API 的完整响应。需要注意的是，首次发起请求时因网络传输、服务器处理存在耗时，数据打印会有短暂延迟。后续请求因浏览器缓存机制，响应速度会显著提升，数据能快速渲染至控制台。获取结果如下。
 
如截图所示，我们确实获取到相应数据了。关于响应数据的具体结构（如状态码（status） 200、响应体（body）、请求头（headers）、标识请求成功的 ok 字段、重定向信息（redirected）及请求 url 等），暂无需深入探究，核心只需确认：通过 then 方法，我们已成功捕获到 Promise 解析后的有效数据。
4.2 catch方法：处理Promise的失败结果
与 then 方法对应，catch 方法专门用于处理 Promise 被拒绝（rejected）的场景。当 Promise 状态从 pending 转为 rejected 时，比如发生网络异常、接口不可用等问题导致 Promise 被拒绝时，catch 方法中传入的回调函数都会被触发，并且该函数可访问到 Promise 携带的错误信息 —— 这些错误信息的具体内容，取决于生成 Promise 的来源（如 fetch 请求、第三方库或自定义 Promise），具体示例如下。
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";
const url = `${BASE_URL}/1`;
fetch(url).then(function (data) {
    console.log(data);
}).catch(function(error){
    console.log(“ERROR:”,error);
});
我们在上一节示例的基础上调用catch方法。然后给catch方法传入一个回调函数捕获错误，该函数接收 error 作为参数，再通过 console.log(error) 打印错误详情。
需要强调的是，对于同一个 Promise，then 与 catch 的回调函数是互斥执行的。Promise 初始状态为 pending，后续如果转为 resolved，则仅执行 then 中的逻辑；如果若转为 rejected，则仅执行 catch 中的逻辑，不会出现两者同时触发的情况。
我们可通过断开网络模拟失败场景：在 fetch 请求无法正常完成时，Promise 状态转为 rejected，控制台会打印对应的错误信息（如 TypeError: Failed to fetch）。结果如下所示。
 
尽管截图中的错误信息较为简洁，但确实已明确标识出 Promise 被拒绝的原因，这正是我们所需要的失败结果。
4.3 补充说明与代码优化
关于 then 与 catch 中的回调函数的参数，存在两个实用细节：一是参数名称可自由定义，与普通函数参数一致，无需遵循固定命名规范；二是实际开发中，通常会使用箭头函数替代传统函数写法，进一步精简代码结构，提升代码的可读性。改写后的示例代码如下所示。
fetch(url)
    .then(response => console.log("API 响应数据：", response))
    .catch(err => console.log("请求错误：", err));
4.4 小结
fetch 发起异步请求后，会返回一个初始状态为 pending 的 Promise 对象；该对象后续会不可逆地转为 resolved 或 rejected 状态：状态为 resolved 时，触发 then 回调函数，可在其中处理 API 响应数据；状态为 rejected 时，触发 catch 回调函数，可在其中捕获并处理错误信息。
需要明确的是，then 与 catch 本质上仍依赖回调函数的逻辑，并未完全消除回调概念。但后续我们将学习如何通过 Promise 链式组合，彻底规避“回调地狱”或“金字塔式嵌套”的问题，实现更优雅的异步编程。
5.Promise链式调用：扁平化代码
在实际开发中，经常会遇到需严格按顺序执行多个异步请求的场景。本文以宝可梦 API 为实践载体，依次发起针对 ID 为 1、2、3、4 的宝可梦数据请求，且必须确保前一个请求成功完成后，再触发下一个请求，以此保证异步操作的执行顺序严谨性。
5.1 传统实现：嵌套回调函数引发的“回调地狱”
要满足多请求顺序执行的需求，最直观的传统思路的是利用回调函数嵌套：在第一个请求的 then 回调函数中发起第二个请求，在第二个请求的 then 回调中发起第三个请求，以此类推，同时为每个请求单独添加 catch 方法，用于捕获该请求可能出现的错误，避免整体流程中断。如下所示。
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";
const url = `${BASE_URL}/1`;
fetch(`${BASE_URL}/1`).then((res1) => {
    console.log("RESPONSE", res1);
    fetch(`${BASE_URL}/2`).then((res2) => {
        console.log("RESPONSE2", res2);
        fetch(`${BASE_URL}/3`).then((res3) => {
            console.log("RESPONSE3", res3);
            fetch(`${BASE_URL}/4`).then((res4) => {
                console.log("RESPONSE4", res4);
            })
        }).catch((error3) => {
            console.log("ERROR3", error3);
        })
    }).catch((error2) => {
        console.log("ERROR2", error2);
    })
}).catch((error1) => {
    console.log("ERROR1", error1);
})
在这段代码中，先发起针对 pokemon/1 的第一个请求，通过箭头函数接收响应数据并命名为 res1，打印对应的结果标识及数据；同时添加 catch 方法，捕获该请求的错误并打印出来。在第一个请求的 then 回调执行完毕后，直接在其内部发起针对 pokemon/2 的请第二个请求，重复上述操作，依次嵌套实现四个请求的顺序执行。只要其中任意一步发送错误，我们就使用catch捕获错误并打印出来。
这种实现方式虽能满足核心需求——网络限速场景下可清晰观察到四个请求按顺序发起与响应，断开网络时也能通过 catch 正常捕获错误，但存在致命缺陷：随着请求数量增加，回调嵌套层级会不断加深，形成典型的“回调地狱”，也被称为“死亡金字塔”。
回调地狱会导致代码可读性急剧下降，后续修改某一步请求逻辑时，需逐层定位嵌套代码，不仅效率低下，还极易引发新的逻辑漏洞。这是异步编程中需要极力规避的问题。
5.2 优化方案：Promise 链式调用实现代码扁平化
得益于 Promise 的核心工作机制，我们可通过“Promise 链式调用”替代传统嵌套回调，将层级叠加的嵌套代码转化为扁平结构，彻底解决回调地狱问题。其核心思路是：在 then 回调函数内部返回一个新的 Promise 对象，也就是下一个请求，然后再then方法外部以一种扁平化的方式继续链式调用 then 方法，实现多异步请求的顺序串联，同时保持代码结构简洁。示例代码如下。
fetch(`${BASE_URL}/1`)
    .then((res1) => {
        console.log("RESPONSE1", res1);
        return fetch(`${BASE_URL}/2`)
    })
    .then((res2) => {
        console.log("RESPONSE2", res2);
        return fetch(`${BASE_URL}/3`);
    })
    .then((res3) => {
        console.log("RESPONSE3", res3);
        return fetch(`${BASE_URL}/4`)
    })
    .then((res4)=>{
        console.log("RESPONSE4", res4);
    })
在这段代码中，我们首先使用 fetch 起第一个请求，在其then 回调中打印当前响应结果并直接返下一个请求——由于这个请求本身会返回一个 Promise 对象，因此可在外部继续衔接调用 then 方法。在新的 then 回调中函数，重复“打印当前响应结果和返回下一个请求”的操作，直至完四个请求的串联。运行结果下图所示。
 
运行这段代码你可以发现，链式调用方案与嵌套回调函数方案的执行行为完全一致：在第一个请求发起后， Promise 处于 pending 状态，直至成功解析并执行对应的 then 回调函数，打印响应结果，然后返回第二个请求对应的 Promise；此时第二个请求才正式发起，重复上述流程，直至四个请求完成。
四个请求严格按顺序发起与响应，网络限速场景下能清晰观察到执行顺序；但代码结构完全扁平化，无任何嵌套层级，可读性和维护性得到极大提升。需要注意，这个示例暂不涉及 catch 方法的优化，我们会在后续章节将专门讲解统一错误处理逻辑。
5.3 小结
从功能层面看，两种方案均能实现多异步请求的顺序执行，且错误捕获（需要单独添加 catch 时）均有效。但从代码质量层面，二者差异显著：嵌套回调形成“死亡金字塔”，层级混乱、维护成本高；链式调用通过扁平化结构，彻底打破嵌套束缚，逻辑清晰，修改某一步请求时无需逐层定位，直接操作对应的 then 回调函数即可。
需要明确的是，Promise 链式调用并未完全消除回调函数，但其通过规范的调用方式，将回调函数从“嵌套叠加”转化为“扁平串联”，从根源上规避了回调地狱的问题，体现了“合理使用工具规范逻辑”的核心思路——开发者仍可手动写出嵌套回调，但遵循 Promise 规范可实现更加优雅的异步编程。
Promise 链式调用的核心价值在于“平衡顺序性与可读性”：既保证了多异步请求的严格顺序执行，满足业务场景需求，又通过扁平化结构优化了代码形态，降低了阅读与维护成本。这一切的实现基础，源于 then 回调函数的返回值特性——返回 Promise 即可接管后续状态流转，为链式衔接提供了底层支撑。
6.Promise中的错误处理机制
在异步编程的实现中，Promise 凭借其链式调用的特性，为开发者解决了回调函数嵌套带来的诸多问题，但错误处理的方式选择，直接决定了代码的简洁度与可维护性。在 Promise 的使用过程中，如果未添加任何 .catch 方法，当 Promise 被拒绝（reject）时，错误将无法被捕获，进而导致程序出现隐藏风险。
为此，很多开发者会陷入一个误区：是否需要为每一个 Promise 单独添加 .catch 方法来处理错误？即在执行每一个 Promise 后都调用一次 catch 方法捕获可能出现的异常。这种写法虽然能够实现错误捕获，却会让代码变得臃肿冗余，形成所谓的 “噩梦级别” 写法，不仅增加了代码量，也降低了代码的可读性。
6.1 Promise链的错误传递：一个catch的“魔法”
Promise 存在一个极为实用的特性——错误传递机制，借助这一特性，我们完全不需要为每个 Promise 单独设置错误捕获。只需要在 Promise 链的末尾添加一个 .catch 方法，即可捕获链中任意一个 Promise 被拒绝时抛出的错误。
在 Promise 链式调用的流程中，无论错误发生在哪个环节，只要某一个 Promise 被拒绝，这个错误就会沿着 Promise 链向上传递，最终被末尾的 catch 方法捕获并处理，整个 Promise 链的后续执行流程也会随之立即终止。例如在一个包含多次异步请求的 Promise 链中，不管是第一个请求、中间某个请求，还是最后一个请求出现错误导致 Promise 被拒绝，末尾的 .catch 处理器都会触发执行预设的错误处理逻辑。我们给上一节示例调用catch方法，并在第三个请求中请求一个不存在的网址http://nope.nppe，代码如下。
 fetch(`${BASE_URL}/1`)
    .then((res1) => {
        console.log("RESPONSE1", res1);
        return fetch(`${BASE_URL}/2`)
    })
    .then((res2) => {
        console.log("RESPONSE2", res2);
        return fetch(`http://nope.nppe`);
    })
    .then((res3) => {
        console.log("RESPONSE3", res3);
        return fetch(`${BASE_URL}/4`)
    })
    .then((res4)=>{
        console.log("RESPONSE4", res4);
    })
    .catch((error)=>{
    console.log("IN THE CATCH! Error:",error);
    })  
在上面的代码中，第三个请求是一个不存在的网址，因此该Promise 被拒绝，后面的第四个 Promise 将终止执行，Promise 链最后面的catch方法中的回调函数就会执行并捕获错误。这段代码的执行结果如下。
 
这种捕获错误的处理方式的优势十分明显，我们可以将所有的错误处理逻辑集中在一起处理，比如通过 console.log 打印错误信息。无论错误出现在 Promise 链的哪个位置，都能通过这一个 .catch 方法完成统一处理，极大简化了代码结构。
6.2 小结
Promise 的价值远不止于简化处理错误，更在于它能有效解决回调函数嵌套带来的 “回调地狱” 问题。在传统的回调函数写法中，多层异步操作会导致代码层层缩进，形成难以阅读和维护的 “死亡金字塔” 结构。
而 Promise 链式调用的特性，能让异步代码实现扁平化书写。即便需要按顺序执行多次异步请求，代码也无需进行多层嵌套。每一个 then 方法只需要返回一个值或一个新的 Promise传递给下一个 then 方法，前一步的操作无需依赖后一步的具体实现逻辑，实现了各步骤之间的解耦。
在这种扁平化的代码结构中，错误处理的优势被进一步放大。所有异步操作的错误都能被末尾的 .catch 方法捕获，开发者无需在每一层嵌套中都设置错误处理逻辑，代码的整洁度和可维护性得到了大幅提升。当然，还有一种更简洁的语法来处理Promise异步代码，那就是async/await。这也是我们接下来要学习的内容。
7.sync/await基础
在 JavaScript 异步编程的演进过程中，async/await 是一项极具价值的JavaScript原生语言改进，它以更简洁、更直观的方式简化了 Promise 的使用流程，让异步代码的书写和阅读更贴近开发者熟悉的同步代码逻辑。作为专门为优化 Promise 而设计的关键字，async 和 await 相辅相成，共同构成了现代 JavaScript 异步编程的主流方案，彻底降低了异步代码的理解成本和编写难度。
7.1 async关键字
要使用 async/await 处理异步操作，首先需要使用async关键字声明异步函数。这是使用 await 关键字的前提，也是构建异步代码的基础。
下面这个函数是一个普通的函数，思考一下如何将该函数变为异步函数。
function Hello() {
    return "hi";
}
对于一个普通的函数而言，比如上面这个函数，其返回值就是其内部return语句指定的内容，可能是字符串、数字、对象等任意常规类型。但如果在函数声明前添加 async 关键字，这个函数就会被标记为异步函数，其返回值会发生根本性的变化——无论函数内部返回的是何种常规值，该异步函数最终都会返回一个 Promise 对象，这是 async 关键字的核心特性。所以我们只需要将上面示例代码中的 Hello() 函数前面添加一个 async 关键字，这个函数就变为了异步函数。
async function Hello() {
    return "hi";
}
在浏览器控制台中运行这个函数，结果如下： 
Hello()函数返回了一个Promise，所以现在Hello()函数变成一个异步函数了。async 关键字的价值，不仅在于将函数转换为异步函数并返回 Promise，更在于它为await关键字提供了合法的运行环境，同时让函数内部可以承载类同步的异步逻辑。我们不再需要频繁地使用 then 和 catch 方法。在 async 异步函数中，我们依然可以使用 Promise 的then、catch等方法，这两个方法完全兼容 Promise 的所有特性，只是这两个方法不再成为处理异步操作的必需选择。
7.2 await关键字
如果说async关键字是开启异步函数大门的钥匙，那么await关键字就是让异步代码呈现同步逻辑的核心。await关键字仅能在async标记的异步函数内部使用，如果在非 async 函数中使用，会直接出现语法错误，这是其使用范围的重要限制。
await 关键字的核心作用是让 async 函数的执行流程“暂停”，等待 await 后面跟随的 Promise 对象完成解析。这里的“暂停”是 async 函数执行流程呈现出的一种表面上的同步表现。在 JavaScript 引擎的底层实现中，这一过程仍然是异步操作，await 并不会真正阻塞 JavaScript 主线程的执行，JavaScript 引擎依然可以在后台并行处理其他任务，不会影响程序的整体运行效率。下面是一个简单的示例说明。
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";
async function getFirstPokemon() {
    const result =await fetch(`${BASE_URL}/1`);
    console.log(result);
}
在上面这段代码中，我们使用 async 定义了一个函数 getFirstPokemon()。在这个函数中，我们使用fetch向宝可梦API，即BASE_URL获取编号为1的宝可梦（${BASE_URL}/1），并且在fetch方法前面添加了关键字 await。然后请求结果保存在变量result变量中，最后最后通使用console.log() 打印result。
我们前面已经了解，fetch 函数的返回值是一个 Promise 对象，而当我们在被 async 关键字声明的异步函数内部时，就可以直接使用 await 关键字来等待这个 Promise 对象解析兑现。
这行带有 await 的 fetch 代码，本质上会暂停当前 async 函数的执行流程（并不会阻塞整个 JavaScript 线程），专门等待对应的 Promise 完成解析并变为成功状态。当被等待的 Promise 对象成功解析（resolve）后，async 函数的执行流程会恢复继续向下运行，而 Promise 解析后返回的值，会直接被赋值给包含 await 关键字的当前变量。这意味着，我们可以直接通过变量获取 Promise 的解析结果，无需像传统 Promise 那样通过 then 方法传入回调函数来提取返回值。
与之形成对比的是，如果在普通函数中直接处理返回 Promise 的异步操作，我们无法获取到 Promise 的最终解析结果，只能得到一个处于 pending 状态的 Promise 对象，无法直接使用异步操作的返回数据，要么依赖 then 方法处理，要么就需要借助 async/await 进行优化。
7.3 async/await的意义
async/await 的组合使用，最大的价值在于彻底简化了异步代码的结构。在传统 Promise 链式调用中，我们需要依赖then方法串联多个异步操作（件6.1节示例），代码始终带有链式调用的语法痕迹；而借助 async/await，我们可以写出完全平铺的代码，无需嵌套，也无需频繁使用then和catch方法，代码的执行顺序从上到下一目了然，完全符合人类的阅读和思考习惯。
从功能等价性来看，async/await 的写法与传统 Promise 的then链式写法是等价的，二者都能实现相同的异步操作逻辑，获取相同的返回结果。但 async/await 剥离了 Promise 的链式语法外衣，让开发者无需关注异步操作的 “串联逻辑”，只需专注于 “要完成的任务本身”，大幅降低了心智负担。
尤其对于复杂的异步流程而言，async/await 的优势更为明显。它让异步代码看起来和同步代码毫无二致，既保留了 Promise 的异步非阻塞特性，又拥有了同步代码的简洁直观，解决了传统 Promise 链式调用在复杂逻辑下可能出现的可读性下降问题。这种现代、简洁的异步编程方式，也让 JavaScript 的异步开发效率得到了显著提升，成为当前处理异步操作的首选方案。
7.4 小结
async/await 作为 JavaScript 原生支持的异步编程语法，是建立在 Promise 基础之上的优化方案，并非对 Promise 的替代。async关键字负责声明异步函数并确保其返回 Promise，await关键字负责在函数内等待 Promise 解析并提取结果，二者配合使用，让异步代码摆脱了复杂的链式调用，以更贴近同步逻辑的形态呈现。这种写法既兼顾了代码的可读性和可维护性，又保留了异步操作的高效性，推动了 JavaScript 异步编程的进一步发展和完善。
8 Async/Await进阶内容
8.1 在async函数中串行执行多个请求
在 async 函数内部，我们可以对任意多个异步请求使用 await 关键字，从而轻松实现按顺序串行执行发起多个请求。对应此前基于 Promise 实现的6.1一节中的示例，我们可以将7.2一节中 getFirstPokemon 函数重命名为 getFourPokemon，使用该函数按顺序获取编号为 1、2、3、4 的四个宝可梦数据。代码如下所示。
async function getFourPokemon() {
    const res1 =await fetch(`${BASE_URL}/1`);
    console.log(res1);

    const res2 =await fetch(`${BASE_URL}/2`);
    console.log(res2);

    const res3 =await fetch(`${BASE_URL}/3`);
    console.log(res3);

    const res4 =await fetch(`${BASE_URL}/4`);
    console.log(res4);
}
这段代码中的四个请求的具体实现思路十分直观。首先我们声明保存所有请求结果的变量，比如 res1、res2、res3、res4，然后先通过 res1 = await fetch(${BASE_URL}/1) 发起第一个请求，并用 console.log 打印返回的请求结果。之后只需复制这段请求代码，依次将请求地址修改为 /2、/3、/4，分别完成 res2 到 res4 的请求与打印操作。在浏览器控制台中执行getFourPokemon函数，结果如下。
 
我们成功获取到了所有请求数据。目前我们暂未涉及 async 函数的错误处理 —— 这部分内容后续会为大家详细讲解。仅从功能实现的角度来看，这种写法完全可行，不仅能够精准达成按顺序串行发起 4 次 fetch 请求的需求，还具备极强的代码可读性。
8.2 await的“同步式”执行特性
由于我们将所有异步逻辑代码都编写在 async 关键字声明的异步函数中，因此可以借助 await 关键字，写出看起来与同步代码完全一致的异步执行逻辑，这也是 async/await 语法相较于传统 Promise 链、回调函数的核心优势之一。
具体来说，在 async 函数的执行流程中，当 JavaScript 引擎执行到带有 await 关键字修饰的 fetch 请求代码时，当前这个 async 函数的内部执行流程会进入临时的 “暂停” 状态。但需要重点明确的是，这种暂停是局部且非阻塞式的，它仅作用于当前这个 async 函数内部，并不会阻塞整个 JavaScript 的单线程执行，不会影响其他外部代码的执行。它只会等待当前对应的 Promise 对象完成解析（resolve），获取到 Promise 的返回值并赋值给对应的变量，例8.1小节中的 res1变量，然后执行后续的打印操作，之后再恢复函数的执行流程。
回顾8.1小节示例，在一个请求（res1）完成后，后续的第二个（res2）、第三个（res3）、第四个（res4）请求，都会重复这一“暂停-等待-解析-执行-恢复”的完整流程：执行到对应行的 await fetch 代码时，先暂停当前 getFourPokemon 函数的执行，等待当前请求对应的 Promise 完成解析并将结果赋值给对应变量，再执行后续的 console.log 打印操作，待打印完成后恢复函数执行流程，继而发起下一个请求，以此类推。运行 getFourPokemon 函数后，我们会在控制台中按 /1、/2、/3、/4 的先后顺序，看到四次对应的响应对象打印结果（见8.1小节截图），这完全符合多请求串行执行的预期。。
8.3 与传统异步写法的对比
async/await 这种串行写法，相比传统的回调函数以及 Promise 链写法，是一次巨大的进步，尤其解决了令人头疼的 “回调地狱”问题。
在实现多请求串行的传统 Promise 链写法中，我们需要持续借助 .then() 方法来传递回调函数，并且要在每一个 .then() 回调函数的内部，手动返回下一个 fetch 异步请求（即一个新的 Promise 对象），最终构建出一条层层衔接、依次执行的 Promise 调用链（相关示例可参考 5.1 小节示例）。虽然这种写法已经比原始回调函数更具逻辑性，但依然需要传递回调和维护 Promise 链的返回关系，代码会存在一定的嵌套感，可读性随请求数量增加而下降。
尽管相较于原始的回调函数写法，这种 Promise 链写法已经具备了更清晰的逻辑条理，但它依旧需要手动传递回调函数来处理每一步的异步解析结果，同时还得严谨维护 Promise 链的返回值关联——即确保每个 .then() 内部都返回下一个异步请求对应的 Promise 对象，才能保障链式调用的有序推进。这就使得代码不可避免地带有一定的嵌套痕迹，而且随着串行请求数量的增多，Promise 链会愈发冗长，代码的阅读难度、理解成本也会随之逐步上升，整体可读性呈现明显的下降趋势。
而采用 async/await 语法实现时，我们既无需手动传递任何回调函数，也不会产生层层嵌套的代码结构。所有请求操作均可按照执行顺序逐行平铺书写，代码整体结构扁平规整、条理清晰，完全契合人类的直觉逻辑 —— 先完成这一步操作，待其执行完毕后，再开展下一步操作。这种写法大幅降低了代码的理解门槛与后期维护成本。需要强调的是，这两种实现方式的核心功能完全一致，均是按串行顺序发起四个 API 请求，二者的差异仅体现在语法表现形式上。
8.4 串行请求的执行特性
在此，有一个至关重要的核心要点需要明确强调：无论是通过 async/await 语法实现的多请求串行，还是通过传统 Promise 链实现的多请求串行，8.1小节中的四个 API 请求均为串行执行，并不会被同时发起与发送。具体而言，下一个请求只有在前一个请求完全执行完毕（即对应的 Promise 对象解析成功，且后续关联的打印操作也已执行完成）之后，才会被正式发起。
在传统 的Promise 链写法中，只有当前一个 fetch 请求对应的 Promise 解析完成，其关联的 .then() 回调函数才会被触发执行，继而在该回调函数内部发起下一个 fetch 异步请求（参见6.1小节示例）。而在 async/await 写法中，只有前一个 await 关键字对应的 Promise 对象完成解析，被暂停的 async 函数执行流程恢复后，才会继续向下执行下一行的 fetch 请求代码，从而发起下一个异步请求（参见8.1小节示例）。
8.5 async/await 是 Promise 的语法糖
尽管 async/await 语法为 JavaScript 异步编程带来了极大的开发便捷性与代码可读性，但它并非 JavaScript 语言引入的全新底层异步机制，本质上只是基于 Promise 实现的语法糖（syntactic sugar），并没有任何脱离现有规范的 “黑魔法”。
在底层执行逻辑中，async/await 完全依托 JavaScript Promise 的核心机制来运作：一方面，被 async 关键字声明的函数，其返回值本质上是一个 Promise 对象 —— 函数内部的返回值会被自动包装为该 Promise 的成功决议值（resolve value），函数内部抛出的错误则会成为该 Promise 的拒绝原因（reject reason）；另一方面，await 关键字的核心作用，是替代 Promise 链中 .then() 方法的回调函数调用，它会暂停当前 async 函数的执行流程，等待目标 Promise 完成状态决议，并直接提取 Promise 成功决议后的值，从而将原本需要链式调用的异步逻辑，转化为更符合直觉的同步式代码书写形式。
因此，要真正熟练且规范地运用 async/await 语法，开发者依然需要扎实掌握 Promise 的核心原理，包括 Promise 的三种状态（pending、fulfilled、rejected）、状态的不可逆性、resolve 与 reject 的触发逻辑、以及 Promise 链的执行规则等关键内容。毕竟整个 async/await 机制都是以 Promise 为底层基石构建的，语法糖的价值仅在于优化异步代码的书写结构，让代码更干净、更直观，进而大幅降低异步逻辑的开发与维护成本。
8.6 async/await的错误处理
在8.1小节示例中实现的 getFourPokemon 函数，仅能在网络通畅、请求地址有效等理想场景下，完成四次宝可梦数据的串行请求与响应结果的打印操作，并未考虑各类请求失败的异常场景 —— 例如请求地址无效导致的 404 错误、网络中断引发的请求超时、服务端返回的 500 系列内部错误，或是跨域配置不当造成的请求被拦截等。
但在真实的前端开发流程中，网络环境的不确定性与服务端的不可控性，决定了错误处理是异步请求逻辑中不可或缺的核心环节。一套健壮的异步请求代码，必须具备完善的异常捕获与兜底处理能力，才能保障程序的稳定性与用户体验。针对这类异常场景的捕获与处理方案，也将是我们下一章节的重点讲解内容。
8.7 小结
本节围绕 async/await 实现多请求串行展开讲解，以 getFourPokemon 函数获取 1-4 号宝可梦数据为示例，我们了解了在 async 函数内可通过多个 await 修饰 fetch 请求实现多请求串行发起，await 会触发函数局部暂停但不阻塞 JavaScript 线程，待对应 Promise 解析成功后函数恢复执行并将兑现值赋值给变量，此写法相比传统 Promise 链无需传递回调、维护链式关系，代码结构扁平直观，二者核心功能一致均为串行执行，下一个请求需在前一个请求完全执行完毕后发起；async/await 本质是 Promise 的语法糖，底层依托 Promise 机制运作，async 函数返回值为 Promise 对象，熟练掌握 Promise 原理是用好该语法的前提；当前实现的函数仅覆盖正常请求场景，未处理地址无效、网络中断等异常情况，错误处理作为异步请求的核心环节将是后续讲解重点。
9 async函数错误处理
在异步 JavaScript 编程领域，错误处理始终是保障程序稳健、可靠运行的核心关键环节。相较于同步代码，异步操作存在更多不可控因素（如网络波动、接口异常、资源加载失败等），而在使用 async 函数（作为 async/await 异步语法的核心载体）进行开发时，为了避免单个异步操作的异常导致整个程序崩溃或流程紊乱，我们更需要搭建一套高效、清晰且具备可维护性的错误处理方案。下文将先从普通 Promise 实例的传统错误处理方式切入，逐步过渡到 async 函数对应的专属错误处理逻辑，全方位、系统性地解析其中的核心知识点与实操要点。
9.1 async函数错误处理
在使用 async 函数时，只需要将可能导致 Promise 被拒绝的异步操作或者任何可能抛出错误的代码包裹在 try 块中，就可以通过对应的 catch 块捕获所有相关错误，这是 async 函数中最核心、最常用的错误处理方式。但是try...catch并不是专门用来处理 Promise 或 async 函数错误的。
我们先来看一个简单的示例。
async function fetchFakeWebsite() {
    const res1 = await fetch(`http://nope.nope`);
    console.log(res1);
}
在这段代码中，我们把 fetchFakeWebsite 函数定义为 async 函数，函数内部通过 await 关键字等待 fetch 方法请求一个不存在的网络地址 http://nope.nope。由于该地址无效且无法访问，这个网络请求必然会失败，使得 fetch 方法返回一个状态为“被拒绝（rejected）”的 Promise 对象。而原代码未做任何异常处理，因此执行该函数时会直接抛出网络请求失败的错误。我们该如何捕获 async 函数抛出的错误示例代码如下。
async function fetchFakeWebsite() {
    try{
        const res1 = await fetch(`http://nope.nope`);
        console.log(res1);
    } catch (e) {
        console.log("我们得到了一个错误：",e);
    }
}
在上面的代码中，我们将 fetch 请求代码包裹在 try 块内，在 catch 块中捕获 try 块中出现的所有错误。将捕获到的错误存储在catch 的参数中，打印自定义提示信息 “我们得到了一个错误：”，同时输出保存错误的对象 e。catch 块的参数名是一个形参，你可以自行定义。
当我们运行 fetchFakeWebsite 函数时，被拒绝的 Promise 错误就会被try/catch成功捕获，控制台会输出预设的提示信息和具体错误（如 TypeError: Failed to fetch），程序不会出现意外中断，实现了优雅的错误兜底。结果如下所示。
 
9.2 统一处理批量异步操作
在实际开发中，async 函数往往包含一系列连续的异步操作（多个 Promise），例如8.1小节中用于获取宝可梦数据的 getFourPokemon 函数，这类函数中任意一步操作都可能因网络中断、请求地址无效等原因抛出异常。针对这种场景，无需为每个 Promise 单独编写一个 try/catch 语句块，而是可以采用 “统一包裹” 的思路：在 async 函数顶部编写一个全局的 try 块，将所有可能出错的异步操作（以及其他易抛出错误的代码）全部包裹在内，再配合一个全局的 catch 块来统一处理所有异常。例如8.1小节中的getFourPokemon 函数中第三个请求的地址改为不存在的http://nope.nope，。代码如下所示。
async function fetchFakeWebsite() {
    try {
        const res1 = await fetch(`${BASE_URL}/1`);
        console.log(res1);

        const res2 = await fetch(`${BASE_URL}/2`);
        console.log(res2);

        const res3 = await fetch(`http://nope.nope`);
        console.log(res3);

        const res4 = await fetch(`${BASE_URL}/4`);
        console.log(res4);

    } catch (e) {
        console.log("我们捕获到了一个错误：", e);
    }
}
运行上面的代码，我们就会捕获 res3 请求返回的被拒绝的 Promise，对应的错误会被全局 catch 块捕获。在 res3请求错被误触发后，程序会立即终止 try 块内后续代码的执行，直接进入 catch 块，输出预设的提示信息 “我们捕获到了一个错误：”和具体的错误详情（如 TypeError: Failed to fetch）。前面已经成功执行的请求操作不会收到影响，只终止后续流程。上面的代码运行结果如下。
 
这种 “统一包裹” 的方式极大简化了批量异步操作的错误处理逻辑，显著提升了代码的可读性与可维护性，其作用与普通 Promise 链中通过单个.catch () 处理错误的效果异曲同工。
9.3 小结
async函数的核心错误处理方案为try/catch语句块（无需依赖Promise的.catch()方法），该方案适配性极强：不仅能捕获变量未定义、类型错误等同步错误，还可精准捕获async函数中因网络异常、地址无效等原因被拒绝的Promise错误（如fetch请求失败）。在实际开发中，只需将async函数内所有易出错的异步操作（如连续的网络请求、数据解析）及其他可能抛出错误的代码统一包裹在单个try块中，即可通过一个catch块集中捕获所有相关错误。错误被捕获后程序不会直接崩溃，开发可在catch块中执行灵活的兜底处理（如输出自定义提示信息、打印具体错误详情、返回默认数据等），有效保障程序的稳健运行。
10.异步编程模式：Promise与async/await的对比与实践
在 JavaScript 异步编程中，Promise 及其衍生的 async/await 语法是处理异步操作的核心方法。理解这两种方式的底层逻辑与适用场景，能帮助我们更灵活地应对不同的异步处理需求。接下来，我们将从语法特性、实际场景出发，对比 Promise 原生调用（.then/.catch）与 async/await 语法糖的使用方式，并聚焦“并行执行异步请求”这一典型场景展开分析，最后过渡到“顺序执行异步操作”的场景。
10.1 异步代码处理的底层一致性与语法差异
从底层实现逻辑来看，Promise 对象的 .then/.catch 方法与 async/await 语法本质上实现的是完全相同的功能——二者都是通过监听 Promise 的状态变化，在状态最终确定后执行对应的处理逻辑。我们前面已经了解，Promise 本身包含三种核心状态：pending（进行中）、resolved（已兑现/成功）、rejected（已拒绝/失败）。无论是直接调用 Promise 的 `.then/.catch` 方法处理异步结果，还是使用 async/await 这种更简洁的语法糖，其底层核心逻辑都是围绕 Promise 的这三种状态来设计分支处理逻辑的。两者的核心区别体现在语法层面：
•	Promise 原生的 then/catch 采用链式调用的形式，是早期处理异步逻辑的主流写法，当需要处理连续的异步操作时，往往需要通过回调函数嵌套或者链式拼接的方式来实现；
•	async/await 作为 ES2017 版本引入的语法糖，能够让异步代码的书写形式更贴近同步代码，执行流程也更加直观，契合人们线性的思维习惯，大幅降低了代码的编写与阅读成本。
但要明确的是，async/await 并非脱离 Promise 独立存在 —— 所有标记为 async 的函数，其返回值本质上都是一个 Promise 对象，await 关键字的作用只是 “暂停” 该函数执行，直到等待的 Promise 状态变为 resolved 或 rejected。因此，理解 Promise 的原生逻辑，尤其是 then/.catch 的调用机制是用好 async/await 的基础；而在某些场景下，直接使用 Promise 原生写法反而更简洁、更符合需求。
10.2 并行执行多个异步请求的场景与实现思路
在实际开发过程中，我们常会遇到需要批量发起异步请求的场景：比如需要同时请求多个无依赖关系的独立资源（例如不同 ID 的宝可梦数据、多款商品的详情接口等），这类请求彼此之间不存在依赖，无需等待前一个请求完成后再发起下一个，我们只需一次性发出所有请求，并且遵循“哪个请求先返回结果，就优先处理哪个结果”的原则。这种“并行执行、按结果返回顺序处理”的需求场景，正是 Promise 原生写法的典型适用场景。
我们来看一段相互独立的并行请求。代码如下所示。
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";
const results=[];

fetch( `${BASE_URL}/1`).then((res)=>results.push(res));
fetch( `${BASE_URL}/2`).then((res)=>results.push(res));
fetch( `${BASE_URL}/3`).then((res)=>results.push(res));
在这个示例中，我们首先创建一个名为 results 的空数组用于存储所有宝可梦接口请求返回的响应结果，随后并行发起针对 ID 为 1 至 4 的 4 个宝可梦接口 fetch 请求，无需等待前一个请求完成。每个请求都调用一个 then 回调函数，在请求成功时将响应对象存入 results 数组。执行这个示例，你可以在浏览器控制台中看到results 数组中元素的顺序由请求完成的时间决定，而非发起顺序。结果如下所示。
 
并行请求场景的关键特点是 “请求发起顺序固定，但返回顺序不固定”。我们按 1、2、3 的顺序发起三个宝可梦接口请求，但因服务器处理速度、网络传输耗时的差异，会导致请求完成的顺序可能是 2、3、1 或 3、1、2 等。而由于每个请求的 then 回调函数是独立绑定的，哪个请求先完成，对应的结果就会先被存入results数组，无需等待其他请求结束。
这种方式的优势在于最大化利用网络资源 —— 所有请求同时发起，总耗时取决于最慢的请求，而非所有请求耗时之和，相比串行请求能大幅提升请求效率。
10.3 使用async/await改写并行请求逻辑
虽然 Promise 的原生then方法（结合fetch）可以很好地实现并行请求，但我们也能借助 async/await 语法实现同样的逻辑，其核心在于不使用 await 阻塞 async 函数的执行流程。
我们可以将 Promise 的原生then方法改为使用独立的 async 函数，在每个函数内部使用 await 等待对应 fetch 请求的结果，然后将结果存入数组和打印完成日志。最后，直接调用这三个 async 函数就能同时发起请求——因为 async 函数调用后会立即返回一个 Promise 对象，且函数内部的异步逻辑会在事件循环中并行执行，不会因为前一个函数的 await 而阻塞后续函数的调用。代码示例如下。
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";
const results=[];

async function getPokemon1(){
    const res=await fetch(`${BASE_URL}/1`);
    results.push(res);
    console.log("响应1已经完成");
    
}

async function getPokemon2(){
    const res=await fetch(`${BASE_URL}/2`);
    results.push(res);
    console.log("响应2已经完成");
}

async function getPokemon3(){
    const res=await fetch(`${BASE_URL}/3`);
    results.push(res);
    console.log("响应3已经完成");
}

getPokemon1();
getPokemon2();
getPokemon3();

console.log("所有的响应都已经完成");
这段代码通过三个独立的 async 函数实现了宝可梦接口的并行异步请求：首先定义了 getPokemon1、getPokemon2、getPokemon3 三个 async 函数。每个函数内部通过 await 等待对应 ID（1、2、3）的宝可梦接口 fetch 请求完成，获取响应对象后将其推入 results 数组，并打印该请求完成的日志。随后直接调用这三个 async 函数（未添加 await），同时发送这三个请求被，函数内部的 fetch 请求并行发起，每个函数内的 await 仅暂停自身函数内的执行流程（等待 fetch 结果），但不会阻塞其他函数的启动。最后一行 console.log("所有的响应都已经完成") 打印日子语句会优先于三个 async 函数内的日志打印语句。因为 async 函数调用后返回的是 Promise 对象，其内部逻辑属于异步宏任务，需等同步代码执行完毕后才会进入事件循环执行，且三个请求完成日志的打印顺序不固定（由接口响应速度决定），即便打印了“所有的响应都已经完成”，实际三个请求也可能尚未完成。
为了避免重复编写多个结构相似的 async 函数，我们可以将独立的 async 函数抽离到一个通用的 async 函数中。该函数接收一个与请求相关的参数（比如宝可梦ID），在函数内部根据这个参数拼接请求地址，发起异步请求并处理返回结果，如将结果存入数组、打印完成日志。之后只需传入不同的请求参数依次调用这个通用函数（调用时不添加 await），就能实现和 Promise 原生写法完全一致的并行效果。所有请求都会被同时发起，请求结果会按请求完成的先后顺序返回，且程序中的同步代码不会被阻塞，仍能正常执行。我们将上面示例中的三个async函数抽离到一个通用的函数中。代码示例如下。
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";
const results = [];

async function getPokemon(num) {
    const res = await fetch(`${BASE_URL}/${num}`);
    results.push(res);
    console.log(`响应${num}已经完成`);
}

getPokemon(1);
getPokemon(2);
getPokemon(3);

console.log("所有的响应都已经完成");
在这段代码中，我们将请求异步函数封装在通用的 async 函数 getPokemon 中。getPokemon 函数接收一个参数，在这个函数内部通过 await 等待对应 ID 的宝可梦接口 fetch 请求完成，将响应存入数组并打印该请求完成日志。我们直接调用 getPokemon，并传入要请求的宝可梦 ID 1、2、3实现并行请求。函数内的 await 仅阻塞自身执行，函数间互不阻塞。最后同步代码打印的“所有的响应都已经完成”会优先执行，且三个请求完成日志的打印顺序由接口响应速度决定。这代码运行结果如下。
 
需要注意的是，直接调用 async 函数，也就是直接调用 await getPokemon ()，并行请求执行会变成串行请求执行：必须等待第一个请求完成，才会发起第二个请求，这就违背了 “并行” 的初衷。
以上我们探讨的是 “并行执行、无序处理” 的异步场景，这种方式适用于请求之间无依赖、无需保证请求结果顺序的场景。但在实际开发中，我们也会遇到需要 “顺序执行异步操作” 的场景 —— 比如第二个请求的参数依赖于第一个请求的结果，或者必须保证结果按请求顺序排列。接下来，我们就来分析这类场景的处理方式。
10.4小节
Promise 的 .then/.catch 方法与 async/await 语法虽语法形式不同，但底层逻辑一致，均基于 Promise 状态变化实现异步处理。在并行执行异步请求时，可通过批量发起独立的 Promise 或直接调用 async 函数来实现，请求结果会按完成顺序处理。而使用 async/await 改写并行请求的核心是避免通过 await 阻塞函数调用，防止请求串行执行。
11. 异步模式：串行异步操作
在异步编程场景中，让多个异步操作严格按照既定顺序依次执行，是开发过程中极为常见且核心的需求。不同于同步代码天然的线性执行逻辑，异步操作（如网络请求、文件 I/O 等）本身不具备有序执行的特性。如果不做特殊处理，极易出现执行顺序混乱的问题。在实际业务中，很多场景都依赖异步操作的有序性：比如在用户数据请求链路中，我们必须先调用接口获取有效的用户身份令牌，只有拿到该令牌后，才能用它作为请求凭证去获取用户的个人数据，最后再依据返回的个人数据去请求对应的系统权限信息；再比如按序号依次调用 /1、/2、/3 这类存在依赖关系的接口时，必须等待前一个接口请求完成并返回结果后，才能发起下一个接口的请求。这种 “一次仅执行一个异步操作、严格遵循先后执行次序” 的逻辑，是异步编程中最基础也最关键的应用场景之一，接下来我们就详细拆解实现这一需求的两种主流思路，对比二者各自的优劣。
11.1 基于Promise链实现异步操作的顺序执行
Promise 的诞生，彻底解决了早期 JavaScript 异步编程中回调函数多层嵌套引发的 “回调地狱” 问题 —— 这种嵌套不仅让代码结构杂乱无章、可读性极差，还会导致调试和维护成本大幅增加。而基于 Promise 构建的 Promise 链，更是实现异步操作顺序执行的经典且基础的方式：它通过.then()方法的链式调用，让前一个 Promise 完成（状态变为 resolved）后，再触发下一个异步操作，以此严格保证多个异步任务的执行次序。
我们来看看下面的的代码示例。
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";
const results = [];

fetch(`${BASE_URL}/1`).then((res) => {
    results.push(res);
    console.log("第一个请求完成");
    return fetch(`${BASE_URL}/2`);
}).then((res) => {
    results.push(res);
    console.log("第二个请求完成");
    return fetch(`${BASE_URL}/3`);
}).then((res) => {
    results.push(res);
    console.log("响应3已经完成");
});
在这段代码中，我们依次请求 /1、/2、/3 三个接口，具体的实现逻辑是这样的：我们先创建第一个 Promise 来发起对 /1 的请求，当这个 Promise 状态变为“已解决（fulfilled）”（也就是 /1 请求成功返回）后，在其 .then() 方法中，返回一个新的 Promise——这个新 Promise 负责发起对接口 /2 的请求；同理，等 /2 请求对应的 Promise 完成后，再在它的 .then() 里返回发起 /3 请求的 Promise。
为了记录每一步的执行结果，我们定义了一个用于存储返回结果的数组results：当 /1 请求完成时，将返回结果存入result数组，并打印 “第一个请求完成” 的提示；当 /2 请求完成时，重复这个过程，将结果追加到result数组并打印 “第二个请求完成”；直到 /3 请求完成，最后统一处理这个结果数组（比如展示数据、做后续计算等）。
整个执行流程的核心是 “链式调用 + 返回新 Promise”：Promise 链会确保只有前一个异步操作完全结束，后一个异步操作才会开始执行。无论我们重复执行多少次 Promise 链，/1、/2、/3 的请求顺序都不会被打乱 —— 永远是 /1 完成后执行 /2，/2 完成后执行 /3，这正是顺序执行的核心要求。
11.2 使用 async/await 简化异步顺序执行代码
尽管 Promise 链能够实现异步操作的顺序执行，但随着异步操作数量的增加，层层串联的.then()调用会让代码的层级感愈发明显，不仅降低了代码的可读性，还显著提高了后续的维护成本。而 async/await 作为 Promise 的语法糖，以近乎同步代码的线性书写方式，既能实现完全一致的异步顺序执行逻辑，又能使代码的可读性与易维护性得到大幅提升
要使用 async/await 实现异步操作的顺序执行，我们需要将所有异步操作代码封装在一个 async 函数中。在这个 async 函数内部，我们完全可以以“同步思维”的方式编写代码。我们可以将11.1小节示例
使用一个 getThreeResourcesInSequence 函数来改写重构。示例代码如下。
async function getThreeResourcesInSequence() {
    const res1 = await fetch(`${BASE_URL}/1`);
    console.log("请求1已经完成");
    const res2 = await fetch(`${BASE_URL}/2`);
    console.log("请求2已经完成");
    const res3 = await fetch(`${BASE_URL}/3`);
    console.log("请求3已经完成");
    const results=[res1,res2.res3];
}
在这段代码中，我们先发起对接口 /1 的请求，用 await 关键字等待请求完成，将返回结果赋值给变量 res1，打印“第一个请求完成”。在第一个请求完成后，再发起对接口 /2 的请求，同样用 await 等待第二个请求完成，将结果赋值给变量 res2，打印 “第二个请求完成”。在第二个请求完成后，继续发起对接口 /3 的请求，使用 await 等待请求三完成，将请求结果存入变量 res3 中，并打印 “第三个请求完成”。最后，我们可以直接创建一个数组results，将res1、res2、res3 按顺序存入，后续即可直接使用这个数组对所有请求结果做统一处理。
在上面这个过程中，await 关键字的作用和 Promise 链中 .then() 的作用本质是一致的 —— 都会暂停其所在函数的执行，直到对应的 Promise 完成，再继续执行后续的请求代码。也就是说，await 天然就保证了“前一个异步操作完成后，再执行下一个” 的顺序性，而代码的写法却完全是线性的，不需要嵌套的 .then() 回调函数，代码逻辑上更符合我们的阅读和编写习惯。
11.3 两种写法的可读性与维护性对比
从功能层面来看，Promise 链与 async/await 实现的异步顺序执行逻辑完全一致 —— 无论是执行次序、结果获取，还是异常处理的底层逻辑，二者均无本质差异；async/await 本质上只是 Promise 的语法糖，其代码最终仍会被编译为基于 Promise 的执行逻辑。
但从代码的整洁度、易读性和维护性来看，async/await 的优势非常明显：
•	Promise 链需持续嵌套.then()方法，且每个.then()内部还要返回新的 Promise；当异步操作数量增加（如从 3 个增至 5 个、10 个），链式调用会愈发冗长，回调嵌套也会导致代码的执行流程变得晦涩不直观。
•	而 async/await 让代码回归 “自上而下” 的线性逻辑，只需按照 “发起请求→等待完成→处理结果→发起下一个请求” 的思路编写即可，无需关注 “返回 Promise”“链接 then” 等细节，代码执行逻辑与我们的思考逻辑完全契合。
如果分别在 11.1 小节和 11.2 小节示例中添加 5 个异步操作的顺序执行，我们需要写 5 层嵌套的 .then()；而用 async/await，只需要 5 行带 await 的线性代码，每一行对应一个异步操作，任何人看代码都能立刻明白执行顺序，这也是为什么在实际开发中，只要场景合适，我们更推荐用 async/await 来实现异步操作的顺序执行。具体代码你可以自己添加
11.4 小节
异步操作顺序执行的核心是 “前一个完成后再启动下一个”：Promise 链通过嵌套.then()并返回新 Promise 实现该逻辑。async/await 作为 Promise 语法糖，以线性同步式写法实现相同逻辑，无需嵌套回调。二者功能等价，但 async/await 在代码整洁度、易读性上更具优势，是实现异步顺序执行的更优选择。
12 Promise.all实现并行异步操作
在异步编程的实际开发中，我们常常会遇到需要处理多个异步操作的场景 —— 比如要发起 10 次、100 次，甚至数量不固定的动态网络请求。如果逐个等待每个 Promise 执行完成，会大幅降低程序的执行效率，而 Promise.all 正是专门用于解决这类批量异步处理问题的核心辅助方法。
12.1 Promise.all的核心定位与基本规则
Promise.all是 JavaScript 提供的专门处理 Promise 数组的内置方法，它的核心作用是将多个独立的 Promise “打包”成一个新的 Promise 对象，其执行规则可以总结为 “全成则成，一败则败”。Promise.all基本语法如下：
Promise.all([promise1, promise2, promise3])
  .then((results) => {
    // results 是一个数组，顺序和传入的 Promise 一一对应
    console.log(results);
  })
  .catch((error) => {
    // 只要有一个 Promise 失败，就会进这里
    console.error(error);
  });
1.	接收参数：Promise.all参数是一个由 Promise 实例组成的数组（也可以是可迭代对象，如 Set）；
2.	返回值：Promise.all参数返回的值是一个全新的 Promise 对象，这个新 Promise 的状态完全由数组内的所有子 Promise 决定；
3.	成功条件：只有当数组中所有 Promise 都被成功兑现（resolved）时，新 Promise 才会是 resolved 状态，此时它的返回值是一个数组，数组内的结果与原 Promise 数组的顺序完全对应（比如第一个 Promise 返回的结果位于返回数组的第一个位置，即使它不是最先完成的）；
4.	失败条件：只要数组中有任意一个Promise 被拒绝（rejected），新 Promise 就会是 rejected 状态，且只会返回这个失败 Promise 的错误信息，不会等待其他未完成的 Promise。
在早期的异步代码中，我们通常通过 then 和 catch 来处理Promise.all的结果，这也是最基础的用法。我们来看一下下面这个示例。
const lotsOfFetchCalls = [
    fetch(`${BASE_URL}/1`),
    fetch(`${BASE_URL}/2`),
    fetch(`${BASE_URL}/3`),
    fetch(`${BASE_URL}/4`),
    fetch(`${BASE_URL}/5`),
    fetch(`${BASE_URL}/6`),
];

Promise.all(lotsOfFetchCalls).then((results) => {
    console.log("Promise.all() is done and resolved!");
    console.log(results);
}).catch((error) => {
    console.log("ONE of the promises rejected!");
    console.log(error);  
});
这段代码用于批量获取 6 个宝可梦的信息。我们首先创建一个名为lotsOfFetchCalls的数组，数组中依次放入 6 个 fetch 请求——每个请求对应一个宝可梦的接口地址（即${BASE_URL}/1到${BASE_URL}/6），每个fetch调用都会返回一个独立的 Promise 实例。需要注意的是，在调用 fetch 方法的瞬间，这 6 个网络请求就已经并行发起。我们将lotsOfFetchCalls 数组作为参数传递给而Promise.all方法得到一个新的 Promise。然后调用 then 处理所有 Promise 成功的场景：比如打印 “Promise.all is done and resolved”，并打印输出汇总后的 results 数组（包含 6 个宝可梦的响应数据），输出结果如下。
 
同时，使用 catch捕获任意一个 Promise 失败的场景，打印 “one of the promises was rejected”，并输出具体的错误信息。如果将任意一个请求改为不存在的URL http://nope.nope，输出结果如下。
 
我们可以看到，任意一个 Promise 失败，Promise.all会立即终止并触发.catch，即使其他 Promise 可以成功完成，也不会再返回任何成功结果 —— 这正是 “all” 的含义：只有全部成功，才算成功。
12.2 使用 async/await 简化 Promise.all
随着 ES8（ECMAScript 2017）正式引入 async/await 语法，这一专为简化异步编程设计的语法糖，让我们能够以更贴近同步代码的线性逻辑、更高的可读性来使用 Promise.all 处理批量异步任务，彻底摆脱传统的通过 .then/.catch 链式调用处理 Promise.all 结果时容易出现的“回调地狱”问题。我们完全可以使用一个 async 函数重构12.1中的示例代码，这样就不需要再使用 then 和 catch 方法。代码示例如下。
const lotsOfFetchCalls = [
    fetch(`${BASE_URL}/1`),
    fetch(`${BASE_URL}/2`),
    fetch(`${BASE_URL}/3`),
    fetch(`http://nope.nope`),
    fetch(`${BASE_URL}/5`),
    fetch(`${BASE_URL}/6`),
];

async function getLotsOfPokemon() {
    try {
        const results = await Promise.all(lotsOfFetchCalls);
        console.log("所有的请求都完成了！");
        console.log(results);
    }
    catch (error) {
        console.log("有一个请求失败了！");
        console.log(error);
    }
}
我们可以定义一个名为 getLotsOfPokemon 的 async 函数包裹所有批量请求的异步代码。 函数内部通过 try/catch 块处理 Promise.all 的执行结果，try 块中使用 await Promise.all (lotsOfFetchCalls) 等待数组内所有 Promise 执行完成，直接获取汇总后的结果数组，并打印 “所有的请求都完成了！” 及结果数组；catch 块则会捕获数组中任意一个 Promise 失败的情况（如代码中第四个请求地址错误导致的失败），打印 “有一个请求失败了！” 并输出具体的错误详情，最后调用这个 getLotsOfPokemon 函数，即可执行完整的批量异步请求流程。这种写法的优势在于，代码逻辑更接近同步编程的思维，没有回调嵌套，可读性和可维护性更高，也是现代 JavaScript 开发中处理批量异步的首选方式。
12.3 Promise.all的适用场景与注意事项
Promise.all的核心价值在于并行处理多个独立的异步操作，并等待所有操作完成后统一处理结果，常见适用场景包括：
•	批量获取数据（如同时请求多个接口的列表数据）；
•	批量上传文件（如一次性上传多张图片）；
•	并行执行多个耗时的异步计算，汇总最终结果。
使用时也需要注意：
•	异步操作的并行性：Promise.all不会改变异步操作的执行方式，只要创建了 Promise（比如调用了 fetch），操作就已经开始，Promise.all只是等待所有操作结束；
•	“一败则败” 的特性：如果业务允许部分操作失败、只获取成功结果，不能直接使用Promise.all，需要先给每个 Promise 单独添加.catch处理错误（比如返回一个默认值），再传入Promise.all；
•	结果顺序的一致性：返回的结果数组顺序与传入的 Promise 数组顺序完全一致，与实际完成顺序无关，这能保证我们可以准确对应每个异步操作的结果。
12.4 小节
Promise.all 作为处理批量 Promise 的核心方法，会接收 Promise 数组并返回一个新的 Promise，遵循 “全成则成，一败则败” 的规则。结合 async/await + try/catch 是使用该方法的最优方式，能让代码更易读、逻辑更清晰。同时该方法适用于并行处理多个独立异步操作且需要汇总所有操作成功后结果的场景，使用时需注意其 “一败则败” 的特性以及结果顺序的一致性。
13 Promise.allSettled()获取全部结果
在前述内容中，我们已经学习了 Promise.all 方法，其核心行为特征可概括为“全成则成，一败则败”：该方法接收一个由 Promise 实例组成的可迭代对象（通常为数组），并返回一个全新的 Promise 实例。只有当传入的所有 Promise 实例均进入已解决（resolved）状态时，这个新返回的 Promise 才会触发解决逻辑；而只要其中任意一个 Promise 实例进入已拒绝（rejected）状态，Promise.all 返回的 Promise 就会立即触发拒绝逻辑，且无法获取任何已成功完成的异步操作结果。
但在实际开发中，开发者往往会面临更具灵活性的业务诉求：需要批量发起多个相互独立的异步操作（例如并行请求多个用户的资料信息、同时上传多张图片文件等）。这类场景下，异步操作彼此无依赖关系，即便部分操作执行失败，业务侧仍希望保留所有成功操作的结果，同时精准知晓失败操作的具体原因，而非因单个操作失败导致所有结果都无法获取。此时 Promise.all “一败则败” 的特性便无法满足这类需求，而 Promise.allSettled 方法正是为这类场景专门设计的异步处理方案。
13.1 Promise.allSettled的核心特性
Promise.allSettled 与 Promise.all 的语法调用形式完全一致：二者均接收一个由 Promise 实例组成的可迭代对象（常见为数组）作为入参，且最终都会返回一个全新的 Promise 实例。Promise.allSettled基本语法如下：
Promise.allSettled([promise1, promise2, promise3])
    .then((results) => {
        console.log(results);
    }).
    catch((error) => {
        console.log(error)
    });
从异步流程的管控逻辑来看，Promise.allSettled 与 Promise.all 的核心行为存在本质差异，Promise.allSettled 的关键特性可高度概括为“全量等待，结果全返”，具体可从以下两个维度展开解析：
1. 状态敲定条件
返回的新 Promise 实例会等待传入的可迭代对象中的所有 Promise 成员的状态全部确定后，才会变成 resolved 状态。这里的“状态敲定”包含两种情况 —— 无论是传入的可迭代对象中的所有 Promise 成员执行成功变为 fulfilled（已兑现）状态，还是执行失败变为 rejected（已拒绝）状态，均属于状态敲定的范畴。
与 Promise.all “一败则败” 的触发逻辑不同，Promise.allSettled 不会因某一个 Promise 成员的拒绝而提前终止等待或触发自身的拒绝逻辑，而是始终保持等待状态，直至所有异步操作完成状态闭环。
2. 返回结果格式
与 Promise.all 直接返回“与入参 Promise 顺序一一对应的成功结果数组”不同，Promise.allSettled 返回的结果数组，其每一个元素都是一个标准化的结果描述对象，该对象的结构由对应 Promise 成员的最终状态决定，且严格遵循 “结果顺序与入参 Promise 顺序一致”的规则：
•	若对应 Promise 执行成功，结果对象的 status 字段值为字符串 fulfilled，同时会通过 value 字段存储该 Promise 成功兑现的结果数据；
•	若对应 Promise 执行失败，结果对象的 status 字段值为字符串 rejected，同时会通过 reason 字段携带该 Promise 被拒绝的具体原因（如网络请求超时、接口返回错误状态码、参数校验失败等各类异常信息）。
这种设计让开发者可以在批量异步操作执行完毕后，完整获取每一个操作的执行状态与结果，无需担心因个别操作失败而丢失全部有效数据。
13.2 Promise.allSettled的实际应用场景示例
我们可以使用 async/await 结合批量获取 GitHub 用户资料的实际业务场景，直观理解 Promise.allSettled 的应用方式，示例代码如下。
async function allSettledDemo() {
    const GITHUB_BASE_URL = "https://api.github.com/users";

    let elieP=fetch(`${GITHUB_BASE_URL}/users/eliep`);
    let joelP=fetch(`${GITHUB_BASE_URL}/users/joelburton`);
    let badUrl=fetch("http://definitelynotarealsite.com");
    let coltP=fetch(`${GITHUB_BASE_URL}/users/colt`);   
    let anotherbadUrl=fetch("http://definitelynotarealsite.com");
    
    let results=await Promise.allSettled([
        elieP,
        joelP,
        badUrl,
        coltP,
        anotherbadUrl
    ]);

    console.log(results);
}
在上面这个示例中，我们发了起多个基于 fetch API 的异步请求，这些请求可分为两类：一类是指向真实存在的 GitHub 用户的有效请求，如 elieP、joelburton、Colt；另一类则是指向完全无效的网络地址的无效请求，如 badUrl、anotherbadUrl。
由于每个 fetch 调用都会返回一个 Promise 实例，因此我们可直接将所有请求对应的 Promise 实例整合为一个数组，并将该数组作为入参传入 Promise.allSettled 方法。随后，我们既可以通过 await 语法以同步写法等待所有异步操作执行完毕，也可以通过 then() 链式调用的异步写法处理最终结果。无论采用哪种方式，最终都能得到一个与入参 Promise 数组顺序完全对应的完整结果数组。结果如下所示。
 
•	对应有效请求的对象：status为 "fulfilled"，value属性存储的是 fetch 请求成功返回的用户资料响应数据；
•	对应无效请求的对象：status为 "rejected"，reason属性会清晰展示错误原因（如“网络请求失败”“URL 不存在”等）。
在获取到 Promise.allSettled 返回的标准化结果数组后，我们可以根据业务需求灵活处理：比如通过筛选操作，将status为 "fulfilled" 的对象汇总到“成功数组”，集中处理有效用户资料；将 status 为 "rejected" 的对象汇总到“失败数组”，分析失败原因（如是否是 URL 拼写错误、网络波动），甚至针对失败请求做重试、给用户提示等后续操作——这种能同时获取“成功 / 失败全量结果”的特性，正是 Promise.allSettled 的核心价值。我们将上面示例中的成功和失败的Promise打印处理，代码示例如下。
......
const fulfilled=results.filter((r)=>r.status==="fulfilled");
const rejected=results.filter((r)=>r.status==="rejected");
console.log(fulfilled);
console.log(rejected);
执行上面的代码，输出结果如下所示。
 
需要注意的是，在上面的示例中我们使用 async 函数结合 await 的方式调用 Promise.allSettled，但它的使用形式并不局限于此：和所有 Promise 方法一样，也可以通过.then链式调用处理返回的结果数组，比如在.then回调中直接筛选成功 / 失败的结果、做数据格式化等操作，完全适配不同的代码风格和开发场景。
13.4 Promise.all vs Promise.allSettled 的核心差异对比
为了更清晰区分二者的适用场景，我总结了Promise.all 和 Promise.allSettled 核心差异：
特性	Promise.all	Promise.allSettled
完成条件	所有 Promise 都成功才会解决	所有 Promise 都完成（成功 / 失败）即解决
失败处理	任意一个失败则整体拒绝，无结果返回	不拒绝，仅在结果对象中标记失败原因
返回结果	成功值数组	结果对象数组
结果结构	value	{ status, value / reason }
顺序	与传入的 Promise 顺序一致	与传入的 Promise 顺序一致
语义	“全都必须成功”	只需要其中一个 Promise 被解决即可
核心适用场景	求所有异步操作必须全部成功的场景	允许部分操作失败，需返回全量结果的场景
13.4 小结
Promise.allSettled 弥补了 Promise.all “一败则败” 的局限性，专门适用于需获取所有异步操作全量结果（无论成功与否）的场景，其核心特性为 “全量等待、结果全返”，返回的结果数组中每个元素均会标注对应异步操作的状态（fulfilled 或 rejected），并携带相应的成功值或失败原因；相较于侧重 “全成功才返回” 的 Promise.all，Promise.allSettled 更侧重 “全完成就返回”，实际开发中可根据业务是否允许部分失败来选择适配的批量异步处理方法。
14 Promise.race：异步任务的竞赛模式
在批量异步任务的处理场景中，除了“全量结果回流”的需求外，还存在一种典型诉求：一次性发起多个异步操作，仅需要获取和处理首个完成状态确定的任务结果，其余未完成的任务则不再关心。这种 “谁先完成，谁就决定最终结果” 的执行逻辑，可被概括为异步任务的竞赛模式，而 Promise.race 正是为这类场景设计的 Promise 辅助方法。
Promise.race 的语法调用形式与 Promise.all、Promise.allSettled 保持一致：都接收一个由 Promise 实例组成的可迭代对象（常见为数组）作为传入参数，并返回一个全新的 Promise 实例。其核心执行规则可总结为：返回的新 Promise 实例，会在入参 Promise 集合中任意一个成员的状态率先敲定（无论是 fulfilled 成功还是 rejected 失败）时，立即跟随该成员的状态完成自身的状态闭环，且不会等待剩余 Promise 成员的执行结果。Promise.race基本如何下所示：
Promise.race([promise1, promise2, promise3])
  .then((result) => {
    // 第一个 fulfilled 的 Promise 的结果
    console.log("赢了的是：", result);
  })
  .catch((error) => {
    // 第一个 settled 的如果是 rejected，就会进这里
    console.error("第一个失败的是：", error);
  });
14.1 成功场景演示：Pokemon API 请求竞赛
我们可以通过调用 Pokemon API 的多组 fetch 请求，直观演示 Promise.race 的竞赛逻辑。将多个指向不同 Pokemon 资源的 fetch 请求（每个请求对应一个 Promise 实例）整合为一个数组，传入 Promise.race 方法，并通过 .then 链式调用处理最终结果，示例代码下。
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";

const lotsOfFetchCalls = [
    fetch('https://nope.nope'),
    fetch(`${BASE_URL}/2`),
    fetch(`${BASE_URL}/3`),
    fetch(`${BASE_URL}/4`),
    fetch(`${BASE_URL}/5`),
    fetch(`${BASE_URL}/6`)
  ];

Promise.race(lotsOfFetchCalls)
  .then(winner => console.log('首个完成的请求结果：', winner))
需要注意的是，这段代码的执行结果不具备确定性。由于每个 fetch 请求的网络传输耗时存在差异，每次执行时率先完成的 Promise 成员都可能不同 —— 可能是 pokemon/1，也可能是 pokemon/2 或 pokemon/3。Promise.race 会严格遵循 “先到先得” 的原则，将首个状态敲定的 Promise 结果作为自身的返回结果，其余未完成的请求则会被忽略。代码运行结果如下。
 
14.2 失败场景演示：首个异常决定整体流程
Promise.race 的竞赛逻辑对失败状态同样适用：当传给 Promise.race 的所有 Promise 组成的集合（通常是数组）里，有某个 Promise 最先完成状态确定，且这个状态是 rejected（失败）时，Promise.race 返回的新 Promise 会立刻触发拒绝逻辑，整个异步流程会直接进入 catch 异常处理环节。
我们在 14.1 示例中 的请求数组中加入一个指向无效地址的 fetch 请求（如 http://nope.nope），即可验证该特性：
const BASE_URL = "https://pokeapi.co/api/v2/pokemon";

const lotsOfFetchCalls = [
  fetch('http://nope.nope'), // 无效地址，会立即失败
    fetch(`${BASE_URL}/2`),
    fetch(`${BASE_URL}/3`),
    fetch(`${BASE_URL}/4`),
    fetch(`${BASE_URL}/5`),
    fetch(`${BASE_URL}/6`)
  ];

Promise.race(lotsOfFetchCalls)
  .then(winner => console.log('首个完成的请求结果：', winner))
  .catch(error => console.log('首个触发的异常：', error))
在这个示例中，指向 http://nope.nope 的请求会因地址无效而快速触发 rejected 状态，且该状态会优先于其他合法的 Pokemon API 请求完成确定。因此，Promise.race 返回的 Promise 会立即变为拒绝状态，执行 catch 中的异常处理代码，而后续的合法请求即便最终能成功完成，其结果也不会被处理。示例运行结果如下所示。
 
如果您的代码执行没有像上面这样报错，是因为Promise.race 只关注所有传入的 Promise 中第一个进入敲定（settled）状态的那个的 Promise，该 Promise 的状态（兑现 / 拒绝）会直接决定 Promise.race 自身的最终状态。如果第一个确定的 Promise 为 fulfilled 状态，Promise.race 便兑现并触发 then 流程。如果第一个确定的 Promise 为 rejected 状态，Promise.race 便拒绝并触发 catch 错误处理流程。其余未完成的 Promise，无论后续成功或失败，结果都会被直接忽略。在上面的示例中，第一个请求返回的 Promise 可能不是最先完成，反而是后面的请求返回的Promise最先完成，因此你的代码执行结果可能如下所示。
 
14.3 小节
Promise.race 的本质是异步任务的竞速执行器，它不关注所有任务的完成情况，只对首个完成状态的任务结果做出响应。无论是成功结果还是失败异常，只要是第一个敲定的状态，就会直接决定 Promise.race 返回实例的最终状态，这一特性使其在 “超时控制”“多源请求择优选择” 等场景中具备不可替代的价值。
15 构建我们自己的 Promise 对象
在前面的内容中，我们使用的 Promise 均由浏览器原生 API（如 fetch）自动返回。但在实际开发中，我们常会遇到基于回调函数设计的 API（如 setTimeout、fs.readFile 等），这类 API 无法直接使用 await 或 .then 进行链式调用。
为了让 Promise 回调函数式操作适配现代异步编程范式，我们需要手动创建 Promise 对象，将回调函数代码封装为支持 Promise 规范的异步操作。这一过程也被称为“回调函数 Promise 化”。
需要提前说明的是，Promise 的创建语法初看会略显抽象，但只要掌握核心逻辑，就能灵活运用。
15.1  Promise基础创建语法：执行器函数与状态控制
创建自定义 Promise 的核心语法是使用 new 关键字调用 Promise 构造函数，其语法结构如下：
const promise = new Promise((resolve, reject) => {
    // 异步操作逻辑
});
该语法有两个核心要点需要掌握：
1.构造函数参数：执行器函数
Promise 构造函数必须接收一个函数作为参数，这个函数被称为执行器函数。执行器函数会在 Promise 实例创建时立即执行，无需手动调用。
2.执行器函数参数：状态控制回调函数
•	执行器函数本身会接收两个预设的回调函数作为参数，分别是 resolve 和 reject。
•	从语法层面来说，这两个参数的名称可以自定义，但 resolve 和 reject 是行业通用的标准命名，遵循该惯例能让代码更具可读性和可维护性。
•	调用 resolve 参数时(值) 时，会将当前 Promise 实例的状态切换为 fulfilled（已兑现），并将传入的参数作为 Promise 的兑现值；
•	调用 reject(原因) 时，会将当前 Promise 实例的状态切换为 rejected（已拒绝），并将传入的参数作为 Promise 的拒绝原因。
15.2 实战示例 1：封装 await 函数实现延迟执行
我们以 setTimeout 为例，演示如何将一个基础的回调式操作封装为返回 Promise 的函数。需求是实现一个 await 函数，接收毫秒数作为参数，延迟指定时间后再执行后续逻辑，且支持 await 调用。
步骤 1：分析原始回调式写法的痛点
setTimeout 的传统用法基于回调函数，若嵌套多层延迟逻辑代码，极易出现“回调地狱”：
// 原始回调写法：延迟 2 秒后打印 hi
setTimeout(() => {
    console.log("hi");
}, 2000);
这种写法无法直接使用 await 实现“同步化”的代码书写逻辑，而通过 Promise 封装即可解决这一问题。
步骤 2：封装返回 Promise 的 wait 函数
我们定义 await 函数，在其内部创建 Promise 实例，并通过 setTimeout 控制 resolve 参数函数的调用时机：
function wait(duration) {
    // 返回 Promise 实例
    return new Promise((resolve, reject) => {
        // 利用 setTimeout 实现延迟
        setTimeout(() => {
            // 延迟结束后调用 resolve，标记 Promise 兑现
            resolve();
        }, duration);
    });
}
这里需要注意一个关键细节：如果在执行器函数中立即调用 resolve，Promise 会被立即兑现，无法实现延迟效果。只有将 resolve 放入 setTimeout 的回调中，才能实现指定时长的延迟调用。
步骤 3：使用 await 调用 wait 函数
我们在 async 函数中调用 await，体验同步化的异步代码书写方式：
async function demo() {
    console.log("hi");
    // 等待 1 秒，不阻塞整个 JavaScript 线程
    await wait(1000);
    console.log("there");
}

demo();
执行该函数后，控制台会先输出 hi，等待 1 秒后再输出 there。需要强调的是，await 仅会暂停当前 async 函数的执行，不会阻塞整个 JavaScript 引擎的线程，其他同步代码仍可正常运行。
15.3 进阶示例 2：带兑现值的 Promise 封装
在实际开发场景中，我们往往需要 Promise 兑现时携带具体数据。此时只需在调用 resolve 时传入指定值，即可在 await 或 .then 中获取该值。
我们修改上述 wait 函数，使其延迟结束后返回一个自定义字符串：
function wait(duration) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            // 传入兑现值
            resolve("this is the resolved value");
        }, duration);
    });
}
随后通过两种方式获取兑现值：
1.使用 await 获取（推荐）
async function demo() {
    const val = await wait(2000);
    console.log(val); // 输出：this is the resolved value
}
2.使用 .then 链式调用获取
wait(2000).then(val => {
    console.log(val); // 输出：this is the resolved value
});
15.4 进阶示例 3：Promise 拒绝逻辑的触发与捕获
除了兑现状态（fulfilled），我们还需要掌握 Promise 拒绝状态（rejected）的触发方式 —— 即调用 reject 函数。下面我们创建一个 randomResolveReject 函数，该函数会在指定延迟后随机触发 resolve 或 reject，以此演示拒绝逻辑的完整流程。
1. 函数封装实现
function randomResolveReject(delay) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const rand = Math.random();
            if (rand < 0.5) {
                // 随机兑现，携带自定义值
                resolve("pickles");
            } else {
                // 随机拒绝，携带错误原因
                reject(new Error("random reject triggered"));
            }
        }, delay);
    });
}
2. 拒绝状态的捕获与处理
我们通过 then/catch 链式调用处理该函数的执行结果，并观察不同状态下的执行逻辑：
randomResolveReject()
    .then(res => {
        console.log("inside then:", res);
    })
    .catch(err => {
        console.log("inside catch:", err.message);
    });
执行该代码后，控制台有 50% 的概率输出 inside then: pickles，也有 50% 的概率输出 inside catch: random reject triggered。
3. 链式调用中的 reject 穿透
Promise 的reject状态具有穿透性：若链式调用的任意环节触发 reject，后续的 then 会被跳过，直接执行最接近的 catch。我们通过多层链式调用验证这一特性：
randomResolveReject()
    .then(res => {
        console.log("first then:", res);
        return randomResolveReject();
    })
    .then(res => {
        console.log("second then:", res);
        return randomResolveReject();
    })
    .then(res => {
        console.log("third then:", res);
    })
    .catch(err => {
        console.log("catch error:", err.message);
    });
在这个链式调用中，只要任意一个 randomResolveReject Promise触发拒绝，整个调用链就会立即终止，并执行最后的 catch 逻辑。注意，我没有给 randomResolveReject 参数，虽然不传参数是可以的，但是我强烈建议你在自己的代码中传入合适的参数，从而提升代码的语义和可读性。
15.5 小结
本小节通过多个示例，详细讲解了自定义 Promise 对象的语法和逻辑，其核心价值可归纳为两点：
1.回调函数 Promise 化：将传统的回调式 API 封装为 Promise 规范的异步操作，支持 await 和链式调用，解决回调地狱问题，提升代码可读性。
2.精准控制异步状态：通过 resolve 和 reject 两个回调函数，开发者可以完全掌控 Promise 的状态切换时机，并按需传递兑现值或错误原因。
虽然本节的示例略显简单，但这些核心逻辑适用于更复杂的实际场景，例如封装第三方回调式 SDK、处理自定义异步任务等。
16 Promise 实战：Node.js 文件读取的 Promise 化改造
在异步编程的实际场景中，回调函数嵌套过深是早期 JavaScript 开发的典型痛点，这一问题在 Node.js 的文件操作、网络请求等异步 API 中尤为突出。本节将以 Node.js的 fs 模块中的 readFile 方法为例，从传统的回调函数写法的缺陷入手，逐步演示如何通过手动封装 Promise 解决回调地狱问题，再基于 Promise 链与 async/await 实现代码的扁平化重构，最终掌握回调函数 Promise 化的通用思路。
即便你对 Node.js 并不熟悉也无需担心，其 JavaScript 语法与浏览器环境基本一致，核心差异在于 Node.js 提供了文件系统（fs 模块）等服务端器专属能力。
16.1 传统回调式文件读取与回调地狱问题
Node.js 的 fs 模块提供了 readFile 方法，用于异步读取文件内容，其核心语法为基于回调函数的设计，接收三个参数：
fs.readFile(filePath, options, callback)
1.	filePath 目标文件的路径；
2.	options 文件编码格式（如 utf8），这是一个可选参数；
3.	callback 回调函数，该函数接收 error 和 data 两个参数 —— 读取失败时 error 为错误对象，读取成功时 data 为文件内容。
1. 读取单文件的回调函数实现
我们以读取 haiku1.txt 为例，实现最基础的异步文件读取操作：
const fs = require('fs');

// 异步读取 haiku1.txt
fs.readFile('./haiku1.txt', 'utf8', (error, data) => {
    if (error) {
        return console.error('文件读取失败：', error);
    }
    console.log('Haiku 1：');
    console.log(data);
});
执行这段代码，控制台会输出 haiku1.txt 中的俳句内容；若文件名拼写错误或文件不存在，则会打印对应的错误信息。这种写法对于单文件读取是可行的，但在按顺序读取多个文件的场景下，会暴露出严重的缺陷。
2. 多文件顺序读取的回调地狱
假设我们需要按顺序读取 haiku1.txt、haiku2.txt、haiku3.txt—— 必须等待前一个文件读取完成后，再读取下一个文件。此时只能通过回调函数嵌套的方式来实现：
fs.readFile('./haiku1.txt', 'utf8', (error, data) => {
    if (error) {
         return console.error('读取 haiku1 失败：', error);
    }
    console.log('Haiku 1：', data);
    // 嵌套读取第二个文件
    fs.readFile('./haiku2.txt', 'utf8', (error, data) => {
        if (error) {
            return console.error('读取 haiku2 失败：', error);
        }
        console.log('Haiku 2：', data);
        // 嵌套读取第三个文件
        fs.readFile('./haiku3.txt', 'utf8', (error, data) => {
            if (error) {
               return console.error('读取 haiku3 失败：', error);
            }
            console.log('Haiku 3：', data);
        });
    });
});
这种多层嵌套的写法被称为回调地狱，其核心问题可归纳为三点：
•	代码可读性差：嵌套层级随任务数量增加而加深，代码结构呈 “金字塔” 状，难以快速梳理执行逻辑；
•	错误处理分散：每个回调函数都需要单独处理错误，无法统一捕获异常；
•	维护成本高：新增或删除任务时，需要修改多层嵌套的代码，极易引发逻辑错误。
16.2 手动封装 Promise 化的文件读取函数
要解决回调地狱问题，核心思路是将回调式 API 封装为返回 Promise 的函数，将回调的 error 和 data 分别映射到 Promise 的 reject 和 resolve 状态。
我们定义一个 readFile 函数，在其内部创建 Promise 实例，并调用 fs.readFile 实现异步读取文件逻辑：
const fs = require('fs');

/**
 * Promise 化的文件读取函数
 * @param {string} filePath - 目标文件路径
 * @returns {Promise<string>} 成功时返回文件内容，失败时返回错误对象
 */
function readFile(filePath) {
    // 返回 Promise 实例
    return new Promise((resolve, reject) => {
        // 调用 Node.js 原生 readFile 方法
        fs.readFile(filePath, 'utf8', (error, data) => {
            if (error) {
                // 读取失败：调用 reject 触发 Promise 拒绝状态
               return reject(error);
            }
            // 读取成功：调用 resolve 触发 Promise 兑现状态，并传递文件内容
            resolve(data);
        });
    });
}
这段封装的核心逻辑可总结为两点：
1.	执行器函数中调用原始回调式 API（fs.readFile）；
2.	根据回调函数的 error 判断状态 —— 有错误则调用 reject，无错误则调用 resolve 并传递数据。
通过这一步封装，我们就可以基于 Promise 的链式调用，替代回调函数的嵌套写法。
16.3 基于 Promise 链的顺序文件读取
Promise 的 .then 方法支持链式调用，且前一个 .then 中返回的 Promise 会作为下一个 .then 的入参。利用这一特性，我们可以实现无嵌套的顺序文件读取，并通过末尾的 .catch 统一捕获所有异常。
// 链式读取三个文件
readFile('./haiku1.txt')
    .then(data => {
        console.log('Haiku 1：', data);
        // 返回下一个文件的读取 Promise
        return readFile('./haiku2.txt');
    })
    .then(data => {
        console.log('Haiku 2：', data);
        // 返回第三个文件的读取 Promise
        return readFile('./haiku3.txt');
    })
    .then(data => {
        console.log('Haiku 3：', data);
    })
    .catch(error => {
        // 统一捕获所有环节的错误
        console.error('文件读取异常：', error);
    });
相较于回调嵌套的写法，Promise 链的优势十分明显：
•	代码扁平化：无论读取多少个文件，代码始终保持同一层级，结构清晰易读；
•	错误处理集中化：所有异步操作的异常都会被末尾的 .catch 捕获，无需重复编写错误处理逻辑；
•	流程可控性强：通过在 .then 中返回 Promise，可以灵活控制任务的执行顺序。
16.4 基于 async/await 的扁平化重构
Promise 链虽然解决了嵌套问题，但仍需编写 .then 链式调用。而 async/await 语法可以进一步将异步代码 “同步化”，是目前最简洁、可读性最高的异步编程方案。
我们定义一个 async 函数 getHaikus，结合 await 实现顺序文件读取，并通过 try/catch 捕获异常：
/**
 * 基于 async/await 读取所有俳句文件
 */
async function getHaikus() {
    try {
        // 等待第一个文件读取完成
        const haiku1 = await readFile('./haiku1.txt');
        console.log('Haiku 1：', haiku1);

        // 等待第二个文件读取完成
        const haiku2 = await readFile('./haiku2.txt');
        console.log('Haiku 2：', haiku2);

        // 等待第三个文件读取完成
        const haiku3 = await readFile('./haiku3.txt');
        console.log('Haiku 3：', haiku3);
    } catch (error) {
        // 捕获所有 await 环节的异常
        console.error('文件读取异常：', error);
    }
}

// 调用 async 函数
getHaikus();
这段代码的执行逻辑与 Promise 链完全一致，但代码风格更接近同步编程，极大降低了理解成本。需要注意的是：
•	await 关键字必须在 async 函数内部使用；
•	所有 await 操作的异常都可以通过 try/catch 统一捕获，与同步代码的错误处理逻辑一致。
16.5 补充内容：Node.js 原生 Promise 化文件 API
需要强调的是，现代 Node.js 已经内置了 Promise 化的 fs 模块 API，开发者无需手动封装。Node.js 提供了 fs.promises 对象，其包含的 readFile 方法会直接返回 Promise 实例，用法如下：
const fs = require('fs').promises;

async function getHaikusNative() {
    try {
        const haiku1 = await fs.readFile('./haiku1.txt', 'utf8');
        console.log('Haiku 1：', haiku1);
        // 后续逻辑与手动封装版本一致
    } catch (error) {
        console.error('读取异常：', error);
    }
}
虽然原生 API 已经足够便捷，但手动封装 Promise 的思路仍然具有重要价值—— 在实际开发中，我们常会遇到第三方库或遗留代码中的回调式 API（这些 API 没有原生 Promise 版本），此时就需要通过本节的封装方法，将其转化为支持 async/await 的异步操作。
16.6 小节
本小节通过 Node.js 文件读取的案例，完整演示了回调函数 Promise 化的全流程，核心要点可归纳为三点：
1.	回调地狱的根源：多层异步任务的顺序执行依赖回调嵌套，导致代码可读性与可维护性下降；
2.	Promise 封装的核心：通过 new Promise() 包裹回调式 API，将 error 映射到 reject，将 data 映射到 resolve；
3.	异步代码的最优写法：基于 async/await 实现同步化的异步编程，结合 try/catch 完成错误处理，是目前最推荐的异步代码编写范式。
文件读取只是一个载体，回调函数 Promise 化的思路适用于所有基于回调的异步操作，是解决回调地狱、提升代码质量的关键技能。

<img width="539" height="777" alt="image" src="https://github.com/user-attachments/assets/998ed649-5fcb-4e71-96e4-5dd5cb2edced" />
