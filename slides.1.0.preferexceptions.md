# Rule 1 - Prefer Exceptions

Sample codes - return error codes from function

```ts

if (deletePage(page) == E_OK) {
    if (registry.deleteReference(page.name) == E_OK) {
        if (configKeys.deleteKey(page.name.makeKey()) == E_OK){
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry failed");
    }
} else {
    logger.log("delete failed");
    return E_ERROR;
}

```

<!--
1. 还是按照和以前一样的方式，主要是先原汁原味的讲解书中的内容，再加上一点个人的理解给给大家。
2. 这是书中的样例代码， 给大家20秒中的时间，大家可以看下这段代码有什么样的问题。
3. 简单过一下代码内容 - 重点是每个方法的调用存在依赖关系，上一个成功后，才能执行下一个。
-->

---

# What problems to return error codes?

1. Deeply nested structures
2. Dependency Magnet

<!--
1. 一是嵌套太深了， 最多是有5层，在我们第一节课中，最寻small的原则，最好不要超过2两层。
2. 二是，书中给出了一个词语，叫依赖磁铁。 在上面的样例代码中，E_OK 这样的状态码一般都会定义中一个枚举类中，可以会有很多其他的方法来使用它们， 当我们需要更新它的名字的时候，所有调用的地方都会受影响。一般为了避免这种情况，我们一般会新增一个code，即使可能已经存在一个类似的存在。这样慢慢也就演变成了一些千奇百怪的冗余代码的存在，并且还谁都不能轻易的去触碰它们。
-->

---

# Use exceptions instead of returned error codes

```ts
try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
} 
catch (Exception e) {
    logger.log(e.getMessage());
}
```

<!--
1. 针对这种情况，书中给出了一种推荐的解决方案 - 就是使用一个try/catch块来包括起来这三个方法的调用，来一起部署这些方法抛出的异常。 这样既减少了代码层级的嵌套，又完美的实现了任何一个方法执行失败都会终止整个流程。
-->

---

# Problem？

[LanguageAssignmentManager](https://gitlab.com/kingland-projects/indy/projects/independence/-/blob/master/Independence/src/com/kingland/independence/languageassignment/LanguageAssignmentManager.java?ref_type=heads#L1152)


<!--
1. 又到了思考时刻，难道仅仅是使用一个try/catch块，就没有任何问题了吗？
2. 大家看一下我随便从系统中找到的一段代码哈，大家看看这个方法中总共有多少个try/catch块，这段阅读起来那是相当费劲的。
3. 这还只是在一个方法中， 如果方法的调用层级很深，每一层级的方法都有这样的try/catch块，那简直就是灾难。
3. 由此可见，如果只是肆无忌惮添加try/catch块，而没有遵循一定的规则的话，也会带来很大的阅读负担。

-->

---

# How to extract try/catch blocks?

1. DO NOT add try/catch to every level of function
2. Error handling is one thing

<!--
1. 那如何解决这样的问题， 我总结归纳书中的观点为两点。
2. 一不要在每一个层级的方法中， 都添加try/catch 块。
3. 二是错误处理就应该被当作已经时间处理。我们之前讲的规则，一个函数应该只做一件事情(do one thing)， 在这个错误处理的函数中同样适用。
4. 那我们分别来看一下这两条规则。

5. 需要强调的一点是，这节课中的内容关于try/catch捕获异常，都是在前几课内容基于函数层级划分的基础上，如何规划好try/catch增加来可读性，并不是关于checked和unchecked异常的讨论，它们是这本书中后续的内容。 其实前几年B老师和Nate叔，还有R总 也都有过lunchlearn， 后续我们也会再一起回顾那部分内容。
-->

---

# DO NOT add try/catch to every level of function

<img src="/images/exceptionlevel.png" class="rounded shadow" />


<!--
1. 为了方便大家理解， 我直接将书中的代码，以图形划的方式展示出来。
2. 这个规则说的也就是，一定要最上的层级统一地去捕获和处理一样，而不是在第三和第二层级，分别去做try/catch 操作。

-->
---

# DO NOT add try/catch to every level of function

```ts
public void delete(Page page) {
	 try {
		deletePageAndAllReferences(page);
	 }catch (Exception e) {
		logError(e);
	 }
 }

private void deletePageAndAllReferences(Page page) throws Exception {
	deletePage(page);
	registry.deleteReference(page.name);
	configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) {
	logger.log(e.getMessage());
}
```

<!--
1. 这是书中样例的代码。

-->
---

# Problem?

<v-click>

How do we know what level to go up to?

</v-click>

<!--
1. 给几秒中提问。
2. 但是有一个问题是， 书中没有明确说明，那到底向上抽取到哪一层级合适？
3. 看书中的例子，我们会理所当然的认为，嗯， 它就应该是这个样子，就应该是在这一层。那换成我们的实际业务代码可能就不清楚它应该在那一层级添加，或者说我们有什么进一步的规则去遵循么。
4. 注意了，书中并没有给出特别清晰的说法，以下是我个人的想法，仅供参考！！
-->

---

# DO NOT add try/cacth to every level of function

### Personal View
https://frankie-talks-thinking.netlify.app/23

- Any dependencies between function execution order?

> 👉 Yes - DO NOT add tryc/acth to every level of function

> 👉 No - Add try/catch in function itself

<!--
1. 我认为我们应该关注的一个重点就是，同一层级的方法中是否有执行的依赖关系，就是后一个方法的调用要依赖于前一个方法的执行结果，如果前一个执行成功它继续执行，否则流程终止。
2. 清楚这一点后， 我们应该遵循的规则是，如果有依赖，不要在当前层级的方法中添加try/catch，应该继续向上寻找，直到，同一层级的多个方法没有依赖，那try/catch块应该添加到各自方法中。
3. 再一起看一下，我之前的分享中关于TB的层级图，假设日料和火锅这一层级有依赖，那就不应该在它们添加try/catch，那就应该向上寻找，假设，餐饮和游戏也有依赖，那就应该继续向上寻找，这时假设娱乐项目选择和家属餐饮没有依赖， 那try/catch就应该添加到娱乐项目选择这一层，否则就应该继续向上直到最顶层。
4. 实际的业务代码，肯定是这种依赖关系和没有依赖关系方法混合在一起的情况，大家可以试着用我这个方法。 出问题了，不要找我哈。


-->

---

### Multiple try/catch
```ts
public void prepareTeamBuilding() {
	try {
		chooseActivities();
	 }catch (ExceptionA e) {
		logError(e);
	 }

    try {
		checkFamilyMembers();
	 }catch (ExceptionB e) {
		logError(e);
	 }
 }
```
<!--
1. 可能会有小伙伴，表示不认同， 既然这两个方法没有依赖，为什么一定要将try/catch块添加到各自的方法中，向这样写到一个方法中，分别做try/catch 有什么问题呢。
2. 问题就是增加了阅读负担， 违反下面的规则， error handing is one thing
-->
---

# Error handling is one thing

1. if the keyword
try exists in a function, it should be the very first word in the function and that there
should be nothing after the catch/finally blocks.

2. Functions should do one thing. Error handing is one thing. Thus, a function that handles
errors should do nothing else.

```ts
public void delete(Page page) {
	 try {
		deletePageAndAllReferences(page);
	 }catch (Exception e) {
		logError(e);
	 }
 }
```

<!--
1. 前面提到了，要遵循 do one thing的原则， 错误处理也应该被当作一件事情来出来。 这种写法就是一个函数中，在处理两件事情。
2. try/catch 块前不要有代码， 是指不要有业务流程的代码减轻阅读负担。但是正常的变量声明等是正常的。
-->
