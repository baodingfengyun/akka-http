@@@ div { .group-java }

This section is only relevant when using the Scala API

本部分只在使用 Scala API 时相关
@@@

@@@ div { .group-scala }
# Case Class 提取
**Case Class Extraction**

The value extraction performed by @ref[Directives](directives/index.md) is a nice way of providing your route logic with interesting request
properties, all with proper type-safety and error handling. However, in some case you might want even more.
Consider this example:

通过 @ref[Directives](directives/index.md) 执行的值提取是为你的路由逻辑提供为感兴趣的请求属性的一种好方法，所以这些属性都具有合适的类型-安全性和错误处理。
但是，有些情况下，你可能会想更多。举个例子：

@@snip [CaseClassExtractionExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/CaseClassExtractionExamplesSpec.scala) { #example-1 }

Here the @ref[parameters](directives/parameter-directives/parameters.md) directives is employed to extract three `Int` values, which are then used to construct an
instance of the `Color` case class. So far so good. However, if the model classes we'd like to work with have more
than just a few parameters the overhead introduced by capturing the arguments as extractions only to feed them into the
model class constructor directly afterwards can somewhat clutter up your route definitions.

这里，`parameters` 指令用于提取三个 `Int` 值，然后这三个值用于构造 `Color` case 类的实例。到目前为止还不错。
但是，如果我们想要处理的模型类不止有几个参数，那么捕获作为提取的参数并将其直接提供给模型类构造函数所带来的开销，可能你的路由定义有些混乱。

If your model classes are case classes, as in our example, Akka HTTP supports an even shorter and more concise
syntax. You can also write the example above like this:

如果你的模型类是 case 类，在我们的例子中，Akka HTTP 支持一种更短且更简洁的语法。也可以像这样写上面的例子：

@@snip [CaseClassExtractionExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/CaseClassExtractionExamplesSpec.scala) { #example-2 }

You can postfix any directive with extractions with an `as(...)` call. By simply passing the companion object of your
model case class to the `as` modifier method the underlying directive is transformed into an equivalent one, which
extracts only one value of the type of your model class. Note that there is no reflection involved and your case class
does not have to implement any special interfaces. The only requirement is that the directive you attach the `as`
call to produces the right number of extractions, with the right types and in the right order.

你可以使用 `as(...)` 调用为任何带有提取的指令添加后缀。通过简单的传递模型 case 类的伴身对象到 `as` 修改器方法，将底层指令转换为等效的指令，
该指令只提取模型类类型的一个值。注意，这里没有反射参与并且你的 case 类没有实现任何特定接口。唯一的要求是，附加的指令 `as` 调用生成正确的提取数量，
具有正确的类型和正确的顺序。

If you'd like to construct a case class instance from extractions produced by *several* directives you can first join
the directives with the `&` operator before using the `as` call:

如果要从 *几个* 指令生成的提取构造 case 类实例，可以先使用 `&` 操作符连接指令，再使用 `as` 调用：

@@snip [CaseClassExtractionExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/CaseClassExtractionExamplesSpec.scala) { #example-3 }

Here the `Color` class has gotten another member, `name`, which is supplied not as a parameter but as a path
element. By joining the `path` and `parameters` directives with `&` you create a directive extracting 4 values,
which directly fit the member list of the `Color` case class. Therefore you can use the `as` modifier to convert
the directive into one extracting only a single `Color` instance.

这里 `Color` 类有另一个成员：`name`，它不作为（查询）参数而是作为路径元素提供。通过将 `path` 和 `parameters` 指令与 `&` 连接起来，创建一个提取 4 个值的指令，
这些值直接贴合 `Color` case 类的成员列表。因此，可以使用 `as` 修改器将指令转换为提取单个 `Color` 实例的指令。

Generally, when you have routes that work with, say, more than 3 extractions it's a good idea to introduce a case class
for these and resort to case class extraction. Especially since it supports another nice feature: validation.

一般来说，当你有处理超过 3 个提取的路由时，最好引入 case 类并使用 case 类提取。特别是它支持另一个很好的特性：验证。

@@@@warning { title="Caution" }
There is one quirk to look out for when using case class extraction: If you create an explicit companion
object for your case class, no matter whether you actually add any members to it or not, the syntax presented above
will not (quite) work anymore. Instead of `as(Color)` you will then have to say `as(Color.apply)`. This behavior
appears as if it's not really intended, so this might be improved in future Scala versions.

当使用 case 类提取时有一个奇怪的地方需要留意：如果为 case 类创建一个显示的伴身对象，不管是否真的添加了任何成员，上面的语法将再也不能（完全）工作。
作为 `as(Color)` 的替代，不得不用 `as(Color.apply)`。这种行为似乎不是真实想要的，所以，在未来的 Scala 版本可能会改进。
@@@@

## Case Class Validation
**Case 类验证**

In many cases your web service needs to verify input parameters according to some logic before actually working with
them. E.g. in the example above the restriction might be that all color component values must be between 0 and 255.
You could get this done with a few @ref[validate](directives/misc-directives/validate.md) directives but this would quickly become cumbersome and hard to
read.

在很多情况下，你的 Web 服务在实际使用输入参数以前需要根据某些逻辑验证它们。例如：在上面的例子中，可以限制所有颜色分量值必须在 0 到 255 之间。  
可以使用一些 @ref[validate](directives/misc-directives/validate.md) 指令完成这项工作，但是这将很快变得繁琐且难以阅读。

If you use case class extraction you can put the verification logic into the constructor of your case class, where it
should be:

如果使用 case 类提供，可以将验证逻辑放入 case 类的构造器里，它应该在：

@@snip [CaseClassExtractionExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/CaseClassExtractionExamplesSpec.scala) { #example-4 }

If you write your validations like this Akka HTTP's case class extraction logic will properly pick up all error
messages and generate a @apidoc[ValidationRejection] if something goes wrong. By default, `ValidationRejections` are
converted into `400 Bad Request` error response by the default @ref[RejectionHandler](rejections.md#the-rejectionhandler), if no
subsequent route successfully handles the request.

如果像这样编写验证，如果出了什么问题，Akka HTTP 的 case 类提取逻辑将正确的获取所有错误消息并生成 @apidoc[ValidationRejection] 。
默认，如果没有后续路由成功处理请求，`ValidationRejections` 将被默认 @ref[RejectionHandler](rejections.md#the-rejectionhandler) 转换为 `400 Bad Request` 错误响应。
@@@